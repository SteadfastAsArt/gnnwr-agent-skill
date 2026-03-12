# GNNWR Spatial Intelligent Analysis

**Version 1.0.0**

> **Note:**
> This document is for AI coding agents (Codex, Gemini CLI, Amp, Claude Code, Cursor, etc.)
> to follow when working with GNNWR spatial regression tasks.
> Humans may also find it useful as a quick reference.

## When to Apply

Reference these guidelines when:
- Analyzing spatial or spatiotemporal data with geographic non-stationarity
- Building GNNWR / GTNNWR regression models
- Generating spatial coefficient maps or diagnostic reports
- Working with large-scale geographic datasets (N > 10k)
- Interpreting geographically varying regression results

## Prerequisites

```bash
pip install gnnwr
```

---

## 1. Quick Start

```python
from gnnwr import models, datasets, utils
import pandas as pd

data = pd.read_csv("data.csv")

train, val, test = datasets.init_dataset(
    data=data, test_ratio=0.2, valid_ratio=0.1,
    x_column=["x1", "x2", "x3"], y_column=["y"],
    spatial_column=["lon", "lat"],  # REQUIRED: geographic coords
    batch_size=32, process_fn="minmax_scale"
)

model = models.GNNWR(train, val, test, use_gpu=True, optimizer="Adam", start_lr=0.01)
model.run(max_epoch=200, early_stop=30)

result = model.reg_result(only_return=True)  # DataFrame: coef_x1, coef_x2, ..., Pred_y
print(model.result())                         # R², AIC, RMSE, F-tests summary
```

### Spatiotemporal (GTNNWR)

```python
train, val, test = datasets.init_dataset(
    data=data, ...,
    spatial_column=["lon", "lat"],
    temp_column=["year", "month"],  # add temporal coords
    use_model="gtnnwr"
)
model = models.GTNNWR(train, val, test, use_gpu=True)
```

### Large-Scale (N > 10k) — KNN Mode

```python
train, val, test = datasets.init_dataset(
    data=data, ..., knn_k=500  # only k nearest neighbor distances
)
# Memory: N=100k full=55GB → knn_k=2000 only 763MB
```

---

## 2. API Reference

### 2.1 init_dataset Key Parameters

| Parameter | Default | Notes |
|-----------|---------|-------|
| `knn_k` | None | KNN sparse distance; None=full matrix |
| `process_fn` | "minmax_scale" | or "standard_scale" |
| `spatial_fun` | BasicDistance | Euclidean; or ManhattanDistance |
| `Reference` | None | "train", "train_val", or custom DataFrame |
| `sample_seed` | 42 | Reproducibility |

### 2.2 Model Hyperparameters

| Parameter | Recommended | Notes |
|-----------|-------------|-------|
| `optimizer` | "Adam" | Also: SGD, AdamW, Adagrad, RMSprop |
| `start_lr` | 0.01–0.1 | Critical tuning point |
| `drop_out` | 0.2 | 0.0–0.5 |
| `dense_layers` | None (auto) | Auto: power-of-2 sequence from input_dim to n_coef |
| `early_stop` | 20–50 | Patience; -1=disabled |
| `batch_norm` | True | Stabilizes training |
| `use_ols` | True | OLS-initialized output layer |

### 2.3 Diagnostics (DIAGNOSIS)

```python
diag = model._test_diagnosis
diag.R2()           # always available
diag.RMSE()         # always available
diag.AIC()          # needs lite=False (auto for N<10k)
diag.AICc()         # corrected AIC
diag.F1_Global()    # GNNWR vs OLS significance
diag.F2_Global()    # spatial weight significance
diag.F3_Local()     # per-variable significance → (dict1, dict2)
```

`lite=True` (auto when N>10k): only R²/RMSE available; Hat-matrix diagnostics skipped.

---

## 3. Visualization Patterns

### 3.1 Folium Interactive Maps (built-in)

```python
viz = utils.Visualize(model, lon_lat_columns=["lon", "lat"], zoom=5)

# Dataset distribution
m1 = viz.display_dataset(name="all", y_column="y")
m1.save("dataset_map.html")

# Coefficient spatial variation — one map per variable
for col in [c for c in result.columns if c.startswith("coef_")]:
    m = viz.coefs_heatmap(data_column=col, steps=20)
    m.save(f"map_{col}.html")

# Custom dot map for any DataFrame
m3 = viz.dot_map(result, "lon", "lat", "denormalized_pred_result", zoom=5)
```

