---
name: paid-media/creative-strategy
description: >
  Use this skill for any creative strategy, creative testing, or creative optimization task
  in paid media. Triggers include: "creative strategy", "ad creative", "creative brief",
  "creative testing", "A/B test creative", "creative fatigue", "creative performance",
  "ad copy", "creative refresh", "asset specs", "what creative should we run", "creative
  is underperforming", "hook rate", "thumb stop", "creative rotation", "creative roadmap",
  "creative scorecard", or any request to plan, evaluate, or improve paid media creative.
  Also use when diagnosing campaigns where spend is on target but conversions are lagging —
  creative is often the cause.
  Requires the paid-media-mcp server to be connected for org-specific data.
---

# Creative Strategy & Testing

Creative is the primary performance lever in paid media — especially on paid social.
A 10% bid adjustment affects results marginally. A new creative concept can move
performance 2–5x. This skill covers creative strategy, structured testing, and
ongoing optimization.

---

## Step 1 — Gather context

Pull from paid-media-mcp:
1. `get_campaign` — objective, platform, funnel stage, budget
2. `get_campaign_performance` — last 30 days: CTR trend, CPA trend, frequency
3. `get_test_learnings` filtered by platform — what has and hasn't worked before
4. `get_asset_specs` for the platform — exact specs for each format
5. `list_first_party_audiences` — targeting context (creative should speak to the audience)
6. `get_benchmarks` — CTR and CPA benchmarks to gauge current creative health

If the user provides a campaign ID with "creative is underperforming" or similar:
also pull `get_watchdog_alerts` — sometimes a tracking issue looks like a creative issue.

---

## Step 2 — Creative health diagnosis

Before recommending new creative, diagnose the current state:

### Fatigue signals
| Signal | What it tells you |
|--------|------------------|
| CTR declining over 3+ weeks with stable spend | Audience has seen the ad too many times |
| Frequency >3 in 7 days (Meta) | Overexposure — users are ignoring it |
| CPA rising without audience or budget change | Creative is fatiguing; algorithm is working harder |
| Impression share stable but CPC rising | Not creative fatigue — likely auction competition |

### Performance diagnosis checklist
- [ ] What is the current CTR vs. benchmark? (`get_benchmarks`)
- [ ] Is CTR declining week-over-week, or stable/improving?
- [ ] What is the frequency over the last 7/14/30 days?
- [ ] Are any individual ads marked "Low" quality by the platform?
- [ ] Have there been any creative changes in the last 30 days? (from `get_test_learnings`)
- [ ] Is performance uniform across placements, or is one placement dragging the average?

---

## Step 3 — Creative framework

### The three creative dimensions

Every paid media ad has three strategic dimensions. Test them in order — the highest-variance
dimension first:

**1. Concept** (biggest impact)
The core creative idea or hook. What story are you telling? What tension or insight opens the ad?
- "Pain-agitate-solve": name the problem, make it worse, then show the solution
- "Social proof": a customer testimonial or result
- "Direct response": clear offer, clear benefit, clear CTA
- "Education": teach something valuable related to the product
- "Humor/pattern interrupt": disrupt the scroll with something unexpected

Test concepts against each other first. Don't test headlines until you know which concept works.

**2. Format** (high impact)
The creative execution: static image, short video, UGC-style, carousel, testimonial, etc.
Format affects the algorithm's distribution and the user's consumption mode.

**3. Copy/messaging** (refine after concept and format are confirmed)
Headlines, body copy, CTA. Only meaningful to test once you have a winning concept and format.

---

## Step 4 — Testing methodology

### A/B testing (controlled experiment)

Use for: testing a specific variable with high confidence.

**Rules**:
- Test ONE variable at a time. Do not change concept, format, and copy simultaneously.
- Run both variants simultaneously — never sequentially (seasonality, auction dynamics change).
- Minimum runtime: 7 days (14+ days preferred for campaigns with <50 conversions/week).
- Statistical significance target: 95% confidence. Check with the org's testing tool.
- Minimum sample size: use the formula: `n = 16σ²/δ²` (σ = standard deviation of your KPI, δ = minimum detectable effect).
- For conversion KPIs: minimum ~100 conversions per variant before calling a winner.

**Platforms with native A/B test tools**:
- Meta: A/B Test in Ads Manager (Campaign level) — proper holdout split
- Google Ads: Campaign Experiments (draft + experiment) — traffic split at auction level
- DV360: Creative rotation with reporting breakdowns
- LinkedIn: A/B testing in Campaign Manager (limited; mostly ad-level rotation)

### Multivariate testing (MVT)

Use for: testing multiple elements simultaneously when sample size is large.

**When MVT is appropriate**:
- >10,000 impressions per day per variant
- Testing copy variables (headlines, CTAs), not concept-level differences
- Platform auto-optimization handles the traffic split (Google RSAs, Meta Advantage+)

**Read RSA asset performance as MVT results**:
- Google RSA: shows "Best", "Good", "Low" per headline/description
- Immediately pin "Best" performers; replace "Low" performers with new tests

### Platform-native creative optimization

| Platform | Feature | How to use |
|----------|---------|------------|
| Meta | Advantage+ Creative | Auto-generates variations; review asset scores weekly |
| Google | RSA asset performance | Replace "Low" assets immediately |
| DV360 | DCO (Dynamic Creative Opt.) | Feed-based; swap variant rows in the feed sheet |
| LinkedIn | Ad rotation | Set to "Rotate and optimize"; check ad-level CTR after 500+ impressions |
| TikTok | Smart Creative | Test UGC vs. produced; check hook rate per ad |

