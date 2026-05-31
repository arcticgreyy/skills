---
name: paid-media/setup
description: >
  Interactive setup wizard for the paid-media-mcp server. Scans the data/
  directory to show what is already populated, then works through each domain
  one at a time — accepting data from BQ queries, pasted spreadsheet content,
  Google Sheets (if that MCP is connected), Jira (if connected), or plain
  conversational answers. Writes valid JSON files directly to data/. Designed
  to run from inside the paid-media-mcp project directory.
  Trigger when the user says anything like: "set up the mcp", "help me fill in
  the data", "run the setup wizard", "configure the mcp", "I need to populate
  the data files", "how do I get started", or similar.
---

# Paid Media MCP — Setup Wizard

You are helping a user set up their paid-media-mcp instance. Your job is to
make this as fast as possible by pulling data from whatever sources are
available, asking focused questions only when you need to, and writing clean
validated JSON files directly to `data/`.

Run this skill from inside the paid-media-mcp project directory.

---

## Step 1 — Scan current state

Read all files in `data/` to determine what is already populated vs. missing
or empty. Check for each of the 13 data files:

```
data/metadata.json
data/accounts.json
data/teams.json
data/team-members.json
data/campaigns.json
data/historical-performance.json
data/attribution-models.json
data/reporting-templates.json
data/assets.json
data/testing.json
data/audiences.json
data/measurement.json
data/platforms.json
```

For files that exist, check whether they contain real data (not just the
example Acme Corp data) by looking for company-specific identifiers.

Display a setup progress summary before starting:

```
Paid Media MCP — Setup Status
──────────────────────────────
✓ metadata            company name set
✓ accounts            3 accounts configured
○ teams               using example data — needs update
○ team-members        using example data — needs update
✗ campaigns           file missing
✗ historical-performance  file missing
○ attribution-models  using example data — needs update
✗ reporting-templates file missing
○ assets              using example data — needs update
✗ testing             file missing
✗ audiences           file missing
✗ measurement         file missing
○ platforms           using example data — needs update

5 complete · 4 need update · 4 missing
```

Ask the user: "Which domain would you like to start with, or should I work
through them in the recommended order?"

**Recommended order** (highest leverage first):
1. metadata — fast, unlocks everything else
2. accounts — needed before campaigns
3. teams + team-members — together
4. campaigns — core data
5. historical-performance — biggest impact on analysis
6. measurement — tracking setup
7. attribution-models
8. audiences
9. testing
10. reporting-templates
11. assets
12. platforms (only if using DV360/SA360/CM360)

---

## Step 2 — Work through each domain

For each domain, follow this flow:

### A. Check what's already there

If the file exists with real data, show a summary and ask:
- "This looks populated. Skip, or update specific fields?"

### B. Identify available sources

Before asking questions, check what data sources are available:

**BigQuery** (best for campaigns and performance):
- Ask: "Do you have campaign data or performance data in BigQuery? If so,
  share your project ID, dataset name, and table name(s) and I'll query them."
- If they confirm: use the import-data workflow inline (see Step 3).

**Google Sheets / Google Workspace** (good for team structure, account lists):
- If Google Workspace MCP is connected, offer: "I can read directly from a
  Google Sheet. Share the URL or file ID."
- Otherwise: "Can you paste the relevant rows from your spreadsheet?"

**Jira** (good for test history, project tracking):
- If Jira MCP is connected, offer: "I can query Jira. Which project contains
  your testing or campaign records?"
- Otherwise: "Can you export the relevant tickets as CSV or paste the key
  details?"

**Platform exports** (CSV from Meta, Google Ads, etc.):
- "You can export your campaign list from the platform UI as CSV. Paste it
  here and I'll convert it."

**Direct answers** (for anything else):
- Ask focused, specific questions — not open-ended ones.

### C. Ask domain-specific questions

Use the question guides below for each domain.

### D. Write the file

Once you have enough information:
1. Generate the complete JSON structure
2. Show it to the user: "Here's what I'll write to `data/{file}.json`. Does
   this look right?"
3. After confirmation (or if the user says "just write it"), write the file
4. Confirm: "Written. Here's a summary of what was saved."

---

## Domain question guides

