---
name: paid-media-agent/review_saturation_brief
description: >
  Use this skill when reviewing the autonomous channel saturation analysis
  produced by the Analyst agent's `analyze_channel_saturation` tool. Triggers
  include: "review saturation brief", "channel is saturated", "saturation
  analysis", "cap budget recommendation", "override saturation", "hill function
  threshold", "85% rule", "diminishing returns", "marginal ROI", "should we
  accept the saturation recommendation", or any situation where the agent has
  flagged a channel as Saturated or Diminishing and a human decision is required
  before the Operator executes capital pruning.
  Requires the paid-media-mcp server to be connected.
---

# Saturation Override Protocol

The Analyst agent runs Hill function saturation analysis across all five channels.
When it flags a channel as **⚠️ Saturated** or **🛑 Diminishing**, it writes an
operator recommendation vector to `analyst_insights` — but takes no action.
This skill walks through the human review gate before any budget cap is executed.

---

## Step 1 — Load the saturation brief

Pull the latest analyst output. The saturation brief arrives as a structured
operator vector embedded in the most recent Analyst insight record.

```
get_analyst_insights(insight_type="channel_saturation", limit=1)
```

The operator vector is a JSON list. One entry per channel. Each entry has the
shape:

```json
{
  "channel":                  "meta",
  "action":                   "cap_budget",
  "current_monthly_spend_usd": 72000,
  "saturation_pct":           90.0,
  "marginal_roi":             0.54,
  "recommended_cap_usd":      68000,
  "reallocation_amount_usd":  4000,
  "reallocation_target":      "tiktok",
  "urgency":                  "high"
}
```

**Action values and what they mean:**

| Action | Meaning | Default response |
|---|---|---|
| `cap_budget` | Spend is at or above 85% of max efficient — freeze expansion, prune to cap | Review and approve or override |
| `freeze_budget_expansion` | 75–85% saturation — hold spend flat, no increases | Usually accept |
| `hold` | Below 75% saturation — no action required | Accept automatically |

**Focus only on channels with `action: "cap_budget"` or `action: "freeze_budget_expansion"`.**

---

## Step 2 — Cross-validate with live data

The autonomous Hill model uses a 30-day rolling spend average from `mmm_channel_contributions`.
Before accepting or overriding, pull live platform truth to confirm the model's baseline is current.

### 2a. Channel efficiency check

```
get_channel_efficiency(date_from="YYYY-MM-01", date_to="YYYY-MM-DD")
```

Compare the MCP-returned `cpa` and `roas` per channel against the saturation brief:

| Check | Green | Red |
|---|---|---|
| CPA trend vs. target | Within CPA tolerance corridor for saturation band | CPA rising faster than band allows |
| ROAS trend | Declining but within expected saturation curve | ROAS collapsed below floor |
| Spend delta | MCP spend ≈ analyst spend input (< 5% gap) | Large discrepancy — data lag or model staleness |

> If the MCP spend figure differs from `current_monthly_spend_usd` in the operator vector
> by more than 10%, the analyst may have used stale MMM inputs. Trigger a fresh run before
> proceeding: `trigger_agent_run(agent="analyst")`.

### 2b. Pacing status check

```
get_pacing_report()
```

For any channel flagged for `cap_budget`, check its current pacing rate. A channel that is
**critically underpacing** (< 75%) is unlikely to actually hit the saturation threshold
regardless of what the model shows. Do not cap a channel that can't spend to its current budget.

---

## Step 3 — Cognitive discrepancy framework

Math and strategy can diverge. Before accepting the agent's capital pruning, answer these
four questions for each flagged channel:

### Q1 — Is there a time-bound strategic commitment overriding the saturation signal?

Examples:
- **Product launch window**: Saturated by Hill function math but required for share-of-voice
  during a 4-week launch sprint
- **Competitive defense**: A key competitor is running an aggressive conquest campaign — the
  saturation cost is the strategic cost of visibility
- **Locked IO or commitment**: A contracted insertion order or upfront commitment means spend
  cannot be reduced without a penalty

**If yes**: Apply a **Strategic Override** (see Step 4). Document the overriding rationale.

### Q2 — Is the saturation signal driven by spend level or creative fatigue?

The Hill function models spend-level saturation. But a channel can show degraded CPA
(which looks like saturation) because of creative fatigue — a distinct cause with a distinct fix.

Check:
```
get_ad_performance(campaign_id="<flagged_campaign>", date_from="...", date_to="...")
```

If the top 3 creatives show **frequency > 5 in the last 14 days** and the bottom-performing
creatives account for > 40% of spend — the issue is creative, not saturation.

