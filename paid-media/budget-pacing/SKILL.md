---
name: paid-media/budget-pacing
description: >
  Use this skill whenever the user asks about budget pacing, spend tracking, or budget
  management for active campaigns. Triggers include: "check pacing", "are we on pace",
  "pacing report", "we're overpacing", "we're underpacing", "budget will run out early",
  "budget won't spend", "daily budget check", "flight pacing", "monthly pacing",
  "end of quarter budget", "budget adjustment", "reallocate budget", "how much have we
  spent", "projected spend", "pace to goal", or any question about whether campaigns
  are spending at the right rate. Also use for budget-related optimization decisions
  mid-flight.
  Requires the paid-media-mcp server to be connected for org-specific data.
---

# Budget Pacing

Pacing is the discipline of ensuring campaigns spend at the right rate — not too fast
(wasting budget by burning out early), not too slow (leaving money on the table at flight end).
Pacing analysis is a daily responsibility for active campaigns.

---

## Step 1 — Pull current pacing data

From paid-media-mcp:
1. `get_team_performance` or `get_campaign_performance` for the current flight period
2. `list_campaigns` filtered by status "active" — get budget and flight dates
3. `get_benchmarks` — for context on typical spend rates by platform

Calculate pacing for each campaign (see formulas below).

---

## Step 2 — Pacing formulas

### The core pacing check

```
Days Elapsed    = Today - Flight Start Date
Days Remaining  = Flight End Date - Today
Total Flight Days = Days Elapsed + Days Remaining

Expected Spend (today) = Total Budget × (Days Elapsed / Total Flight Days)
Actual Spend (to date) = Sum of spend from performance data

Pacing %        = Actual Spend / Expected Spend × 100

Interpretation:
  85–115%  = ON PACE  ✓
  >115%    = OVERPACING — will exhaust budget early
  <85%     = UNDERPACING — will miss delivery; budget left unspent
```

### Projected total spend

```
Daily Burn Rate   = Actual Spend / Days Elapsed
Projected Total   = Daily Burn Rate × Total Flight Days

Projected vs. Budget = (Projected Total / Total Budget - 1) × 100
```

### Daily budget check (always-on campaigns)

For always-on campaigns with a monthly budget:

```
Days in Month      = e.g. 30
Day of Month       = Today's date
Budget Spent       = MTD spend
Daily Budget       = Monthly Budget / Days in Month
Expected MTD Spend = Daily Budget × Day of Month

Pacing % = Actual MTD Spend / Expected MTD Spend × 100
```

---

## Step 3 — Pacing status classification

| Status | Pacing % | Action |
|--------|----------|--------|
| ✅ On Pace | 90–110% | Monitor. No action needed. |
| ⚠️ Slight Overpace | 110–125% | Watch daily. Consider tightening bids. |
| 🔴 Overpacing | >125% | Reduce daily budget or tighten bids now. |
| ⚠️ Slight Underpace | 75–90% | Watch daily. Check for delivery issues. |
| 🔴 Underpacing | <75% | Diagnose delivery issue. Increase daily budget or loosen bids. |
| 🔴 Critical Underpace | <50% | Immediate action. Campaign may not spend. |

---

## Step 4 — Diagnosing pacing problems

### Overpacing causes and fixes

| Cause | Diagnosis | Fix |
|-------|-----------|-----|
| Daily budget set too high | Spend = budget cap daily | Reduce daily budget to Expected Daily Spend |
| Bid too aggressive | CPC/CPM lower than forecast | Reduce bid or add bid cap |
| Audience too broad | Cheap inventory; high volume | Narrow targeting or add exclusions |
| Campaign front-loaded | Platform spending early in month | Switch to "standard" delivery pacing in platform settings |

**Quick fix for overpacing**:
Calculate the new daily budget needed to land on target:
```
New Daily Budget = (Total Budget - Actual Spend) / Days Remaining
```
Set this as the new daily budget in the platform.

### Underpacing causes and fixes

| Cause | Diagnosis | Fix |
|-------|-----------|-----|
| Audience too narrow | Small reach; budget can't clear | Expand audience or add lookalike layer |
| Bid too conservative | Winning few auctions | Increase bid or switch to auto-bid |
| Ad disapproved | Check platform notifications | Fix creative/copy and resubmit |
| Budget-limited but CPM too high | Reaching budget cap early but frequency is low | Something is wrong — check ad delivery |
| Flight not started correctly | Start date in past but spend = $0 | Check campaign/ad set status; ensure ads are active |

**Platform-specific underpacing notes**:
- **Meta**: If an ad set is "Learning" (< 50 optimization events), performance and delivery
  will be inconsistent. Give it 7 days before making pacing changes.
- **Google Ads**: Check "Search Impression Share Lost (Budget)" — if >20%, budget is limiting.
  Increase daily budget or improve Quality Score.
