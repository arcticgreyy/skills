---
name: paid-media/audience-strategy
description: >
  Use this skill for any audience planning, segmentation, or targeting task in paid media.
  Triggers include: "audience strategy", "who should we target", "audience segmentation",
  "lookalike audience", "lookalike strategy", "customer match", "1P audiences", "first-party
  audiences", "suppression list", "exclusion audiences", "audience overlap", "retargeting
  audience", "B2B targeting", "account-based targeting", "ABM", "job title targeting",
  "company targeting", "audience refresh", "audience size", "audience quality",
  "third-party data", "data provider", "audience layer", or any question about which
  audiences to target, exclude, or build across any paid media platform.
  Requires the paid-media-mcp server to be connected for org-specific data.
---

# Audience Strategy

Audience strategy determines who sees your ads. It is the targeting layer that makes
creative and budget decisions meaningful. A well-built audience strategy — across
prospecting, nurture, retargeting, and suppression — is the difference between
efficient paid media and expensive spray-and-pray.

---

## Step 1 — Pull existing audience context

From paid-media-mcp:
1. `list_first_party_audiences` — all 1P segments available and on which platforms
2. `get_lookalike_strategy` — existing LAL seeds, sizes, and best performers
3. `list_third_party_audience_layers` — available 3P overlays and default layers
4. `get_audience_library` — full picture: data providers, onboarding platforms
5. `list_campaigns` — what audiences are currently in use (avoid duplication/overlap)
6. `get_test_learnings` filtered by audience type — past audience test results
7. `get_accounts_in_open_pipeline` — current pipeline accounts (for suppression)

---

## Step 2 — Audience architecture

### The four audience tiers

Structure audiences across four tiers. Every campaign should know which tier it addresses.

```
TIER 1 — PROSPECTING (cold audience, no prior relationship)
Goal: reach new potential customers at scale
Sources: 3P data layers, platform interest/behavior targeting, LAL audiences, 
         B2B firmographic targeting (LinkedIn), contextual targeting

TIER 2 — WARM PROSPECTING (engaged but not converted)
Goal: re-engage people who have shown interest but haven't converted
Sources: website visitors (pixel-based), video viewers, social engagers,
         content downloaders, webinar attendees

TIER 3 — RETARGETING (high intent, near conversion)
Goal: convert people who have demonstrated purchase intent
Sources: product/pricing page visitors, cart abandoners (e-comm), lead form openers,
         high-value page visitors, demo requesters who didn't complete

TIER 4 — SUPPRESSION (do not show ads to)
Goal: avoid wasting spend on people who cannot or should not convert
Sources: existing customers, current pipeline accounts, recent converters,
         employees, low-quality sources (if identifiable)
```

### B2B audience architecture

For B2B programs, add an account-level layer:

```
ACCOUNT TIER — TARGET ACCOUNT LIST (ABM)
Goal: concentrate spend on a defined list of named accounts
Method: Company list targeting (LinkedIn), Customer Match (Google), CRM list (Meta)
Audience: ICP accounts that are not yet customers or in active pipeline

CONTACT TIER — WITHIN TARGET ACCOUNTS
Goal: reach the right personas inside target accounts
Method: Job function + seniority + company size (LinkedIn); CRM contact list
Key personas: Economic buyer (VP/C-suite), Technical evaluator (Manager/Director/IC), 
              Champion (end user / practitioner)
```

---

## Step 3 — First-party audience strategy

### Priority 1P segments to build and maintain

**Suppression (highest priority — build first)**:
- Existing customers: suppress from all prospecting campaigns
- Current pipeline accounts: suppress top-of-funnel ads (use `push_audience_suppression`)
- Recent converters: 30–90 day post-conversion window
- Employees: exclude from public advertising

**Retargeting (build next)**:
- All website visitors (7 days, 30 days, 90 days — separate segments)
- High-value page visitors (pricing, contact, demo request) — separate segment
- Video viewers (25%, 50%, 75%, 100% thresholds — separate segments)
- Lead form openers (not completers) — high intent

**Prospecting seeds (for lookalikes)**:
- Best customers by LTV or deal size (CRM export)
- Marketing qualified leads (MQLs) from CRM
- Closed-won opportunities (B2B)
- High-value page visitors (for lookalike audiences on social)

### 1P audience health checks

For each active 1P segment, verify:
- [ ] Size sufficient for platform delivery (Meta: >1,000; LinkedIn: >300 companies; Google: >1,000)
- [ ] Refresh cadence: is the list being updated? (check `list_first_party_audiences` for cadence field)
- [ ] Match rate: what % of CRM records match platform users?
  - Meta Customer Match: typically 50–70% match rate for B2B email lists
  - LinkedIn: typically 40–60%
  - Google Customer Match: typically 50–80%
