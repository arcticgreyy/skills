---
name: paid-media/paid-social
description: >
  Use this skill for any paid social media task across Meta (Facebook/Instagram),
  LinkedIn, or TikTok. Triggers include: planning or building a paid social campaign,
  setting up Meta Ads, LinkedIn Campaign Manager, or TikTok Ads Manager, writing social
  ad copy or creative briefs, diagnosing paid social performance, setting up Conversion
  APIs (Meta CAPI, LinkedIn CAPI, TikTok Events API), explaining platform differences,
  B2B social strategy, lead generation on social, audience targeting on social platforms,
  or any question containing the words "Meta", "Facebook", "Instagram", "LinkedIn",
  "TikTok", "paid social", "social ads", "CAPI", or "walled garden".
  Requires the paid-media-mcp server to be connected for org-specific data.
---

# Paid Social — Meta, LinkedIn & TikTok

Paid social operates differently from search and programmatic. The user is not
expressing intent — you are interrupting their feed. Creative is the primary lever,
audience is the targeting mechanism, and measurement is structurally harder because
every platform reports inside its own walled garden.

---

## Step 1 — Establish platform, objective, and audience

If not already provided, ask in a single message:

- **Platform(s)**: Meta (Facebook/Instagram), LinkedIn, TikTok, or multiple
- **Objective**: awareness, consideration, lead generation, conversions, retargeting
- **Audience type**: B2B (company/title targeting) or B2C (interest/demographic)
- **Funnel stage**: upper (prospecting), mid (nurture/retargeting), lower (conversion)

Pull context from paid-media-mcp (run in parallel):
1. `get_team` — objectives, KPIs, and platform expertise
2. `list_campaigns` filtered by platform — existing social activity to avoid overlap
3. `list_first_party_audiences` filtered by platform — available 1P segments
4. `get_benchmarks` for the platform and objective
5. `get_test_learnings` filtered by platform — apply past winners, avoid known losers
6. `list_attribution_models` — confirm which attribution window applies to this platform
7. `get_measurement_overview` (if available) — CAPI match rates, pixel health

---

## Platform Reference

### Meta (Facebook + Instagram)

**Campaign structure**: Campaign → Ad Set → Ad
- Campaign: objective (Awareness, Traffic, Engagement, Leads, Sales, App Promotion)
- Ad Set: budget, schedule, audience, placements, optimization event
- Ad: creative + destination

**Key 2026 features**:
- **Advantage+ Campaigns**: Meta's AI-driven campaign type. Minimal manual targeting —
  Meta finds the audience from your creative and pixel data. Best for conversion
  objectives with strong pixel history (500+ events/week). Do not use if audience
  segmentation is critical to the brief.
- **Advantage+ Audience**: Broad targeting with a suggested audience hint. The algorithm
  expands beyond your defined targeting when it finds better performance. Always set a
  suggested audience even in Advantage+ mode.
- **Advantage+ Creative**: Auto-generates ad variations (cropping, background, text
  overlays). Upload multiple asset types; Meta optimizes the combination.
- **Lead Ads / Instant Forms**: Native lead capture without leaving the platform.
  Higher form-fill rate but lower lead quality than landing page leads. Use with
  CRM sync to pass back offline conversion events for CAPI.

**Bidding**:
- Lowest cost (no bid cap): default, maximizes volume
- Bid cap: use when CPL target is hard. Set ~20% above your target initially.
- Cost cap: average cost across the campaign, not per impression. More stable delivery.
- ROAS target: e-commerce/catalog. Requires conversion data.

**Attribution**:
- Default: 7-day click + 1-day view. View-through inflates conversions significantly.
- For B2B or longer cycles: 28-day click, 1-day view (if available on your account).
- Platform-reported conversions will always be higher than GA4 or MTA model.
- The discrepancy is normal — document your Meta-to-GA4 ratio and use it consistently.

**iOS 14+ / CAPI**:
- Pixel-only implementation is significantly degraded on iOS. CAPI is required.
- CAPI match rate target: >80%. Check via Events Manager → Event Match Quality.
- Pass: email (SHA-256), phone (SHA-256), fbclid, external_id.
- Deduplication: ensure event_id matches between browser pixel and CAPI server event.

**B2B on Meta**:
- Meta's B2B targeting is weak. Use 1P CRM lists (customer match) as core targeting.
- Lookalikes seeded from high-value customers outperform interest targeting for B2B.
- Exclusions are critical: suppress existing customers, current pipeline accounts.
- Lead quality is typically lower than LinkedIn — use qualification questions in forms.

