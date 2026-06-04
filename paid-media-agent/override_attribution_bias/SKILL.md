---
name: paid-media-agent/override_attribution_bias
description: >
  Use this skill when the Watchdog agent flags a forensic attribution anomaly —
  specifically a Trap A (CRM overwrite / Salesforce bulk import that silently
  rewrites digital attribution) or Trap B (organic session surge masking paid
  lift). Triggers include: "attribution bias alert", "CRM overwrite detected",
  "Salesforce attribution issue", "lead source changed", "orphaned token",
  "timestamp divergence", "forensic trap triggered", "watchdog alert",
  "organic surge masking", "paid lift contaminated", "correction weight",
  "calibrate Meridian priors", "attribution correction", or any situation where
  a channel's MMM posterior ROI may have been contaminated by downstream CRM
  data corruption and a correction must be injected before the next optimization
  cycle.
  Requires the paid-media-mcp server to be connected.
---

# Forensic Exception Handling — Attribution Bias Override

The Watchdog agent's Task 37 Forensic State Machine detects two categories of
attribution contamination that silently corrupt the Analyst's MMM priors:

- **Trap A — CRM overwrite**: Salesforce bulk import rewrites digital lead sources
  to offline values (e.g., "Trade Show", "SDR Cold Outreach"), stealing credit
  from the paid channel that actually drove the click.
- **Trap B — Organic surge masking**: A 5× organic session spike with flat paid
  spend creates a false counterfactual that makes paid lift appear larger than it
  is — inflating MMM ROI posteriors for the affected channel.

Both patterns require immediate investigation and, if confirmed, a correction
weight injection before the next MMM optimization run.

---

## Step 1 — Confirm and scope the alert

### 1a. Pull active Watchdog alerts

```
get_watchdog_alerts(status="open", limit=10)
```

Each alert contains:
- `alert_type`: `"trap_a_crm_overwrite"` | `"trap_b_organic_masking"` | `"trap_c_revenue_concentration"`
- `flagged_platform`: the paid channel implicated
- `anomaly_week`: the date range of the anomaly
- `severity`: `"critical"` | `"warning"` | `"info"`
- `description`: human-readable summary of what was detected

Identify all open Trap A and Trap B alerts. Trap C (revenue concentration) does not
require this forensic workflow — handle separately.

### 1b. Pull signal capture health for the flagged period

```
check_signal_capture_health()
```

If `gclid` or `fbclid` capture rates dropped during the same window as the anomaly,
the overwrite may have affected more records than the individual alert indicates.
A capture rate drop > 10% below the baseline during the anomaly week is a strong
corroborating signal.

### 1c. Locate the anomaly records in BigQuery

Run this query to pull all anomalies from the flagged window. Replace the date range
and platform with values from the alert:

```sql
SELECT
  anomaly_id,
  anomaly_type_enum,
  token_type,
  flagged_platform,
  crm_lead_source,
  claimed_channel,
  expected_channel,
  estimated_pipeline_value,
  confidence_score,
  detection_method,
  anomaly_week
FROM `{project}.{dataset}.data_attribution_anomalies`
WHERE anomaly_week BETWEEN DATE('{window_start}') AND DATE('{window_end}')
  AND flagged_platform = '{platform}'
  AND anomaly_type_enum IN ('orphaned_token', 'timestamp_divergence', 'phantom_conversion')
ORDER BY confidence_score DESC, estimated_pipeline_value DESC
LIMIT 100;
```

**Anomaly type guide:**

| `anomaly_type_enum` | What happened | Detection method |
|---|---|---|
| `orphaned_token` | A `gclid` or `fbclid` in the session exists, but the CRM lead_source is an offline value | `sql_orphan_scan` — session-to-CRM join |
| `timestamp_divergence` | CRM `lead_source_updated_at` is AFTER the click timestamp — source was rewritten retroactively | `timestamp_compare` |
| `phantom_conversion` | Platform reports a conversion but no matching CRM lead exists | `pixel_crm_mismatch` |

**Trap A signature**: Multiple `orphaned_token` + `timestamp_divergence` records for the
same `flagged_platform` in the same `anomaly_week`, with `crm_lead_source` set to an
offline value (e.g., "Trade Show", "Content Syndication", "SDR Cold Outreach").

**Trap B signature**: `phantom_conversion` records where `estimated_pipeline_value` is
NULL (no CRM lead matched) but `flagged_platform` shows elevated conversion counts —
indicating the platform is claiming conversions that aren't in the CRM.

---

## Step 2 — Investigation path by anomaly type

### Trap A — CRM Overwrite Investigation

#### 2a. Isolate the overwrite event

The key fingerprint: `systemmodstamp = lead_source_updated_at = created_at` on records
with offline `lead_source` values, where the session table shows a paid click token.

