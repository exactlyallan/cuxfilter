# Data Layer

Generated dashboards should choose the lightest dataframe layer that fits the
data, workload, and user context. Make GPU/CPU and local/distributed boundaries
explicit. Do not hide data movement: converting between cuDF, pandas, Polars,
and Dask collections can be expensive and should happen only at intentional
boundaries.

## Source Notes

- cuDF 26.04 docs describe cuDF as a GPU dataframe library for loading,
  joining, aggregating, filtering, and related dataframe operations:
  https://docs.rapids.ai/api/cudf/stable/
- `cudf.pandas` accelerates pandas code on GPU for supported operations and
  falls back to pandas for unsupported operations:
  https://docs.rapids.ai/api/cudf/stable/cudf_pandas/
- Dask docs advise using pandas when data fits comfortably in RAM because Dask
  adds parallelism overhead:
  https://docs.dask.org/en/latest/dataframe-best-practices.html
- Dask cuDF docs recommend the Dask DataFrame API with the `"cudf"` backend,
  Dask-CUDA for GPU clusters, Parquet for large data, and avoiding eager
  `compute()` on collections too large for one GPU:
  https://docs.rapids.ai/api/dask-cudf/stable/
- The historical cuxfilter Dask-cuDF notebook says to use cuDF when the data
  fits in single-GPU memory and Dask-cuDF when distributing across multiple
  GPUs, exceeding single-GPU memory, or analyzing many files:
  https://github.com/rapidsai/cuxfilter/blob/main/notebooks/cuxfilter_with_dask_cudf/README.md
- Polars GPU Engine accelerates Polars Lazy API query execution; Polars
  visualization docs show Polars dataframes working with Altair, hvPlot,
  Matplotlib, Seaborn, and Plotly paths:
  https://docs.pola.rs/user-guide/lazy/gpu/
  https://docs.pola.rs/user-guide/misc/visualization/

## Backend Choice

These are practical heuristics, not benchmark promises. File size is only a
weak proxy: compressed Parquet, CSV parsing, string columns, temporary columns,
joins, groupby state, indexes, browser tables, and copied chart data can all
make the in-memory working set much larger than the source file.

| Situation | Prefer | Why |
| --- | --- | --- |
| Tiny arrays, toy examples, small static charts, or quick schema inspection | pandas or Polars CPU | GPU and Dask startup/data-transfer overhead is not worth it. A size-10 array should not spin up cuDF unless the point is to test the GPU path. |
| Small to moderate data that fits comfortably in CPU RAM and is already pandas-shaped | pandas, with `cudf.pandas` as a low-change acceleration toggle when useful | Preserves pandas compatibility while allowing a GPU path without rewriting the dashboard. |
| Small to moderate data where the user prefers Polars or clean lazy plans | Polars eager/lazy CPU, with Polars GPU Engine as a collection-time toggle for supported lazy queries | Keeps one Polars pipeline and moves GPU acceleration to the execution boundary. |
| Large single-machine data where the estimated working set fits one GPU with margin | cuDF | Best fit for RAPIDS-native filtering, joins, groupby, datetime prep, and HoloViz/Datashader paths. |
| Data is larger than one GPU, spread across many files, or needs multi-GPU execution | Dask DataFrame with `dataframe.backend="cudf"` plus Dask-CUDA | Lets partitions stream across one or more GPUs; keep Dask lazy until reducing to a visualizable result. |
| Data fits CPU RAM but not GPU memory | pandas/Polars CPU, or Dask CPU if parallel/out-of-core execution is needed | Avoid forcing GPU memory pressure. Use GPU only after filtering/aggregation reduces the data enough. |
| Final chart/table is small after filtering or aggregation | Convert late to pandas/Polars as required by the plotting library | Reduce first, then cross library boundaries. |

## Size and Memory Heuristics

