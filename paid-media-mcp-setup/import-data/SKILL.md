---
name: paid-media/import-data
description: >
  Refreshes campaign metadata or performance data in an existing paid-media-mcp
  setup. Imports from BigQuery, a pasted platform export, or a Google Sheet.
  Designed to be run regularly (weekly, monthly) to keep the MCP's data
  current without re-running the full setup wizard. Run from inside the
  paid-media-mcp project directory.
  Trigger when the user says anything like: "refresh the data", "update
  campaigns", "import performance", "sync from BQ", "update the MCP data",
  "campaigns have changed", "re-import", "pull latest data", or similar.
---

# Paid Media MCP — Import Data

You are refreshing data in an existing paid-media-mcp setup. The existing
JSON files should already be populated from `/paid-media/setup` — your job
is to update specific files with fresh data without touching anything else.

Run this skill from inside the paid-media-mcp project directory.

---

## Step 1 — Identify what to refresh

Ask (or infer from context):
```
What data needs updating?
  1. campaigns        — campaign list, budgets, statuses
  2. performance      — daily metrics (impressions, clicks, spend, conversions)
  3. both             — campaigns + performance together
  4. other            — audiences, accounts, or another domain
```

If the user says something like "sync from BQ" or "pull latest data", default
to option 3 (both).

---

## Step 2 — Determine the data source

Ask which source to use. Try in this order:

**BigQuery (preferred for campaigns and performance)**
- "Do you want to pull from BigQuery? Share your project ID, dataset, and
  table name(s) — or confirm if they're the same as last time."
- Read `data/campaigns.json` and `data/historical-performance.json` to check
  if there are BQ connection details stored in the `notes` field of any
  existing entry (some users store this as a comment).

**Pasted export (fallback)**
- "You can export from the platform UI and paste here. For campaigns: include
  Campaign ID, Name, Status, Budget, Start Date. For performance: include
  Date, Campaign ID/Name, Impressions, Clicks, Spend, Conversions."

**Google Sheet (if Workspace MCP is connected)**
- Offer to read directly from a sheet URL.

---

## Step 3 — Import campaigns

### From BigQuery

1. Verify access: `SELECT COUNT(*) FROM \`{project}.{dataset}.{table}\``

2. Show the column mapping and ask for confirmation if anything is ambiguous:
   ```
   Mapping your columns to MCP fields:
     id            ← campaign_id
     name          ← campaign_name
     platform      ← platform  (values: google_ads, meta, dv360, ...)
     account_id    ← account_id
     team_id       ← [derived from account_id — checking accounts.json]
     status        ← status (values: active, paused, ended, draft, archived)
     objective     ← objective
     budget.amount ← daily_budget or total_budget
     budget.type   ← [inferred from which budget column is populated]
     start_date    ← start_date (YYYY-MM-DD)
     end_date      ← end_date (YYYY-MM-DD, nullable)
   ```
   Flag columns that need value mapping (e.g. platform name "Facebook" → "meta").

3. Query:
   ```sql
   SELECT *
   FROM `{project}.{dataset}.{table}`
   WHERE status NOT IN ('removed', 'deleted')
   ORDER BY name
   ```

4. Read existing `data/campaigns.json` to identify:
   - **New campaigns** to add
   - **Changed campaigns** to update (status, budget, end date)
   - **Removed campaigns** — ask: "These campaigns are in the file but not in
     BQ. Mark as `archived` or remove them?"

5. Merge changes: update existing entries in place, add new ones, handle
   removed ones per the user's choice.

### From pasted CSV

Parse the pasted content. Auto-detect column headers. Show the mapping and
ask for confirmation. Apply the same merge logic as above.

### Value normalization

Platform names vary by export source. Auto-map common variants:
```
Facebook / Facebook Ads / META   → meta
Google / Google Ads / GOOGLE     → google_ads
Display & Video 360 / DV360      → dv360
Search Ads 360 / SA360           → sa360
LinkedIn Ads / LINKEDIN          → linkedin
TikTok For Business / TIKTOK     → tiktok
Twitter / X / TWITTER_X          → twitter_x
Pinterest Ads / PINTEREST        → pinterest
Snapchat / SNAPCHAT              → snapchat
Amazon Advertising / AMAZON      → amazon
```

If a platform value doesn't match any known variant, ask the user.

---

## Step 4 — Import performance data

### From BigQuery

1. Ask for the date range to import:
   - "Last 30 days" (default)
   - "Last 90 days"
   - "Custom range (YYYY-MM-DD to YYYY-MM-DD)"
   - "All available data"

2. Query:
   ```sql
   SELECT
     FORMAT_DATE('%Y-%m-%d', date) AS date,
     campaign_id,
     SUM(impressions)      AS impressions,
     SUM(clicks)           AS clicks,
     SUM(spend)            AS spend,
     SUM(conversions)      AS conversions,
     SUM(conversion_value) AS conversion_value
   FROM `{project}.{dataset}.{table}`
   WHERE date BETWEEN '{date_from}' AND '{date_to}'
   GROUP BY date, campaign_id
   ORDER BY date DESC, campaign_id
   ```
   Adjust column names to match the actual schema.

3. Read existing `data/historical-performance.json`:
   - Identify overlapping date range
   - For overlapping dates: ask "Replace overlapping dates or keep existing?"
     (default: replace — the BQ data is the source of truth)
   - Append new dates

4. Validate: every `campaign_id` in performance records should exist in
   `data/campaigns.json`. Flag orphaned campaign IDs and ask whether to
   add the missing campaigns or drop those records.

### From pasted export

Parse the pasted CSV. Required columns: Date, Campaign ID or Name, and at
least one of: Impressions, Clicks, Spend.

If the export uses campaign names instead of IDs, attempt to match against
campaigns in `data/campaigns.json` by name. Warn about any unmatched names.

### Benchmarks (optional)

After importing performance, offer to recalculate benchmarks:
"Want me to calculate average CTR, CPC, CPA, and ROAS from the imported data
and save them as benchmarks? These are used as targets in performance analysis."

If yes, calculate per platform + objective and write to the `benchmarks` array
in `historical-performance.json`.

---

## Step 5 — Write and confirm

Before writing any file:
1. Show a diff summary:
   ```
   campaigns.json changes:
     + 3 new campaigns added
     ~ 7 campaigns updated (status or budget changed)
     ! 2 campaigns in file not found in source — marking as archived

   historical-performance.json changes:
     + 2,240 new records (2026-05-01 to 2026-05-31)
     ~ 310 records replaced (overlapping dates)
     Total records after update: 12,450
   ```

2. Ask for confirmation or write immediately if the user said "just do it."

3. Write files.

4. Remind to rebuild:
   ```
   Data updated. To apply changes:
     npm run build
     → restart Claude Desktop (Settings → Developer → restart paid-media)

   Or if running in dev mode (npm run dev), changes apply automatically.
   ```

---

## Notes

- Never delete data without explicit confirmation. If in doubt, mark as
  `archived` rather than removing.
- Keep campaign IDs stable across imports. If a BQ row uses a different ID
  format than what's in the file, ask before changing IDs — other files
  (performance records, tests) reference campaign IDs.
- For large imports (1000+ campaigns, 100K+ performance records), warn the
  user that the MCP loads everything into memory at startup — consider limiting
  the performance history to the most relevant date range (e.g. 12 months).