### 3.2 Matplotlib Static Maps (publication-ready)

```python
import matplotlib.pyplot as plt
import numpy as np

result = model.reg_result(only_return=True)

fig, axes = plt.subplots(2, 3, figsize=(18, 12))
coef_cols = [c for c in result.columns if c.startswith("coef_")]

for ax, col in zip(axes.flat, coef_cols):
    sc = ax.scatter(
        result["lon"], result["lat"],
        c=result[col], cmap="RdYlBu_r", s=5, alpha=0.8,
        vmin=result[col].quantile(0.02), vmax=result[col].quantile(0.98)
    )
    ax.set_title(col.replace("coef_", "β_"), fontsize=14)
    ax.set_xlabel("Longitude")
    ax.set_ylabel("Latitude")
    plt.colorbar(sc, ax=ax, shrink=0.8)

plt.suptitle("Spatially Varying Coefficients (GNNWR)", fontsize=16)
plt.tight_layout()
plt.savefig("coefficients_map.png", dpi=300, bbox_inches="tight")
```

### 3.3 Residual Spatial Distribution

```python
result["residual"] = result["denormalized_pred_result"] - result[y_column]

fig, ax = plt.subplots(figsize=(10, 8))
sc = ax.scatter(
    result["lon"], result["lat"],
    c=result["residual"], cmap="coolwarm", s=5,
    vmin=-result["residual"].abs().quantile(0.95),
    vmax=result["residual"].abs().quantile(0.95)
)
ax.set_title("Spatial Residual Distribution")
plt.colorbar(sc, ax=ax, label="Residual")
plt.savefig("residuals_map.png", dpi=300, bbox_inches="tight")
```

### 3.4 Prediction vs Observed Scatter

```python
fig, ax = plt.subplots(figsize=(8, 8))
ax.scatter(result[y_column], result["denormalized_pred_result"], s=3, alpha=0.5)
lim = [result[y_column].min(), result[y_column].max()]
ax.plot(lim, lim, "r--", linewidth=2, label="1:1 line")
ax.set_xlabel("Observed"); ax.set_ylabel("Predicted")
ax.set_title(f"GNNWR: R²={model._test_diagnosis.R2().item():.4f}")
ax.legend()
plt.savefig("pred_vs_obs.png", dpi=300, bbox_inches="tight")
```

### 3.5 GeoPandas + Contextily (with basemap)

```python
import geopandas as gpd
import contextily as ctx

gdf = gpd.GeoDataFrame(result, geometry=gpd.points_from_xy(result.lon, result.lat), crs="EPSG:4326")
gdf_web = gdf.to_crs(epsg=3857)

fig, ax = plt.subplots(figsize=(12, 10))
gdf_web.plot(column="coef_x1", ax=ax, cmap="RdYlBu_r", legend=True,
             markersize=5, alpha=0.7, legend_kwds={"shrink": 0.6})
ctx.add_basemap(ax, source=ctx.providers.CartoDB.Positron)
ax.set_title("β_x1 Spatial Variation")
ax.set_axis_off()
plt.savefig("coef_basemap.png", dpi=300, bbox_inches="tight")
```

---

## 4. Workflow Checklist

1. **EDA**: Check spatial distribution, feature correlations, OLS baseline
2. **Data split**: `init_dataset` with appropriate ratios and `sample_seed=42`
3. **Train**: Start with defaults, tune `start_lr` and `early_stop`
4. **Diagnose**: R², RMSE, F1 (GNNWR vs OLS), F2 (spatial weight significance)
5. **Visualize**: Coefficient maps (spatial non-stationarity), residual maps (model adequacy), pred vs obs
6. **Interpret**: Where do coefficients vary most? Which variables show strongest non-stationarity? (F3_Local)
7. **Report**: Model summary table + coefficient maps + diagnostic statistics

---

## 5. Common Pitfalls

- **Forgot `spatial_column`**: Model degenerates to global regression
- **N > 10k without `knn_k`**: OOM on distance matrix; use `knn_k=500–2000`
- **`start_lr` too high**: Loss explodes; start with 0.01
- **No `early_stop`**: Overfitting; always set early_stop=20–50
- **Interpreting normalized coefficients**: Use `reg_result()` which returns denormalized predictions; coefficients are on normalized scale
- **GTNNWR without `temp_column`**: Silently falls back to GNNWR behavior
