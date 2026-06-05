# Cross-Filter Patterns

Cross-filter state should have one owner. Avoid duplicating filter logic across
several widgets and callbacks.

## HoloViews Linked Selections

Use this for notebook-first Panel dashboards. It is the default replacement for
historical cuxfilter-style linked dashboard behavior.

```python
import holoviews as hv

linker = hv.link_selections.instance()

scatter = df.hvplot.scatter(x="x", y="y", rasterize=True, responsive=True)
bars = df.hvplot.hist(y="value", bins=50, responsive=True)
line = df.hvplot.line(x="timestamp", y="metric", rasterize=True, responsive=True)

linked_view = linker(scatter + bars + line).cols(2)
```

Use Panel for layout around the linked object:

```python
import panel as pn

dashboard = pn.template.FastListTemplate(
    title="GPU Cross-Filter Dashboard",
    sidebar=[backend_indicator],
    main=[linked_view],
)
dashboard.servable()
```

For selection-aware downstream data, apply the linked-selection filter and keep
any materialized output bounded. Prefer showing the filtered result directly in
a notebook first; if you need a browser table, materialize only a small sample.

```python
selected = linker.filter(df)
selected.head(1000)
```

## Panel Widget Filters

Use widgets when the filter is not naturally expressed as brushing/selecting a
plot, such as a symbol selector or global time range.

```python
def filter_frame(df, symbols, time_range):
    start, end = time_range
    out = df
    if symbols:
        out = out[out["symbol"].isin(symbols)]
    out = out[(out["timestamp"] >= start) & (out["timestamp"] <= end)]
    return out
```

Use `pn.bind` to connect widgets to a pure filter helper and to each chart
builder. This keeps Panel-specific code at the layout edge.

```python
filtered = pn.bind(
    filter_frame,
    df=df,
    symbols=symbols_widget,
    time_range=time_range_widget.param.value,
)

scatter = pn.bind(make_scatter, filtered)
hist = pn.bind(make_histogram, filtered)
```

Use `pn.rx` when the filter is clearer as a reactive expression, especially
when chaining several widget-driven transformations. Use
`widget.param.value_throttled` for sliders and range controls that expose it and
would otherwise trigger expensive chart regeneration on every drag. Use the
same filtering source for every chart regeneration.

## Dash Callback Filters

Dash should use one filtering helper and then fan out to figures.

```python
def filter_frame(df, symbols, start, end):
    out = df
    if symbols:
        out = out[out["symbol"].isin(symbols)]
    if start and end:
        out = out[(out["timestamp"] >= start) & (out["timestamp"] <= end)]
    return out
```

Convert cuDF or Polars to pandas only after filtering and only if the selected
Plotly path cannot consume the dataframe directly.

## Streamlit Session State

Streamlit should keep widget state in `st.session_state` and pass it into one
filter helper. Cache the loaded dataframe.

```python
@st.cache_resource(show_spinner=False)
def load_data(path):
    df, backend = read_table(path)
    return df, backend
```

Avoid storing large filtered dataframes in session state. Store only scalar
filter choices.
