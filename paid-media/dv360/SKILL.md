---
name: paid-media/dv360
description: >
  Use this skill whenever the user wants help with display advertising, programmatic
  campaigns, DV360, Display & Video 360, or the Google Marketing Platform (GMP) stack.
  Triggers include: planning a display campaign, setting up DV360 line items, configuring
  CM360 trafficking, running ADH queries, building a programmatic media plan, deploying
  rich media or DOOH formats, troubleshooting frequency capping, setting up PAIR, working
  with Ads Data Hub, or any question about GMP stack strategy. Also use when the user
  mentions "DV360", "Display & Video 360", "CM360", "Campaign Manager", "Ads Data Hub",
  "ADH", "programmatic", "display", "CTV", "connected TV", "DOOH", "rich media",
  "private marketplace", "PMP", "programmatic guaranteed", "PG deal", "Floodlight",
  "begin-to-render", "PAIR", or anything related to enterprise display campaign management.
---

# DV360 / GMP Enterprise Display – 2026 Strategy Framework

## The Stack: Who Does What

Before any campaign work, align on which tool owns which function:

| Tool | Role |
|------|------|
| **CM360** | Ad server and source of truth. Hosts master Floodlight tags, enforces global cross-channel frequency caps, measures using begin-to-render (eliminates ghost impressions). |
| **GA4** | Post-click behavior. Builds predictive behavioral segments that feed back into DV360 targeting. |
| **DV360** | Programmatic buying engine (DSP). Executes bids across open exchange, PMPs, and PG deals. |
| **ADH** | Privacy-safe data clean room. Combines CM360/DV360 log-level data with first-party CRM data via SQL for custom attribution without exposing raw user data. |

---

## Phase 1 — Planning: Funnel Architecture & Privacy Infrastructure

Google has migrated all targeting to the Line Item level (campaign and insertion-order level targeting is sunset). Structure must be deliberate from day one.

### Step 1: Deploy Privacy Infrastructure

Set up **PAIR** (Publisher Advertiser Identity Reconciliation) in DV360. PAIR lets you securely match encrypted first-party CRM data with a publisher's first-party data — no third-party cookies required. This is the foundation for scale in a cookieless environment.

### Step 2: Establish Global Frequency Cap in CM360

Set a cross-channel frequency cap at the CM360 **advertiser level** (not the line item level) so line items don't compete against each other. Example: maximum 3 exposures per user per week across all placements.

### Step 3: Architect Line Items by Funnel Stage

Build three distinct programmatic buckets:

**Prospecting — Upper Funnel**
- Targeting: Google Affinity segments, broad contextual
- Exclusion: Suppress existing CRM lists to avoid wasted spend on existing customers

**Behavioral — Mid Funnel**
- Targeting: Custom Intent audiences, in-market segments, GA4 lookalike audiences built from high-value site visitors
- Goal: Feed the retargeting pool

**Retargeting — Lower Funnel**
- Structured by recency:
  - 1–7 days: highest bid, most aggressive messaging
  - 8–30 days: reduced bid, softer re-engagement creative
- Use specific GA4 events and Floodlight conversion signals as the seed for audience membership

---

## Phase 2 — Creation: The Multi-Format Media Mix

A comprehensive enterprise display campaign is not just banners. Deploy the full format matrix:

| Format | Inventory Source | Best Use Case |
|--------|-----------------|---------------|
| Responsive Display Ads (RDAs) | Open Exchange | Scale & efficiency — machine-assembled from your text, images, and native layouts |
| Native Display & Video | Premium publisher feeds | Mid-funnel engagement — blends into publisher layout and content style |
| High-Impact Rich Media | Custom Marketplace / Studio | Branding & interactivity — built in Google Web Designer/Studio (3D objects, expanding skins, swipeable lookbooks) |
| Programmatic Video & CTV | YouTube, Hulu, Connected TV | Top-of-funnel reach — unskippable storytelling on the big screen |
| Programmatic Audio | Spotify, Pandora, YouTube Music | Brand recall during screenless moments (podcasts, playlists) |
| Digital Out-of-Home (DOOH) | Billboards, transit, retail screens | Mass market awareness — triggered dynamically by time of day, location, or local weather |

### Trafficking Workflow

1. **Upload to CM360 first** — Create placements and upload creatives into CM360. This wraps ads in the master tracking layer before they go anywhere near a DSP.
2. **Sync to DV360** — Push CM360 placements into your DV360 advertiser account.
3. **Build Line Items** — One Line Item per format + audience pairing. Use **Custom Bidding Algorithms** tied directly to GA4 conversion values rather than maximizing raw impressions.

---

## Phase 3 — Optimization: Steering the Stack

Optimization at this level is not about CTR. It is about cross-tool data analysis.

### Audit Begin-to-Render Placements (CM360)

Pull placement reports weekly. If a publisher site shows a high ad download rate but a low render rate, block that placement. The begin-to-render standard means you only pay for ads that actually load on-screen — ghost impressions (loaded but never displayed) are eliminated at the serving layer.

### Run Path-to-Conversion Queries in ADH

Pull log-level data from Ads Data Hub. Analyze:
- Are display exposures driving a lift in branded search queries in GA4?
- If a line item shows zero direct conversions but appears in 40% of successful conversion paths as an early touchpoint — **protect its budget**. Last-click attribution will mislead you here.

Use SQL in ADH to build custom attribution models that reflect your actual conversion paths.

### Execute Asset Swaps via Creative Status

Check responsive and rich media asset scores in CM360. Immediately drop any asset rated "Low." Upload replacements into CM360 — they propagate live to active DV360 line items without requiring a new trafficking pass.

### Adjust Custom Bidding via Off-Funnel Signals

Symptom: Mid-funnel behavioral line items are spending heavily but not feeding the retargeting pool.

Fix: Modify your Custom Bidding scripts to:
- **Penalize** sessions with low dwell time
- **Reward** high-engagement page views tracked by GA4

The retargeting pool is only as good as the behavioral signals feeding it — garbage in, garbage out.

---

## Common Situations & What to Do

**"My DV360 line items are competing with each other and driving up CPMs"**
→ Global frequency cap is either not set or set at the wrong level. Set it at the CM360 advertiser level, not per line item. Also audit audience overlap — if prospecting and retargeting audiences aren't mutually exclusive, you're bidding against yourself.

**"My display campaigns show lots of impressions but zero conversions"**
→ Check begin-to-render rates first — high impressions with low render rates means you're buying ghost inventory. Then check ADH path-to-conversion data before concluding display isn't working; it may be assisting conversions that close via search.

**"I want to run DOOH but don't know how to trigger it dynamically"**
→ DOOH line items in DV360 support dynamic creative triggers based on time of day, DMA-level weather feeds, and proximity to a point of interest. Configure trigger rules in the line item settings before attaching creative.

**"Third-party cookies are gone and my retargeting audiences have collapsed"**
→ Two steps: (1) Activate PAIR with key publisher partners so first-party data matching replaces cookie-based identification. (2) Shift retargeting seeds to GA4 conversion events and Floodlight tags, both of which are first-party signals that survive the cookie deprecation.

**"ADH queries are returning null or incomplete path data"**
→ Verify CM360 and DV360 are linked to the same ADH instance and that log export is enabled on both. ADH data typically has a 24–48 hour lag. Also confirm your SQL is joining on the correct user key (not cookie-based if those are zeroed out).