- Start with the simplest backend that keeps the workflow interactive.
- Prefer rough size bands over hard rules:
  - Tiny/small examples and report charts: stay CPU unless testing GPU code.
  - Moderate data: prefer pandas/Polars with a CPU/GPU toggle if the user may
    compare paths or run on mixed hardware.
  - Large, GPU-friendly data: use cuDF when the estimated working set fits GPU
    memory with safety margin.
  - Larger-than-device, many-file, or multi-GPU workloads: use Dask with a cuDF
    backend and reduce before materializing.
- Treat GPU startup, host-device transfer, and dataframe conversion as fixed
  costs. They dominate tiny examples and can erase any GPU speedup.
- Use cuDF for large single-GPU workloads only when the input plus temporary
  columns, joins, groupby state, and dashboard overhead fit device memory with
  margin. Interactive visualization can require much more memory than raw
  column byte counts suggest.
- Use Dask/Dask-cuDF when partitioning is required for memory, many files, or
  multi-GPU execution. Do not use Dask just to make small local data look more
  scalable; scheduling overhead is real.
- For Dask cuDF partition sizing, start from the RAPIDS guidance: roughly
  1/32-1/16 of single-GPU memory for shuffle-heavy workloads and 1/16-1/8
  otherwise, then tune from actual memory and dashboard behavior.
- Prefer Parquet/Arrow for large data. CSV is acceptable for examples and user
  supplied files, but it is usually the first thing to replace when scale
  matters.
- Reduce before visualizing. Filtering, aggregation, projection, downsampling,
  and Datashader rasterization are data-layer decisions, not only chart
  styling choices.

## Backend-Toggled Loader

Prefer generated code that can switch backend without rewriting dashboard
logic. Choose the default from rough data size and user context; keep the
selected backend visible in the dashboard.

```python
from pathlib import Path

def read_table(path, *, backend="auto"):
    """Load a table with an explicit CPU/GPU backend choice.

    backend:
      - "auto": try cuDF, fall back to pandas.
      - "cudf": require cuDF.
      - "pandas": use pandas.
    """
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
```

## Conversion Helpers

```python
def backend_name(df):
    return df.__class__.__module__.split(".")[0]

def to_pandas(df):
    """Convert only when a plotting library cannot consume the source frame."""
    if hasattr(df, "to_pandas"):
        return df.to_pandas()
    return df

def is_gpu_frame(df):
    return backend_name(df) == "cudf"

def unique_values(df, column, *, limit=None):
    """Return sorted Python values for widget choices across common backends."""
    values = df[column].dropna().unique()

    if hasattr(values, "to_arrow"):
        out = values.to_arrow().to_pylist()
    elif hasattr(values, "to_list"):
        out = values.to_list()
    elif hasattr(values, "tolist"):
        out = values.tolist()
    else:
        out = list(values)

    out = sorted(out)
    return out[:limit] if limit else out
```

## Rules

- Prefer parquet/arrow for large data. CSV is acceptable for examples but slower.
- Make backend choice visible and easy to change. `cudf.pandas`, Polars
  `collect(engine="gpu")`, and explicit loader backend parameters are useful
  when users need CPU/GPU switching without code rewrites.
- Use cuDF groupby, filtering, datetime parsing, and joins while data is on GPU.
- Convert to pandas only for Plotly, Bokeh, or Streamlit components that cannot
  consume cuDF directly.
- For Dash/Streamlit callbacks, keep the raw dataframe cached and perform
  filtering in one helper function.
- For Polars GPU Engine requests, use `references/polars-gpu-viz.md`. Do not
  introduce `polars[gpu]` as a new dependency by default for cuDF-oriented
  examples.

## Common Prep Patterns

```python
def ensure_datetime(df, column):
    if hasattr(df, "to_pandas"):
        import cudf
        df[column] = cudf.to_datetime(df[column])
    else:
        import pandas as pd
        df[column] = pd.to_datetime(df[column])
    return df

def top_n_categories(df, column, n=20):
    counts = df.groupby(column).size().reset_index(name="count")
    counts = counts.sort_values("count", ascending=False).head(n)
    return counts
```
