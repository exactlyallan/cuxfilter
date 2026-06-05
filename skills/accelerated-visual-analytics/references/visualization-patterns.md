# Visualization Patterns

Use this reference when choosing the visualization stack, chart type, or
HoloViz pattern for a GPU-accelerated cross-filter dashboard.

Official docs consulted:

- RAPIDS API docs: https://docs.rapids.ai/api/
- HoloViz background: https://holoviz.org/learn/background.html
- HoloViz interlinked plots tutorial:
  https://holoviz.org/tutorial/Interlinked_Plots.html
- HoloViews linked brushing:
  https://holoviews.org/user_guide/Linked_Brushing.html
- hvPlot docs: https://hvplot.holoviz.org/
- Panel reactive API:
  https://panel.holoviz.org/explanation/api/reactive.html
- Datashader introduction: https://datashader.org/getting_started/Introduction.html
- Datashader FAQ: https://datashader.org/FAQ

## Default Stack

For notebook-first cross-filtered large-data exploration, choose:

`cuDF or backend-toggled dataframe path -> hvPlot/HoloViews/GeoViews -> Datashader -> Panel`

This is the maintained replacement for the useful historical cuxfilter pattern:
dataframe-native charting, linked selection support, scalable rasterization,
and notebook-friendly layout.

## RAPIDS API Notes

- The current RAPIDS API docs list cuDF, dask-cuDF, cuGraph, cuxfilter,
  Dask-CUDA, RMM, cuVS, and related packages as RAPIDS APIs or libraries.
- The same docs include an inactive projects section. Do not recommend inactive
  RAPIDS projects for new generated visualization examples.
- cuxfilter may still appear in RAPIDS API docs, but this skill treats it as
  historical only. Do not install it or import it.

## Library Roles

| Library | Use when | Data boundary |
| --- | --- | --- |
| hvPlot | One-line charts from cuDF, pandas, Polars, Dask, Xarray, and related data structures. | Native for supported dataframe types. |
| HoloViews | Composable plots, overlays, layouts, operations, and linked brushing. | Native via HoloViews data interfaces. |
| GeoViews | HoloViz-native geographic objects, tiles, and linked geo views. | Prefer for HoloViz geospatial dashboards. |
| Datashader | Scatter, line, heatmap, graph, or geo views above about 100k marks. | Native cuDF support. |
| Panel | Dashboard layout, widgets, templates, tables, indicators, and servable apps. | Hosts plots/widgets; data remains in chosen backend. |
| Colorcet | Perceptually uniform HoloViz palettes. | Palette only; no data boundary. |
| Param | Reusable stateful Panel components and declarative app parameters. | State layer only; no data boundary. |
| Plotly | Teams expect Plotly charts or Dash apps. | Convert late if required by the target version/library path. |
| Bokeh | Lower-level interactive charts and custom tools. | Usually pandas/ColumnDataSource boundary. |
| Streamlit | Fast standalone prototype with common Python app ergonomics. | Usually pandas boundary for display/widgets. |
| PyDeck | deck.gl-specific geospatial layers, 3D extrusions, or custom WebGL maps. | Usually pandas/Arrow boundary. |
| cuGraph | Graph analytics before graph visualization. | Native cuDF edge lists. |

## Decision Heuristics

1. Multi-chart linked dashboard: use Panel/HoloViews/Datashader first.
2. Very large scatter/line/heatmap/graph: include Datashader.
3. Single quick chart from a dataframe: use hvPlot.
4. Dash deployment requirement: use Plotly Dash and convert at plot boundary
   only if the selected Plotly path cannot consume the dataframe directly.
5. Streamlit requirement: use Streamlit, cache data, and filter through one
   stateful helper.
6. Spatial points: use hvPlot/GeoViews plus tiles and Datashader for volume.
7. deck.gl layers, custom WebGL maps, or 3D polygon extrusions: use PyDeck or
   Plotly depending on target UI.
8. Graph analytics: compute layout/metrics with cuGraph, render with
   Datashader/HoloViews or PyDeck depending on scale.

## HoloViz Cross-Filtering

- Use `hv.link_selections.instance()` for linked brushing across HoloViews or
  hvPlot-generated objects that share meaningful dimensions, indexes, or
  explicit `index_cols`.
- Keep all linked charts in the same linked-selection call or the same
  `link_selections` instance.
