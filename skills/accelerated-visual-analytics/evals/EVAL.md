# Evaluation Guidance

Use these evals to test whether the skill replaces historical cuxfilter
visualization workflows with maintained NVIDIA GPU visualization patterns.

## Questions

- Migrate cuxfilter dashboard sections from
  `evals/files/rapids-viz-guide-original.ipynb`.
- Construct a maintained visualization guide from
  `evals/files/rapids-viz-guide-base.ipynb`.
- Recommend the right HoloViz, Datashader, Plotly Dash, Streamlit, cuDF, Polars,
  or Dask route for large-data visualization prompts.
- Answer negative small-chart prompts without forcing GPU dashboard machinery.

## Behaviors

- The agent should activate
  `accelerated-visual-analytics` for large-data visualization,
  linked dashboard, cuxfilter migration, and Polars GPU visualization prompts.
- The agent should not import `cuxfilter`, recommend installing it, or assume
  the cuxfilter repository is present.
- The agent should preserve analytical intent when migrating old notebooks:
  charts, filters, linked behavior, graph/spatial context, and tables should
  map to maintained direct-library equivalents.
- The agent should use backend-aware data loading or CPU/GPU toggles when data
  size, hardware, or review environment is uncertain.
- The agent should reduce or bound data before browser display, Plotly/Dash
  conversion, Streamlit display, or table rendering.
- The agent should prefer notebook-first HoloViz/Panel/hvPlot/HoloViews/
  Datashader patterns unless the prompt asks for a standalone Dash or Streamlit
  application.

## Notes

- `evals/files/rapids-viz-guide-original.ipynb` is intentionally historical and
  contains cuxfilter usage. It should be treated as migration input only.
- `evals/files/rapids-viz-guide-base.ipynb` is a cleaned construction fixture.
  It contains setup and data-prep intent but no completed dashboard solution.
- `examples/rapids-viz-guide-successful.ipynb` is a human reference and should
  not be referenced by eval prompts as an input fixture.