---

### LinkedIn Campaign Manager

**Campaign structure**: Campaign Group → Campaign → Ad
- Campaign Group: budget and schedule (or set at campaign level)
- Campaign: objective, audience, format, bid
- Ad: creative

**Objectives**:
- Brand Awareness, Website Visits, Engagement (top of funnel)
- Video Views, Lead Generation (mid funnel)
- Website Conversions, Job Applicants (lower funnel)

**Why LinkedIn for B2B**:
- Only platform with company, seniority, job function, and skills targeting
- Company list targeting: upload a CSV of target accounts (ABM)
- Matched Audiences: retarget website visitors, contact lists, account lists
- Lead Gen Forms: native forms that pre-fill from LinkedIn profile — highest B2B form-fill rates

**Ad formats**:

| Format | Best use | Notes |
|--------|----------|-------|
| Single Image Ad | Brand, traffic, conversions | Standard feed placement |
| Video Ad | Awareness, storytelling | 15–30s performs best; captions required (85% muted) |
| Carousel Ad | Multi-product, multi-message | 2–10 cards; each card has own CTA |
| Document Ad | Content download (no gate) | PDF preview in-feed; drives engagement not clicks |
| Thought Leader Ad | Credibility, brand | Boost an employee's organic post as an ad |
| Conversation Ad | Mid-funnel nurture | InMail-style; declining performance due to filtering |
| Lead Gen Form | Lead capture | Pre-filled from profile; CPL premium but quality higher |
| Event Ad | Webinar/event registration | Integrates with LinkedIn Events |

**Bidding**:
- Maximum Delivery: default, broadest reach within budget
- Target Cost: average CPL/CPC target — more predictable, slightly less reach
- Manual Bid: full control, can cap CPCs tightly

**Audience targeting** (B2B specific):
- Job title: broad — senior titles have smaller audiences. Use job function + seniority instead.
- Company list: upload named accounts (ABM). Minimum 300 companies for delivery.
- Seniority: VP+ for enterprise deals; Manager+ for mid-market.
- Skills: underused but powerful — targets practitioners not just titles.
- Retargeting: website visitors (needs Insight Tag), video viewers, Lead Gen Form openers.

**Measurement**:
- Insight Tag: JavaScript tag for website tracking and retargeting.
- CAPI (Conversions API): server-side events. Requires `li_fat_id` click ID parameter.
- Attribution: default 1-day view + 30-day click. Adjust to 7-day click for B2B.
- LinkedIn over-reports vs. GA4 — typical discrepancy is 30–50%.

**LinkedIn-specific notes**:
- Frequency cap: 1 impression per member per day by default. Raise to 2–3 for retargeting.
- Audience Network: LinkedIn's off-platform inventory. Disable for B2B (low quality).
- Minimum audience size for delivery: 50,000 members. Smaller audiences will not deliver.

---

### TikTok Ads Manager

**Campaign structure**: Campaign → Ad Group → Ad
- Campaign: objective and campaign-level budget
- Ad Group: audience, placements, optimization, budget, schedule
- Ad: creative

**Objectives**:
- Reach, Traffic, Video Views, Community Interaction (upper funnel)
- Lead Generation, App Promotion (mid funnel)
- Conversions, Catalog Sales (lower funnel)

**Ad formats**:

| Format | Best use | Notes |
|--------|----------|-------|
| In-Feed Ad | Standard — appears in For You Page | 9:16 vertical video, 15–60s; first 3s critical |
| TopView | Mass reach + brand awareness | First ad seen when app opens; premium cost |
| Branded Hashtag Challenge | Viral UGC campaigns | High cost, high reach; engagement metric |
| Spark Ad | Boost organic content | Amplify creator or brand organic posts; high trust signal |
| Collection Ad | E-commerce / product | Video + product tiles; native shopping |

**Creative is everything on TikTok**:
- Hook in the first 2–3 seconds or the user scrolls.
- Native-looking content outperforms polished production ads.
- UGC-style (creator-first, authentic, speaking to camera) is the dominant format.
- Text overlays and captions are required — most watch without sound.
- Use TikTok Creative Center to see top-performing ads in your vertical.
- Recommended: test 3–5 creative variations per ad group and let the platform optimize.

**Audience**:
- Interest and behavior targeting: broad but effective for B2C.
- Custom Audiences: website visitors (Pixel), customer list upload.
- Lookalike Audiences: seeded from customer list or pixel events.
- B2B on TikTok: limited professional targeting. Use interest/behavior + retargeting.
  Best suited for B2B brands with strong brand/creator content.