- [ ] Attribution: are actions from matched audiences being tracked?

---

## Step 4 — Lookalike strategy

Lookalike audiences (LAL / Lookalike / Similar Audiences) let platforms find new users who
resemble your best existing customers. They are the most efficient prospecting method
when the seed audience is strong.

### Seed selection (highest impact decision)

The seed defines everything. A poor seed produces a poor lookalike.

| Seed type | Quality | Why |
|-----------|---------|-----|
| Closed-won customers (B2B) | ⭐⭐⭐⭐⭐ | Represents highest-quality outcomes |
| High-LTV customers | ⭐⭐⭐⭐⭐ | Value-weighted toward best buyers |
| MQLs or SQLs | ⭐⭐⭐⭐ | Strong intent signal |
| Demo/trial completers | ⭐⭐⭐⭐ | High product engagement |
| All leads (unqualified) | ⭐⭐ | Too broad; includes low-quality signals |
| All website visitors | ⭐ | Too diluted; use high-intent page visitors instead |

**Minimum seed sizes**:
- Meta: minimum 100 matches; recommend 1,000+ for stable lookalikes
- Google: minimum 1,000 matches
- LinkedIn: minimum 300 company matches for Lookalike Companies

### Expansion percentage

Smaller % = more similar to seed, smaller audience, higher CPM.
Larger % = broader reach, lower CPM, more diluted.

| Expansion | Typical use | Notes |
|-----------|-------------|-------|
| 1% | Initial test; highest precision | Small audience — may have delivery issues <$5K/month |
| 2–3% | Standard prospecting | Good balance of precision and scale |
| 5% | Scale phase; proven concept | Broader; monitor quality drop-off |
| 10% | Brand awareness at scale | Essentially "interest" targeting at this point |

**From `get_lookalike_strategy`**: check `best_performing_expansion` for each seed.
Use historical data — don't assume 1% is always better.

### Lookalike platform differences

**Meta**:
- Lookalike Audience: 1–10% of a country's population
- Can create from pixel events, page engagements, video views, or customer list
- Advantage Lookalike: Meta auto-optimizes the expansion during delivery
- Exclude the seed audience from the lookalike campaign (avoid showing to existing customers)

**LinkedIn**:
- Lookalike Audiences: based on company characteristics of your matched list
- Good for account-based expansion: "companies similar to our target account list"
- Limited to company-level signals (not individual behavior)

**Google**:
- Similar Audiences: deprecated for targeting. Use "optimized targeting" on PMax instead.
- Customer Match + Target CPA/ROAS: Google's equivalent — feed the signal, let the algorithm expand

**TikTok**:
- Lookalike Audience: 0–10% expansion of a custom audience
- Best seeded from high-value website events (purchase, lead form complete)

---

## Step 5 — Third-party data layers

3P data augments prospecting when 1P data is insufficient in size or when reaching
new audiences outside existing customer profiles.

### When to use 3P data

- Early-stage program with small 1P lists
- Entering a new market or vertical with no existing customers
- Awareness campaigns requiring broad reach
- B2B prospecting in a new industry vertical

### 3P data categories (B2B)

| Category | Use case | Providers |
|----------|----------|-----------|
| Firmographic | Target by company size, revenue, industry | Bombora, Dun & Bradstreet |
| Intent data | Target companies actively researching your category | Bombora, TechTarget, G2 |
| Job function / seniority | Reach specific personas | LinkedIn (native), ZoomInfo |
| Technology stack | Target companies using specific tools | Bombora, BuiltWith |
| Account-based | Reach specific named accounts | 6sense, Demandbase |

### 3P data categories (B2C)

| Category | Use case | Providers |
|----------|----------|-----------|
| Demographic | Age, income, household size | Oracle Data Cloud, Experian |
| Interest / lifestyle | Enthusiast audiences | Oracle, Acxiom, IRI |
| Purchase behavior | In-market buyers | Nielsen, Mastercard Insights |
| Geographic | Hyper-local targeting | Precisely, SafeGraph |

**From `list_third_party_audience_layers`**: check which layers are available, contracted,
and marked as default or best-performer before recommending new purchases.

---

## Step 6 — B2B account-based targeting

For B2B programs, the most effective approach combines account list targeting with
persona-level demographic targeting.

### Building a target account list

1. Define the Ideal Customer Profile (ICP): industry, company size, revenue, geography,
   technology stack, stage of business
