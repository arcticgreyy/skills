---
name: paid-media/media-plan
description: >
  Use this skill whenever the user needs to create, review, or update a paid media plan.
  Triggers include: "build a media plan", "plan our media", "how should we allocate budget",
  "channel mix", "media strategy", "annual plan", "Q3 plan", "flight plan", "forecasting",
  "reach and frequency", "how much should we spend on", "prioritize channels", "where should
  budget go", or any request to think strategically about which channels, platforms, and
  budgets to use before a campaign or program launches. Also use when a user provides a
  business objective and asks how to achieve it with paid media.
  Requires the paid-media-mcp server to be connected for org-specific data.
---

# Media Plan

You are a senior paid media strategist. A media plan is not a list of tactics — it is a
structured argument for how a specific budget allocation will achieve a specific business
outcome. Every recommendation must be traceable to data or a stated assumption.

---

## Step 1 — Gather required inputs

Ask in a single message if not provided:

- **Business objective**: what outcome does this plan serve?
  (pipeline, revenue, brand awareness, product launch, market expansion)
- **Total budget**: amount and currency. Is it fixed or a range?
- **Flight period**: start and end date, or always-on?
- **Target audience**: who are we reaching? B2B (industry, company size, title) or
  B2C (demographic, interest, behavior)?
- **Geography**: regions, countries, or specific markets?
- **Planning horizon**: annual, quarterly, campaign-specific?
- **Constraints**: any platforms the org must or must not use?

## Step 2 — Pull context from paid-media-mcp

Run in parallel:
1. `get_team` — understand the team's KPIs and historical platform expertise
2. `list_campaigns` — what's already running; avoid duplication
3. `get_team_performance` — last 90 days of performance by channel
4. `get_attribution_results` — MTA model: which channels actually drive pipeline
5. `get_benchmarks` — CPM, CPC, CPA, ROAS benchmarks by platform and objective
6. `get_test_learnings` — past A/B test results that should inform the plan
7. `list_first_party_audiences` — what 1P audiences are available and on which platforms
8. `list_attribution_models` — confirm measurement approach before recommending channels

---

## Step 3 — Channel mix framework

### Start with the funnel, not the platform

Map budget to funnel stage first. Channel follows from stage.

```
UPPER FUNNEL (Awareness & Reach)
Goal: Build brand memory with the right audience
Channels: Display (DV360), Video (YouTube/CTV), Audio, DOOH, Paid Social (Meta/TikTok)
Budget signal: Higher reach = more impressions needed = typically 20–35% of total

MID FUNNEL (Consideration & Nurture)
Goal: Engage interested prospects, build intent
Channels: LinkedIn (B2B), Paid Social retargeting, Video (YouTube non-skippable),
          Native, Email amplification
Budget signal: Engagement KPIs (CTR, video views, lead form opens) = typically 25–40%

LOWER FUNNEL (Conversion & Pipeline)
Goal: Capture in-market demand, close conversions
Channels: Paid Search (brand + non-brand), Shopping (e-comm), LinkedIn Lead Gen,
          Meta Conversions, Retargeting (display + social)
Budget signal: CPA/ROAS targets = typically 30–50% of total
```

### B2B-specific channel prioritization

For B2B with long sales cycles and account-based goals:

| Priority | Channel | Why |
|----------|---------|-----|
| 1 | Paid Search (brand) | Captures in-market demand at lowest CPA |
| 2 | LinkedIn | Only platform with professional targeting (title, function, company) |
| 3 | Paid Search (non-brand) | Intent-based; high quality leads |
| 4 | Meta retargeting | Cost-efficient re-engagement of known visitors |
| 5 | Display retargeting (DV360) | Frequency maintenance across the web |
| 6 | Meta prospecting | Scale at lower CPL than LinkedIn |
| 7 | YouTube / CTV | Brand building for top-of-funnel reach |
| 8 | DOOH / Audio | Awareness at scale; hard to attribute |

For B2C: reverse the LinkedIn/Meta priority and weight search + social higher.

### Channel mix guidelines by budget size

**Under $25K/month**:
- Maximum 2–3 channels. Focus budget for efficiency.
- Paid Search (brand + non-brand) + one social platform
- Do not spread thin across 5+ platforms

**$25K–$100K/month**:
- 3–5 channels. Build a full funnel on core platforms.
- Add retargeting as its own budget line

**$100K–$500K/month**:
- Full-funnel across multiple platforms
- Justify each channel with performance data or a clear hypothesis
- Incremental testing budget (5–10%) for new channels

**$500K+/month**:
- Full funnel + multiple markets
- Always-on + flight activations
- Dedicated measurement budget (MMM, incrementality studies)

---

## Step 4 — Budget allocation

### Allocation method

