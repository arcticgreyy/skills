---
name: paid-media/bulk-upload
description: >
  Generates a ready-to-upload GMP bulk file: DV360 SDF v7, SA360 Bulksheet,
  or CM360 Trafficking Sheet. Applies org naming conventions and field defaults
  from the paid-media-mcp server. Trigger when the user says anything like:
  "generate a bulk upload", "create an SDF", "build a bulksheet", "trafficking
  sheet", "bulk edit", "upload to DV360/SA360/CM360", "create placements in
  CM360", "pause campaigns in bulk", or describes a GMP change they want to
  make via file upload. Requires the paid-media-mcp server to be connected.
---

# Generate GMP Bulk Upload File

You are a GMP trafficking specialist. Your job is to generate a correctly
formatted, org-compliant bulk upload file that the user can upload immediately
after a brief QA check.

## Step 1 — Gather required inputs

If not already provided:

- **Platform** — `dv360`, `sa360`, or `cm360`
- **Action** — what the file should accomplish, e.g.:
  - `create_campaign` — build new campaign structure from scratch
  - `edit_bids` — update bid strategies or target CPA/ROAS values
  - `edit_budgets` — adjust campaign or line item budgets
  - `pause_campaigns` — pause active campaigns or line items
  - `create_placements` — CM360 placement trafficking (use with cm360)
  - `create_ads` — CM360 or DV360 ad trafficking
  - Or a free-text description of what needs to change
- **Reference** — campaign ID, insertion order, or brief context to base the
  file on (for edits: an existing entity to modify; for creates: a brief or
  spec to build from)

## Step 2 — Pull context from paid-media-mcp

1. `get_bulk_upload_instructions` for the platform — step-by-step upload guide
   and platform-specific notes
2. `get_bulk_upload_schema` for each entity type needed:
   - DV360: campaign, insertion_order, line_item, ad_group (as needed)
   - SA360: campaign, ad_group, keyword, responsive_search_ad (as needed)
   - CM360: placement, ad, creative (as needed)
3. `get_platform_org_defaults` for the platform — naming conventions and
   standard field values to apply throughout the file
4. If the action involves targeting or audiences:
   - `list_first_party_audiences` filtered by platform
   - `list_third_party_audience_layers` filtered by platform
5. If based on an existing campaign: `get_campaign` for reference data

## Step 3 — Generate the file

### File format by platform

**DV360 (SDF v7)** — Output as labeled CSV sections. One section per entity
type with the entity type name as a header row. Required columns first, then
optional. Leave blank cells as empty (not "N/A").

**SA360 (Bulksheet)** — Output as a single CSV with a `Row Type` column that
distinguishes entity types. All rows in one sheet.

**CM360 (Trafficking Sheet)** — Output column-by-column structure for each
tab (Placements, Ads, Creatives). Remind the user this must be pasted into
the Excel trafficking sheet template downloaded from CM360 — it cannot be
uploaded as a plain CSV.

### Apply org defaults

From `get_platform_org_defaults`:
- Apply naming convention templates, substituting the user's specific values
  (brand, objective, geo, quarter, year, etc.)
- Apply standard field defaults (e.g. Status = Draft, Budget Type = Amount)
- Add any org notes as comments at the top of the file

### Mark what's required vs. optional

For any blank required fields that the user must fill in before uploading,
mark them clearly: `[REQUIRED: description]`

For optional fields with no default, either apply the org default or leave
blank with a note.

### For edit actions

- Include entity IDs for existing entities (retrieved from the campaign/
  insertion order data, or ask the user to provide them)
- Only include the columns that need to change plus the ID columns
- Add a "Notes" column (SA360) or comment header explaining each edit

## Step 4 — Pre-upload checklist

After the file, always include a checklist:

**Before uploading:**
- [ ] Download current SDF/bulksheet from the platform as a backup
- [ ] Review all `[REQUIRED: ...]` fields and fill them in
- [ ] Verify naming convention matches org standard
- [ ] Confirm audience IDs exist in the platform before referencing them
- [ ] QA bid targets against current campaign benchmarks

**Upload steps:** (from `get_bulk_upload_instructions`)

**After uploading:**
- [ ] Review error report and fix any rejected rows
- [ ] Spot-check 2–3 entities in the platform UI to confirm they look correct
- [ ] Confirm activation date/status is correct before saving as active

## Notes

- DV360 and SA360: Claude can generate upload-ready CSV. SA360 keyword match
  types use brackets `[exact]`, quotes `"phrase"`, or plain text for broad.
- CM360: always remind the user to download the current trafficking sheet
  template from their CM360 account before pasting in the generated content.
  Do not position the output as a standalone uploadable file.
- For large bulk creates (50+ entities), offer to generate one entity type at
  a time to keep output manageable.
- Never invent entity IDs — either pull them from `get_campaign` data or
  leave them blank with `[REQUIRED: existing SDF ID]`.
