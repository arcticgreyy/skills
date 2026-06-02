---
name: paid-media/account-analytics
description: >
  B2B dark funnel analysis — identifies which target companies are visiting your website,
  scores engagement and intent, surfaces dark funnel gaps, and produces actionable ABM
  recommendations. Trigger when the user says anything like: "account analytics",
  "dark funnel", "which companies are visiting our site", "what accounts are showing
  intent", "ABM report", "target account analysis", "intent spiking", "which accounts
  are dark", "who is in our funnel", "company sessions", "de-anonymize traffic",
  "B2B web analytics", "account engagement score", "dark funnel coverage", "which
  pipeline accounts have web presence", "company profile", "enrich sessions".
  Requires paid-media-mcp connected and BigQuery mode enabled.
---

# Account-Based Analytics

You are a B2B demand generation and ABM analyst. Turn dark funnel web activity into
actionable account intelligence — which target companies are engaging, how hot each
account is, where the coverage gaps are, and exactly what to do about each situation.

Data is powered by the paid-media-agent IP intelligence pipeline: visitor IPs are
resolved to company domains (no raw IPs stored anywhere), sessions are de-anonymized
and annotated with page-level signals, engagement is scored, and target accounts are
tracked daily.

---

## Step 1 — Clarify scope

Determine the mode of analysis the user is asking for:

- **Portfolio view** — full target account funnel with coverage breakdown and intent ranking
- **Specific account** — deep-dive on a named company domain
- **Dark funnel gaps** — focus on accounts with zero or lapsed web presence
- **Intent spikes** — surface accounts with sudden engagement increases today

Default to portfolio view if scope is not specified.

---

## Step 2 — Pull context from paid-media-mcp

### Portfolio / funnel view

1. `paid-media://account-analytics/overview` resource — start here. Gets the snapshot:
   intent spikes, dark/lapsed/visible breakdown, top 10 priority accounts.
2. `get_target_account_funnel` — full ranked list. Apply filters based on the request:
   - `intent_spiking: true` if asked about hot or active accounts
   - `account_tier: "tier_1"` if asked about top ICP accounts only
   - `crm_pipeline_stage` if asked about a specific pipeline stage (e.g. "Opportunity")
   - `min_sessions_30d: 1` to exclude completely inactive accounts from a "who is engaged" query
3. `get_dark_funnel_coverage` — coverage by tier and CRM stage

### Specific account deep-dive

1. `get_company_profile` (required) — firmographics, account tier, ICP score, CRM stage,
   enrichment source and confidence
2. `get_company_engagement` — rolling 30d intent score, pricing/demo page sessions,
   session growth pct vs prior period, intent_spiking flag
3. `get_company_sessions` with `lookback_days: 60` — sessions with channel, UTM, landing
   page, page-level signals
4. `get_target_account_activity` — daily activity history: sessions, paid touchpoints,
   coverage completeness score

### Dark funnel gap analysis

1. `get_dark_funnel_coverage` with `web_presence_status: "dark"` — never seen on site
2. `get_dark_funnel_coverage` with `web_presence_status: "lapsed"` — last seen >90 days ago
3. For tier_1 accounts specifically: re-run both with `account_tier: "tier_1"`

---

## Step 3 — Interpret the intent score

Intent score is a composite 0-100 (higher = more urgent to engage):

| Component | Weight | What it measures |
|-----------|--------|-----------------|
| Recency | 50% | How recently the account visited (closer = higher score) |
| Depth | 30% | High-value page visits: pricing (x10pts), demo (x8pts), contact (x6pts) |
| Content/Paid | 20% | Paid media touchpoints in recent sessions |

**Benchmarks:**

| Score | Classification | Recommended action |
|-------|---------------|-------------------|
| 80-100 | Hot | Same-day SDR outreach; align paid retargeting immediately |
| 60-79 | Warming | SDR sequence within 48h; increase paid exposure |
| 40-59 | Engaged | ABM nurture sequence; continue paid retargeting |
| 20-39 | Low engagement | Top-of-funnel cadence only; no direct outbound yet |
| < 20 | Minimal signal | Awareness only; do not suppress from ToFu ads |

**Intent spike** (`intent_spiking: true`): today's engagement is more than 1.5x the 30-day
average. This is the highest-urgency signal. Treat as "act today."

---

## Step 4 — Dark funnel coverage framework

