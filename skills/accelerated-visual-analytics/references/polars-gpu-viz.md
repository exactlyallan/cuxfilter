# Polars GPU Visualization

Use this reference when the user has a Polars `DataFrame` or `LazyFrame`, asks
for Polars GPU Engine, or wants visualization after a Polars query pipeline.

Official docs consulted:

- Polars GPU support:
  https://docs.pola.rs/user-guide/gpu-support/
- Polars visualization:
  https://docs.pola.rs/user-guide/misc/visualization/
- RAPIDS Polars GPU Engine:
  https://rapids.ai/polars-gpu-engine/
- cuDF Polars GPU Engine docs:
  https://docs.rapids.ai/api/cudf/stable/cudf_polars/
- cuDF Polars GPU Engine usage:
  https://docs.rapids.ai/api/cudf/stable/cudf_polars/usage/

## Role

Polars GPU Engine accelerates Polars Lazy API execution with RAPIDS cuDF. It is
a data-prep/query acceleration path, not a separate browser rendering engine.
Use it to scan, filter, join, aggregate, and materialize a smaller result before
visualization.

## Rules

- Use Polars GPU Engine when the user already works in Polars, asks for Polars,
  or provides a `LazyFrame`.
- Trigger GPU execution with `.collect(engine="gpu")` or a configured
  `pl.GPUEngine`.
- Use `pl.GPUEngine(raise_on_fail=True)` when the user requires proof that the
  query ran on the GPU. Otherwise unsupported plans may fall back to CPU.
- Treat Polars GPU Engine as Open Beta and currently single-GPU. Avoid promises
  that every expression or full pipeline will run on GPU.
- Do not introduce `polars[gpu]` as a default dependency for cuDF-oriented
  examples. Mention installation only when the user asks for a Polars path.
- Keep query work lazy as long as possible, then materialize a result that is
  appropriate for the visualization layer.

## Basic Pattern

```python
import polars as pl

query = (
    pl.scan_parquet("data/*.parquet")
    .filter(pl.col("total_amount") > 15)
    .group_by("pickup_hour")
    .agg(pl.len().alias("rides"))
)

result = query.collect(engine="gpu")
```

Fail-loud GPU execution:

```python
engine = pl.GPUEngine(raise_on_fail=True)
result = query.collect(engine=engine)
```

## Visualization Paths

HoloViz/hvPlot:

```python
import hvplot.polars  # noqa: F401

plot = result.hvplot.bar(x="pickup_hour", y="rides", responsive=True)
```

Dense HoloViz views:

```python
scatter = result.hvplot.scatter(
    x="x",
    y="y",
    rasterize=True,
    cnorm="eq_hist",
    cmap="viridis",
    responsive=True,
)
```

Altair:

```python
chart = result.plot.point(x="x", y="y", color="category")
```

Use Altair or Polars `.plot` for small-to-medium charts. Do not send millions
of raw rows to Vega-Lite.

Plotly:

```python
import plotly.express as px

fig = px.scatter(result, x="x", y="y", color="category")
```

Plotly v6+ can work with Polars through Narwhals. For older Plotly paths or
libraries that cannot consume Polars directly, convert late and after filtering.

## Dashboard Guidance

- For notebook-first cross-filter dashboards, prefer Panel plus hvPlot/HoloViews
  after materializing a filtered Polars result.
- For massive interactive filtering where every brush should recompute against
  the full raw dataset, prefer the cuDF/HoloViz path unless the user explicitly
  wants to keep the pipeline in Polars.
- Keep browser tables bounded with `result.head(1000)` or an explicit sample.
- Explain clearly whether acceleration happened in Polars query execution,
  Datashader rasterization, or both. They are different layers.

## Boundary Helpers

```python
def to_pandas(df):
    if hasattr(df, "to_pandas"):
        return df.to_pandas()
    return df
```

Use this only at boundaries that require pandas, and only after reducing the
data to the smallest practical result.
