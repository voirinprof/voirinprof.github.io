---
title: "Rasterio Cheat Sheet"
permalink: /en/rasterio/
lang: en
date: 2026-10-16T08:48:13-04:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: start
---

This cheat sheet covers essential **Rasterio** operations for reading, inspecting, processing, and writing georeferenced rasters. The examples assume Rasterio and NumPy are installed (`pip install rasterio numpy`) and the input file exists.

```python
import numpy as np
import rasterio
```

## Open and inspect a raster

Use a `with` context manager so the file is properly closed after use.

```python
with rasterio.open("image.tif") as src:
    print(src.crs)          # coordinate reference system
    print(src.bounds)       # spatial extent
    print(src.width, src.height)
    print(src.count)        # number of bands
    print(src.dtypes)       # data type of each band
    print(src.nodata)       # nodata value, if defined
    print(src.transform)    # pixel-to-coordinate transform
    print(src.res)          # resolution in CRS units
    print(src.profile)      # settings for creating a similar raster
```

`rasterio.open()` opens a file in read mode by default. Rasterio band indexes start at **1**.

## Read bands and handle nodata

```python
with rasterio.open("image.tif") as src:
    band_1 = src.read(1)                    # array (rows, columns)
    rgb = src.read([1, 2, 3])               # array (bands, rows, columns)
    masked_band = src.read(1, masked=True)

    height, width = src.shape
    band_count = src.count
```

With `masked=True`, nodata pixels are masked when the file defines a nodata value. For statistics, use NumPy functions that support masked arrays, such as `np.ma.mean(masked_band)`. Also check `src.nodata` and the validity mask if the source does not correctly identify missing pixels.

## Convert between rows, columns, and coordinates

```python
with rasterio.open("image.tif") as src:
    row, col = src.index(x, y)     # coordinates to row and column
    x, y = src.xy(row, col)        # pixel center to coordinates
```

`x` and `y` must be in the raster's CRS. `src.index()` returns the corresponding cell; `src.xy()` returns the cell center by default.

## Read a window from a large raster

Windows let you load only part of a raster at a time.

```python
from rasterio.windows import Window

with rasterio.open("image.tif") as src:
    window = Window(col_off=100, row_off=200, width=256, height=256)
    subset = src.read(1, window=window, masked=True)
    subset_bounds = rasterio.windows.bounds(window, src.transform)
    subset_transform = src.window_transform(window)
```

To process an entire raster in blocks, `src.block_windows(1)` provides windows suited to its storage layout:

```python
with rasterio.open("image.tif") as src:
    for _, window in src.block_windows(1):
        block = src.read(1, window=window, masked=True)
        # Process the block before moving to the next one.
```

## Clip a raster to a geometry

`rasterio.mask.mask()` clips the raster to a geometry expressed in the raster's CRS. `crop=True` adjusts the output extent to the clip.

```python
import geopandas as gpd
from rasterio.mask import mask

zones = gpd.read_file("study_area.gpkg")

with rasterio.open("image.tif") as src:
    zones = zones.to_crs(src.crs)
    geometries = zones.geometry
    clipped_data, clipped_transform = mask(
        src,
        [geom.__geo_interface__ for geom in geometries if geom is not None],
        crop=True,
        filled=False,
    )

    profile = src.profile.copy()
    profile.update(
        height=clipped_data.shape[1],
        width=clipped_data.shape[2],
        transform=clipped_transform,
    )
```

With `filled=False`, the result is a masked array, so pixels outside the geometry are not replaced with a nodata value. Check that the vector layer contains geometries and that they overlap the raster.

## Reproject a raster

Reprojection changes the CRS and recalculates the target raster grid. Choose a resampling method that fits the data: `nearest` for categories, `bilinear` or `cubic` for continuous data.

```python
from rasterio.warp import calculate_default_transform, reproject, Resampling

target_crs = "EPSG:32198"

with rasterio.open("source.tif") as src:
    target_transform, target_width, target_height = calculate_default_transform(
        src.crs,
        target_crs,
        src.width,
        src.height,
        *src.bounds,
    )

    profile = src.profile.copy()
    profile.update(
        crs=target_crs,
        transform=target_transform,
        width=target_width,
        height=target_height,
    )

    with rasterio.open("reprojected.tif", "w", **profile) as dst:
        for band in range(1, src.count + 1):
            reproject(
                source=rasterio.band(src, band),
                destination=rasterio.band(dst, band),
                src_transform=src.transform,
                src_crs=src.crs,
                dst_transform=target_transform,
                dst_crs=target_crs,
                resampling=Resampling.nearest,
            )
```

The source CRS must be known. For continuous data, replace `Resampling.nearest` with a suitable method and account for nodata propagation.

## Write a new raster

Copy the source profile to preserve its geospatial settings, then update dimensions and data type as needed.

```python
with rasterio.open("image.tif") as src:
    profile = src.profile.copy()
    profile.update(dtype="float32", nodata=-9999)

    data = src.read(1, masked=True).astype("float32")
    result = data.filled(-9999)

    with rasterio.open("result.tif", "w", **profile) as dst:
        dst.write(result, 1)
```

The profile must match the number of bands, dimensions, and type of the data being written. A multiband array uses the axis order `(bands, rows, columns)`.

## Calculate an index from two bands

Rasterio reads the pixels and NumPy applies the calculation to the arrays. This example calculates NDVI from a near-infrared band and a red band:

```python
with rasterio.open("image.tif") as src:
    red = src.read(3, masked=True).astype("float32")
    near_infrared = src.read(4, masked=True).astype("float32")

    denominator = near_infrared + red
    ndvi = np.ma.masked_where(
        np.ma.getmaskarray(red)
        | np.ma.getmaskarray(near_infrared)
        | (denominator == 0),
        (near_infrared - red) / np.ma.masked_equal(denominator, 0),
    )
```

Band numbers depend on the satellite product. Check the source metadata and band conventions before selecting bands.

## Key points

- Open rasters with `with rasterio.open(...)` and close them before reusing or moving files.
- Check the CRS, resolution, dimensions, band count, and nodata value before processing.
- Read large files by windows to limit memory use.
- Preserve or correctly update the profile, transform, and CRS when writing.
- Choose resampling based on the data and keep vector layers in the raster CRS when clipping.

## Official documentation

- [Rasterio documentation](https://rasterio.readthedocs.io/en/stable/)
- [Rasterio quickstart](https://rasterio.readthedocs.io/en/stable/quickstart.html)