```
detect_crm_null_fields(date_from="{window_start}", date_to="{window_end}")
```

This surfaces CRM records with missing media identifiers (`gclid`, `fbclid`,
`li_fat_id`) in the flagged window. High null rates (> 30% for a period that showed
normal click capture earlier) indicate a batch import event.

#### 2b. Query the account journey for affected leads

For each `crm_lead_id` in the anomaly records (these are opaque CRM system IDs,
not PII), check whether the identity graph has the paid click that should have
been credited:

```
query_account_journey(
  account_domain="{affected_company_domain}",
  lookback_days=90
)
```

If the account journey shows a `gclid` or `fbclid` touchpoint that predates the
CRM `created_at` timestamp, this is definitive confirmation: the paid click existed
and was subsequently overwritten by the CRM import.

#### 2c. Quantify the misattribution

From the anomaly records, sum:
- `estimated_pipeline_value` — total ARR at risk from misattributed leads
- Count of `orphaned_token` + `timestamp_divergence` records by `flagged_platform`

This gives the financial scope of the overwrite.

**Decision threshold:**
- **≥ 3 records, confidence_score ≥ 0.70**: Proceed to correction injection (Step 3)
- **1-2 records, mixed confidence**: Flag for CRM team review; hold correction
- **All records confidence < 0.60**: Alert CRM team but do not inject correction —
  insufficient signal

---

### Trap B — Organic Surge Masking Investigation

#### 2a. Confirm the organic spike is unpaid

```
get_daily_performance(
  date_from="{window_start}",
  date_to="{window_end}",
  platforms=["all"]
)
```

If paid spend was flat (< 5% week-over-week change) during the organic spike, the
spike is external — not adstock carry-over. This is the Trap B pattern.

#### 2b. Check for correlated paid signals

```
check_signal_capture_health()
```

If `gclid` and `fbclid` capture rates were normal during the spike, the organic
traffic was genuinely unpaid. If capture rates were degraded, some organic-attributed
sessions may actually be paid sessions with broken tracking.

#### 2c. Identify the spike source

Cross-reference the spike date range against:
- PR/content releases (viral organic content)
- Competitor outages (branded search spikes)
- Bot/scraper activity (typically shows zero engagement depth)

```
get_company_sessions(
  account_domain="<any_high-volume_domain>",
  date_from="{spike_start}",
  date_to="{spike_end}"
)
```

If sessions from the spike period show session depth of 1 (single page, immediate
bounce) at scale — this is bot/scraper traffic. Do NOT inject a correction for bot
traffic. The correct action is filtering, not MMM prior adjustment.

**Decision threshold for Trap B correction:**
- Organic spike > 3× the 30-day rolling average
- Paid spend flat (< 5% change)
- No bot signal (session depth > 1.5 average)
- Spike lasted ≥ 3 days

If all four conditions are met, the spike has inflated the MMM's counterfactual
baseline, making paid lift appear larger than reality. Proceed to correction.

---

## Step 3 — Calibration pivot: inject correction weights

This step adjusts the Meridian prior arrays so the contaminated period does not
bias the next optimization cycle.

### 3a. Retrieve the current correction weight vector

The `v_attribution_correction_weights` view computes a `correction_multiplier`
per channel per week (range: 0.60 – 1.0, where 1.0 = no correction needed):

```sql
SELECT
  flagged_platform         AS channel,
  anomaly_week,
  correction_multiplier,
  orphaned_token_count,
  timestamp_divergence_count,
  phantom_conversion_count,
  total_anomaly_count,
  estimated_pipeline_at_risk
FROM `{project}.{dataset}.v_attribution_correction_weights`
WHERE anomaly_week BETWEEN DATE('{window_start}') AND DATE('{window_end}')
ORDER BY anomaly_week, channel;
```

This is the authoritative correction signal. The multiplier is already computed —
you are not deriving it manually.

### 3b. Interpret the correction multiplier

| Correction Multiplier | Interpretation | Action |
|---|---|---|
| 0.90 – 1.0 | Minor contamination (< 10% of conversions affected) | Low urgency; note in next MMM run briefing |
| 0.75 – 0.90 | Moderate contamination | Flag before next optimization cycle; inject as soft prior adjustment |
| 0.60 – 0.75 | Severe contamination | Block next optimization cycle for affected channel until correction is confirmed |
| < 0.60 | Should not occur; indicates data quality failure | Escalate; do not run optimization |

### 3c. Inject the correction into the Meridian prior configuration

The Meridian prior adjustments live in the private skill file
`agents/analyst/skills/private_meridian_priors.md`. The correction workflow
creates a **temporary override** for the contaminated period.

Write the correction as a structured note that the MMM analyst engine reads at
the next Meridian run. Use `trigger_agent_run` with the correction payload:

