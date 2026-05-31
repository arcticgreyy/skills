---
name: paid-media/dv360-dynamic-display
description: >
  Use this skill whenever the user wants to build, architect, or optimize dynamic
  creative for display campaigns using DV360 or the Google Marketing Platform. Triggers
  include: setting up Dynamic Creative Optimization (DCO), building a dynamic feed matrix,
  personalizing display ads by audience or signal, configuring Ads Creative Studio, using
  DV360 Ad Canvas for data-driven creative, binding CM360 Floodlight u-variables to creative
  elements, building personalized ad sequences or storyboards, swapping creative variants
  without re-trafficking, or running ADH queries against dynamic creative performance. Also
  use when the user mentions "DCO", "dynamic creative", "personalize display", "dynamic feed",
  "feed sheet", "creative variants", "Ad Canvas", "Google Web Designer", "GWD", "Ads Creative
  Studio", "Studio", "dynamic countdown", "weather trigger", "variant fatigue", or any request
  to serve different creative to different audiences within a single ad framework.
---

# DV360 Dynamic Display — DCO Strategy Framework

## What This Skill Does

Architect, storyboard, and execute data-driven dynamic ad campaigns that programmatically
swap creative components in real time based on first-party business data (GA4, CRM,
Floodlight u-variables) and optional environmental signals — scaling thousands of creative
permutations inside a single master ad framework using DV360 Ad Canvas and CM360.

---

## Phase 1 — Data Architecture & Signal Mapping

Dynamic creative is a structural relationship between **Data Signals** (the "Why") and
**Creative Variants** (the "What"). Map this relationship before opening any design tool,
typically in a Google Sheet linked to Ads Creative Studio or CM360.

### Step 1: Identify Your Data Signals

**First-Party Data (highest priority)**
- GA4 Audience Segments — e.g., Lapsed Buyers vs. High-Value Cart Abandoners
- CM360 Floodlight Custom Variables (u-variables) — pass real-time business data like
  SKU number, membership tier, or last-searched product category directly into the creative

**Third-Party & Environmental Data (optional guardrails)**
- Publisher contextual signals
- Geo-location data
- Weather API triggers — e.g., show a heat-themed creative only when the local weather
  index exceeds a defined threshold for that DMA

Layer environmental signals on top of first-party signals, never instead of them. First-party
data produces the highest-relevance variants; environmental signals add situational context.

### Step 2: Build the Dynamic Feed Schema

Map every creative element to a structural variable before designing anything. Example matrix:

| Signal (Trigger) | Image | Headline | CTA |
|-----------------|-------|----------|-----|
| GA4: Cart Abandoner (SKU_102) | Product_102_Hero.png | "Still thinking about the [Product_Name]?" | "Complete Purchase" |
| GA4: Loyalty Member — Gold tier | Gold_Rewards_Bg.jpg | "Exclusive: 20% off for Gold Members" | "Claim Reward" |
| Weather: >30°C + Location: LA | Summer_Chill_Video.mp4 | "Beat the LA heat today." | "Order Now" |
| **Default (fallback)** | Brand_Generic_Bg.jpg | "Discover what's new." | "Shop Now" |

The Default row is mandatory — see Phase 2 for fallback rules.

---

## Phase 2 — Creative Storyboarding & Rule Definition

DV360's Creative Workspace Storyboard feature lets you map sequential user experiences across
multiple exposures so users don't see the same ad style repeatedly.

### Step 1: Establish the Narrative Flow

Structure touchpoints by recency or funnel progression:

**Touchpoint 1 — Prospecting**
Generic brand-building dynamic assets. Drive via third-party contextual interest targeting.
No personalization yet — the user hasn't signaled intent.

**Touchpoint 2 — Consideration**
Introduce personalized dynamic text based on the specific category or product page visited
in GA4. Show the user you know what they were looking at without being aggressive.

**Touchpoint 3 — Conversion**
Serve high-intent dynamic elements showing the exact abandoned item. Add a dynamic countdown
timer tied to a promotional end date or inventory signal if available.

### Step 2: Define the Fallback Strategy

Always provision a **Default Row** in the dynamic feed. The programmatic engine falls back to
this row when:
- A user doesn't match any defined audience or business data silo
- A feed API call times out or returns an error

The default creative must be broad, brand-compliant, and safe for any context. Never leave
this row empty — a blank fallback means a broken ad impression.

### Step 3: Set Content Selection Rule Priority

When a user matches multiple rules simultaneously (e.g., they are both in the LA geo and a
Gold Loyalty Member), define explicit priority:

```
Priority 1: First-party loyalty/CRM signals
Priority 2: Behavioral signals (cart abandon, category browse)
Priority 3: Environmental/contextual signals
Priority 4: Default fallback
```

Configure this priority order inside Ads Creative Studio's **Content Selection Rules** before
generating the master template.

---

## Phase 3 — Technical Execution

### Step 1: Choose Your Creative Build Path

**Option A — Scale & Speed (Ad Canvas)**
Use DV360's native Ad Canvas to build a Data-Driven Creative layout from templates. Media
traders and designers can view the structural layout simultaneously. Best for responsive
display and native formats where speed to market matters more than pixel-perfect interactivity.

**Option B — Complex Interactive / Rich Media (Google Web Designer)**
Build a custom shell in Google Web Designer (GWD). Bind text fields, background image
containers, and video containers directly to the CM360 dynamic tracking tags. Required for
high-impact formats (expanding skins, interactive 3D objects, swipeable lookbooks) where
the template system is too constrained.

### Step 2: Connect Profiles in Ads Creative Studio

1. Upload the dynamic feed sheet (Google Sheet or CSV)
2. Set column data types:
   - Destination URL column → Exit URL
   - Headline columns → Text
   - Image/video columns → Asset
   - Timer columns → Date/Time or Integer
3. Set Content Selection Rules with the priority order from Phase 2
4. Transform the dynamic shell into a **Master Template**
5. Sync the Master Template directly to your CM360 advertiser repository

### Step 3: Traffic via CM360 → DV360

1. Confirm the dynamic creative is visible in CM360 as a live placement
2. Pull the CM360 tracking tag and attach it to the corresponding DV360 Line Item
3. Verify u-variable pass-through in a test impression before scaling spend

---

## Phase 4 — Optimization: Steering the Creative Matrix

### Audit Variant Fatigue via CM360 Dynamic Reporting

Pull a Dynamic Creative performance report in CM360. Track impression volume alongside CTR
for individual asset combinations. If a specific variant's CTR collapses after 3–4 exposures
to the same user, pause that row in the feed matrix. You can isolate and pause individual
variant rows without touching the broader campaign line items.

### Extract Attribution Depth via ADH

Run SQL in Ads Data Hub linking log-level dynamic creative data to conversion pathways:
- Did showing the exact SKU (product-level personalization) drive higher incremental ROI
  than showing only a category image?
- Which creative variant appeared most often as an early touchpoint in multi-touch
  conversion paths — even if it never received last-click credit?

Use these findings to promote high-performing variant logic up the Content Selection Rule
priority stack.

### Dynamic Swap Optimization (No Re-Trafficking Required)

To swap underperforming assets:
1. Drop a replacement image into the linked dynamic feed sheet, or
2. Edit a headline variant directly inside Ads Creative Studio

The system refreshes active creative components across all live programmatic buys
automatically — no creative re-review by publisher exchanges, no new trafficking pass
in CM360.

This is the core operational advantage of a feed-based DCO setup: creative iteration
is decoupled from media trafficking. The media team controls budgets and targeting;
the creative team controls the feed.

---

## Common Situations & What to Do

**"We have hundreds of SKUs — how do we avoid managing hundreds of feed rows?"**
→ Group SKUs into intent clusters (category-level personalization) rather than one row per
SKU. Reserve individual SKU rows only for your top-revenue products or highest cart-abandon
items. Category rows handle the long tail. Fewer rows = easier to maintain and faster to
optimize.

**"The fallback creative is serving to the majority of users"**
→ Your audience signal is either too narrow or not firing correctly. Check that GA4 audiences
are shared to DV360 (via audience linking in Google Ads), that Floodlight u-variables are
passing values on the relevant pages, and that the feed's Content Selection Rules are set
to match the actual audience names in DV360 — even a case-mismatch can break the lookup.

**"A variant is performing well but I want to test a new headline without risking performance"**
→ Don't edit the existing row. Add a new row to the feed with the same targeting rule and
new headline, then set a traffic split percentage in Ads Creative Studio's rule settings.
This is effectively an A/B test within the feed without creating separate line items.

**"Weather triggers aren't firing even though the conditions are met"**
→ Verify the weather data provider is still connected and the API is returning values for the
relevant DMAs. Weather triggers require a live feed connection — they don't cache. Also
confirm the threshold logic (e.g., ">30°C" vs. ">86°F") matches the unit the data provider
returns, which is a common silent failure point.

**"CM360 dynamic report shows impressions but the variant breakdown is blank"**
→ Dynamic reporting requires the creative to be tagged with the dynamic profile ID, not just
the standard CM360 placement tag. Verify that the tag pulled from CM360 was generated from
the dynamic creative placement type, not a standard display placement.
