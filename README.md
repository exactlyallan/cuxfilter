# cuxfilter Sunset and Accelerated Visual Analytics Skill

This repository no longer contains the historical cuxfilter Python package.
The former package code, build recipes, release machinery, and Sphinx
documentation have been removed as part of the cuxfilter sunset.

The maintained replacement artifact is a portable Codex skill for accelerated
visual analytics:

```text
skills/accelerated-visual-analytics/
```

The skill helps agents migrate cuxfilter-era notebooks and dashboard concepts
to maintained NVIDIA accelerated data science and visualization patterns using
cuDF, Polars GPU Engine, HoloViz, Panel, hvPlot, HoloViews, GeoViews,
Datashader, Plotly Dash, Streamlit, Bokeh, and PyDeck.

## Repository Layout

```text
.
├── skills/
│   └── accelerated-visual-analytics/
│       ├── SKILL.md
│       ├── references/
│       ├── templates/
│       ├── examples/
│       ├── evals/
│       ├── BENCHMARK.md
│       └── skill-card.md
├── examples/
│   ├── legacy-notebooks/
│   └── legacy-doc-notebooks/
├── CONTRIBUTING.md
├── SECURITY.md
└── LICENSE
```

## Historical Notebooks

Historical cuxfilter notebooks have been preserved under `examples/` for
migration reference:

- `examples/legacy-notebooks/`: former top-level notebooks and supporting
  assets.
- `examples/legacy-doc-notebooks/`: notebook content that previously lived
  inside Sphinx user-guide docs.

These notebooks may still import or describe `cuxfilter`. Treat them as
historical input for migration, not as maintained runnable examples.

## Skill Evals

The skill includes evals and fixtures under:

```text
skills/accelerated-visual-analytics/evals/
```

Useful checks:

```bash
python3 -m json.tool skills/accelerated-visual-analytics/evals/evals.json >/tmp/evals.json.checked
python3 -m json.tool skills/accelerated-visual-analytics/evals/files/rapids-viz-guide-base.ipynb >/tmp/base.ipynb.checked
python3 -m json.tool skills/accelerated-visual-analytics/examples/rapids-viz-guide-successful.ipynb >/tmp/success.ipynb.checked
git diff --check
```

Do not commit generated eval results.
