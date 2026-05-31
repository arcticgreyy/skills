---
name: paid-media/create-cm360-ct
description: >
  Use this skill whenever the user needs to create CM360 click trackers for paid social
  campaigns or generate a CM360 bulk upload sheet for walled-garden platforms. Triggers
  include: building click trackers for Meta, TikTok, or LinkedIn, generating a CM360
  campaign bulk sheet, creating tracking ads or 1x1 placements, appending UTM parameters
  and dynamic macros to destination URLs, configuring absolute vs. unique click counting,
  setting up CM360 tracking for paid social, or any request to produce a CM360-formatted
  upload file for click tracking. Also use when the user mentions "click tracker", "CM360
  bulk sheet", "tracking ad", "1x1 placement", "site-served tracker", "dynamic click tracker",
  "Floodlight de-duplication", "cache-buster macro", "walled garden tracking", "UTM append",
  or asks how to get CM360 tags into Meta Ads Manager, TikTok Ads Manager, or LinkedIn
  Campaign Manager.
---

# CM360 Click Trackers — Bulk Sheet Generation

## Why Click Trackers (Not Standard Tags)

Walled gardens (Meta, TikTok, LinkedIn) do not execute JavaScript, so standard CM360 display
tags cannot fire. Instead, you deploy **Click Trackers** — redirect URLs that CM360 generates
from 1x1 placements or Site-Served Tracker placements. The output is a plain URL you paste
into the "Website URL" field in the platform's ad manager. CM360 intercepts the click,
records it, appends your parameters, and forwards the user to the final destination.

---

## Phase 1 — Parameter Architecture & Counting Logic

Establish both before touching the bulk sheet.

### URL Architecture

Every Click-Through URL in the bulk sheet is a fully assembled string:

```
{Base URL} + {Static UTMs} + {Platform Dynamic Macros} + {CM360 Key-Values}
```

Example:
```
https://www.brand.com/landing-page
  ?utm_source=facebook
  &utm_medium=paidsocial
  &utm_campaign={{campaign.name}}
  &utm_content={{ad.name}}
  &u1={{adset.id}}
  &u2=VariantA
```

| Parameter layer | Purpose | Example |
|----------------|---------|---------|
| Static UTMs | GA4 attribution | `utm_source=facebook&utm_medium=paidsocial` |
| Platform dynamic macros | Cross-reference inside platform reporting | `{{campaign.name}}`, `{{ad.name}}` — Meta syntax; `__CAMPAIGN_NAME__` — TikTok |
| CM360 u-variables | Map to creative variants, internal IDs, or audience segments | `u1={{adset.id}}&u2=VariantA` |

Use platform-specific macro syntax per destination:
- **Meta**: `{{campaign.name}}`, `{{adset.name}}`, `{{ad.name}}`
- **TikTok**: `__CAMPAIGN_NAME__`, `__AID__`, `__CID__`
- **LinkedIn**: `{campaignName}`, `{creative}`

### Click-Counting Method

Ask the user which counting method applies before building the sheet:

**Absolute Counting**
Every click is recorded — 4 clicks from the same user = 4 recorded clicks. Append a
cache-buster macro (`[timestamp]`) to the URL string to force fresh redirect processing
and prevent caching from swallowing clicks. Best for traffic validation and matching raw
publisher click totals.

**Unique Counting (De-duplication)**
To enforce de-duplicated attribution:
1. In **reporting**: use the Unique Reach Click metric instead of Total Clicks
2. In the **Floodlight configuration**: if the click tracker lands on a page with a
   Floodlight activity, set that activity to *Unique* (1 conversion per user per 24-hour
   window) or *Session* (1 conversion per browser session)

Do not try to enforce uniqueness at the click tracker level itself — uniqueness is enforced
downstream via Floodlight counting logic.

---

## Phase 2 — Bulk Sheet Schema

Use the CM360 Campaign Spreadsheet Upload format. Placements are configured as **1x1** or
**Site-Served Trackers**. Ad Type must be **Static Click Tracker** or **Dynamic Click Tracker**
(prefer Dynamic — it allows URL updates inside CM360 without breaking the live social ad).

### Required Columns

