# Skills Collection

Custom skills for Claude Code and compatible AI agent environments, covering session management and paid media workflows.

## Skills

### `productivity`
General-purpose productivity and knowledge management skills:

- **save-session**: Summarizes conversations and saves structured notes to a knowledge base — capturing decisions, actions taken, facts learned, and open items. Triggers on "save session", "save this chat", "log what we did", and similar phrases.

**Path**: `productivity/{skill-name}/SKILL.md`

**Setup**: For `save-session`, configure your knowledge base path in the skill file (default: `~/knowledge-base/_meta/sessions/`). Optionally configure a reindex command if your knowledge base supports it.

### `paid-media`
A collection of skills for paid media campaign management:

- **analyze-performance**: Analyze campaign performance metrics
- **attribution-report**: Generate attribution reports
- **bulk-upload**: Handle bulk campaign uploads
- **create-campaign**: Create new campaigns
- **create-cm360-ct**: Generate CM360 click tracker bulk sheets for walled-garden platforms
- **create-report**: Generate campaign reports
- **dv360**: Enterprise display campaign strategy using the full GMP stack
- **dv360-dynamic-display**: Dynamic Creative Optimization (DCO) using DV360 Ad Canvas and CM360
- **optimize-campaign**: Optimize existing campaigns
- **ppc**: Google Ads / PPC campaign strategy and optimization

**Path**: `paid-media/{skill-name}/SKILL.md`

### `paid-media-mcp-setup`
Skills for setting up and configuring paid media MCP integration:

- **import-data**: Import data for paid media campaigns
- **setup**: Set up paid media MCP integration

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
└── productivity/
    └── save-session/
        └── SKILL.md
```

## Contributing

Feel free to fork and adapt these skills for your own use. If you create something useful, consider sharing it back!