---

## Step 5 — Creative specs by platform

Pull exact specs with `get_asset_specs` for the target platform. Key specs to always confirm:

### Meta (Facebook + Instagram)

| Format | Aspect Ratio | Recommended Size | Video Length | Notes |
|--------|-------------|-----------------|-------------|-------|
| Feed Image | 1:1 | 1080×1080 | — | Also accepts 4:5 for more feed space |
| Feed Video | 4:5 | 1080×1350 | 15s–60s | Hook in first 3s; captions required |
| Stories/Reels | 9:16 | 1080×1920 | 15s–30s | Keep key content in center 1080×1420 safe zone |
| Carousel | 1:1 | 1080×1080 per card | — | 2–10 cards; each has own headline + CTA |
| Collection | 1:1 | 1080×1080 | — | Lead image/video + product grid |

Text: Headline 40 chars, Primary text 125 chars (truncated after). Place key info in first 125 chars.

### LinkedIn

| Format | Aspect Ratio | Recommended Size | Video Length | Notes |
|--------|-------------|-----------------|-------------|-------|
| Single Image | 1.91:1 | 1200×627 | — | Also accepts 1:1 for mobile |
| Video | 16:9 or 1:1 | 1920×1080 | 3s–30min (30s for awareness) | Captions essential (85% muted) |
| Carousel | 1:1 | 1080×1080 | — | 2–10 cards; download/swipe |
| Document | Variable | PDF, 1–10 pages | — | Displays inline; high engagement |

Headline: 150 chars. Intro text: 600 chars (truncated at ~150 chars in feed). CTA: choose from preset list.

### TikTok

| Format | Aspect Ratio | Recommended Size | Video Length | Notes |
|--------|-------------|-----------------|-------------|-------|
| In-Feed | 9:16 | 1080×1920 | 9s–60s (15–30s optimal) | Safe zone: keep text/logo out of bottom 20% |
| TopView | 9:16 | 1080×1920 | 5s–60s | First 3s must be compelling |
| Spark Ad | Original | Native format | Varies | Boosting organic — use creator's original specs |

Brand safety zone: leave 130px safe zone at top and 250px at bottom for platform UI overlays.

### Google Display (via DV360/Google Ads)

| Format | Size | Notes |
|--------|------|-------|
| Responsive Display | Multiple assets | Upload 15 images, 5 logos, 5 headlines, 5 descriptions, 5 videos |
| HTML5 | 300×250, 728×90, 160×600, 300×600 | Build in Google Web Designer |
| Native | 1:1.91 image | 1200×628; headline 25 chars, description 90 chars |

---

## Step 6 — Creative rotation and refresh cadence

### When to refresh creative

| Platform | Fatigue signal | Refresh trigger |
|----------|---------------|-----------------|
| Meta | Frequency >3/7d, CTR declining | Refresh 1–2 ads per week in fatiguing ad sets |
| LinkedIn | CTR <0.4% after 1,000 impressions | Replace bottom-performing ad; keep top performer |
| TikTok | Hook rate <25% (2s VV / impr.) | Pause and replace within the week |
| Google Display | CTR declining 3 weeks straight | Replace "Low" assets; upload 3 new options |

**Rotation strategy**:
- Always have 3–5 ads active per ad group. Never rely on a single ad.
- When a top performer fatigues, don't start from scratch — evolve it:
  - Change the hook (first 3 seconds) while keeping the same concept
  - Change the visual treatment while keeping the same copy
  - Test the winning concept in a new format
- Only retire a concept when TWO evolved versions have failed.

### Creative roadmap

Structure creative work in waves:

**Wave 1 (Launch)**: 3–5 concept variants across 1–2 formats. Pure exploration.
**Wave 2 (4–6 weeks)**: Double down on the winning concept. Test 3 copy variants.
**Wave 3 (8–12 weeks)**: Evolve winning creative with new hooks. Test new format.
**Wave 4+**: Seasonal/promo overlays. Refresh with updated proof points.

---

## Step 7 — Output

### For a creative brief

Produce a structured brief with:
- **Concept**: the core creative idea (one sentence pitch)
- **Audience**: who specifically is this talking to (use `list_first_party_audiences` data)
- **Platform + Format**: where it runs and in what format
- **Hook**: the opening line or visual in the first 2–3 seconds
- **Message**: what you want the viewer to think, feel, or do
- **CTA**: exact call-to-action text and destination
- **Specs**: pulled from `get_asset_specs`
- **Success metric**: what CTR/hook rate/CVR would make this creative successful?
- **Test hypothesis**: if this is a test, what is being compared and why?

### For a creative performance review

Produce a **Creative Scorecard** with:
- Top 3 performing ads (CTR, CPA, hook rate) and why they work
- Bottom 3 performing ads and root cause diagnosis
- Fatigue assessment per ad group
- Recommended retirements (with replacement concept direction)
- Recommended new tests (concept, format, copy — one variable each)
- Reference past test learnings from `get_test_learnings`

---

## Notes

- Reference `get_test_learnings` before recommending any creative direction. Never
  recommend something that has already been tested and failed on this account.
- When a campaign underperforms, check measurement health first (use `detect_crm_null_fields`
  or `check_signal_capture_health`) before blaming the creative. Signal loss can look like
  creative failure.
- For B2B creative: specificity outperforms aspiration. "Reduce invoice processing time by 40%"
  outperforms "Transform your finance team." Name the exact problem and the exact outcome.
- UGC/creator content outperforms polished production on Meta and TikTok. On LinkedIn,
  thought leadership content (articles, carousels, talking head video) performs best.