### metadata.json
```
- What is the company name?
- What industry are you in? (optional, helps Claude give relevant context)
- What currency does your team report in? (default: USD)
- When does your fiscal year start? (MM-DD format, default: 01-01)
```

### accounts.json
Each entry = one ad account. Ask:
```
- What ad platforms do you run? (Meta, Google Ads, DV360, SA360, LinkedIn,
  TikTok, Amazon, etc.)
- For each platform: what is the account name, account ID (as shown in the
  platform UI), and which team manages it?
```
Tip: "You can export your account list from your MCC/agency dashboard. Paste
the rows and I'll convert them."

### teams.json + team-members.json
Do these together. Ask:
```
Teams:
- What are your media teams called? (e.g. Performance, Brand, Social, Programmatic)
- For each team:
  - What is their primary objective? (e.g. drive conversions, build awareness)
  - What are their 3 key KPIs?
  - What platforms do they manage?
  - Which accounts (by account ID) does this team own?

Team members:
- For each team, who are the members?
- For each person: name, email, role/title, which teams they're on, which
  platforms they specialize in.
```
Tip: "If you have a team roster in Google Sheets or Jira, I can read it
directly if those MCPs are connected."

### campaigns.json
Best sourced from BQ or platform export. Ask:
```
- Do you have campaigns in BigQuery? (project, dataset, table)
- Or: can you export your active campaign list from the platform as CSV and
  paste it here?
- For each campaign I need: ID, name, platform, account ID, team, status,
  objective, budget (amount + type), start date, end date (if not always-on).
```
Generate IDs if the source doesn't have them (`camp_{platform}_{slug}`).

### historical-performance.json
Best sourced from BQ. Ask:
```
- Do you have daily performance data in BigQuery?
  If yes: project ID, dataset, table name. Expected columns:
  date, campaign_id, impressions, clicks, spend, conversions, conversion_value
- How far back should we import? (e.g. last 90 days, last 12 months)
- Do you have any benchmarks (avg CTR, CPC, CPA, ROAS by platform and
  objective) you want to set as targets?
```
If no BQ: "Can you export a date-range performance report from the platform
and paste it? Columns needed: Date, Campaign ID (or name), Impressions, Clicks,
Spend, Conversions, Conversion Value."

Map campaign names to IDs from campaigns.json if the export uses names.

### measurement.json
Walk through each sub-section:
```
Tag management:
- What TMS do you use? (Google Tag Manager, Tealium, Adobe Launch, manual, etc.)
- What is your container ID?
- Is your implementation client-side, server-side, or hybrid?
- If hybrid/server-side: what is the server-side endpoint URL?

Pixels and Conversion APIs:
- Which platform pixels are installed? For each: pixel ID, which events fire,
  client-side or server-side or both?
- Do you have any Conversion APIs running? (Meta CAPI, Google Enhanced
  Conversions, etc.) For each: which events, estimated match rate?

CM360 (if applicable):
- Do you run CM360? If so: account ID, Floodlight config ID.
- What u-variables do you use? (u1 through u100 — what does each capture?)

Data layer:
- Do you have a data layer? What analytics platform? (GA4, Adobe Analytics)
- What key variables does it capture? (pageType, transactionId, revenue, userId, etc.)
- Are first-party cookies implemented? Cookie domain? Duration?

Measurement partners:
- Do you use any MMM, incrementality, or brand lift partners? For each:
  name, type, status, which channels they cover, how often they run.
```

### attribution-models.json
```
- What attribution model does each platform report on by default?
  (last-click, data-driven, linear, time-decay, first-click, etc.)
- What conversion window does each platform use? (e.g. 7-day click, 1-day view)
- Which conversion events count as primary conversions? Secondary?
- Do you use any cross-channel attribution tools? (Northstar, Rockerbox, etc.)
- Are there any known discrepancies you track between platforms?
  (e.g. Meta reported vs. GA4 — how do you reconcile?)
```