```
trigger_agent_run(
  agent="analyst",
  run_config={
    "tools": ["run_mmm_model"],
    "override_params": {
      "attribution_correction_weights": {
        "{channel}": {
          "week": "{anomaly_week}",
          "correction_multiplier": {correction_multiplier},
          "source": "forensic_trap_{trap_type}",
          "applied_by": "{operator_name}",
          "applied_at": "{timestamp}"
        }
      },
      "exclude_contaminated_weeks": ["{anomaly_week}"]
    }
  }
)
```

This tells the Meridian engine to:
1. Scale down the conversion signal for the affected channel + week by the `correction_multiplier`
2. Exclude the contaminated week from the baseline period used to compute incremental lift

**For Trap A specifically**: The correction also prevents the overwritten CRM attribution
from feeding the MTA model's last-touch credit pool. The `orphaned_token` records in
`data_attribution_anomalies` are already excluded from the standard attribution path
joins — but the Meridian MMM doesn't see session-level data. The correction multiplier
is the bridge.

**For Trap B specifically**: The exclude window prevents the organic spike from inflating
the counterfactual baseline in the BSTS incrementality model. A higher counterfactual
baseline understates the paid channel's true lift.

### 3d. Verify the correction was applied

After the next Analyst run completes:

```
get_attribution_run_history(limit=3)
```

Check the most recent MMM run's `correction_applied` flag and the `roi_mean` for the
affected channel. If the correction worked correctly:
- The channel's `roi_mean` should be **lower** than the contaminated pre-correction run
  (Trap A: channel was over-credited; correction removes phantom conversions)
- OR the channel's `roi_mean` should be more **stable** across periods (Trap B: organic
  spike no longer artificially elevates the counterfactual)

---

## Step 4 — Exception handling sign-off

```
FORENSIC EXCEPTION REVIEW — [DATE]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ALERT DETAILS
Trap type:         [ ] Trap A (CRM overwrite)  [ ] Trap B (organic masking)
Flagged channel:   _______________________
Anomaly window:    _______ to _______
Anomaly count:     _______  records
Est. pipeline ARR at risk: $____________
Avg confidence score: ____________

INVESTIGATION OUTCOME
[ ] Trap A: Paid click tokens (gclid/fbclid) confirmed in session table
[ ] Trap A: CRM lead_source rewritten AFTER click timestamp (timestamp_divergence)
[ ] Trap A: Bulk import event identified (high null field rate in detect_crm_null_fields)
[ ] Trap B: Organic spike confirmed (> 3× 30-day rolling avg)
[ ] Trap B: Paid spend was flat during spike (< 5% WoW change)
[ ] Trap B: Bot traffic ruled out (avg session depth > 1.5)

CORRECTION DECISION
Correction multiplier from v_attribution_correction_weights: ____________
[ ] Multiplier ≥ 0.90 — Note only; no prior injection required
[ ] Multiplier 0.75–0.90 — Inject as soft adjustment before next MMM run
[ ] Multiplier < 0.75 — Block next optimization cycle; inject correction first

CALIBRATION ACTION
[ ] trigger_agent_run submitted with override_params correction weights
[ ] exclude_contaminated_weeks list confirmed accurate
[ ] CRM operations team notified of overwrite event (Trap A only)
[ ] Next MMM roi_mean reviewed post-correction run

Notes / escalation:
_______________________________________________
_______________________________________________

Signed off by: _______________________ Date: _______
```

---

## Notes

- **Do not run `execute_system_budget_reallocation` while a forensic correction is pending.**
  An optimization cycle with contaminated priors will execute in the wrong direction.
  Always complete the correction + re-run cycle before the next Operator execution.
- **Trap A does not necessarily mean the CRM team acted maliciously.** Bulk imports from
  trade shows, webinars, or content syndication platforms routinely overwrite lead sources
  because the CRM field isn't protected. The forensic trap catches the pattern, not intent.
  Notify CRM ops with the anomaly list so they can protect the `LeadSource` field for
  digitally-originating leads going forward.
- **The `correction_multiplier` is floored at 0.60.** Below 0.60, the data is too
  contaminated to correct mathematically. At that point, the safest path is to exclude
  the channel from the optimization package for one cycle and treat the ROI posterior
  as uninformative (widened credible interval).
- **`crm_lead_id` fields in `data_attribution_anomalies` are opaque system IDs** — they
  are CRM account/lead record IDs, not emails or personal identifiers. They can be passed
  to CRM ops for record-level investigation without exposing PII.
- Watchdog alert records are retained in `watchdog_alerts` until manually closed. After
  completing this workflow, close the alert:
  ```
  get_pending_approvals()
  # Locate the watchdog alert resolution action and approve it
  ```

---

*© 2026 @arcticgreyy — Business Source License 1.1. Persistent attribution required.*
*See paid-media-agent/LICENSE and paid-media-agent/NOTICE for terms.*