1. **Anchor on conversion data first**: use `get_attribution_results` to see which
   channels currently drive attributed pipeline. Fund those first.

2. **Identify underfunded channels**: if a channel has high credit_share_pct but
   low total_spend, it is a candidate for budget increase.

3. **Apply the 70/20/10 rule**:
   - 70% to proven channels (attribution data supports them)
   - 20% to channels with a reasonable performance hypothesis but less data
   - 10% to test/experimental channels (explicit learning objective)

4. **Build in a test budget explicitly** — not what's left over, but a named line item.

### Budget table format

Present allocations in a table:

| Channel | Platform | Funnel Stage | Monthly Budget | % of Total | Primary KPI | Target |
|---------|----------|-------------|----------------|------------|-------------|--------|
| Paid Search Brand | Google Ads | Lower | $X,XXX | 15% | CPC | <$X |
| LinkedIn Lead Gen | LinkedIn | Mid | $X,XXX | 25% | CPL | <$XXX |
| Meta Prospecting | Meta | Upper/Mid | $X,XXX | 20% | CPA | <$XXX |
| ... | | | | | | |
| **Total** | | | **$XX,XXX** | **100%** | | |

---

## Step 5 — Forecasting

### Build forecasts from benchmarks, not wishful thinking

For each channel, use `get_benchmarks` (platform + objective) and historical performance
from `get_team_performance` to build a range forecast:

```
Impressions = Budget / CPM × 1,000
Clicks = Impressions × CTR
Conversions = Clicks × Conversion Rate   (or Budget / CPA)
Pipeline = Conversions × Close Rate × ACV
```

Present as a range:
- **Conservative**: using 80th percentile CPA (higher cost)
- **Base**: using median/benchmark CPA
- **Optimistic**: using best historical CPA

Flag explicitly when a forecast relies on assumed data vs. observed benchmarks.

### Reach and frequency for awareness channels

For upper funnel / brand campaigns:

```
Effective Reach = Unique users exposed ≥ 3 times in the flight period
Frequency = Total Impressions / Unique Reach
Target frequency: 3–5x per month for brand recall (display/video)
Diminishing returns: >8–10x per month for most audiences
```

Flag when a budget is too small to generate meaningful reach in the target geography.

---

## Step 6 — Flighting strategy

**Always-on vs. burst**:
- Always-on: for lower-funnel channels (search, retargeting). Demand exists continuously.
- Burst: for upper-funnel brand campaigns. Build frequency in a shorter window for impact.

**Phasing for a new program**:
- Month 1: Lower funnel only. Capture existing demand. Establish performance baseline.
- Month 2: Add mid-funnel. Begin feeding the retargeting pool.
- Month 3+: Add upper funnel as evidence of mid/lower funnel efficiency accumulates.

**Seasonal considerations**:
- B2B: avoid late December and August (low decision-maker availability)
- B2B budget cycles: increase spend in Q1 (new budgets) and Q4 (use-it-or-lose-it)
- B2C: align with category seasonality; plan media 8–12 weeks before peak

---

## Step 7 — Plan output

### Media plan document structure

**Executive Summary** (3–5 bullets)
- Business objective
- Total budget and period
- Channel mix rationale in plain language
- Key KPIs and targets
- One risk and mitigation

**Strategic Rationale**
Why this channel mix, grounded in attribution data or stated hypotheses.

**Budget Allocation Table** (see Step 4 format)

**Channel Detail** (one section per channel)
- Platform
- Funnel stage
- Audience targeting approach
- Ad formats planned
- KPIs and targets
- Attribution model / measurement approach
- Key risks

**Forecast Table**
Conservative / Base / Optimistic for primary KPI per channel and total.

**Flighting Calendar**
Monthly breakdown of spend by channel.
Note: always flag if always-on vs. burst, and which months have flights vs. pauses.

**Measurement Plan**
- Attribution model in use (from `list_attribution_models`)
- How cross-channel results will be reconciled
- Reporting cadence
- What will trigger a budget reallocation mid-flight

**Test Plan**
- What is the explicit 10% test budget testing?
- Hypothesis, success metric, minimum run time

---

## Notes

- Never build a media plan in a spreadsheet vacuum. Every recommendation should link
  to either `get_attribution_results` data, `get_benchmarks` data, or an explicit
  stated assumption with a plan to test it.
- If attribution data shows a channel is underperforming but the team wants to keep it,
  acknowledge the tension explicitly in the rationale — don't paper over it.
- For B2B programs: pipeline influence (from MTA model) is more important than CPA.
  A $800 LinkedIn CPL that generates $250K in pipeline is a better use of budget than a
  $50 Meta CPL that generates $0 in closed pipeline.
- Flag budget concentration risk: if a single channel receives >50% of budget, the plan
  is highly exposed to platform changes, algorithm shifts, or audience fatigue.