**Measurement**:
- TikTok Pixel: client-side tracking. Heavily impacted by iOS 14+.
- Events API (CAPI equivalent): server-side events. Pass `ttclid` click ID.
- Measurement partner integration: available via Adjust, AppsFlyer, Kochava for app.
- Attribution: 7-day click + 1-day view by default. View-through on TikTok is especially
  aggressive — 1-day view is often more than 50% of reported conversions.

---

## Walled Garden Attribution Reality

**The fundamental problem**: Meta, LinkedIn, and TikTok each claim credit for conversions
using their own attribution windows, often counting the same conversion multiple times.

**What to expect**:
- Meta reports: 40–60% more conversions than GA4 shows from Meta sessions
- LinkedIn reports: 30–50% higher than GA4
- TikTok reports: highest inflation, especially with view-through enabled
- All three combined can report 2–4x total conversions vs. actual CRM entries

**How to manage**:
1. Set a consistent internal attribution model (from `list_attribution_models`).
   Use it for all optimization decisions.
2. Track platform-reported metrics for within-platform A/B testing and optimization.
3. Never add up platform-reported conversions — you'll double-count.
4. Use the MTA model results from `get_attribution_results` for cross-channel budget decisions.
5. Document your platform-to-CRM reconciliation ratio per platform per quarter.

---

## Step 2 — Build the campaign

### For a new campaign brief, include:

**Campaign Overview**
Name (org naming convention from `get_platform_org_defaults`), platform, objective,
audience, budget, flight dates, funnel stage.

**Audience Strategy**
- Prospecting: define targeting parameters specific to the platform (see above)
- Suppression: which 1P lists to exclude (from `list_first_party_audiences`)
- Retargeting: which pixel events or engagement audiences to target
- B2B: which account lists or job function + seniority combinations to use

**Creative Requirements**
Per-platform specs pulled from `get_asset_specs`. For each format:
- Dimensions and aspect ratios
- Video length range
- File size limits
- Text character limits
- Any platform-specific rules (safe zone for captions, etc.)

**Measurement Plan**
- Which conversion events to track and how (pixel vs. CAPI)
- Attribution window to use
- How results will be reconciled against MTA model

**Launch Checklist**
- [ ] Pixel/Insight Tag/TikTok Pixel firing and verified
- [ ] CAPI connected and match rate >80%
- [ ] Conversion events mapped correctly
- [ ] Suppression audiences uploaded and applied
- [ ] 1P audience lists refreshed (check `list_first_party_audiences` for refresh cadence)
- [ ] UTM parameters on all destination URLs
- [ ] ttclid / fbclid / li_fat_id parameters being captured server-side
- [ ] Deduplication confirmed (event_id matching browser + server events)
- [ ] QA ad in platform preview before going live

---

## Step 3 — Optimization

### Weekly optimization routine

**Meta**:
1. Check ad-level performance — pause any ad with CTR < 0.5% after 1,000+ impressions
2. Review Breakdown by Age/Gender/Placement — exclude poor performers
3. Check frequency — >3 in 7 days signals fatigue; refresh creative
4. Review cost per result vs. target — adjust bid strategy if >20% off target
5. Check Pixel Event Match Quality — alert if below 80%

**LinkedIn**:
1. Check demographic report — is the right seniority/function clicking?
2. Review audience size — if <50K, delivery will suffer; expand targeting
3. Check CTR by ad format — video vs. single image vs. document
4. Review Lead Gen Form completion rate (target: >10%)
5. Check Insight Tag firing in Campaign Manager → Event tracking

**TikTok**:
1. Check creative performance — hook rate (2s video views / impressions), hold rate
2. Pause any creative with hook rate <25% after 5,000+ impressions
3. Review placement performance — disable Pangle (TikTok's third-party network) if it's underperforming
4. Check Events API status — alert if pixel-only delivery rate drops

---

## Notes

- Never optimize toward a platform's reported conversion metric directly. Use
  `get_attribution_results` from the MTA model for cross-channel budget decisions.
- For B2B: LinkedIn CPL will always be higher than Meta CPL. This is expected.
  Quality-adjust by tracking MQL rate and Opportunity rate per channel.
- Creative refresh cadence: Meta needs new creative every 2–4 weeks for active campaigns.
  LinkedIn can run longer (4–8 weeks) before fatigue sets in. TikTok needs the most
  frequent refresh — test 3–5 creatives simultaneously and replace the bottom performer weekly.