- **DV360**: Check if a flight's pacing mode is "ASAP" vs. "Even". Even pacing spreads spend
  uniformly. If your line item isn't spending, check inventory availability and frequency caps.
- **LinkedIn**: Minimum daily budget requirements ($10 for most objectives) mean very small
  campaigns may hit delivery limits. Also check audience size — must be >50K.

---

## Step 5 — Platform pacing behaviors

Each platform handles pacing differently:

### Google Ads
- **Standard delivery**: evenly distributes budget throughout the day (recommended)
- **Accelerated delivery**: deprecated. No longer available for most campaign types.
- **Campaign budget vs. shared budget**: shared budget can be cannibalized by other campaigns.
  Check if a shared budget is causing the underpace.
- **Smart campaigns / PMax**: Google controls pacing internally. If underspending, check
  bid strategy constraints (target CPA too low, target ROAS too high).

### Meta (Facebook/Instagram)
- **Accelerated delivery**: available at ad set level; useful for time-sensitive offers
  but can exhaust budgets very quickly.
- **Campaign Budget Optimization (CBO)**: Meta distributes budget across ad sets.
  If one ad set is underpacing, Meta has decided the others are more efficient.
  Check ad set minimum/maximum spend limits.
- **Dayparting**: if ad set runs only certain hours, budget won't be fully used in
  non-active hours. Pacing formula needs adjustment for active hours only.

### DV360
- **Flight pacing**: "Even" (default), "Ahead", "ASAP". Even is safest.
- **Insertion Order vs. Line Item budget**: line items can exhaust their allocation before
  the IO flight ends if not set to "auto-budget".
- **Inventory scarcity**: if a line item targets a narrow private marketplace deal, limited
  inventory can cause underpacing. Check win rate in Reporting.

### LinkedIn
- **Standard vs. Accelerated pacing**: Standard = even delivery; Accelerated = as fast as possible.
- **Campaign Group budget**: similar to CBO — LinkedIn allocates across campaigns.
  Set budgets at campaign level for more control.

### TikTok
- **Standard vs. Accelerated**: Standard recommended for sustained flights.
- **Smart Budget**: TikTok's equivalent of CBO. Can cause erratic pacing early in flight.
- Learning phase: first 20 optimization events per ad group. Pacing is unpredictable.

---

## Step 6 — End-of-period pacing

### End of quarter / end of month pressure

When significant budget remains in the last week of a flight:

**Options in order of preference**:
1. **Increase daily budget**: divide remaining budget by days left. Cleanest option.
2. **Switch to accelerated delivery**: only if you need to burn budget fast and
   don't care about efficiency degradation.
3. **Expand targeting temporarily**: add an additional audience layer or loosen
   geographic targeting to access more inventory.
4. **Launch a new ad set/campaign**: with the remaining budget and broad targeting,
   targeting top-of-funnel reach.
5. **Reallocate to an always-on campaign**: if the budget is flexible, move it to
   a proven performer rather than burning it on a poorly-pacing campaign.

**When NOT to force spend**:
- Do not artificially inflate spend by lowering quality thresholds.
- Do not switch to awareness/reach objectives if the campaign is set up for conversions —
  the data will pollute your performance history.
- If <5 days remain and <30% of budget is spent: acknowledge the miss. Don't make
  reckless changes that damage data quality for the next flight.

---

## Step 7 — Pacing output format

### Pacing report table

Present for each active campaign:

| Campaign | Platform | Budget | Spent | Expected | Pacing % | Projected | Days Left | Status | Action |
|----------|----------|--------|-------|----------|----------|-----------|-----------|--------|--------|
| EMEA Brand | DV360 | $50,000 | $22,000 | $25,000 | 88% ⚠️ | $44,000 | 10 | Underpace | Increase daily budget to $2,800 |
| US Search | Google | $30,000 | $18,500 | $17,000 | 109% ✅ | $33,300 | 9 | On Pace | Monitor |
| LinkedIn LG | LinkedIn | $20,000 | $5,000 | $10,000 | 50% 🔴 | $10,000 | 20 | Critical | Check ad status + audience size |

### Summary metrics
- Total budget across flights: $XXX,XXX
- Total spent: $XXX,XXX (XX% of budget)
- Projected total spend: $XXX,XXX (XX% of budget)
- Campaigns overpacing: N
- Campaigns underpacing: N
- Reallocation opportunity: $X,XXX (from over-pacing campaigns to under-pacing)

---

## Notes

- Pacing checks should happen at least daily for campaigns with daily budgets >$1,000.
  For smaller campaigns, weekly is sufficient.
- Platform-reported pacing in dashboards often has a 24–48h data lag. Always verify
  with the most recent data from `get_campaign_performance`.
- For automated reallocation based on attribution data, use `reallocate_media_budget` —
  but only after checking pacing first. Don't reallocate TO a campaign that's already overpacing.
- Never adjust daily budgets by more than 20% at a time. Larger changes can knock smart
  bidding algorithms out of their learning state.
