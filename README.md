# Accelerated Visual Analytics Skill (formerly cuxfilter)

The repository contains a skill to help agents utilize best practices
for NVIDIA accelerated data science visualization, as well as migrate
cuxfilter-era notebooks and dashboard concepts.
The new focus is utilizing the excellent visualization libraries such as
cuDF, Polars GPU Engine, HoloViz, Panel, hvPlot, HoloViews, GeoViews,
Datashader, Plotly Dash, Streamlit, Bokeh, and PyDeck.

This repository no longer contains the historical cuxfilter Python package.
as the former has been sunset.
The last release of [cuxfilter was 26.06](https://github.com/rapidsai/cuxfilter/tree/release/26.06).

The replacement artifact is a portable skill for accelerated
visual analytics:

```text
skills/accelerated-visual-analytics/
```

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

## Skill Evals

The skill includes evals and fixtures under:

```text
skills/accelerated-visual-analytics/evals/
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

Thanks for using cuxfilter!
