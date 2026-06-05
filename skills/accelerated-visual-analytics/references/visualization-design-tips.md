# Visualization Design Tips

Use this reference when generated dashboard code needs chart selection, layout,
color, accessibility, or source/mapping transparency decisions. Apply the
guidance in the artifact; do not turn the response into a design tutorial unless
the user asks for design rationale.

Sources:

- Nielsen Norman Group, "10 Usability Heuristics for User Interface Design":
  https://www.nngroup.com/articles/ten-usability-heuristics/
- Tableau, "Data Visualization Tips and Best Practices":
  https://www.tableau.com/visualization/data-visualization-best-practices
- Johns Hopkins Libraries, "Designing Effective Data Visualizations":
  https://guides.library.jhu.edu/datavisualization/design
- University of Missouri, "Visualization Best Practices":
  https://udair.missouri.edu/visualization-chart-best-practices/
- The Data Visualisation Catalogue:
  https://datavizcatalogue.com/
- Colorcet:
  https://colorcet.holoviz.org/

## Agent Rules

- Start from the analytical task and column roles, not from the plotting library
  that happens to be convenient.
- Make the primary question visible first. Put secondary charts, controls, and
  tables after the main view.
- Keep linked state understandable: active filters, brush selection, backend,
  row count, table cap, and sample/aggregate status should be visible.
- Use the user's domain language for titles, labels, legends, and controls.
  Preserve raw column names only when the user is inspecting schema.
- Remove visual noise before adding explanation. Prefer clear labels, ordering,
  grouping, and units over decorative styling.
- Keep color mappings stable across charts and interaction states.
- Make data movement and data mapping inspectable. Generated dashboards should
  show what data is loaded, filtered, aggregated, sampled, or converted.

## Chart Selection

Start with the analytical task, not the available plotting library.

| Task | Prefer | Avoid or use carefully |
| --- | --- | --- |
| Change over time | Line chart, area chart for cumulative volume, horizon/Datashader line for dense series | Bar charts for high-frequency time series; smoothed lines without disclosure |
| Compare categories | Bar chart, dot plot, sorted lollipop chart | Pie/donut charts for precise comparison |
| Rank values | Sorted horizontal bar chart | Alphabetical sorting unless the category has natural order |
| Distribution | Histogram, density plot, box plot, violin plot | Mean-only summaries when spread, skew, or outliers matter |
| Relationship between variables | Scatterplot, hexbin, rasterized scatter, bubble chart when size adds meaning | Dual axes; unbounded raw scatter when points overlap heavily |
| Part-to-whole | 100% stacked bar, stacked bar, treemap for rough hierarchy, donut only for very few categories | Pie/donut with more than 3-5 slices; exploded or 3D effects |
| Geographic pattern | Choropleth for regional rates, point/hex/tile map for locations, flow map for movement | Maps when geography is not the insight; raw counts on choropleths |
| Network relationships | Node-link graph, adjacency matrix, edge bundling for dense relationships | Graphs when a table or grouped bar answers the question more clearly |
| Exact lookup | Table with sorting, filtering, sticky headers, and aligned numeric columns | Dense chart when users need exact values |

Use The Data Visualisation Catalogue only as a lookup when the task is unusual
or when comparing candidate chart families by function.

## Hierarchy, Navigation, and Noise

- Put global controls in a stable sidebar or header.
- Put the primary chart first, supporting charts next, and bounded detail tables
  last.
- Keep current state near the charts it affects: selected symbols, date range,
  aggregation level, row counts, and backend.
- Sort categories by value unless there is a natural or domain-specific order.
- Keep axis labels, legends, units, date granularity, and color mappings
  consistent across linked views.
- Avoid heavy gridlines, decorative backgrounds, unnecessary borders, duplicated
  legends, 3D effects, and labels for every mark.
- Provide clear reset or clear-selection affordances when selections can become
  hard to see.

## Color Selection

- Categorical data: use a distinct qualitative palette with stable category
  mapping.
- Sequential numeric data: use a perceptually ordered low-to-high palette such
  as Viridis, Cividis, or a suitable Colorcet palette.
- Diverging numeric data: use a two-sided palette only when there is a meaningful
  midpoint such as zero, target, or baseline.
- Dense Datashader/HoloViz views: prefer Colorcet or HoloViz-compatible
  perceptually uniform palettes that reveal structure across the full value
  range.
- Keep default marks neutral and reserve one or two accent colors for emphasis.
- Do not use color as the only channel for meaning. Reinforce with labels,
  position, shape, line style, pattern, or grouping when interpretation matters.
- Keep color mappings consistent across charts and dashboard states. If
  "member" is blue in one view, it should not become orange in another.
- Test contrast and color-vision-deficiency behavior for key states, especially
  selections, warnings, and categorical distinctions.
- Avoid rainbow palettes unless the chosen palette is intentionally
  perceptually designed for the data and there is a clear reason.

## Data and Mapping Transparency

Every generated dashboard should make the data contract inspectable.

- Show the data source, file/path/table, update date if known, and whether the
  view is full data, sampled data, filtered data, or aggregated data.
- Expose key transformations: filters, joins, derived fields, bin sizes,
  resampling interval, normalization, rates versus raw counts, and
  missing-value handling.
- Make encodings clear in labels, captions, or comments: x, y, color, size,
  facet, line style, geometry, and selection state.
- Label units and denominators. Do not make percent, count, rate, and normalized
  score visually interchangeable.
- Start bar-chart quantitative axes at zero. If a line chart uses a truncated
  axis, make the scale choice clear.
- For maps, prefer normalized rates on choropleths and disclose boundaries,
  projection assumptions, and aggregation level when known.
- For linked selections, show whether summaries and tables reflect all data,
  active filters, active brush selection, or a bounded sample.
- Surface backend and performance boundaries when relevant: CPU/GPU backend,
  pandas/cuDF/Polars/Dask path, rows loaded, rows displayed, and table cap.

## Pre-Final Checklist

- Chart types match the analytical task and data shape.
- The primary view answers the user's main question before secondary detail.
- Filters, selections, row counts, backend, and table caps are visible.
- Color encodes meaning consistently and accessibly.
- Labels, legends, axes, units, and denominators are clear.
- Source, transformations, and visual encodings are transparent.
- Dense views use aggregation, binning, or Datashader instead of unbounded raw
  marks.
- Tables and maps appear only when they answer the question better than a
  simpler chart.
