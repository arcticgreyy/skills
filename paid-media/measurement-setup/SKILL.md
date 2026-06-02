---
name: paid-media/measurement-setup
description: >
  Use this skill for any measurement, tracking, or tag implementation task. Triggers include:
  "set up tracking", "implement pixels", "GTM audit", "tag audit", "check our tracking",
  "set up Conversion API", "CAPI setup", "Enhanced Conversions", "server-side GTM",
  "data layer", "pixel health", "why are conversions not tracking", "tracking is broken",
  "measurement setup", "attribution isn't working", "first-party cookies", "ITP",
  "deduplication", "event match quality", "signal loss", "privacy-safe measurement",
  or any question about how ad platform tracking is implemented or whether it's working.
  Requires the paid-media-mcp server to be connected for org-specific data.
---

# Measurement Setup & Tracking Audit

Measurement is the foundation of attribution. If signals are missing or degraded,
everything downstream — MTA models, optimization decisions, budget allocations — is
unreliable. This skill covers both initial setup and ongoing health audits.

---

## Step 1 — Understand the current state

Pull context from paid-media-mcp:
1. `get_measurement_overview` — tag management, pixels, conversion APIs, data layer
2. `get_website_data_capture` — data layer, analytics platform, first-party cookies
3. `list_identity_namespaces` — all identity signal types the org should be capturing
4. `get_watchdog_alerts` (status: "open") — active tracking issues already detected
5. `check_signal_capture_health` — current capture rates per namespace

If the user reports a specific tracking problem, start with `get_watchdog_alerts` —
the Watchdog agent may have already diagnosed it.

---

## Step 2 — Tag management audit

### Which TMS is in use?

Determine the tag management system:
- **Google Tag Manager (GTM)**: most common. Client-side, server-side, or hybrid.
- **Tealium**: enterprise CDPs and data governance.
- **Adobe Launch / Tags**: Adobe stack environments.
- **Segment**: CDP-native. Handles tag management and data routing together.
- **Manual (no TMS)**: hardcoded tags. High maintenance, high error rate.

### GTM-specific audit steps

**Client-side GTM**:
1. Verify the GTM container snippet fires on all pages (check via GTM Preview mode)
2. Confirm dataLayer is initialized before the GTM snippet
3. Review tags by type: check for duplicate tags, misfiring triggers, missing variables
4. Check tag firing order — conversion tags should fire after transaction data is in dataLayer
5. Test in GTM Preview mode: submit a test conversion and verify all tags fire correctly

**Server-side GTM (sGTM)**:
1. Confirm the sGTM container URL is configured in all client-side tags
2. Verify gclid, dclid, fbclid, li_fat_id, ttclid are being forwarded in requests
3. Check the Monitoring panel in sGTM — are all expected requests arriving?
4. Confirm PII (email, phone) is being hashed before sending to ad platforms
5. Verify each platform client is configured and forwarding correctly

**Server-side GTM capture rate check**:
Use `check_signal_capture_health` and look for the `platform_click_id.*` namespace rates.
Target: >95% of sessions with paid media traffic should carry the relevant click ID.
If <90%: investigate whether the GTM container is firing on all landing page types.

---

## Step 3 — Pixel and tag inventory

For each platform the org runs, confirm:

### Google (gclid / Enhanced Conversions / Google Ads tag)

| Check | Target | How to verify |
|-------|--------|---------------|
| Google Ads tag fires on all pages | ✓ | Google Tag Assistant |
| Auto-tagging enabled in Google Ads | ✓ | Google Ads → Account Settings → Auto-tagging |
| gclid captured in dataLayer on landing | ✓ | `check_signal_capture_health` (namespace: platform_click_id.google.gclid) |
| Enhanced Conversions configured | ✓ | Google Ads → Goals → Conversions → Enhanced Conversions |
| Hashed email passed with conversions | ✓ | Google Tag Assistant → Verify EC data |
| Conversion deduplication active | ✓ | Order ID passed as transaction_id |

