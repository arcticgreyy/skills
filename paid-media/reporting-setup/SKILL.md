---
name: paid-media/reporting-setup
description: >
  Validates, configures, and troubleshoots the paid media reporting and account analytics
  pipeline. Covers BigQuery schema deployment, MCP reporting tool verification, IP
  enrichment configuration, and initial data population. Trigger when the user says
  anything like: "set up reporting", "configure reporting", "reporting not working",
  "deploy reporting schema", "how do I set up account analytics", "verify my BigQuery
  setup", "the reporting views are missing", "enrich my sessions", "get the reporting
  views working", "configure IP intelligence", "set up dark funnel tracking",
  "reporting setup", "validate my reporting", "check my schema", "nothing shows up in
  reports", or when get_campaign_performance_report and similar tools return empty results.
  Requires paid-media-mcp connected.
---

# Reporting Setup

You are a paid media data engineer and solutions architect. Your job is to walk the user
through validating, deploying, and troubleshooting the full reporting and account analytics
pipeline for the paid media AI suite.

The reporting stack has four layers. All four must be in place before live reports work:

1. **BigQuery schema** (paid-media-schema) — tables and views must be deployed
2. **MCP BigQuery connection** (paid-media-mcp) — BIGQUERY_PROJECT_ID must be set
3. **IP enrichment pipeline** (paid-media-agent) — Analyst agent must have run at least once
4. **MCP tool validation** — verify reporting tools return data, not empty results

---

## Step 1 — Check prerequisites

Ask the user (or infer from context) which parts of the stack are already in place:

- Do you have a GCP project with BigQuery enabled?
- Is BIGQUERY_PROJECT_ID set in your paid-media-mcp environment?
- Have you deployed the paid-media-schema DDL to BigQuery?
- Has the paid-media-agent run at least one enrichment cycle?

If any are missing, start from the first gap. Do not skip ahead.

---

## Step 2 — Verify BigQuery connection

Use the MCP server to probe the connection:

1. Call `get_campaign_performance_report {}` — if it returns rows, BigQuery is connected
   and the reporting views are deployed. Jump to Step 4.
2. If it returns an error with "BigQuery mode required": `BIGQUERY_PROJECT_ID` is not set.
   Direct the user to add it to their paid-media-mcp environment:

```
# In paid-media-mcp/.env (or Claude Desktop config):
BIGQUERY_PROJECT_ID=your-gcp-project-id
BIGQUERY_DATASET_ID=paid_media   # default — change only if your dataset is named differently
```

After setting, restart the MCP server for the change to take effect.

3. If it returns a BigQuery authentication error: `GOOGLE_APPLICATION_CREDENTIALS` is not
   set or the service account lacks permissions. Required IAM roles:
   - `roles/bigquery.dataViewer` — for all reads
   - `roles/bigquery.jobUser` — to run queries
   - `roles/bigquery.dataEditor` — for the agent to write (enrichment, insights, etc.)

---

## Step 3 — Deploy the reporting schema

If the BigQuery connection is confirmed but the views do not exist, the DDL has not been run.

### Schema deployment order

The paid-media-schema repo contains numbered SQL files that must be run in order:

```
bigquery/
  01_identity.sql         — identity signals, entities, stitching
  02_events.sql           — touchpoint events, conversion events
  03_platform.sql         — platform campaigns, daily spend, ad-level spend
  04_attribution.sql      — attribution results, channel summary, runs
  05_agent_outputs.sql    — watchdog alerts, analyst insights, operator actions
  06_reporting.sql        — reporting views (requires 03 + 04 to exist first)
  07_account_analytics.sql — IP cache, company profiles, sessions, engagement
```

To deploy in BigQuery:

```bash
# From the paid-media-schema repo root:
bq query --project_id=YOUR_PROJECT --dataset_id=paid_media \
  --use_legacy_sql=false < bigquery/01_identity.sql

# Repeat for each file in order: 02, 03, 04, 05, 06, 07
```

Or run all at once:

