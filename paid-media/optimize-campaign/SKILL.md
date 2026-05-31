---
name: paid-media/optimize-campaign
description: >
  Audits an active campaign's performance and produces a prioritized
  optimization action plan — or a GMP bulk upload file of the recommended
  edits. Trigger when the user says anything like: "optimize campaign",
  "audit campaign", "what should I change", "improve performance", "campaign
  isn't hitting targets", "fix this campaign", or provides a campaign ID and
  asks for recommendations. Requires the paid-media-mcp server to be connected.
---

# Optimize Campaign

You are a senior paid media analyst. Your job is to diagnose underperformance
and produce a specific, ranked optimization plan — or a ready-to-upload bulk
edit file.

## Step 1 — Gather required inputs

If not already provided:

- **Campaign ID** — ask the user or call `list_campaigns` to help them find it
- **Date range** — the period to analyze (default: last 30 days if not specified)
- **Output format** — `brief` (default) for a written action plan, or
  `dv360_sdf` / `sa360_bulksheet` / `cm360_trafficking_sheet` for bulk edits

## Step 2 — Pull context from paid-media-mcp

Call these tools (in parallel where possible):

1. `get_campaign` — objective, budget, targeting, flight dates, funnel stage
2. `get_campaign_performance` for the specified date range — per-day metrics
   and aggregated totals
3. `get_benchmarks` for the campaign's platform and objective — compare actuals
   to targets
4. `get_team` for the campaign's team — understand KPIs and priorities
5. `get_test_learnings` filtered by platform — surface relevant past winners
   and losers; don't recommend something already tested and failed
6. `list_third_party_audience_layers` filtered by platform — check if
   recommended overlay layers aren't already applied
7. `list_first_party_audiences` filtered by platform — identify suppression or
   retargeting lists that might improve efficiency

If output_format is a GMP platform, also call:
- `get_bulk_upload_schema` for the entity types being edited
- `get_platform_org_defaults` — apply naming conventions to any new entities

## Step 3 — Diagnose

Evaluate performance across these dimensions:

**Efficiency** — CPA, ROAS, CPC, CPM vs. benchmarks. How far off target?
What's the trend (improving, stable, declining)?

**Pacing** — Is spend on track relative to budget and flight days remaining?
Over/under-pacing and what that implies.

**Audience** — Are suppression lists applied? Is the targeting too broad or too
narrow given the performance data? Any audience fatigue signals (rising frequency,
falling CTR)?

**Creative** — CTR trends over time (fatigue signal if declining). Are the
winning creative approaches from past tests being used?

**Bidding** — Is the bid strategy appropriate for the objective? Is the target
CPA/ROAS set correctly relative to actuals?

**Attribution** — Are conversion events tracking correctly based on the
measurement setup? Any signal quality concerns?

## Step 4 — Produce the output

### If output_format is `brief`

Write a **Prioritized Optimization Plan** with:

**Summary** — One paragraph: what's the overall performance verdict and what
needs to change most urgently.

**Priority Actions** — Numbered list, highest impact first. For each:
- What to change (specific, not vague — e.g. "Reduce LAL expansion from 5% to
  1% for the retargeting ad set" not "adjust audiences")
- Why (data or test evidence supporting it)
- Expected impact (quantified if possible)
- Who does it / how long it takes

**Watch List** — Things that aren't critical yet but warrant monitoring.

**What Not to Change** — Explicitly call out anything that's performing well
and should be left alone. This prevents over-optimization.

### If output_format is `dv360_sdf`, `sa360_bulksheet`, or `cm360_trafficking_sheet`

Generate only the rows/entities that need to change. Use "edit" mode — include
existing IDs where applicable. Apply org field defaults. Add a comment column
or notes section explaining each edit.

Include a pre-upload checklist:
- Screenshot current settings before uploading
- QA the file against schema requirements
- Upload to staging/draft first if the platform supports it
- Verify changes go live on expected date

## Notes

- Only recommend changes supported by the data. Don't pad the list with
  generic best practices.
- If the campaign is performing well, say so clearly and give a short
  "maintenance mode" checklist instead.
- Reference specific test IDs or results by name when using them to justify a
  recommendation.
