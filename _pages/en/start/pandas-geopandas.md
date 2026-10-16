---
title: "pandas and GeoPandas Cheat Sheet"
permalink: /en/pandas-geopandas/
lang: en
date: 2026-10-16T08:45:34-04:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: start
---

This cheat sheet covers useful **pandas** operations for working with tables and **GeoPandas** operations for vector data analysis. The examples assume the libraries are installed (`pip install pandas geopandas`) and the input files exist.

```python
import pandas as pd
import geopandas as gpd
```

## pandas: working with tables

### Read, write, and inspect

```python
df = pd.read_csv("data.csv")
df = pd.read_excel("data.xlsx")

df.head()                    # first rows
df.info()                    # columns and data types
df.describe()                # numeric summary statistics
df.shape                     # row and column counts
df.columns                   # column names
df.to_csv("output.csv", index=False)
```

For other formats, pandas also provides functions such as `read_json()` and `read_parquet()`.

### Select and filter

```python
df["population"]                         # one column (Series)
df[["name", "population"]]               # multiple columns (DataFrame)
df.loc[df["population"] > 1000]          # filter by condition
df.loc[df["population"] > 1000, ["name", "population"]]
df.iloc[0:5, 0:3]                        # select by position
df[df["name"].str.contains("lake", case=False, na=False)]
```

Use `.loc` to select by labels or conditions and `.iloc` to select by position.

### Create, modify, and clean columns

```python
df["density"] = df["population"] / df["area_km2"]
df = df.rename(columns={"zone_name": "name"})
df = df.drop(columns=["unused_column"])
df = df.dropna(subset=["population"])    # remove rows without a population
df["population"] = df["population"].fillna(0)
df["date"] = pd.to_datetime(df["date"], errors="coerce")
```

### Sort, summarize, and combine

```python
df = df.sort_values("population", ascending=False)

summary = (
    df.groupby("region", as_index=False)
      .agg(total_population=("population", "sum"),
           mean_population=("population", "mean"),
           count=("name", "count"))
)

result = df.merge(regions_table, on="region", how="left")
```

`merge()` combines tables using a key. Make sure the key has the same type and meaning in each table; use `validate=` when you know the expected relationship between rows.

## GeoPandas: working with vector data

A `GeoDataFrame` is a pandas table that also contains a geometry column and a coordinate reference system (CRS).

### Read, inspect, and write

```python
gdf = gpd.read_file("zones.gpkg")

gdf.head()
gdf.columns
gdf.crs                         # coordinate reference system
gdf.geometry                    # active geometry column
gdf.geom_type.value_counts()    # geometry types
gdf.total_bounds                # xmin, ymin, xmax, ymax

gdf.to_file("output.gpkg", driver="GPKG")
```

GeoPandas reads and writes formats such as GeoPackage, Shapefile, and GeoJSON. A CSV with coordinates can be converted to a `GeoDataFrame`:

```python
points = gpd.GeoDataFrame(
    df,
    geometry=gpd.points_from_xy(df["longitude"], df["latitude"]),
    crs="EPSG:4326",
)
```

### Check and reproject coordinates

```python
gdf = gdf.to_crs("EPSG:32198")  # reproject to a CRS suitable for the analysis
```

`set_crs()` declares the CRS of existing coordinates; it does not transform them. `to_crs()` transforms coordinates and requires a known source CRS. For distance or area calculations, use a projected CRS suitable for your region, not longitude/latitude coordinates.

### Filter and calculate with geometries

```python
selection = gdf[gdf["population"] > 1000]
selection = gdf[gdf.geometry.notna() & ~gdf.geometry.is_empty]

gdf["area_m2"] = gdf.geometry.area
gdf["length_m"] = gdf.geometry.length
gdf["centroid"] = gdf.geometry.centroid
gdf["buffer_zone"] = gdf.geometry.buffer(500)
```

Areas, lengths, and distances use the units of the CRS. Geometries should therefore use an appropriate projected CRS for reliable metric results.

### Common spatial operations

```python
zones = gpd.read_file("zones.gpkg")
study_area = gpd.read_file("study_area.gpkg").to_crs(zones.crs)

# Match each point to the polygon that contains it.
matched = gpd.sjoin(
    points.to_crs(zones.crs),
    zones[["zone_name", "geometry"]],
    how="left",
    predicate="within",
)

# Group polygons and combine their geometries.
dissolved = zones.dissolve(by="region", as_index=False)

# Keep the parts of features inside a study area.
clipped = gpd.clip(gdf, study_area)

# Combine attributes and geometries based on overlap.
intersection = gpd.overlay(layer_a, layer_b, how="intersection")
```

For `sjoin()`, `clip()`, and `overlay()`, the layers must use the same CRS. The `sjoin()` predicate (`within`, `intersects`, `contains`, etc.) defines the spatial relationship. `dissolve()` combines geometries in each group; `clip()` cuts features to a boundary; `overlay()` performs a topological operation between two layers.

### Quick visualization

```python
gdf.plot(column="population", cmap="OrRd", legend=True)
```

`plot()` creates a static map with Matplotlib. To explore an interactive map directly in a notebook, try `gdf.explore()`.

## Key points

- Inspect columns and data types before starting an analysis.
- Use `merge()` for attribute joins and `sjoin()` for spatial joins.
- Check the CRS before geometry operations; reproject layers when needed.
- Check for empty or invalid geometries and missing values.
- Save geospatial data in an appropriate format, such as GeoPackage.

## Official documentation

- [pandas documentation](https://pandas.pydata.org/docs/)
- [GeoPandas documentation](https://geopandas.org/en/stable/)
