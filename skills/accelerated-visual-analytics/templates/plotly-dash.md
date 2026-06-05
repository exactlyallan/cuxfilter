# Template: Plotly Dash

Use this when the user explicitly wants Dash or when deployability through a
Dash/Flask-style app matters more than notebook ergonomics.

```python
from pathlib import Path

from dash import Dash, Input, Output, dcc, html
import plotly.express as px


CONFIG = {
    "path": "data.parquet",
    "backend": "auto",
    "time_col": "timestamp",
    "category_col": "category",
    "metric_col": "value",
    "title": "GPU Cross-Filter Dashboard",
}


def read_table(path, *, backend="auto"):
    path = Path(path)
    suffix = path.suffix.lower()

    if backend not in {"auto", "cudf", "pandas"}:
        raise ValueError("backend must be 'auto', 'cudf', or 'pandas'")

    if backend in {"auto", "cudf"}:
        try:
            import cudf

            if suffix in {".parquet", ".pq"}:
                return cudf.read_parquet(path), "cudf"
            if suffix in {".arrow", ".feather"}:
                return cudf.read_feather(path), "cudf"
            if suffix == ".csv":
                return cudf.read_csv(path), "cudf"
        except Exception as exc:
            if backend == "cudf":
                raise
            print(f"Falling back to pandas because cuDF load failed: {exc}")

    import pandas as pd

    if suffix in {".parquet", ".pq"}:
        return pd.read_parquet(path), "pandas"
    if suffix in {".arrow", ".feather"}:
        return pd.read_feather(path), "pandas"
    if suffix == ".csv":
        return pd.read_csv(path), "pandas"
    raise ValueError(f"Unsupported file type: {suffix}")


def to_pandas(df):
    if hasattr(df, "to_pandas"):
        return df.to_pandas()
    return df


df, backend = read_table(CONFIG["path"], backend=CONFIG["backend"])


def top_categories(df, category_col, n=20):
    counts = df.groupby(category_col).size().reset_index(name="count")
    counts = counts.sort_values("count", ascending=False).head(n)
    return counts


def filter_frame(df, categories):
    out = df
    if categories:
        out = out[out[CONFIG["category_col"]].isin(categories)]
    return out


def bounded_rows(df, n=10000):
    """Keep Plotly conversion bounded after GPU/CPU filtering."""
    return to_pandas(df.head(n))


category_options = [
    {"label": value, "value": value}
    for value in to_pandas(top_categories(df, CONFIG["category_col"]))[
        CONFIG["category_col"]
    ].tolist()
]

app = Dash(__name__)
app.layout = html.Div(
    [
        html.H1(CONFIG["title"]),
        html.Div(f"Data backend: {backend}"),
        dcc.Dropdown(
            id="category-filter",
            options=category_options,
            multi=True,
            placeholder=f"Filter {CONFIG['category_col']}",
        ),
        dcc.Graph(id="bar-chart"),
        dcc.Graph(id="time-chart"),
    ]
)


@app.callback(
    Output("bar-chart", "figure"),
    Output("time-chart", "figure"),
    Input("category-filter", "value"),
)
def update_figures(categories):
    filtered = filter_frame(df, categories)

    counts = to_pandas(top_categories(filtered, CONFIG["category_col"]))
    bar = px.bar(
        counts,
        x=CONFIG["category_col"],
        y="count",
        title=f"Top {CONFIG['category_col']} values",
    )

    pdf = bounded_rows(filtered)
    time_fig = px.line(
        pdf,
        x=CONFIG["time_col"],
        y=CONFIG["metric_col"],
        title=f"{CONFIG['metric_col']} over time",
    )

    return bar, time_fig


if __name__ == "__main__":
    app.run(debug=True)
```

## Run

```bash
python app.py
```

Dash/Plotly may require pandas-like data at figure construction time. Filter
with the selected backend first, then convert a reduced or bounded dataframe.