**Enhanced Conversions setup**:
- Capture customer email at conversion (form submit, checkout, sign-up)
- Hash with SHA-256 (normalized: lowercase, trimmed) before sending
- Pass as `user_data.email_address` in the conversion tag
- Also pass phone (`user_data.phone_number` in E.164 format) if available

### Meta (fbclid / Pixel / CAPI)

| Check | Target | How to verify |
|-------|--------|---------------|
| Meta Pixel fires on all pages (PageView) | ✓ | Meta Pixel Helper browser extension |
| Key events firing (Lead, Purchase, etc.) | ✓ | Events Manager → Test Events |
| fbclid captured in first-party cookie (_fbc) | ✓ | `check_signal_capture_health` (platform_cookie.meta.fbc) |
| CAPI connected and sending events | ✓ | Events Manager → Event Match Quality |
| Event Match Quality score | >80% | Events Manager |
| Browser + server events deduplicated | ✓ | event_id matches browser and server |
| Hashed PII sent in CAPI payload | ✓ | email, phone, name, zip as user_data |

**CAPI setup checklist**:
1. Choose implementation: direct API, partner integration (Shopify, HubSpot), or sGTM Meta client
2. Generate system user and access token in Business Settings
3. Configure event endpoint: `https://graph.facebook.com/v19.0/{pixel_id}/events`
4. Pass these user_data fields: `em` (SHA-256 email), `ph` (SHA-256 phone), `fbc` (_fbc cookie), `fbp` (_fbp cookie), `client_ip_address`, `client_user_agent`
5. Pass `event_id` = same value as browser pixel event_id for deduplication
6. Verify in Events Manager → Test Events → Server tab

### LinkedIn (li_fat_id / Insight Tag / CAPI)

| Check | Target | How to verify |
|-------|--------|---------------|
| Insight Tag fires on all pages | ✓ | LinkedIn Insight Tag Helper |
| li_fat_id captured from URL and stored | ✓ | `check_signal_capture_health` (platform_click_id.linkedin.li_fat_id) |
| Conversion events configured | ✓ | Campaign Manager → Analyze → Conversion Tracking |
| CAPI (Conversions API) connected | ✓ | Campaign Manager → Analyze → Conversions API |
| Deduplication enabled | ✓ | eventId passed in both browser and CAPI |

**LinkedIn CAPI setup**:
- Endpoint: `https://api.linkedin.com/rest/conversionEvents`
- Required: `conversion` (conversion URN), `eventId` (dedup), `occurredAt` (epoch ms)
- User data: `userInfo.firstName` (SHA-256), `lastName` (SHA-256), `email` (SHA-256), `title`, `companyName`, `countryCode`
- `li_fat_id` must be passed as `userInfo.linkedInId` when available — highest match signal

### TikTok (ttclid / Pixel / Events API)

| Check | Target | How to verify |
|-------|--------|---------------|
| TikTok Pixel fires on all pages | ✓ | TikTok Pixel Helper |
| ttclid captured on landing | ✓ | `check_signal_capture_health` (platform_click_id.tiktok.ttclid) |
| Key events firing (CompletePayment, SubmitForm) | ✓ | TikTok Events Manager → Test Events |
| Events API configured | ✓ | Events Manager → Events API |
| Hashed PII in Events API payload | ✓ | email, phone as SHA-256 |

---

## Step 4 — Data layer review

The data layer is the contract between the website and the tag manager.

### Required variables for paid media measurement

```javascript
window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
  event:            'purchase',         // or 'generate_lead', 'form_submit', etc.
  transaction_id:   'ORDER-12345',      // CRITICAL: deduplication key
  value:            299.00,             // Revenue/deal value
  currency:         'USD',
  // User data (for Enhanced Conversions / CAPI)
  user_data: {
    email:          'user@company.com',  // hash this in GTM before sending
    phone:          '+12125551234',      // E.164 format; hash before sending
    user_id:        'internal-user-123', // non-PII internal ID
  },
  // Click IDs (capture from URL on landing)
  gclid:            '{{ captured from URL }}',
  fbclid:           '{{ captured from URL }}',
  li_fat_id:        '{{ captured from URL }}',
  ttclid:           '{{ captured from URL }}',
});
```