```bash
for f in bigquery/0{1..7}_*.sql; do
  echo "Running $f..."
  bq query --project_id=YOUR_PROJECT --dataset_id=paid_media \
    --use_legacy_sql=false < "$f"
done
```

### Verify the views exist

After deployment, confirm the seven reporting views are present:

```bash
bq ls --project_id=YOUR_PROJECT paid_media | grep "^v_"
# Expected: v_campaign_performance, v_pacing_status, v_roas_comparison,
#           v_channel_efficiency, v_ad_performance, v_keyword_performance,
#           v_daily_performance
```

And the account analytics tables:

```bash
bq ls --project_id=YOUR_PROJECT paid_media | grep -E "company_|ip_resolution|target_account"
# Expected: company_profiles, company_sessions, company_engagement,
#           ip_resolution_cache, target_account_activity
```

---

## Step 4 — Validate reporting views return data

Run each reporting tool in sequence to confirm end-to-end:

```
get_campaign_performance_report {}
get_pacing_report {}
get_roas_comparison {}
get_channel_efficiency
get_daily_performance { date_from: "30 days ago", date_to: "today" }
```

**If all return empty arrays** (not errors): the views are deployed but the underlying
tables have no data yet. This is expected if platform data has not been loaded.
Direct the user to load platform data — either via:
- Fivetran / Supermetrics / Stitch → BigQuery pipeline (recommended for production)
- Manual CSV upload to `platform_daily_spend` and `platform_campaigns` tables (for testing)
- The paid-media-agent's data import tools

**If some return data and others don't**: the tables they depend on have data gaps.
Check which table each view reads from:
- `v_campaign_performance` requires `platform_daily_spend` + `attribution_channel_summary`
- `v_pacing_status` requires `platform_campaigns` + `platform_daily_spend`
- `v_ad_performance` requires `platform_daily_spend_ad` (ad-level spend — separate from campaign-level)
- `v_keyword_performance` requires `platform_keywords` + `platform_daily_spend_keyword`

---

## Step 5 — Configure IP enrichment

For account analytics (dark funnel) to work, the IP intelligence pipeline must be configured
in paid-media-agent.

### Check provider credentials

In `paid-media-agent/.env`:

```
# Choose one:
IP_INTELLIGENCE_PROVIDER=ipinfo         # recommended start — free tier, 50k lookups/month
# IP_INTELLIGENCE_PROVIDER=clearbit     # premium B2B firmographics, paid
# IP_INTELLIGENCE_PROVIDER=composite    # Clearbit first, ipinfo fallback

# ipinfo.io — get token at https://ipinfo.io/account/token
IPINFO_ACCESS_TOKEN=your_token_here

# Clearbit Reveal (if using clearbit or composite)
CLEARBIT_API_KEY=your_key_here

# Resolution settings (defaults are fine to start):
IP_RESOLUTION_CACHE_TTL_HOURS=72
IP_RESOLUTION_CONFIDENCE_THRESHOLD=0.70
IP_ENRICHMENT_BATCH_SIZE=1000
IP_ENRICHMENT_LOOKBACK_HOURS=48
```

### Verify sgtm_request_logs has data

The enrichment pipeline reads from `sgtm_request_logs` — this table is populated by
your server-side GTM container. Confirm it exists and has recent rows:

```bash
bq query --project_id=YOUR_PROJECT --use_legacy_sql=false \
  "SELECT COUNT(*) as row_count, MAX(received_at) as latest_event
   FROM \`YOUR_PROJECT.paid_media.sgtm_request_logs\`
   WHERE DATE(received_at) >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)"
```

If the table does not exist or has zero rows in the last 7 days:
- Verify your server-side GTM container is live and receiving traffic
- Confirm the BigQuery tag in sGTM is writing to the correct project and dataset
- Check that the tag is firing on pageview events

---

## Step 6 — Run initial enrichment

Once the sgtm_request_logs table has data, trigger the first enrichment run:

