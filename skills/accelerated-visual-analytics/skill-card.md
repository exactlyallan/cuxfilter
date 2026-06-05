# Skill Card: accelerated-visual-analytics

## Purpose

Generate or migrate accelerated visual analytics notebooks and dashboards
without depending on cuxfilter.

## Intended Use

- Migrate historical cuxfilter dashboards and notebooks to maintained direct
  library patterns.
- Generate notebook-first linked visual analytics over large tabular datasets.
- Recommend HoloViz, Panel, hvPlot, HoloViews, GeoViews, Datashader, Plotly
  Dash, Streamlit, Bokeh, PyDeck, cuDF, Polars GPU Engine, and Dask routes.

## Out of Scope

- Reimplementing or preserving the cuxfilter Python API.
- Installing cuxfilter or assuming the old package is present.
- Production service deployment beyond runnable dashboard starting points.

## Key Guardrails

- Keep CPU/GPU and library conversion boundaries explicit.
- Prefer backend toggles when hardware or data scale is uncertain.
- Reduce or bound data before browser tables and app-framework conversions.
- Treat historical cuxfilter notebooks as migration input, not maintained code.

## Validation Assets

- `evals/evals.json`
- `evals/EVAL.md`
- `evals/files/rapids-viz-guide-original.ipynb`
- `evals/files/rapids-viz-guide-base.ipynb`
- `examples/rapids-viz-guide-successful.ipynb`

## License

Apache-2.0, inherited from the repository license.
