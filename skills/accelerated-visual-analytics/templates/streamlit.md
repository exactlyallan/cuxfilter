# Template: Streamlit

Use this when the user wants the quickest standalone app. Streamlit is less
natural for dense linked charts than Panel/HoloViews, but it is ergonomic for
filters plus a few coordinated views.

```python
from pathlib import Path

import streamlit as st


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
            st.warning(f"Falling back to pandas because cuDF load failed: {exc}")

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


@st.cache_resource(show_spinner=False)
def load_data(path, backend_choice):
    df, backend = read_table(path, backend=backend_choice)
    return df, backend


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
    """Keep Streamlit display conversion bounded after filtering."""
    return to_pandas(df.head(n))


st.set_page_config(page_title=CONFIG["title"], layout="wide")
st.title(CONFIG["title"])

df, backend = load_data(CONFIG["path"], CONFIG["backend"])
st.caption(f"Data backend: {backend}")

category_values = to_pandas(top_categories(df, CONFIG["category_col"]))[
    CONFIG["category_col"]
].tolist()
selected_categories = st.sidebar.multiselect(
    CONFIG["category_col"],
    options=category_values,
    default=category_values[:5],
)

filtered = filter_frame(df, selected_categories)
plot_rows = bounded_rows(filtered)

left, right = st.columns(2)
with left:
    st.subheader("Category counts")
    counts = to_pandas(top_categories(filtered, CONFIG["category_col"]))
    st.bar_chart(counts, x=CONFIG["category_col"], y="count")

with right:
    st.subheader("Metric over time")
    st.line_chart(
        plot_rows,
        x=CONFIG["time_col"],
        y=CONFIG["metric_col"],
    )

st.subheader("Sample rows")
st.dataframe(bounded_rows(filtered, n=1000), use_container_width=True)
```

## Run

```bash
streamlit run app.py
```

Do not store large filtered dataframes in `st.session_state`. Cache the source
data and keep session state limited to filter values.
