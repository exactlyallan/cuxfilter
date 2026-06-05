# cuxfilter Migration

This skill does not preserve the cuxfilter API. It preserves the workflow:
load a large dataframe, define linked charts, filter interactively, and inspect
patterns quickly.

The snippets in this file describe historical input patterns only. Generated
code must not import cuxfilter, and cuxfilter does not need to be installed.

## Concept Mapping

| cuxfilter concept | Replacement pattern |
| --- | --- |
| `cuxfilter.DataFrame.from_dataframe(df)` | Keep the original cuDF/pandas dataframe and pass it directly to hvPlot/HoloViews/Panel helpers. |
| `cuxfilter.DataFrame.from_arrow(path)` | `read_table(path)` from `references/data-layer.md`. |
| `cux_df.dashboard(charts, sidebar=...)` | `pn.template.FastListTemplate(...)` or `pn.Column`/`pn.Row` around HoloViews linked plots and widgets. |
| `cuxfilter.charts.bar(...)` | cuDF `groupby` plus `hvplot.bar`, or Plotly bar in Dash. |
| `cuxfilter.charts.scatter(...)` | `hvplot.scatter(..., rasterize=True)` for dense Bokeh-backed views. |
| `cuxfilter.charts.line(...)` | `hvplot.line(..., rasterize=True)` with optional time aggregation. |
| `cuxfilter.charts.heatmap(...)` | Datashader/hvPlot heatmap, hexbin, or rasterized 2D aggregate. |
| `cuxfilter.charts.range_slider(...)` | Panel range widget, Dash range slider, or Streamlit slider. |
| `cuxfilter.charts.date_range_slider(...)` | Panel `DatetimeRangePicker` or framework equivalent. |
| `cuxfilter.charts.multi_select(...)` | Panel `MultiChoice`, Dash dropdown `multi=True`, or Streamlit multiselect. |
| `cuxfilter.charts.drop_down(...)` | Panel `Select`, Dash dropdown, or Streamlit selectbox. |
| `cuxfilter.charts.choropleth(...)` | GeoViews/HoloViews polygons for linked brushing; PyDeck or Plotly when deck.gl, custom WebGL, or 3D extrusion is required. |
| `cuxfilter.charts.graph(...)` | cuGraph for graph/layout plus Datashader/HoloViews rendering. |
| `cuxfilter.charts.view_dataframe(...)` | Panel `Tabulator` or Dash DataTable over a sample/head view. |
| `cuxfilter.layouts.*` | Explicit Panel rows/columns or templates. |
| `cuxfilter.themes.*` | Panel template theme plus explicit palette choices. |

## Migration Response Pattern

When a user provides cuxfilter code:

1. Identify data loading, chart declarations, layout, and runtime call.
2. Replace data loading with the backend-aware loader from
   `references/data-layer.md`.
3. Replace chart declarations with direct-library chart functions.
4. Replace dashboard construction with Panel/Dash/Streamlit layout.
5. Preserve the analytical intent, not the API shape.

## Example

Old:

```python
cux_df = cuxfilter.DataFrame.from_dataframe(df)
charts = [
    cuxfilter.charts.bar("month"),
    cuxfilter.charts.scatter(x="lng", y="lat"),
]
d = cux_df.dashboard(charts, layout=cuxfilter.layouts.two_by_two)
d.app()
```

New:

```python
import hvplot.cudf  # noqa: F401
import panel as pn
import holoviews as hv

linker = hv.link_selections.instance()

bars = df.hvplot.hist(y="month", bins=12, responsive=True)
points = df.hvplot.scatter(
    x="lng",
    y="lat",
    rasterize=True,
    cnorm="eq_hist",
    cmap="viridis",
    responsive=True,
)

dashboard = pn.Column(linker(points + bars).cols(2))
dashboard.servable()
```
