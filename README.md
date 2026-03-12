# gnnwr-agent-skill

Universal agent skill for spatial intelligent regression with [GNNWR](https://github.com/zjuwss/gnnwr) (Geographically Neural Network Weighted Regression).

Compatible with **Claude Code**, **Codex**, **Gemini CLI**, **Cursor**, **Amp**, **Cline**, **Trae**, **OpenClaw**, and other coding agents.

## What it does

Provides AI coding agents with complete API reference and workflow guidance for:

- **GNNWR / GTNNWR** model training and evaluation
- **Spatial coefficient mapping** (folium interactive maps, matplotlib publication figures, GeoPandas basemaps)
- **Diagnostic interpretation** (R², AIC, F-tests, residual analysis)
- **Large-scale datasets** (KNN sparse distance, O(n·k²) DIAGNOSIS)

## Install

### One-command install (recommended)

```bash
npx skills add SteadfastAsArt/gnnwr-agent-skill
```

This auto-detects installed agents and creates the appropriate symlinks.

### Manual install

| Agent | File | Destination |
|-------|------|-------------|
| Claude Code | `SKILL.md` | `~/.claude/skills/gnnwr-spatial-analysis/SKILL.md` |
| Codex / Gemini CLI / Amp | `AGENTS.md` | `~/.agents/skills/gnnwr-spatial-analysis/AGENTS.md` |
| Cursor | `rules/*.md` | `.cursor/rules/gnnwr-spatial-analysis.md` |

```bash
# Claude Code (global)
mkdir -p ~/.claude/skills/gnnwr-spatial-analysis
cp SKILL.md ~/.claude/skills/gnnwr-spatial-analysis/

# Codex / Gemini CLI / Amp (universal)
mkdir -p ~/.agents/skills/gnnwr-spatial-analysis
cp AGENTS.md ~/.agents/skills/gnnwr-spatial-analysis/

# Cursor (project-level)
mkdir -p .cursor/rules
cp rules/gnnwr-spatial-analysis.md .cursor/rules/
```

## Structure

```
gnnwr-agent-skill/
├── SKILL.md         <- Claude Code (frontmatter + full reference)
├── AGENTS.md        <- Codex / Gemini CLI / Amp (universal format)
├── rules/           <- Cursor rules
│   └── gnnwr-spatial-analysis.md
├── LICENSE
└── README.md
```

## Triggers

The skill activates on keywords: spatial regression, GWR, GNNWR, GTNNWR, spatial non-stationarity, geographic weighting, coefficient mapping, PM2.5 spatial modeling, land price spatial analysis.

## Prerequisites

```bash
pip install gnnwr
```

## License

MIT