**If yes**: Override the cap. Recommend creative refresh. Saturation cap on a creatively
fatigued channel will suppress the channel right as new creative is ready to deploy.

### Q3 — Is the reallocation target actually ready to absorb the freed capital?

The operator vector specifies a `reallocation_target`. Before approving the reallocation:

```
get_pacing_report()
get_channel_efficiency(...)
```

Verify the target channel is:
- Not already overpacing (> 110%)
- Not itself approaching its 85% saturation threshold
- Has active campaigns in a state to absorb additional budget without burning it in off-target placements

**If the target is not ready**: Accept the cap but hold the reallocation. Park capital in
reserve rather than pushing it into a channel that can't deploy it efficiently.

### Q4 — Is the marginal ROI calculation plausible given current market conditions?

The saturation brief reports `marginal_roi` = approximation of the derivative of the Hill
function at current spend, scaled by posterior ROI mean.

Cross-reference against `get_roas_comparison()`. If the analyst's computed marginal_roi
diverges significantly from the MCP-reported current-period ROAS, the MMM posterior may be
stale. A posterior that's more than 60 days old should be treated as directional, not precise.

```
get_attribution_run_history(limit=5)
```

Check the most recent `mmm_run_id` timestamp. If last MMM run is > 45 days ago, flag this
for re-run before the cap executes.

---

## Step 4 — Sign-off matrix

Complete this checklist before routing any action to the Operator agent.

### For each channel with `action: "cap_budget"` or `action: "freeze_budget_expansion"`:

```
SATURATION REVIEW — [CHANNEL] — [DATE]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

DATA VALIDATION
[ ] MCP spend matches analyst input (< 10% gap)
[ ] Current CPA is within expected saturation band for this saturation_pct
[ ] Pacing rate > 80% (channel is actually spending to its budget)
[ ] MMM run date is < 45 days old

STRATEGIC CONTEXT
[ ] No time-bound launch or IO commitment requiring elevated spend
[ ] Degradation is spend-level saturation, not creative fatigue
[ ] Reallocation target is ready to absorb freed capital

DECISION (select one):
[ ] ACCEPT — Execute agent's recommended cap as written
[ ] PARTIAL OVERRIDE — Accept cap at different level: $________/month
[ ] FULL OVERRIDE — Hold spend flat; document reason below
[ ] DEFER — Re-run analyst with fresh MMM posterior before deciding

Override rationale (required if PARTIAL or FULL override selected):
_______________________________________________
_______________________________________________
```

### After sign-off: routing instructions

**If ACCEPT:**
The operator vector is already formatted for the Operator agent. Pass the `action_id` from
`get_analyst_insights` to route the package for execution:

```
get_pending_approvals()
```

Locate the matching entry and approve it through the standard approval interface.

**If PARTIAL OVERRIDE:**
Do NOT pass the operator vector directly. Instead, call `reallocate_media_budget` with
the manually corrected target:

```
reallocate_media_budget(
  channel="meta",
  new_monthly_budget_usd=<your_override_cap>,
  rationale="Saturation brief accepted at modified level — strategic launch commitment through [date]"
)
```

**If FULL OVERRIDE:**
Document the rationale in `get_analyst_insights` comment field. No execution action.
Schedule re-review in 7 days.

**If DEFER:**
```
trigger_agent_run(agent="analyst")
```

Re-review when the new insight record is written (typically within 24 hours of the next
scheduled Analyst run).

---

## Notes

- The 85% diminishing return rule is a hard freeze on budget expansion — it does not require
  immediate spend reduction. A `cap_budget` recommendation means cap at the `recommended_cap_usd`
  level, not zero the channel.
- For LinkedIn specifically: due to its 24.4-day adstock half-life, a saturation cap paired
  with a flighting cool-down achieves a compounding efficiency gain. Check whether an
  `approve_flighting_schedule` workflow should run in parallel.
- This review gate must complete before the Operator agent can execute. `OPERATOR_REQUIRE_APPROVAL=true`
  is enforced in code — the Operator will queue the action and wait.
- `MAX_BUDGET_SHIFT_PCT=10` — any cap recommendation that implies a shift of more than 10%
  from current budget will be rejected at pre-flight. If the analyst's recommended_cap_usd
  implies > 10% reduction, the Operator will flag it and return an error rather than partially executing.

---

*© 2026 @arcticgreyy — Business Source License 1.1. Persistent attribution required.*
*See paid-media-agent/LICENSE and paid-media-agent/NOTICE for terms.*