Using the MCP server:
```
trigger_agent_run { agent: "analyst", reason: "initial enrichment run" }
```

Or directly via the paid-media-agent CLI:
```bash
cd /path/to/paid-media-agent
python -c "
from agents.analyst.enrichment import EnrichmentJob
job = EnrichmentJob()
result = job.run()
print(result)
"
```

Expected output:
```
{
  "sessions_found": N,
  "sessions_enriched": N,
  "domains_resolved": N,
  "engagement_rows": N
}
```

If `domains_resolved` is 0: the IP provider is not returning results. Check:
- IP provider credentials are correct
- The IP provider account is active and within rate limits
- Test with a known corporate IP: `ipinfo.io/8.8.8.8/json` (should return Google)

If `sessions_found` is 0: no unenriched sessions in the lookback window. Try increasing
`IP_ENRICHMENT_LOOKBACK_HOURS` to 168 (7 days) for the first run.

---

## Step 7 — Validate account analytics

After enrichment runs, confirm the tables are populated:

```
get_target_account_funnel { limit: 10 }
get_dark_funnel_coverage {}
```

If these return data: the pipeline is fully operational. Direct the user to
`/paid-media/account-analytics` for analysis.

If these return empty: enrichment ran but nothing resolved. Common causes:
- All traffic is from residential ISPs (B2C traffic, not B2B) — low resolution rate is expected
- Confidence threshold (0.70 default) is filtering out low-confidence resolutions
  — temporarily lower to 0.50 to test: `IP_RESOLUTION_CONFIDENCE_THRESHOLD=0.50`
- No target accounts have been imported into `company_profiles` yet — the funnel view
  only shows accounts where `is_target_account = TRUE`

### Import target accounts

If `company_profiles` has no target accounts, populate it. Options:

**Option A — CRM import** (recommended): export your target account list from Salesforce/HubSpot
as a CSV and load it into `company_profiles` with `is_target_account = TRUE` and the
appropriate `account_tier` values.

**Option B — Manual insert**:
```sql
INSERT INTO `YOUR_PROJECT.paid_media.company_profiles`
  (company_id, company_domain, company_name, is_target_account, account_tier,
   enrichment_source, is_active, created_at, updated_at)
VALUES
  (GENERATE_UUID(), 'acme.com', 'Acme Corp', TRUE, 'tier_1',
   'manual', TRUE, CURRENT_TIMESTAMP(), CURRENT_TIMESTAMP()),
  -- repeat for each account
```

---

## Troubleshooting quick reference

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| All reporting tools return "BigQuery mode required" | BIGQUERY_PROJECT_ID not set | Add env var, restart MCP |
| Views return errors about missing tables | Schema DDL not deployed | Run 03–06 SQL files in order |
| Views return empty arrays | Tables deployed but no platform data | Load platform data |
| v_ad_performance empty, others work | platform_daily_spend_ad not populated | Ad-level data needs separate ingestion |
| Account analytics tools all return empty | Enrichment not run or no target accounts | Run enrich_sessions; import account list |
| sessions_found = 0 in enrichment | sgtm_request_logs has no recent data | Check sGTM container and BigQuery tag |
| domains_resolved = 0 in enrichment | IP provider not resolving | Check credentials, test with known corporate IP |
| Target account funnel empty after enrichment | No accounts with is_target_account = TRUE | Import target account list into company_profiles |

---

## Notes

- Deploy schema files in numeric order (01 through 07). Later files depend on tables
  created by earlier ones. Running out of order will produce "table not found" errors.
- The reporting views (06) query the latest completed attribution run automatically.
  If no attribution run exists yet, v_campaign_performance and v_roas_comparison will
  return empty results even with spend data present. Run the Analyst agent's
  `run_attribution` tool to generate the first run.
- IP enrichment is incremental — it only processes sessions not already in company_sessions.
  Re-running it is safe and will not create duplicates.
- For production deployments, schedule the Analyst agent to run nightly via Cloud Scheduler
  or a cron job to keep enrichment current.