2. Pull named accounts from CRM (pipeline + customer list for suppression)
3. Add ICP-matching accounts from 3P sources (ZoomInfo, Bombora, 6sense)
4. Remove: existing customers, current pipeline (suppress), disqualified accounts

### Account-level targeting by platform

**LinkedIn Company List**:
- Upload CSV of company names; LinkedIn matches to company pages
- Match rate: 60–80% of named companies
- Minimum 300 companies for delivery
- Layer job function + seniority on top for persona-level reach within accounts

**Google Customer Match (Account Level)**:
- Upload company domains; matches to logged-in Google users at those companies
- Match rate lower than LinkedIn (30–50%)
- Best for retargeting: show search ads to known accounts researching keywords

**Meta Company List**:
- Requires B2B-validated data partner (LinkedIn data not exportable to Meta directly)
- Upload hashed employee emails from the target accounts
- Match rate varies widely; typically 20–40% for B2B lists

**DV360 / Programmatic**:
- IP-based targeting: use account domains to match IP address ranges
- ABM data providers (6sense, Demandbase) offer DV360-compatible segment integration
- Best for display: reach the account broadly across the web

### B2B suppression workflow

When an account moves from Prospect → Open Opportunity:
1. `get_accounts_in_open_pipeline` returns current pipeline domains
2. `push_audience_suppression` adds those domains to exclusion lists on DV360/LinkedIn
3. Stops top-of-funnel ads from reaching accounts already in active pipeline
4. (Operator agent runs this automatically on a daily schedule)

---

## Step 7 — Audience overlap management

Audience overlap causes campaigns to bid against themselves and inflates CPMs.

### Detecting overlap

- **Meta**: Audience Overlap tool in Ads Manager (compare two audiences)
- **Google**: Audience Insights in Google Ads
- **LinkedIn**: No native overlap tool — manage by design

### Overlap rules

1. Prospecting and retargeting audiences **must** be mutually exclusive.
   Always exclude retargeting lists from prospecting ad sets.

2. Different retargeting tiers should be mutually exclusive:
   - Tier 3 (high intent) excludes should suppress Tier 2 (warm)
   - Tier 2 excludes should suppress Tier 1 (prospecting)

3. Suppression lists apply to ALL tiers:
   - Existing customers excluded everywhere
   - Current pipeline excluded from upper/mid funnel
   - Recent converters excluded from conversion-objective campaigns (typically 30 days)

### Exclusion stack (apply in every campaign)

```
Always exclude:
  ✗ Existing customers
  ✗ Current pipeline accounts (from get_accounts_in_open_pipeline)
  ✗ Employees
  ✗ Recent converters (30 days)

In prospecting campaigns, also exclude:
  ✗ All website visitors (last 30 days)
  ✗ Video viewers (25%+ completion, last 30 days)
  ✗ CRM leads and contacts

In awareness campaigns only (no exclusion of website visitors):
  ✗ Existing customers only
  ✗ Current pipeline
```

---

## Step 8 — Output

### Audience strategy document

**Audience inventory** (from `list_first_party_audiences`):
Table of all available 1P segments with platform availability, size, and refresh cadence.

**Gap analysis**:
- Segments that should exist but don't
- Segments that exist but aren't being used
- Segments with stale refresh cadence

**Targeting recommendations per campaign/tier**:
- Tier 1 (Prospecting): recommended targeting approach and LAL seeds
- Tier 2 (Warm): which engagement signals to retarget
- Tier 3 (Retargeting): specific high-intent audiences
- Suppression: complete exclusion stack

**Lookalike strategy**:
- Seed audiences in priority order
- Recommended expansion percentages
- Platforms to create LAL on
- Historical performance notes from `get_lookalike_strategy`

**B2B account-level plan** (if applicable):
- Target account list source and size
- Platform coverage (LinkedIn / Google / DV360 / Meta)
- Suppression list for in-pipeline accounts

---

## Notes

- Never recommend a new 3P data purchase without first checking `list_third_party_audience_layers`
  and `get_data_providers`. The org may already have a contract for the segment you're recommending.
- For LALs: always exclude the seed audience from the lookalike campaign. Showing lookalike ads
  to existing customers wastes budget and degrades the audience's purpose.
- Audience size on LinkedIn must be >50,000 members for consistent delivery. If targeting is
  too narrow, either expand seniority range or add adjacent job functions.
- For B2B account-based suppression: the Operator agent runs `push_audience_suppression` daily.
  Check `get_pending_approvals` to see if any suppression actions are awaiting approval.
