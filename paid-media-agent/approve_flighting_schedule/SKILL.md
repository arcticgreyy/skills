---
name: paid-media-agent/approve_flighting_schedule
description: >
  Use this skill when reviewing and authorizing an adstock-driven pulsing
  schedule produced by the Analyst agent's `evaluate_adstock_decay_pacing` tool.
  Triggers include: "approve flighting schedule", "authorize cool-down",
  "adstock pulsing", "flighting recommendation", "cool-down window", "LinkedIn
  pulse", "halo harvest", "residual spend", "pulse on pulse off", "COOL_DOWN
  action", "flighting authorization", or any situation where the Analyst has
  produced a 14-day flighting plan and the Operator needs human sign-off before
  executing spend level changes.
  Requires the paid-media-mcp server to be connected.
---

# Pacing & Flighting Authorization Gate

The Analyst agent's adstock engine produces a 14-day programmatic flighting
schedule — PULSE_ON / COOL_DOWN / HOLD actions per channel, with exact budget
percentages and floor checks. The schedule is mathematically optimal but cannot
execute without explicit human authorization. This skill is the authorization gate.

---

## Step 1 — Frame the core trade-off

Before any validation, state the trade-off in plain terms. Pull the latest
adstock brief from analyst outputs:

```
get_analyst_insights(insight_type="adstock_decay_pacing", limit=1)
```

The flighting schedule section of the output contains an operator vector with
this shape per channel:

```json
{
  "channel": "linkedin",
  "half_life_days": 24.4,
  "current_residual_pct": 100.0,
  "residual_spend_equiv_weekly": 10017,
  "cool_down_eligible": true,
  "week_1": {
    "action": "COOL_DOWN",
    "budget_pct": 30,
    "implied_weekly_usd": 3005,
    "floor_check": "PASS"
  },
  "week_2": {
    "action": "PULSE_ON",
    "budget_pct": 100,
    "implied_weekly_usd": 10017,
    "floor_check": "PASS"
  }
}
```

**State the trade-off explicitly for each COOL_DOWN channel:**

> "The Analyst agent has detected a **{half_life_days}-day** adstock half-life on
> **{channel}** and proposes a **{cool_down_duration}-day** cool-down at **{budget_pct}%**
> of normal spend to harvest a **${residual_spend_equiv_weekly:,}/week** residual halo.
> This reduces live spend by **${reduction_weekly:,}/week** while the $
> {residual_spend_equiv_weekly:,} in carry-over effect continues working in the market.
> **Do you authorize this budget drop?**"

**Action values:**

| Action | Meaning |
|---|---|
| `COOL_DOWN` | Drop to floor spend (typically 25–40% of normal) to harvest adstock halo |
| `PULSE_ON` | Full spend or surge (100–120%) to build new adstock inventory |
| `HOLD` | No change — maintain current spend level |

**If the channel has no COOL_DOWN this cycle**: No authorization needed for that channel.
HOLD and PULSE_ON actions execute automatically unless `OPERATOR_REQUIRE_APPROVAL=true` is
set system-wide.

---

## Step 2 — Pre-flight validation checks

Run these checks before authorizing any COOL_DOWN or spend reduction.

### 2a. Platform floor check

Confirm the proposed cool-down spend does not breach platform minimum budgets.
**Platform floors enforced in the Operator's pre-flight sweep:**

| Platform | Daily Floor | Weekly Equivalent |
|---|---|---|
| Google Ads | $5.00/day | $35/week |
| TikTok Ads | $20.00/day | $140/week |
| Meta | $1.00/day | $7/week |
| LinkedIn | $10.00/day | $70/week |
| Reddit Ads | $5.00/day | $35/week |

The `floor_check` field in the operator vector already reflects this. If `floor_check: "FAIL"`,
the Operator will reject the schedule at pre-flight — do not authorize until the schedule
is rebuilt with floor-safe numbers.

Also check the private decay config floor (`min_floor_spend_weekly_usd`) for awareness channels:

