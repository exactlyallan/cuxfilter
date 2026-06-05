# Benchmark: accelerated-visual-analytics

## Evaluation Setup

- Source harness: not yet run.
- Agent: not yet run.
- Judge model: not yet run.
- Evaluation date: not yet run.
- Test dataset: `evals/evals.json` (8 positive cases, 1 negative case).
- Metrics: intended to follow the NVIDIA GPU data science skill benchmark
  convention: LLM judge score on a 0-100 scale, score delta vs no-skill
  baseline, token usage, duration, and runtime checks when applicable.

## Summary

No benchmark run has been completed yet. This gap is explicit so the skill does
not imply measured quality before it has been evaluated.

## Planned Coverage

| Task | Purpose |
| --- | --- |
| `crossfilter-dashboard-core` | Validate HoloViz-first Panel/hvPlot/HoloViews/Datashader dashboard generation with backend-aware loading and CPU fallback. |
| `visualization-library-routing` | Validate library choice guidance for dense charts, Dash, HoloViz-native geospatial, deck.gl/3D choropleth, and graph cases. |
| `cuxfilter-migration-without-cuxfilter` | Validate migration away from cuxfilter when cuxfilter is not installed and mixed `datashade=True`/`rasterize=True` defaults are avoided. |
| `nbbo-timeseries-explorer` | Validate a realistic time-series explorer prompt with spread, density, symbol filtering, and dense line rendering. |
| `polars-gpu-viz-pipeline` | Validate Polars GPU Engine routing for lazy query acceleration followed by bounded HoloViz/Altair/Plotly visualization. |
| `backend-size-routing-and-toggle` | Validate rough data-size routing, GPU working-set caveats, and CPU/GPU toggle guidance. |
| `rapids-viz-guide-cuxfilter-migration` | Validate migration from the historical RAPIDS visualization guide notebook without requiring cuxfilter. |
| `rapids-viz-guide-base-construction` | Validate construction of a maintained visualization guide from a cleaned base notebook fixture. |
| `negative-small-static-chart` | Validate that the skill does not trigger for tiny static reporting charts. |

## Notes

- This skill replaces the cuxfilter pattern, not the cuxfilter API.
- Evaluation should penalize any answer that imports `cuxfilter`, recommends
  installing it, or assumes the cuxfilter repository is locally present.
- Evaluation should penalize active recommendations of inactive RAPIDS projects
  from the RAPIDS API docs for new visualization examples.
- When this folder is folded into a larger repo, wire these evals into the
  repository-level benchmark harness rather than adding a separate harness here.
