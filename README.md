# gnnwr-spatial-analysis

A Claude Code skill for spatial intelligent regression with [GNNWR](https://github.com/zjuwss/gnnwr) (Geographically Neural Network Weighted Regression).

## What it does

Provides Claude Code with complete API reference and workflow guidance for:

- **GNNWR / GTNNWR** model training and evaluation
- **Spatial coefficient mapping** (folium interactive maps, matplotlib publication figures, GeoPandas basemaps)
- **Diagnostic interpretation** (R², AIC, F-tests, residual analysis)
- **Large-scale datasets** (KNN sparse distance, O(n·k²) DIAGNOSIS)

## Install

```bash
npx skills add SteadfastAsArt/gnnwr-claude-skill
```

Or manually:

```bash
# Global (all projects)
cp SKILL.md ~/.claude/skills/gnnwr-spatial-analysis/SKILL.md

# Project-level
cp SKILL.md /path/to/project/.claude/skills/gnnwr-spatial-analysis/SKILL.md
```

## Triggers

The skill activates when Claude detects keywords like: spatial regression, GWR, GNNWR, spatial non-stationarity, geographic weighting, coefficient mapping, PM2.5 spatial modeling, land price spatial analysis.

## Prerequisites

```bash
pip install gnnwr
```

## License

MIT