**Common data layer issues**:
- `transaction_id` not included → deduplication breaks, conversions are double-counted
- PII pushed to dataLayer as plaintext → hashing must happen in GTM or server-side
- dataLayer push fires before GTM loads → events missed
- Different event names across environments (staging vs. prod) → mismatched reporting

---

## Step 5 — First-party cookie and identity setup

**Why this matters**: ITP (Safari's Intelligent Tracking Prevention) expires third-party
cookies after 24 hours and JavaScript-set first-party cookies after 7 days. Server-set
cookies (HttpOnly, from sGTM or your own server) are exempt.

### First-party cookie setup

For each critical identifier, ensure it's stored as a server-set first-party cookie:

| Signal | Cookie name | Server-set? | Duration target |
|--------|-------------|------------|-----------------|
| GA4 Client ID | `_ga` | No (JS) | 395 days — ITP caps at 7 days on Safari |
| Meta _fbc | `_fbc` | No (JS) | 90 days — ITP caps at 7 days on Safari |
| Internal User ID | `uid` | Yes (recommend) | Session or persistent |
| gclid | `_gcl_aw` | No (JS) | 90 days — ITP caps at 7 days on Safari |

**Recommended**: Deploy sGTM and configure it to set first-party cookies server-side
for all click IDs. This exempts them from ITP and extends their lifespan.

### sGTM first-party cookie setup:
1. Deploy sGTM on a subdomain of your site (e.g., `metrics.yourdomain.com`)
2. Configure the GA4 client to forward requests to sGTM
3. In sGTM, use the `Set cookie` tag to write `_fbc`, `_ga`, etc. as server-set cookies
4. Set `SameSite=None; Secure; HttpOnly` for cross-site compatibility

---

## Step 6 — Deduplication setup

Deduplication prevents the same conversion from being counted twice (once by browser
pixel, once by CAPI/server-side event).

**The pattern**:
1. Generate a unique `event_id` at the moment of conversion (e.g., SHA-256 of order ID + timestamp)
2. Send the same `event_id` from both browser pixel and server-side event
3. The platform deduplicates: if it receives two events with the same `event_id` within
   a deduplication window (Meta: 48 hours; Google: 24 hours), it counts one.

**Dedup check**:
- In Meta Events Manager: Events → Details → see "Deduplication" column
- If >5% of events show "Deduplicated", your setup is working
- If 0% are deduplicated and you're running both browser + CAPI, something is wrong

---

## Step 7 — Output

### For a new measurement setup

Produce a **Measurement Implementation Plan**:
1. Current state inventory (what exists, what's missing)
2. Priority setup order (what to implement first for maximum signal recovery)
3. Implementation steps per platform (use the checklists above)
4. Data layer specification (what variables need to be added)
5. Testing protocol (how to verify each tag before going live)
6. Identity namespace coverage map (from `list_identity_namespaces` + `get_identity_signal_coverage`)

### For a tracking audit / break diagnosis

Use the `diagnose_tracking_drop` prompt — it runs the Watchdog diagnostic workflow
systematically and produces an anomaly report. If the Watchdog alerts are already active,
read `get_watchdog_alerts` first.

Produce an **Anomaly Report**:
1. Which signals are degraded (from `check_signal_capture_health`)
2. When the degradation started (from `watchdog_capture_rate_log` trend)
3. Root cause hypothesis (tag change, browser update, platform change)
4. Affected attribution data (% of conversions now unattributed)
5. Immediate fix steps
6. Verification steps (how to confirm the fix worked)

---

## Notes

- Never hardcode a "this works" statement without verifying via `check_signal_capture_health`.
  Tracking breaks silently and frequently.
- Server-side measurement (sGTM + CAPI for all platforms) is the correct long-term
  architecture for any program spending >$10K/month. Prioritize it.
- Signal loss from iOS is permanent unless you implement sGTM with server-set cookies.
  Acknowledge this to stakeholders rather than explaining away attribution discrepancies.
- For B2B: hashed email is the highest-confidence stitching signal. Ensure it's captured
  at every form submission and passed to all CAPI implementations.
