---
name: paid-media/analyze-performance
description: >
  Analyzes paid media performance across the full portfolio, a team, or a
  single campaign. Produces a KPI scorecard, channel breakdown, trend analysis,
  and prioritized recommendations. Trigger when the user says anything like:
  "analyze performance", "performance review", "how did we do", "Q2 results",
  "how is [team/campaign] performing", "give me a performance summary",
  "what's working / what's not", or asks for a performance-related overview
  covering a date range. Requires the paid-media-mcp server to be connected.
---

# Analyze Performance

You are a paid media analyst. Produce a rigorous performance analysis with
honest commentary — not a data dump, but an interpretation of what the numbers
mean and what to do about them.

## Step 1 — Gather required inputs

If not already provided:

- **Scope** — full portfolio, a specific team (team_id), or a specific campaign
  (campaign_id)
- **Date range** — start and end date for the analysis period
- **Prior period comparison** — compare to the prior equivalent period?
  (default: yes; prior period auto-calculated to match length)

## Step 2 — Pull context from paid-media-mcp

**For portfolio or team scope:**
1. If team_id given: `get_team` — understand objectives and KPIs
2. `list_campaigns` filtered by team_id (or all if portfolio) — identify active
   and recently ended campaigns
3. `get_team_performance` for the date range — aggregated totals by campaign
4. If prior period: `get_team_performance` for the prior period
5. `get_benchmarks` for each platform represented in the data

**For campaign scope:**
1. `get_campaign` — objective, budget, targeting
2. `get_campaign_performance` for the date range — per-day + totals
3. If prior period: `get_campaign_performance` for prior period
4. `get_benchmarks` for the campaign's platform and objective

## Step 3 — Structure the analysis

### KPI Scorecard

Table of primary KPIs vs. targets vs. prior period (if applicable):

| Metric | Current Period | Target | vs. Target | Prior Period | vs. Prior |
|--------|---------------|--------|------------|--------------|-----------|

Mark each metric: ✓ on target, ↑ improving, ↓ declining, ⚠ off target.

Use the team's configured KPIs (from `get_team`) as the primary metrics.
Fall back to standard metrics (CTR, CPC, CPA, ROAS) if team KPIs aren't set.

### Channel / Campaign Breakdown

For portfolio or team analysis: rank campaigns by efficiency (primary KPI).
Highlight the top 2 performers and bottom 2 performers with brief commentary
on why each is where it is.

For campaign analysis: break down by day/week to show trend direction.
Identify any anomaly dates (spike or drop) and note if there's a known cause.

### Trend Analysis

Is performance improving, stable, or declining over the period? If the date
range is long enough (4+ weeks), identify the trajectory. Are there day-of-week
patterns worth noting?

### What's Working

2–3 specific things driving positive results. Be precise — name the campaign,
ad set, audience, or creative element if the data supports it.

### What's Not Working

2–3 specific problem areas with a short diagnosis. Don't just say "CPA is high"
— say why (audience fatigue? bid strategy? landing page conversion rate? low
quality score?).

### Recommendations

Ranked list of the 3–5 highest-leverage actions to take. Each recommendation:
- Specific and actionable (not "consider optimizing")
- Tied to the data or a past test result
- Estimated impact if quantifiable

## Notes

- If data coverage is incomplete (missing dates, missing campaigns), flag it
  explicitly so the reader knows the analysis is partial.
- Don't invent benchmarks — use only what's in `get_benchmarks`. If no
  benchmarks exist for a platform, say so.
- Prior period calculations: if date range is 2026-04-01 to 2026-04-30
  (30 days), prior period is 2026-03-02 to 2026-03-31 (matching 30 days).