| Channel | Min Floor (weekly) | Context |
|---|---|---|
| Google Ads | $3,500 | Protects brand query share of voice |
| LinkedIn | $2,000 | Maintains InMail delivery pipeline |
| Meta | $2,500 | Prevents audience exit from retargeting pool |
| TikTok | $1,500 | Minimum for algorithm memory |
| Reddit Ads | $800 | Community presence floor |

For any COOL_DOWN, confirm `implied_weekly_usd` ≥ both the platform floor AND the channel's
`min_floor_spend_weekly_usd`. The higher of the two is the binding constraint.

### 2b. Active campaign pacing check

```
get_pacing_report()
```

If a channel is **overpacing** (> 115%) going into a COOL_DOWN week, the effective spend
reduction will be smaller than the schedule assumes — the platform may absorb the budget
drop without noticeably reducing delivery. Flag this and adjust the schedule start date if
the channel is mid-flight overpace.

If a channel is **critically underpacing** (< 60%), a COOL_DOWN would compound the
underspend. Override to HOLD until pacing normalizes.

### 2c. Target account tracking check

For LinkedIn specifically, a COOL_DOWN pause means reduced InMail and Sponsored Content
delivery to target accounts currently in active sales stages. Before authorizing:

```
get_target_account_activity(lookback_days=7)
```

Check whether any tier-1 target accounts have shown engagement spikes in the last 7 days.
An account that just entered the identity graph should not be dropped into a COOL_DOWN window.

**If tier-1 accounts are active:** Delay the COOL_DOWN by 7 days. The halo harvest will
still be available — LinkedIn's 24.4-day half-life means waiting one week costs less than
3% of the residual.

### 2d. Organic session baseline check

Pull the organic/direct session trend to confirm the channel's adstock is genuinely above
the cool-down threshold and not already in decay:

```
get_daily_performance(
  date_from="<14_days_ago>",
  date_to="<today>",
  platforms=["<channel>"]
)
```

If spend has already been declining organically for > 7 days, the analyst's residual
calculation may be overstated. If `current_residual_pct` was computed from a spend series
that already includes a gap, the harvest window may be shorter than the schedule shows.

---

## Step 3 — Construct the execution payload

Once validation passes, translate the flighting schedule into the Operator agent's
`task27.v1` package format. The adstock schedule is NOT a standard MMM optimization package —
it requires manual construction of the execution payload.

### 3a. Build the channel_campaign_map

Map each channel to its active campaign IDs. These are required by the Operator's mutation
loop to identify which campaigns to adjust.

```
list_campaigns(status="active", platform="<channel>")
```

Build the map:

```json
{
  "channel_campaign_map": {
    "linkedin":   ["campaign_id_1", "campaign_id_2"],
    "google_ads": ["campaign_id_3"],
    "meta":       ["campaign_id_4", "campaign_id_5"],
    "tiktok":     ["campaign_id_6"],
    "reddit_ads": ["campaign_id_7"]
  }
}
```

Only include channels with COOL_DOWN or PULSE_ON actions (not HOLD).

### 3b. Build the execution package

The adstock flighting schedule must be packaged in task27.v1 schema for `execute_system_budget_reallocation`.

**Compute week 1 shift values for each COOL_DOWN channel:**

```
current_weekly_usd  = residual_spend_equiv_weekly  (from operator vector)
proposed_weekly_usd = implied_weekly_usd            (week_1 from operator vector)
shift_usd_weekly    = proposed_weekly_usd - current_weekly_usd
shift_pct           = (shift_usd_weekly / current_weekly_usd) × 100

# Convert to monthly equivalents for the task27.v1 package:
current_monthly_usd  = current_weekly_usd × 4.33
proposed_monthly_usd = proposed_weekly_usd × 4.33
shift_usd_monthly    = proposed_monthly_usd - current_monthly_usd
shift_pct_monthly    = shift_pct  # same percentage
```

> ⚠️ **10% shift cap**: `MAX_BUDGET_SHIFT_PCT=10`. If the COOL_DOWN implies a shift
> larger than 10% of the channel's current monthly budget, the Operator's pre-flight
> will reject it. In this case, authorize a partial COOL_DOWN (e.g., 70% budget instead
> of 30%) and schedule the second reduction for the following week.

**Full execution package:**

