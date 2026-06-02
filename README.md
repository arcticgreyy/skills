# Skills Collection

Custom skills for Claude Code and compatible AI agent environments, covering session management, paid media workflows, and PMO / project management operations.

## The paid media AI suite

These skills are part of a four-component toolkit for building a Claude-powered paid media workflow:

| Component | What it does |
|---|---|
| **[paid-media-schema](https://github.com/arcticgreyy/paid-media-schema)** | Shared data schema — identity namespace registry, BigQuery DDL, and JSON-file schemas for platform-agnostic attribution |
| **[paid-media-mcp](https://github.com/arcticgreyy/paid-media-mcp)** | MCP server — connects Claude to campaign data, identity signals, attribution results, and autonomous agent outputs |
| **[paid-media-agent](https://github.com/arcticgreyy/paid-media-agent)** | Autonomous agents — Watchdog (data governance), Analyst (MTA modeling), Operator (media execution) running on schedule |
| **[paid-media skills](https://github.com/arcticgreyy/skills/tree/main/paid-media)** ← this repo | Interactive skills — day-to-day campaign strategy and execution via Claude Code |

**How they fit together:**
1. Deploy [paid-media-schema](https://github.com/arcticgreyy/paid-media-schema) — run the BigQuery DDL to create your data tables
2. Set up [paid-media-mcp](https://github.com/arcticgreyy/paid-media-mcp) — clone, fill in your data files or connect BigQuery
3. Run `/paid-media-mcp-setup/setup` to populate your MCP data domains
4. Deploy [paid-media-agent](https://github.com/arcticgreyy/paid-media-agent) on Cloud Run — for automated monitoring, modeling, and optimization
5. Use `/paid-media/*` skills for day-to-day campaign work — they use your MCP data automatically when connected

---

## Skills

### `pmo`
Project management and operations skills:

- **jira-triage**: Validates data quality on newly submitted Jira tickets, searches for historical duplicates via JQL, drafts a professional triage comment, and proposes a status transition — with a mandatory human approval gate before anything is posted. Triggers on ticket IDs (e.g. `PROJ-123`), "triage this ticket", "review this Jira", and similar phrases.

**Path**: `pmo/{skill-name}/SKILL.md`

**Setup**: Before first use, provide your project key, required fields list, issue types, and status mappings (In Progress / Needs Info / Canceled equivalents). The skill prompts for any missing config at runtime.

### `productivity`
General-purpose productivity and knowledge management skills:

- **save-session**: Summarizes conversations and saves structured notes to a knowledge base — capturing decisions, actions taken, facts learned, and open items. Triggers on "save session", "save this chat", "log what we did", and similar phrases.

**Path**: `productivity/{skill-name}/SKILL.md`

**Setup**: For `save-session`, configure your knowledge base path in the skill file (default: `~/knowledge-base/_meta/sessions/`). Optionally configure a reindex command if your knowledge base supports it.

### `paid-media`
Campaign strategy and execution skills. These work standalone, but give significantly richer output when [paid-media-mcp](https://github.com/arcticgreyy/paid-media-mcp) is connected — Claude can reference your actual campaign data, team structure, historical performance, and institutional knowledge instead of working from general principles.

**Strategy & Planning**
- **media-plan**: Build data-driven media plans — channel mix, budget allocation, forecasting, flighting strategy
- **paid-social**: Meta, LinkedIn, and TikTok campaign strategy, setup, and optimization
- **ppc**: Google Ads / paid search campaign strategy and optimization
- **dv360**: Enterprise display and programmatic campaign strategy using the full GMP stack
- **dv360-dynamic-display**: Dynamic Creative Optimization (DCO) using DV360 Ad Canvas and CM360

**Execution & Trafficking**
- **create-campaign**: Create campaign briefs or GMP bulk upload files (DV360 SDF, SA360 Bulksheet, CM360)
- **bulk-upload**: Generate ready-to-upload GMP bulk files for DV360, SA360, or CM360
- **create-cm360-ct**: Generate CM360 click tracker bulk sheets for walled-garden platforms (Meta, TikTok, LinkedIn)

**Audience & Creative**
- **audience-strategy**: First-party audiences, lookalike strategy, suppression logic, B2B account-based targeting
- **creative-strategy**: Creative testing methodology, asset specs, performance diagnosis, creative roadmap

**Measurement & Attribution**
- **measurement-setup**: Tag audit, pixel health, Conversion API (CAPI) setup, data layer review
- **attribution-report**: Attribution model analysis, signal quality assessment, cross-channel comparison

**Reporting & Optimization**
- **analyze-performance**: Campaign performance analysis with KPI scorecard, trend analysis, and recommendations
- **optimize-campaign**: Campaign audit and prioritized optimization action plan
- **budget-pacing**: Pacing analysis, over/under-pacing diagnosis, daily budget adjustments
- **create-report**: Generate audience-appropriate reports (executive, media team, client, internal)

**Path**: `paid-media/{skill-name}/SKILL.md`

### `paid-media-mcp-setup`
Skills for setting up and maintaining the [paid-media-mcp](https://github.com/arcticgreyy/paid-media-mcp) server. Run these from inside the `paid-media-mcp` project directory.

- **setup**: Interactive wizard that scans your `data/` directory, identifies what's missing, and walks through populating each domain — from BigQuery, pasted platform exports, Google Sheets, or plain conversation
- **import-data**: Refresh campaign metadata or performance data in an existing setup — designed to run regularly (weekly or monthly) without touching anything else

**Path**: `paid-media-mcp-setup/{skill-name}/SKILL.md`

---

## Installation

Each skill lives in its own directory with a `SKILL.md` file. To use:

1. Clone or download this repository
2. Copy the skill directories you want to your agent's skills folder
3. Configure any paths or settings specific to your setup (noted in each skill)

## Structure

```
.
├── README.md
├── paid-media/
│   ├── analyze-performance/
│   │   └── SKILL.md
│   ├── attribution-report/
│   │   └── SKILL.md
│   ├── bulk-upload/
│   │   └── SKILL.md
│   ├── create-campaign/
│   │   └── SKILL.md
│   ├── create-cm360-ct/
│   │   └── SKILL.md
│   ├── create-report/
│   │   └── SKILL.md
│   ├── dv360/
│   │   └── SKILL.md
│   ├── dv360-dynamic-display/
│   │   └── SKILL.md
│   ├── optimize-campaign/
│   │   └── SKILL.md
│   └── ppc/
│       └── SKILL.md
├── paid-media-mcp-setup/
│   ├── import-data/
│   │   └── SKILL.md
│   └── setup/
│       └── SKILL.md
├── pmo/
│   └── jira-triage/
│       └── SKILL.md
└── productivity/
    └── save-session/
        └── SKILL.md
```

## Contributing

Feel free to fork and adapt these skills for your own use. If you create something useful, consider sharing it back!
