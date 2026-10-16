---
title: "Advanced GeoPandas, pandas, Matplotlib, Rasterio, and NumPy"
permalink: /en/advanced-vector-raster/
lang: en
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: start
---

## Working Further with Vector and Raster Data

This page presents advanced techniques for processing and visualizing geospatial data in Python. The examples come from the standalone scripts in [`gmq580_advanced_vector_raster`](https://github.com/voirinprof/gmq580_advanced_vector_raster), organized in `src_vector/` and `src_raster/`. The sample data include business occupants and districts in Sherbrooke, as well as Landsat imagery.

## GeoPandas: Spatial Joins and Analysis

**GeoPandas** extends pandas tables with geometry operations. In addition to attribute joins, `gpd.sjoin()` matches features based on their spatial relationship. The predicate (`within`, `intersects`, `contains`, and others) defines the matching rule; `how` determines which features from the left layer are retained.

In the repository example, occupants are represented by points and districts by polygons. After both layers are put in the same coordinate reference system, a `within` join associates each point with its district:

```python
import geopandas as gpd

# Both layers must use the same CRS for the spatial join.
districts = gpd.read_file("districts.gpkg").to_crs("EPSG:32198")
occupants = gpd.GeoDataFrame(
    occupants_table,
    geometry=gpd.points_from_xy(occupants_table["lng"], occupants_table["lat"]),
    crs="EPSG:4326",
).to_crs("EPSG:32198")

joined = gpd.sjoin(
    occupants,
    districts[["NOM", "geometry"]],
    how="left",
    predicate="within",
)
counts = joined.groupby("NOM").size().reset_index(name="occupant_count")
districts = districts.merge(counts, on="NOM", how="left")
districts["occupant_count"] = districts["occupant_count"].fillna(0)
```

**Benefit:** points and polygons are matched by geometry rather than by a shared key. A left join also helps identify points that did not match any polygon. A point exactly on a boundary may behave differently with `within` and `intersects`, so choose the predicate according to the analysis question. See [`src_vector/05_sjoin_predicates.py`](https://github.com/voirinprof/gmq580_advanced_vector_raster/blob/master/src_vector/05_sjoin_predicates.py).

## pandas: Optimizing Attribute Tables

**pandas** provides the tabular data structures and attribute operations also used by GeoPandas. For large tables, data types affect memory use. Text columns with few distinct values can be stored as `category`; numeric types can be downcast when their precision is sufficient. Converting date columns to `datetime` makes them easier to filter and analyze.

The repository applies these optimizations to the occupants table:

```python
import pandas as pd

before_mb = occupants.memory_usage(deep=True).sum() / 1e6

for column in ["type_etablissement", "secteur_nom", "classe_nom"]:
    if column in occupants.columns:
        occupants[column] = occupants[column].astype("category")

occupants["lng"] = occupants["lng"].astype("float32")
occupants["lat"] = occupants["lat"].astype("float32")
occupants["date_creation"] = pd.to_datetime(occupants["date_creation"])

after_mb = occupants.memory_usage(deep=True).sum() / 1e6
print(f"Memory reduction: {(1 - after_mb / before_mb) * 100:.1f}%")
```

**Benefit:** suitable data types can reduce memory usage and make processing more efficient. Make sure numeric precision remains sufficient and categorical columns really have few unique values. See [`src_vector/02_memory_optimization.py`](https://github.com/voirinprof/gmq580_advanced_vector_raster/blob/master/src_vector/02_memory_optimization.py).

## Matplotlib: Comparing Multiple Maps

**Matplotlib** provides fine-grained control over figures and axes. Together with GeoPandas, it can display several mapped indicators side by side, each with its own colour map, legend, and title.

The repository example compares the number of occupants, their density, and the number of major chains by district:

```python
import matplotlib.pyplot as plt

fig, axes = plt.subplots(1, 3, figsize=(16, 6), layout="constrained")
variables = ["nb_occupants", "densite_occupants", "nb_grandes_chaines"]
titles = ["Occupant count", "Density (per km²)", "Major chains"]
cmaps = ["YlOrRd", "Blues", "Purples"]

for ax, column, title, cmap in zip(axes, variables, titles, cmaps):
    districts.plot(
        column=column,
        ax=ax,
        cmap=cmap,
        legend=True,
        legend_kwds={"shrink": 0.6, "label": title},
        edgecolor="white",
        linewidth=0.5,
    )
    ax.set_title(title)
    ax.set_axis_off()

fig.suptitle("District-level analysis")
fig.savefig("comparison.png", dpi=150, bbox_inches="tight")
```

**Benefit:** comparable maps in one figure make it easier to interpret multiple variables and avoid comparing images created with different settings. `layout="constrained"` helps prevent overlap between maps, titles, and legends. See [`src_vector/10_matplotlib_subplots.py`](https://github.com/voirinprof/gmq580_advanced_vector_raster/blob/master/src_vector/10_matplotlib_subplots.py).

## Rasterio: Reading and Processing a Raster by Windows

**Rasterio** reads and writes rasters and provides access to their georeferencing, bands, and metadata. For images too large to fit in memory, a `Window` reads one portion at a time.

The repository script iterates over Landsat tiles, reads only the red and near-infrared bands, and calculates NDVI for each tile:

```python
import numpy as np
import rasterio
from rasterio.windows import Window

tile_size = 256
ndvi_sum = 0.0
pixel_count = 0

with rasterio.open("landsat.tif") as src:
    for row_off in range(0, src.height, tile_size):
        for col_off in range(0, src.width, tile_size):
            width = min(tile_size, src.width - col_off)
            height = min(tile_size, src.height - row_off)
            window = Window(col_off, row_off, width, height)

            red = src.read(1, window=window).astype("float32")
            nir = src.read(4, window=window).astype("float32")
            ndvi = (nir - red) / (nir + red + 1e-10)

            ndvi_sum += ndvi.sum()
            pixel_count += ndvi.size

mean_ndvi = ndvi_sum / pixel_count
```

**Benefit:** an entire raster can be processed without loading all its bands into memory at once. Choose window sizes that fit available memory; if the image uses a `nodata` value, mask or exclude those pixels from statistics. See [`src_raster/03_windowed_reading.py`](https://github.com/voirinprof/gmq580_advanced_vector_raster/blob/master/src_raster/03_windowed_reading.py).

## NumPy: Vectorized Raster Calculations

**NumPy** represents raster bands as arrays and applies operations element-wise, without a Python loop over every pixel. This is useful for calculating spectral indices such as NDVI, the normalized difference vegetation index:

```python
import numpy as np

# red and nir are raster arrays with the same shape.
red = np.clip(red.astype("float32"), 0, None)
nir = np.clip(nir.astype("float32"), 0, None)

ndvi = (nir - red) / (nir + red + 1e-10)
vegetation_pct = (ndvi > 0.3).mean() * 100
```

**Benefit:** a concise expression calculates the index for all pixels at once, generally much faster than an explicit loop. `clip` follows the negative-value handling shown in the repository, and the small denominator term avoids division by zero. In production, account for `nodata` pixels as well; the repository demonstrates masked arrays in [`src_raster/08_nodata_masked_arrays.py`](https://github.com/voirinprof/gmq580_advanced_vector_raster/blob/master/src_raster/08_nodata_masked_arrays.py).

## Exploring the Examples

The [`gmq580_advanced_vector_raster`](https://github.com/voirinprof/gmq580_advanced_vector_raster) repository groups scripts by topic in `src_vector/` and `src_raster/`. Each script is standalone; prepare the sample data as described in the README. Further examples cover partial reads with Pyogrio, CRS management, invalid geometries, zonal statistics, reprojection, and Cloud Optimized GeoTIFF writing.