```json
{
  "schema_version":           "task27.v1",
  "operator_approval_required": true,
  "mmm_run_id":               "<adstock_run_id_from_analyst_insights>",
  "max_shift_pct_policy":     10,
  "recommendations": [
    {
      "channel":                 "linkedin",
      "direction":               "decrease",
      "recommended_shift_pct":   -8.5,
      "recommended_shift_usd":   -865.0,
      "new_target_budget_usd":   9152.0
    }
  ]
}
```

**Mandatory fields per recommendation:**

| Field | Type | Source |
|---|---|---|
| `channel` | string | From operator vector `channel` |
| `direction` | "increase" or "decrease" | `week_1.budget_pct` < 100 → "decrease"; > 100 → "increase" |
| `recommended_shift_pct` | float | Computed shift % (negative for COOL_DOWN) |
| `recommended_shift_usd` | float | Computed shift in USD (negative for COOL_DOWN) |
| `new_target_budget_usd` | float | `proposed_monthly_usd` from Step 3b |

### 3c. Submit for execution

```
execute_system_budget_reallocation(
  action_id="flighting_<YYYY-MM-DD>_<channel>",
  execution_package=<package from 3b>,
  channel_campaign_map=<map from 3a>
)
```

The Operator will:
1. Validate `schema_version` = "task27.v1"
2. Confirm `operator_approval_required` = true
3. Run all-or-nothing pre-flight sweep (shift cap, platform floor, campaign map)
4. Execute mutations sequentially across channels
5. Write to `operator_action_log` and `operator_pending_approvals`

If pre-flight fails, **zero mutations are applied**. The full error list is returned.
Resolve each error before retrying.

---

## Step 4 — Authorization sign-off checklist

Complete before submitting the execution package:

```
FLIGHTING AUTHORIZATION — [DATE]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CHANNELS PROPOSED FOR COOL_DOWN THIS CYCLE:
[ ] [channel 1]  Week 1: ___% → ${implied_weekly_usd}/wk  Floor check: PASS/FAIL
[ ] [channel 2]  Week 1: ___% → ${implied_weekly_usd}/wk  Floor check: PASS/FAIL

PRE-FLIGHT VALIDATION
[ ] All floor_check fields = PASS
[ ] No channel is critically underpacing (< 60%)
[ ] No tier-1 target accounts with spikes in last 7 days (LinkedIn)
[ ] Shift % ≤ 10% for each channel (task27.v1 cap enforced)
[ ] channel_campaign_map built and verified for all COOL_DOWN channels

WEEK 2 PLAN
[ ] Week 2 PULSE_ON schedule reviewed and calendar-blocked
[ ] Creative assets are ready for Week 2 surge (especially TikTok)

DECISION:
[ ] AUTHORIZE — Submit execution package as built
[ ] MODIFY — Adjust schedule: ______________________________
[ ] HOLD — Do not execute this cycle; reason: ______________

Authorized by: _________________________  Date: ___________
```

---

## Notes

- **Week 2 matters as much as Week 1.** A COOL_DOWN without a committed PULSE_ON date
  turns a deliberate adstock harvest into accidental spend underpacing. Block the
  Week 2 PULSE_ON before authorizing the Week 1 COOL_DOWN.
- **LinkedIn has a 4-week on / 2-week off cadence.** Do not initiate a COOL_DOWN
  mid-pulse unless the residual is already ≥ 65%. The cool-down gate requires: recent
  surge + high residual + awareness channel type (all three must be true).
- **TikTok cool-downs are full pauses.** The schedule recommends 0% spend for TikTok
  COOL_DOWN windows (fast 7.3-day decay; no halo worth harvesting at floor spend).
  A full pause on TikTok requires creative refresh before PULSE_ON. Confirm new
  creative is ready.
- If an organic anomaly is flagged in the adstock brief (organic spike without paid
  correlation), do NOT initiate a COOL_DOWN — the spike may indicate a PR or viral
  event that warrants a retargeting surge instead.

---

*© 2026 @arcticgreyy — Business Source License 1.1. Persistent attribution required.*
*See paid-media-agent/LICENSE and paid-media-agent/NOTICE for terms.*