| Section | Column | Example Value | Notes |
|---------|--------|---------------|-------|
| Campaign | Campaign ID | `1234567` | Active CM360 Campaign ID |
| Site | Site ID | `9876543` | Site Directory ID for the social network (e.g., Meta/Facebook entry in CM360's site directory) |
| Placement | Placement Name | `US_FB_Prospecting_1x1` | Follow org naming convention; map to the corresponding social ad set |
| | Placement Dimensions | `1x1` | Standard for all click trackers |
| | Placement Cost Structure | `CPC` or `CPM` | Maps to cost reporting templates |
| Ad | Ad Name | `PROS_Video_VariantA_CT` | Identifies the tracking mechanism; align with creative naming convention |
| | Ad Type | `Dynamic Click Tracker` | Dynamic allows post-launch URL edits without issuing new tags |
| | Ad Status | `Active` | Goes live immediately on upload processing |
| Creative | Creative Name | `PROS_Video_VariantA_TrackingCreative` | Placeholder asset name attached to the ad |
| | Creative Type | `Tracking` | Flags this as a non-visual, tracking-only asset |
| Landing Page | Creative Click-Through URL | `https://www.brand.com/page?utm_source=facebook...` | Fully assembled URL: Base + UTMs + macros + u-variables |

### Naming Convention

Use a consistent pattern across all rows so bulk reports remain filterable:

```
{Market}_{Platform}_{Audience}_{Format}_{Variant}_CT
```

Example: `US_FB_Prospecting_Video_VariantA_CT`

---

## Phase 3 — Automated Build Workflow

### Step 1: Ingest the Content Feed

Request the following from the user (accept as pasted table, CSV, or described list):

- Destination URL(s) — base URLs before parameter appending
- Platform(s) — Meta, TikTok, LinkedIn, or mixed
- Campaign names and ad set names as they appear in the platform
- Creative names / variant identifiers
- CM360 Campaign ID and Site IDs for each platform

If the user has an existing bulk sheet template or prior upload to reference, ask for it
to carry forward Campaign ID and Site ID values.

### Step 2: Apply Counting Rule

- **Absolute**: Append `&cb=[timestamp]` to each Click-Through URL
- **Unique**: Note in the sheet's comments column that the corresponding Floodlight activity
  must be set to Unique or Session counting; do not modify the URL itself

### Step 3: Assemble the Click-Through URL Strings

For each row, concatenate:

```
{Base URL}?{static UTMs}&{platform macros}&{u-variables}{optional cache-buster}
```

Validate each assembled URL:
- No double `?` characters
- Macros use the correct syntax for the destination platform
- Special characters in campaign names are URL-encoded if used in static UTMs
- Total URL length is under 2,000 characters (LinkedIn limit; other platforms are higher)

### Step 4: Populate and Export the Bulk Sheet

Fill all required columns per the schema above. One row per ad/creative variant. Export as
`.csv` or `.xlsx`.

Upload path in CM360:
```
CM360 > [Campaign] > New > Bulk Upload > Select file
```

### Step 5: Retrieve and Deploy Tags

After CM360 processes the upload, it generates a **Click Tracker Tag** (a redirect URL) for
each row. Export these from CM360 and paste them into the platform:

| Platform | Field |
|----------|-------|
| Meta Ads Manager | Website URL on the ad creative |
| TikTok Ads Manager | Landing Page URL on the ad |
| LinkedIn Campaign Manager | Destination URL on the ad creative |

---

## Output Format

Produce the bulk sheet as a labeled CSV with one header row and one data row per
ad/creative variant. If the user needs multiple platforms, produce a separate section (or
tab) per platform — CM360 Site IDs differ by platform and mixing them in a single upload
section causes processing errors.

Include a **pre-upload checklist** at the end of the output:

```
Pre-Upload Checklist
─────────────────────────────────────────────────────
[ ] Campaign ID confirmed active in CM360
[ ] Site IDs verified for each platform (Meta / TikTok / LinkedIn)
[ ] All Click-Through URLs validated: no double-?, correct macro syntax, <2000 chars
[ ] Counting rule applied: Absolute (cache-buster appended) OR Unique (Floodlight noted)
[ ] Ad Type = Dynamic Click Tracker on all rows
[ ] Creative Type = Tracking on all rows
[ ] File exported as .csv or .xlsx (not .gsheet)
[ ] Uploaded via CM360 > Campaign > New > Bulk Upload
[ ] After processing: Click Tracker Tags exported and pasted into platform ad URLs
```

---

## Common Situations & What to Do

**"CM360 click counts don't match Meta's click counts"**
→ This is expected. Meta counts all link clicks including those that don't complete the
redirect (e.g., user clicks but closes the tab). CM360 only records clicks that complete
the redirect to the landing page. The discrepancy is normal and typically 10–30%. Document
the ratio as your platform-to-CM360 reconciliation factor rather than treating one as wrong.

**"The click tracker URL is being flagged as suspicious by the platform's ad review"**
→ CM360 redirect domains are whitelisted by Meta and TikTok for verified CM360 advertisers.
If review still flags it, confirm your CM360 account has the correct platform integration
enabled (Meta Advertiser ID linked to CM360 advertiser), then re-submit.

**"We need to update the destination URL after the tracker is already live in the platform"**
→ This is why Ad Type should always be Dynamic Click Tracker. Edit the Click-Through URL
directly in CM360 under the placement → ad → creative. The tag URL in the platform stays
the same; CM360 redirects to the new destination on next click. No need to pause the ad
or issue a new tag.

**"We have 200+ ad variants — how do we avoid building this row by row?"**
→ Build the Click-Through URL as a formula in a Google Sheet (CONCAT or TEXTJOIN), then
bulk-fill the rest of the CM360 columns by dragging down from a template row. Campaign ID,
Site ID, dimensions, ad type, creative type, and status are the same across all rows —
only placement name, ad name, creative name, and the assembled URL change per variant.

**"LinkedIn is rejecting the CM360 tag URL"**
→ Check that the CM360 advertiser is configured with LinkedIn as an integrated platform.
Also verify the URL length — LinkedIn enforces a stricter 2,000-character limit, and
dynamic macro values can expand the URL significantly at serve time. Shorten variable
names in u-variables if needed.