- Use the linked-selection object's `.filter(df)` method only for bounded,
  selection-aware tables or downstream computations. Never stream millions of
  selected rows into a browser table.
- Use Panel widgets for explicit controls that are not natural brush
  selections, such as symbol lists, date ranges, metric selectors, and
  aggregation level.

## Panel Reactivity

- Use `pn.bind` when wrapping existing pure functions or hvPlot calls with
  widget values.
- Use `pn.rx` for more declarative reactive expressions or when chaining
  several widget-driven transformations.
- Prefer `widget.param.value_throttled` for sliders and ranges that would
  otherwise regenerate expensive Datashader/HoloViews views on every drag.
- Keep filtering in one pure helper, then bind that helper into every chart
  builder. Do not duplicate filtering conditions inside several callbacks.

## Large Data

- Prefer `rasterize=True` for Bokeh-backed hvPlot scatter, line, quadmesh, and
  related dense views because it keeps values available for colorbars and
  inspection.
- Do not combine `datashade=True` and `rasterize=True` on one hvPlot call.
- Use `cnorm="eq_hist"` for dense scatter distributions when equalized density
  reveals structure better than linear normalization.
- Keep data as cuDF, Polars, or the user's chosen dataframe through filtering
  and aggregation. Convert to pandas only for bounded tables, Plotly/PyDeck
  boundaries, or small metadata lists.
- When the right CPU/GPU boundary is uncertain, generate a backend toggle
  rather than separate CPU and GPU dashboards.

## Chart Patterns

Aggregate bar:

```python
def aggregate_bar(df, category, value=None, agg="count"):
    if value is None or agg == "count":
        out = df.groupby(category).size().reset_index(name="count")
        y = "count"
    else:
        out = df.groupby(category)[value].agg(agg).reset_index()
        y = value
    return out.sort_values(y, ascending=False)

bars = aggregate_bar(df, "category").hvplot.bar(x="category", y="count")
```

Histogram:

```python
hist = df.hvplot.hist(y="duration", bins=50, responsive=True)
```

Dense scatter:

```python
scatter = df.hvplot.scatter(
    x="x",
    y="y",
    rasterize=True,
    cnorm="eq_hist",
    cmap="viridis",
    responsive=True,
)
```

Dense line:

```python
line = df.hvplot.line(
    x="timestamp",
    y="value",
    rasterize=True,
    responsive=True,
)
```

Heatmap:

```python
heatmap = df.hvplot.hexbin(
    x="x",
    y="y",
    C="metric",
    reduce_function="mean",
    gridsize=80,
    cmap="viridis",
    responsive=True,
)
```

Temporal range filter:

```python
import panel as pn

date_range = pn.widgets.DatetimeRangePicker(
    name="Time range",
    start=df["timestamp"].min(),
    end=df["timestamp"].max(),
    value=(df["timestamp"].min(), df["timestamp"].max()),
)
```

Categorical filter:

```python
def unique_values(df, column, *, limit=None):
    values = df[column].dropna().unique()
    if hasattr(values, "to_arrow"):
        out = values.to_arrow().to_pylist()
    elif hasattr(values, "to_list"):
        out = values.to_list()
    elif hasattr(values, "tolist"):
        out = values.tolist()
    else:
        out = list(values)
    out = sorted(out)
    return out[:limit] if limit else out

choices = unique_values(df, "symbol", limit=1000)
symbols = pn.widgets.MultiChoice(name="Symbols", options=choices, value=choices[:5])
```

Data table:

```python
table = pn.widgets.Tabulator(to_pandas(df.head(1000)), pagination="remote")
```

Geo polygons:

```python
geo_view = polygons.hvplot.polygons(
    geo=True,
    tiles=True,
    c="metric",
    cmap="viridis",
    responsive=True,
)
```

Use PyDeck or Plotly only when the task requires deck.gl behavior, custom WebGL
map styling, or 3D polygon extrusions.

Graph analytics:

```python
import cugraph

graph = cugraph.Graph()
graph.from_cudf_edgelist(edges, source="src", destination="dst")
layout = cugraph.force_atlas2(graph)
```

After layout, join node coordinates back to edges and render as datashaded
segments or points.

## Do Not Overbuild

If the user's dataset is small and they only need one chart, recommend plain
pandas, Polars, hvPlot, Altair, or Plotly unless the user specifically wants an
NVIDIA GPU data science example.
