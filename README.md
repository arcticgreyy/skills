# Skills Collection

Custom skills for Claude Code and compatible AI agent environments, covering session management, paid media workflows, and PMO / project management operations.

## The paid media agent suite

These skills are part of a three-piece toolkit for building a Claude-powered paid media workflow:

| Component | What it does |
|---|---|
| **[paid-media-mcp](https://github.com/arcticgreyy/paid-media-mcp)** | MCP server — connects Claude to your campaign data, team structure, performance history, and institutional knowledge |
| **[paid-media-mcp-setup skills](https://github.com/arcticgreyy/skills/tree/main/paid-media-mcp-setup)** ← this repo | Setup wizard and data import skills — populate your MCP data files and keep them current |
| **[paid-media skills](https://github.com/arcticgreyy/skills/tree/main/paid-media)** ← this repo | Campaign strategy and execution skills — DV360, DCO, PPC, CM360 click trackers, and more |

**How they fit together:**
1. Set up [paid-media-mcp](https://github.com/arcticgreyy/paid-media-mcp) — clone the repo and fill in your data files
2. Run `/paid-media/setup` (from this repo) to walk through populating each data domain
3. Use `/paid-media/import-data` on a regular cadence to keep campaign and performance data current
4. Use `/paid-media/*` skills for day-to-day campaign work — they use your MCP data automatically when it's connected

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

- **analyze-performance**: Analyze campaign performance metrics
- **attribution-report**: Generate attribution reports
- **bulk-upload**: Handle bulk campaign uploads
- **create-campaign**: Create new campaigns
- **create-cm360-ct**: Generate CM360 click tracker bulk sheets for walled-garden platforms (Meta, TikTok, LinkedIn)
- **create-report**: Generate campaign reports
- **dv360**: Enterprise display campaign strategy using the full GMP stack
- **dv360-dynamic-display**: Dynamic Creative Optimization (DCO) using DV360 Ad Canvas and CM360
- **optimize-campaign**: Optimize existing campaigns
- **ppc**: Google Ads / PPC campaign strategy and optimization

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
