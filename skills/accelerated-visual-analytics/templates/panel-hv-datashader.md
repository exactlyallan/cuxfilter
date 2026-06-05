# Template: Panel + HoloViews + hvPlot + Datashader

Use this as the default notebook-first cuxfilter-pattern replacement. Replace the
`CONFIG` values with the user's dataset path and column names.

```python
# Cell 1: Imports and configuration
from pathlib import Path

import holoviews as hv
import panel as pn
import hvplot

hvplot.extension("bokeh")
pn.extension("tabulator")

try:
    import hvplot.cudf  # noqa: F401
except Exception:
    # CPU fallback still works through hvplot.pandas.
    pass

import hvplot.pandas  # noqa: F401

CONFIG = {
    "path": "data.parquet",
    "backend": "auto",
    "time_col": "timestamp",
    "x_col": "x",
    "y_col": "y",
    "category_col": "category",
    "metric_col": "value",
    "title": "GPU Cross-Filter Dashboard",
}
```

```python
# Cell 2: Data loading
def read_table(path, *, backend="auto"):
    path = Path(path)
    suffix = path.suffix.lower()

    if backend not in {"auto", "cudf", "pandas"}:
        raise ValueError("backend must be 'auto', 'cudf', or 'pandas'")

    if backend in {"auto", "cudf"}:
        try:
            import cudf

            if suffix in {".parquet", ".pq"}:
                return cudf.read_parquet(path), "cudf"
            if suffix in {".arrow", ".feather"}:
                return cudf.read_feather(path), "cudf"
            if suffix == ".csv":
                return cudf.read_csv(path), "cudf"
        except Exception as exc:
            if backend == "cudf":
                raise
            print(f"Falling back to pandas because cuDF load failed: {exc}")

    import pandas as pd

    if suffix in {".parquet", ".pq"}:
        return pd.read_parquet(path), "pandas"
    if suffix in {".arrow", ".feather"}:
        return pd.read_feather(path), "pandas"
    if suffix == ".csv":
        return pd.read_csv(path), "pandas"
    raise ValueError(f"Unsupported file type: {suffix}")


def to_pandas(df):
    if hasattr(df, "to_pandas"):
        return df.to_pandas()
    return df


df, backend = read_table(CONFIG["path"], backend=CONFIG["backend"])
df.head()
```

```python
# Cell 3: Optional type cleanup
time_col = CONFIG["time_col"]

if time_col in df.columns:
    if backend == "cudf":
        import cudf

        df[time_col] = cudf.to_datetime(df[time_col])
    else:
        import pandas as pd

        df[time_col] = pd.to_datetime(df[time_col])
```

```python
# Cell 4: Chart builders
def make_dense_scatter(df, x_col, y_col, metric_col=None):
    kwargs = {
        "x": x_col,
        "y": y_col,
        "rasterize": True,
        "cnorm": "eq_hist",
        "responsive": True,
        "height": 420,
        "cmap": "viridis",
        "title": f"{y_col} by {x_col}",
    }
    if metric_col and metric_col in df.columns:
        kwargs["c"] = metric_col
    return df.hvplot.scatter(**kwargs)


def make_time_series(df, time_col, metric_col):
    return df.hvplot.line(
        x=time_col,
        y=metric_col,
        rasterize=True,
        responsive=True,
        height=260,
        title=f"{metric_col} over time",
    )


def make_category_bars(df, category_col):
    counts = df.groupby(category_col).size().reset_index(name="count")
    counts = counts.sort_values("count", ascending=False).head(20)
    return counts.hvplot.bar(
        x=category_col,
        y="count",
        responsive=True,
        height=260,
        rot=45,
        title=f"Top {category_col} values",
    )


def make_metric_histogram(df, metric_col):
    return df.hvplot.hist(
        y=metric_col,
        bins=50,
        responsive=True,
        height=260,
        title=f"{metric_col} distribution",
    )
```

```python
# Cell 5: Linked dashboard
x_col = CONFIG["x_col"]
y_col = CONFIG["y_col"]
time_col = CONFIG["time_col"]
category_col = CONFIG["category_col"]
metric_col = CONFIG["metric_col"]

plots = []

if x_col in df.columns and y_col in df.columns:
    plots.append(make_dense_scatter(df, x_col, y_col, metric_col))

if time_col in df.columns and metric_col in df.columns:
    plots.append(make_time_series(df, time_col, metric_col))

if category_col in df.columns:
    plots.append(make_category_bars(df, category_col))

if metric_col in df.columns:
    plots.append(make_metric_histogram(df, metric_col))

if not plots:
    raise ValueError("No configured chart columns were found in the dataframe.")

linker = hv.link_selections.instance()
linked = linker(hv.Layout(plots).cols(2))

backend_indicator = pn.widgets.StaticText(
    name="Data backend",
    value=backend,
)

row_count = pn.indicators.Number(
    name="Rows loaded",
    value=len(df),
    format="{value:,}",
)

sample_table = pn.widgets.Tabulator(
    to_pandas(df.head(1000)),
    pagination="remote",
    page_size=25,
    height=320,
)

# For explicit widgets such as symbol selectors or global date ranges, bind one
# pure filter helper with pn.bind and pass that filtered frame into every chart
# builder. Keep HoloViews linked brushing and widget-driven filtering as two
# clearly owned layers of state.

dashboard = pn.template.FastListTemplate(
    title=CONFIG["title"],
    sidebar=[backend_indicator, row_count],
    main=[linked, pn.pane.Markdown("### Sample rows"), sample_table],
)

dashboard.servable()
dashboard
```

## Run

- Notebook: execute all cells and display the final `dashboard`.
- Script: save the code to `app.py`, replace the last line with
  `dashboard.servable()`, then run `panel serve app.py --show`.

## Notes

- The linked-selection path replaces historical cuxfilter dashboard-level
  filtering without requiring cuxfilter.
- hvPlot `rasterize=True` invokes Datashader for dense scatter/line views while
  preserving values for colorbars and inspection.
- Plot libraries may still need pandas conversion for tables or non-HoloViews
  components. Keep those conversions late and small.
