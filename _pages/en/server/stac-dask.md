---
title: "STAC, odc-stac, Dask, and xarray"
permalink: /en/stac-dask/
lang: en
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: gisserver
---

## Analyzing Satellite Imagery at Scale

Working with a time series of satellite imagery involves discovering the data, loading and aligning it, and performing calculations without necessarily keeping everything in memory. **STAC**, **odc-stac**, **Dask**, and **xarray** each address part of this problem and can be combined in one pipeline.

The examples on this page are based on the [stac_dask_samples](https://github.com/voirinprof/stac_dask_samples) repository. They use Sentinel-2 scenes over the Sherbrooke area and demonstrate the path from search to an NDVI composite.

## STAC: Discovering and Describing Data

**STAC** (*SpatioTemporal Asset Catalog*) is a standard for describing and discovering geospatial data by its spatial extent, time range, and properties. A catalog exposes **Collections** (groups of data) and **Items** (individual observations or scenes). An Item contains metadata and **assets**—links to associated files, such as image bands.

With `pystac-client`, you can query a STAC catalog without manually browsing its files:

```python
from pystac_client import Client

catalog = Client.open("https://earth-search.aws.element84.com/v1")
items = catalog.search(
    collections=["sentinel-2-l2a"],
    bbox=[-72.15, 45.30, -71.65, 45.65],
    datetime="2024-06-01/2024-08-31",
    query={"eo:cloud_cover": {"lt": 20}},
).item_collection()

print(f"{len(items)} scenes found")
```

**Benefit:** find scenes that cover a given area and time range before transferring raster data. Metadata can also be used to filter or rank scenes, for example by cloud cover. The repository demonstrates searching and inspecting Items in `01_stac/`.

## odc-stac: Loading Items as Aligned Data

**odc-stac** connects STAC Items to raster data loading. It selects the requested assets and can reproject data, align it to a common grid, and crop it to the area of interest. It returns an xarray Dataset; with *chunks* configured, data is loaded lazily using Dask.

```python
import odc.stac

ds = odc.stac.load(
    items,
    bands=["red", "nir"],
    crs="EPSG:32198",
    resolution=10,
    bbox=[-72.15, 45.30, -71.65, 45.65],
    chunks={"x": 512, "y": 512, "time": 1},
    groupby="solar_day",
)
```

**Benefit:** turn a collection of scenes and files into aligned multidimensional data without manually handling the download and reprojection of each band. Creating the Dataset does not necessarily mean that all pixels have been downloaded: computations are often deferred until `.compute()`.

## Dask: Computing in Chunks, When Needed

**Dask** provides chunked arrays and builds a task graph for execution. Instead of running each operation immediately, it can combine multiple operations and execute them with a local or distributed scheduler. `.compute()` triggers the calculation and returns a concrete result.

```python
import dask.array as da

red = da.ones((10_000, 10_000), chunks=(512, 512))
nir = red * 1.5
ndvi = (nir - red) / (nir + red)

# The computation runs here, on the required chunks.
mean_ndvi = ndvi.mean().compute()
```

**Benefit:** process data chunk by chunk instead of requiring an entire large array to fit in memory. To take advantage of lazy computation, group results into a single `dask.compute(...)` call when possible. Chunk size matters: chunks that are too small add scheduling overhead, while chunks that are too large increase memory pressure.

## xarray + Dask: Working with Dimensions and Coordinates

**xarray** represents multidimensional data with named dimensions, coordinates, and attributes. A raster time series, for example, can have `time`, `y`, and `x` dimensions. When the underlying data are Dask arrays, xarray operations are also lazy and execute chunk by chunk.

```python
# ds is the Dataset loaded with odc-stac.
red = ds["red"].astype("float32") / 10_000
nir = ds["nir"].astype("float32") / 10_000
ndvi = (nir - red) / (nir + red + 1e-10)

# Greenest-pixel composite for the time range.
greenest_pixel = ndvi.max(dim="time")

# Materialize the result when needed.
result = greenest_pixel.compute()
```

**Benefit:** express operations in terms of meaningful dimensions, such as a temporal mean with `.mean(dim="time")`, instead of manipulating anonymous array indices. Combining xarray and Dask lets you write time-series analyses while retaining chunked computation.

## Bringing the Technologies Together

The general workflow is:

1. **STAC** discovers Items matching the area, time range, and filters.
2. **odc-stac** loads the bands and organizes them in an aligned xarray Dataset.
3. **xarray** describes and combines transformations, such as the NDVI calculation.
4. **Dask** builds and executes computations on chunks when requested.
5. Computed results can be visualized or saved, for example as Zarr.

The [stac_dask_samples](https://github.com/voirinprof/stac_dask_samples) repository contains detailed scripts for STAC search and inspection, loading with odc-stac, cloud masking, Dask schedulers, xarray time series, and Zarr storage. The `05_pipeline/01_pipeline_ndvi.py` script combines these steps into a complete pipeline. Follow the repository README to create the environment and run the examples.
