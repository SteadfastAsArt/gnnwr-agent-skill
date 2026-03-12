---
title: GNNWR Spatial Analysis
tags: gnnwr, spatial regression, GWR, geographically weighted, coefficient mapping, non-stationarity
---

# GNNWR Spatial Analysis Rules

When working with GNNWR spatial regression tasks, follow these rules:

## Data Preparation
- Always pass `spatial_column=["lon", "lat"]` to `datasets.init_dataset()` — omitting it degenerates the model to global regression
- For GTNNWR, always include `temp_column` and set `use_model="gtnnwr"`
- Use `knn_k=500–2000` when N > 10,000 to avoid OOM on distance matrices
- Default to `process_fn="minmax_scale"` and `sample_seed=42` for reproducibility

## Model Training
- Start with `optimizer="Adam"`, `start_lr=0.01`, `early_stop=30`
- Leave `dense_layers=None` for automatic architecture sizing
- Always set `early_stop` > 0 to prevent overfitting
- Use `use_gpu=True` when available

## Diagnostics
- Use `model.result()` for summary, `model.reg_result(only_return=True)` for full DataFrame
- `DIAGNOSIS.lite=True` auto-activates for N > 10k — only R²/RMSE available
- For full diagnostics (AIC, F-tests), ensure N < 10k or set `lite=False` explicitly

## Visualization
- Folium maps: `utils.Visualize(model)` for interactive coefficient heatmaps
- Matplotlib: scatter with `cmap="RdYlBu_r"`, clip outliers at 2nd/98th percentile
- GeoPandas + Contextily for basemap overlays in publications
- Always include: coefficient maps, residual spatial distribution, pred vs obs scatter

## Common Mistakes
- N > 10k without `knn_k` → OOM
- `start_lr` too high → loss explosion
- Reading normalized coefficients as real values → use `reg_result()` for denormalized predictions
- GTNNWR without `temp_column` → silently falls back to GNNWR
