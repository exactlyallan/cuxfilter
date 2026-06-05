---
name: accelerated-visual-analytics
description: Generate accelerated visual analytics notebooks and dashboards without cuxfilter, using cuDF or Polars GPU Engine with HoloViz (Panel, hvPlot, HoloViews, GeoViews, Datashader), Plotly Dash, Streamlit, Bokeh, and PyDeck. Use for linked charts, cuxfilter replacement, CUDA-X Data Science visualization, fast visual exploration of large tabular data, visualization library choice, Polars visualization, or migration away from cuxfilter.
---

# Accelerated Visual Analytics Implementer's Guide

## Compatibility

- Release target: NVIDIA GPU data science library stack 26.04+.
- Requires NVIDIA Volta or newer for the GPU path. Generated examples must also
  include a pandas CPU fallback so they can be read and lightly exercised on
  machines without a local GPU.
- cuxfilter is intentionally absent. Do not install it, import it, or depend on
  its documentation being available.

## Naming

Use NVIDIA library-first wording in user-facing answers. Keep literal
RAPIDS/rapidsai URLs, package names, and historical source names only when they
are still the actual source locations or when explaining migration context.

## Role

Use this skill to create notebook-first, GPU-accelerated visual analytics over
large tabular datasets. It replaces the practical cuxfilter pattern with direct
use of maintained libraries: cuDF for large GPU-suitable data, `cudf.pandas` or
Polars GPU Engine when the user needs a CPU/GPU toggle without rewriting the
pipeline, HoloViews/hvPlot/Datashader/Panel by default, GeoViews/Colorcet/Param
when they fit the task, and Plotly Dash or Streamlit when a standalone dashboard
application is better suited. The user knows their data and analytical goal;
your job is to generate a runnable starting point with correct GPU/CPU
boundaries and linked filtering behavior.

## Use When

- The user asks for a cross-filter dashboard, linked multi-chart view, or
  notebook data explorer.
- The user mentions cuxfilter or asks how to replace cuxfilter.
- The dataset is large enough that GPU acceleration or Datashader matters.
- The user asks which NVIDIA GPU-compatible visualization library to use for a
  specific chart or dashboard.
- The user asks how Polars GPU Engine fits into visualization workflows.

Do not use this skill for a small one-off chart unless the user explicitly
wants NVIDIA GPU visualization guidance.

## Required Inputs

Before emitting code, identify or ask for:

1. Data source: file path, URL, or dataframe variable.
2. Column roles: temporal, categorical, numeric, geospatial, graph edge/node.
3. Output form: notebook by default, standalone app if requested.
4. Framework preference: Panel/HoloViews/Datashader by default, Dash or
   Streamlit when requested.
5. Rough data scale: row count, file size, format, and expected working set if
   known. File size is not GPU memory usage; leave headroom for decompression,
   strings, temporary columns, groupby/join state, and dashboard copies.
6. GPU availability: choose a backend based on scale and workflow, and always
   include a CPU path or backend toggle.

If the data source is available locally, inspect the schema before choosing
charts. If not, ask for representative column names and types.

## Critical Rules

- Never import `cuxfilter` in generated code.
- Keep GPU/CPU boundaries explicit. For ambiguous data sizes, prefer a backend
  toggle over hard-coding a single path. Use cuDF when the estimated working set
  fits GPU memory with margin and operations are GPU-friendly; use pandas or
  Polars CPU for tiny/small data; use the user's Polars pipeline when present.
- Use Polars GPU Engine for LazyFrame query acceleration, then visualize the
  materialized result with hvPlot, HoloViews, Panel, Altair, or Plotly.
- Do not recommend inactive RAPIDS projects from the RAPIDS API docs for new
  generated visualization examples.
- Use hvPlot for dataframe-first charts. Use HoloViews directly when composing,
  overlaying, linking, or applying HoloViews operations.
- Use Panel for app layout, widgets, templates, and servable dashboards.
- Use Datashader/rasterization for scatter, line, heatmap, graph, or
  geospatial views above about 100k marks. For Bokeh-backed hvPlot views,
  prefer `rasterize=True`; do not set both `datashade=True` and
  `rasterize=True` on the same chart.
- Keep cross-filter state single-sourced. Prefer HoloViews
  `hv.link_selections.instance()` for linked brushing; use Panel `pn.rx` or
  `pn.bind` for explicit widget-driven filtering; use one filter function for
  Dash/Streamlit callbacks.
- For HoloViz-native geospatial dashboards, prefer hvPlot/GeoViews with tiles
  and Datashader. Use PyDeck only when the user needs deck.gl behavior, 3D
  extrusions, or a non-HoloViz map stack.
- Default to colorblind-safe palettes: Viridis/Cividis for sequential data,
  Okabe-Ito for small categorical sets, and Colorcet for perceptually uniform
  HoloViz palettes when available.
- Write comments that explain data movement, GPU memory, and library boundary
  decisions.
- Emit a runnable starting point, not a production service.

## cuxfilter Absence

Treat cuxfilter only as migration history. If a user supplies old cuxfilter
code, preserve the analytical intent and rewrite it with direct-library
patterns. Do not tell the user to install cuxfilter or rely on local cuxfilter
examples being present.

## Reference Routing

Load only the references needed for the current request:

- `references/data-layer.md`: backend-size heuristics, CPU/GPU toggles,
  cuDF/pandas/Polars/Dask boundaries, and conversions.
- `references/visualization-patterns.md`: library choice, HoloViz defaults,
  RAPIDS API notes, and chart-by-chart patterns.
- `references/visualization-design-tips.md`: agent-facing chart selection,
  hierarchy, color, accessibility, and data-mapping guardrails.
- `references/polars-gpu-viz.md`: Polars GPU Engine query acceleration and
  Polars-compatible visualization paths.
- `references/cross-filter-patterns.md`: linked selections and callback state.
- `references/cuxfilter-migration.md`: mapping old cuxfilter concepts to this
  skill's direct-library output.

Use the templates when generating complete artifacts:

- `templates/panel-hv-datashader.md`: default notebook template.
- `templates/plotly-dash.md`: standalone Dash app template.
- `templates/streamlit.md`: standalone Streamlit app template.

## Default Workflow

1. Inspect or request schema.
2. Estimate rough data scale and choose the lightest backend/framework that
   satisfies the user request.
3. Select chart types from `references/visualization-patterns.md`.
4. Check `references/visualization-design-tips.md` for chart fit, color,
   hierarchy, and source/mapping transparency.
5. Use the matching template and parameterize paths, columns, and charts.
6. Add CPU fallback and comments at every GPU-to-CPU conversion.
7. Include run instructions for the exact output form.
8. If replacing cuxfilter code, include a short migration note showing the old
   concept and the new direct-library equivalent.

## Expected Output

For code-generation tasks, return:

- Files created or code cells to paste.
- Required packages already assumed by the selected framework.
- A clear run command or notebook execution instruction.
- Any GPU-memory or CPU-fallback caveats.

For advisory tasks, return:

- Recommended stack.
- Reasoning in one short paragraph.
- The relevant chart or migration pattern to use next.