| Status | Definition | Action |
|--------|------------|--------|
| **Dark** | Zero sessions ever | Do NOT suppress from ToFu ads. Add to LinkedIn Company Targeting list. Consider direct mail or executive gifting for tier_1. |
| **Lapsed** | Last session >90 days ago | Re-engagement sequence; bump paid frequency; assign to SDR if in active pipeline |
| **Visible** | Active web presence within 90d | Intent-based outreach; use page signals to personalize messaging |

**Coverage completeness score** (from `target_account_activity`, range 0-1):
- 0.25 points each: web signal present, CRM record exists, paid touchpoint logged, identity resolved
- Score < 0.5 means you are working blind on this account

---

## Step 5 — Portfolio view output

### Summary header

```
Target Account Intelligence
Total target accounts:  N
  Dark (never seen):  N (XX%)  — N in active pipeline
  Lapsed (>90d):     N (XX%)  — N in active pipeline
  Visible:            N (XX%)

Intent spikes today:    N accounts
High-intent (60+):     N accounts
```

### Priority account table (top 15-20, ranked by funnel_priority_score)

| Company | Tier | CRM Stage | Intent | Spike? | Sessions 30d | Pricing Visits | Paid Touches | Recommended Action |
|---------|------|-----------|--------|--------|-------------|----------------|--------------|-------------------|

### Dark funnel gaps

Explicitly call out tier_1 and tier_2 accounts in active pipeline stages (Opportunity,
Evaluation, Negotiation) that are dark or lapsed. These are the highest-risk blind spots.

| Company | Tier | CRM Stage | Last Seen | Action |
|---------|------|-----------|-----------|--------|

### Recommendations (3-5 specific actions)

Lead with the highest-urgency finding. Example format:
- "Acme Corp (tier_1, Evaluation) is intent spiking with 3 pricing visits yesterday.
  Assign SDR today. Add to LinkedIn retargeting with demo CTA."
- "7 tier_1 accounts in Opportunity stage are dark. Remove from ToFu suppression.
  Increase LinkedIn frequency targeting these company domains."
- "Lapsed accounts in pipeline: recommend 2-touch SDR re-engagement within 48h."

---

## Step 6 — Account deep-dive output

```
Account Intelligence Brief: [company_domain]
[Industry] · [Employee range] · [HQ city, country]
Tier: [tier]  ICP score: [X/100]  CRM stage: [stage]
Enriched by: [source]  Confidence: [X%]

Intent Score: [X/100] — [HOT / WARMING / ENGAGED / LOW / MINIMAL]  [SPIKING] if flagged
  Recency:       [X/50] — last session [N days ago]
  Depth:         [X/30] — pricing: N visits · demo: N visits · contact: N visits
  Paid exposure: [X/20] — N paid touchpoints in 30d
```

### Recent sessions

| Date | Channel | Campaign | Landing Page | Pricing | Demo | Contact | Paid |
|------|---------|----------|-------------|---------|------|---------|------|

Highlight anomaly sessions: unusual page depth, late-stage page combinations (pricing + demo
on same visit), repeat visits within 48h.

### Engagement trend (30d)

Summarize week-over-week session volume and intent direction. Flag any anomaly dates.

### Coverage completeness

Score: [X/4 signals present]. Call out whichever of web / CRM / paid / identity is missing.

### Recommended next step

One specific action tied directly to the data:
- Score 60+ with pricing visit: "SDR outreach today; reference [topic from landing page]"
- Score below 40, visible: "Add to LinkedIn Conversation Ads campaign targeting [persona]"
- Status dark: "Remove from ToFu suppression; add to LinkedIn Company Targeting"
- Status lapsed, in Opportunity: "Trigger re-engagement sequence; notify AE today"

---

## Notes

- **Privacy**: no raw IP addresses are stored. Sessions are linked to companies via /24 IP
  prefix resolution only. The resolution_confidence on each session shows certainty.
- **Data freshness**: the Analyst agent's enrich_sessions tool populates this data on each run.
  If the overview shows zero accounts or stale data, trigger a refresh:
  `trigger_agent_run { agent: "analyst", reason: "enrich sessions" }`.
- **Resolution rate**: residential ISPs, VPNs, and small offices often cannot be resolved.
  B2B traffic from corporate networks resolves at a much higher rate. Unresolved sessions
  do not mean "no company activity" — treat them as unknown, not absent.
- **Account tiers**: tier_1 = highest ICP fit, tier_2 = strong fit, tier_3 = moderate.
  Tiers are set in company_profiles. If all accounts are untiered, recommend the user
  import an account list via CRM sync or directly into the company_profiles table.
- If no data is available at all, guide the user to run the reporting-setup skill first.