### audiences.json
```
First-party audiences:
- What 1P audience segments do you have built? For each:
  name, what it is (purchasers, cart abandoners, site visitors, CRM list, etc.),
  approximate size, which platforms it's available on, how often it refreshes.

Data providers:
- Do you have any contracts with 3P data providers?
  (Oracle, Experian, Bombora, Acxiom, etc.)
  For each: contract status, what segments, which platforms.

Onboarding platforms:
- Do you use any data onboarding / clean room platforms?
  (LiveRamp, Habu, Salesforce DMP, etc.)

Lookalike strategy:
- For which audiences do you build lookalikes?
- What seed sizes do you use? (1%, 2%, 5%, 10%)
- Which seed + size combinations perform best?

Third-party layers:
- What 3P audience overlay segments do you apply to campaigns?
  (in-market segments, demographic layers, B2B firmographic data, etc.)
  For each: name, provider, which platforms, is it a default layer?
```

### testing.json
```
Methodology:
- What confidence level do you require to call a test winner? (90%, 95%, 99%)
- Do you require statistical significance, or use other winner criteria?
- What's your minimum test duration?

Tools:
- What tools do you use for A/B testing?
  (Meta native, Google Ads Experiments, VWO, Optimizely, internal calc, etc.)

Test history:
- What tests have you run in the past? For each:
  name, what was tested, which variant won, what was the result?
- Are there any tests currently running?
- Any tests planned but not started?
```
If Jira is connected: "I can query Jira for test-related tickets. What project
and label/epic do you use to track tests?"

### reporting-templates.json
```
- What recurring reports do you produce? (weekly team, executive monthly,
  client quarterly, etc.)
- For each report:
  - Who is the audience? (executive, media team, client, internal)
  - What sections does it include?
  - What KPIs are featured?
  - What dimensions does it break down by? (platform, campaign, date)
  - What cadence? (weekly, monthly, quarterly)
```

### assets.json
```
- Where do you store creative assets? (Google Drive, Dropbox, DAM system, etc.)
  URL and access instructions?
- What types of assets do you produce? (social static, video, display HTML5,
  native, copy/headlines, etc.)
- For each type: where is it stored, what naming convention do you use?
- Do you have a brand guidelines or copy guidelines document? URL?
```
Skip detailed per-platform specs for now — those can be added later.

### platforms.json
Only needed if using DV360, SA360, or CM360:
```
- Do you use DV360? SA360? CM360?
- For each platform you use:
  - What naming convention do you use for campaigns / line items / ad groups?
  - What are your standard field defaults when creating new entities?
    (e.g. status = Draft, budget type = Amount, etc.)
```

---

## Step 3 — BigQuery import inline

When the user provides BQ details for campaigns or performance:

1. Confirm the connection: "Let me check I can access that table."
   Run a `SELECT * FROM table LIMIT 1` to verify schema.

2. Show the column mapping:
   ```
   Your columns → MCP fields
   ─────────────────────────
   campaign_id → id
   campaign_name → name
   platform_name → platform (needs value mapping)
   ...
   ```
   Flag any columns that don't map cleanly and ask how to handle them.

3. Query the data:
   - For campaigns: `SELECT * FROM table WHERE status != 'removed'`
   - For performance: `SELECT * FROM table WHERE date >= DATE_SUB(CURRENT_DATE(), INTERVAL 90 DAY)`
     (or the range the user specified)

4. Transform rows to the MCP JSON schema.

5. Write to the file.

---

## Step 4 — Validate and finish

After writing each file, validate the JSON:
- All required fields are present
- IDs are consistent (campaign IDs in performance records exist in campaigns)
- Platform values are from the valid set
- No obvious placeholder values remain (XXXXXXX, YOUR_COMPANY, etc.)

When all requested domains are done, show the final status summary and remind
the user to rebuild and restart:

```
Setup complete for this session.

Files written:
  ✓ data/metadata.json
  ✓ data/accounts.json  (3 accounts)
  ✓ data/teams.json     (2 teams)
  ✓ data/team-members.json  (8 members)
  ✓ data/campaigns.json (24 campaigns)
  ✓ data/historical-performance.json  (90 days, 6,840 records)

Still to do:
  ○ data/attribution-models.json
  ○ data/audiences.json
  ○ data/measurement.json

Next steps:
  npm run build
  → then restart Claude Desktop (Settings → Developer → restart paid-media)

Run /paid-media/setup again to continue with remaining domains, or
run /paid-media/import-data to refresh campaign or performance data later.
```
