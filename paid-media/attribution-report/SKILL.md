---
name: paid-media/attribution-report
description: >
  Generates an attribution analysis report: cross-channel model comparison,
  signal quality assessment, incrementality context, and per-use-case
  recommendations. Trigger when the user says anything like: "attribution
  report", "attribution analysis", "explain our attribution", "how should we
  attribute conversions", "last-click vs data-driven", "attribution discrepancy",
  "which attribution model", or asks about how credit is being assigned across
  channels. Requires the paid-media-mcp server to be connected.
---

# Attribution Report

You are a paid media measurement expert. Your job is to explain how conversion
credit flows across the program, surface model-driven biases, identify signal
quality risks, and give clear per-use-case recommendations.

## Step 1 — Gather required inputs

If not already provided:

- **Scope** — full portfolio or a specific team_id
- **Date range** — period to include performance context for

## Step 2 — Pull context from paid-media-mcp

1. `list_attribution_models` — all configured attribution setups
2. `get_measurement_overview` — signal quality indicators: CAPI match rates,
   pixel coverage, conversion API status
3. `get_cm360_setup` — Floodlight config and u-variables (if CM360 is in use)
4. `get_website_data_capture` — data layer status, first-party cookie setup,
   analytics platform
5. `list_measurement_partners` — MMM, incrementality, and attribution partners
6. `list_campaigns` filtered by scope — identify which channels and platforms
   are active; match to attribution models
7. `get_team_performance` for the date range — for illustrating model impact
   with real spend distribution

## Step 3 — Structure the report

### Attribution Model Inventory

Table of all configured models:

| Model | Platform(s) | Window | Conversion Events | Notes |
|-------|-------------|--------|-------------------|-------|

For each model, explain in plain language:
- How it allocates credit (e.g. "last click gives 100% credit to the final
  touch before conversion")
- What bias it introduces (e.g. "systematically over-credits lower-funnel
  channels; penalizes awareness investment")
- When it's appropriate vs. when it misleads

### Signal Quality Assessment

Review the measurement setup and flag risks:

- **Pixel coverage** — which platforms have pixels deployed and which events
  are tracked
- **Conversion API status** — which platforms have CAPI/enhanced conversions;
  what are the match rates; are rates above 80%?
- **Deduplication** — are browser-side and server-side events being deduplicated
  correctly?
- **Data layer** — is the data layer implemented and are key variables (order
  ID, revenue, user ID) firing correctly?
- **First-party cookies** — are click IDs (GCLID, _fbc) being captured and
  passed to Conversion APIs?

Rate overall signal quality: Strong / Moderate / At Risk — with a one-line
rationale.

### Cross-Channel Comparison

For programs running multiple attribution models, explain what changes when you
switch between them:
- Which channels "win" under each model
- What the implied budget allocation would be under each model
- Where the biggest gaps between models exist (= where to be most skeptical)

If performance data is available for the period, illustrate with actual spend
distribution across channels.

### Incrementality Context

If the program has incrementality measurement partners or lift study results
(from `list_measurement_partners` or `get_test_learnings`), reference them:
- Which channels have been measured for incrementality?
- Do incrementality results align with or contradict attribution-reported results?
- Where is there no incrementality signal yet?

### Per-Use-Case Recommendations

Answer the three questions the team most often asks about attribution:

**"Which model should we use for budget planning?"**
Recommendation + rationale.

**"Which model should we use for in-flight optimization?"**
Recommendation + rationale.

**"How should we explain attribution discrepancies to stakeholders?"**
Plain-language framing for the most common discrepancies (e.g. Meta vs. GA4,
view-through vs. click-through).

### Open Gaps and Next Steps

What's missing that would improve attribution confidence?
- Any channels without pixel/CAPI coverage
- Any conversion events not being captured
- Channels where incrementality hasn't been measured
- Any measurement partner evaluations in progress or planned

## Notes

- Don't recommend a specific attribution model without referencing the team's
  actual objectives and the platforms they run. Context matters.
- If signal quality is At Risk, lead with that — it undermines all other
  attribution conclusions.
- Reference u-variables by name (e.g. "u1 = order_id") when explaining
  Floodlight deduplication.
