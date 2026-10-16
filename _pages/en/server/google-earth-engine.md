---
title: "Google Earth Engine with Python"
permalink: /en/google-earth-engine/
lang: en
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: gisserver
---

## Overview

**Google Earth Engine (GEE)** is a cloud platform for exploring and analyzing geospatial data at scale. It provides access to a large catalog of satellite imagery and geographic datasets, and runs processing on Google's infrastructure instead of requiring all data to be downloaded to your computer.

This page is based on the Colab notebook [“Three ways to create a Sentinel-2 composite”](https://colab.research.google.com/drive/1yH1dolerbJT7wXCtE1H9AZIJGcVvFZnJ?usp=sharing), which compares several composites over Sherbrooke in summer 2025.

## The Python Client and Server-Side Processing

The `ee` Python library provides a client interface to Earth Engine. Calls such as `filterBounds`, `map`, `median`, and `clip` create objects and operations describing the analysis. Pixel calculations run on Earth Engine servers when a result is requested or exported; source images are not loaded into Python memory with each call.

```python
import ee

ee.Authenticate()
ee.Initialize(project="YOUR_GOOGLE_CLOUD_PROJECT")

aoi = ee.Geometry.Rectangle([-72.05, 45.33, -71.80, 45.47])
collection = (
    ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED")
    .filterBounds(aoi)
    .filterDate("2025-06-01", "2025-09-01")
    .filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", 60))
)
composite = collection.median().clip(aoi)
```

Here, `collection` and `composite` are client-side Earth Engine objects describing server-side computation. Calling `median()` does not by itself download every scene into Python.

## Computation Graphs and Lazy Evaluation

Earth Engine builds a **computation graph** from chained operations. This is **lazy evaluation**: steps are described before they are executed. The server can optimize the workflow and calculate only the data required for the requested result.

In the notebook, `map`, `updateMask`, `normalizedDifference`, `median`, `mosaic`, and `qualityMosaic` are chained this way. For example, cloud masking and adding an NDVI band can be described by a function applied to each image:

```python
classes_to_remove = [0, 1, 3, 8, 9, 10]

def prepare_image(image):
    scl = image.select("SCL")
    valid_mask = (
        scl.remap(classes_to_remove, [0] * len(classes_to_remove), 1)
        .eq(1)
    )
    image = image.updateMask(valid_mask)
    ndvi = image.normalizedDifference(["B8", "B4"]).rename("NDVI")
    return image.addBands(ndvi)

prepared_collection = collection.map(prepare_image)
median_composite = prepared_collection.median().clip(aoi)
```

`map(prepare_image)` describes the operation to apply to collection images; it does not create a Python loop that downloads scenes one by one.

### Retrieving Small Values on the Client

`getInfo()` forces server-side evaluation and synchronously transfers the result to Python. The notebook uses it to retrieve dates and cloud percentages, which are small metadata values. Avoid using it to retrieve large images or entire collections:

```python
dates = collection.aggregate_array("system:time_start").getInfo()
```

A useful rule is to keep images and large results on the server and transfer only small values needed for display or inspection.

## Exploring the Data Catalog

The **[Google Earth Engine Data Catalog](https://developers.google.com/earth-engine/datasets/catalog/)** lists collections and products available in GEE. Each dataset page describes coverage, bands, properties, resolution, available dates, and the ID used in code.

The notebook uses [Sentinel-2 MSI: MultiSpectral Instrument, Level-2A (SR) Harmonized](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED), identified in the API as `COPERNICUS/S2_SR_HARMONIZED`. Before building a query, inspect its bands, the classes of its `SCL` band, and scene properties such as `CLOUDY_PIXEL_PERCENTAGE`.

The cloud percentage property is calculated for the entire scene, not just the area of interest. It is therefore a first-pass filter and does not replace pixel-level cloud masking.

## Three Ways to Create a Composite

After masking unwanted pixels, the notebook compares three operations on the collection:

```python
median_composite = prepared_collection.median().clip(aoi)

mosaic_composite = (
    prepared_collection
    .sort("CLOUDY_PIXEL_PERCENTAGE", False)
    .mosaic()
    .clip(aoi)
)

greenest_pixel_composite = (
    prepared_collection
    .qualityMosaic("NDVI")
    .clip(aoi)
)
```

* **Per-pixel median:** calculates the median of each band for each pixel in the time range, reducing the influence of extreme values.
* **Mosaic:** combines images by filling masked pixels with available observations. Image ordering determines which overlapping pixels take priority.
* **`qualityMosaic("NDVI")`:** chooses, for each pixel, the image with the highest NDVI and takes the corresponding bands from that observation. The resulting “greenest pixel” composite does not necessarily represent a single date.

Comparing outputs with the same visualization stretch, as in the notebook, helps reveal differences between methods.

## Exporting: An Asynchronous Task

An export is not an immediate download into the Python session. `ee.batch.Export` creates an **asynchronous task** that runs on Earth Engine. Start the task and then monitor it until it finishes; an export to Google Drive will appear in the specified Drive folder.

```python
task = ee.batch.Export.image.toDrive(
    image=median_composite,
    description="sentinel2_composite_sherbrooke",
    folder="earth_engine_exports",
    fileNamePrefix="sentinel2_composite",
    region=aoi,
    scale=10,
    maxPixels=1e13,
    fileFormat="GeoTIFF",
)
task.start()

print(task.status())
```

The initial state is usually `READY` or `RUNNING`. To monitor the task from Python, check its status periodically:

```python
import time

while task.active():
    print(task.status()["state"])
    time.sleep(30)

print(task.status())
```

A task may take time, fail, or require parameters suited to the output size and projection. Check its final state and any error messages; once complete, retrieve the file from Google Drive. Export parameters such as `region`, `scale`, `crs`, and `maxPixels` define the output raster's extent and resolution.

## Key Takeaways

* Methods in the `ee` library describe operations executed on Earth Engine servers; image objects are not automatically loaded into the Python client.
* Operations are lazily evaluated through a computation graph, until a result is requested or an export task is run.
* `getInfo()` transfers data to the client and should be reserved for small values and metadata.
* The Earth Engine Data Catalog helps identify collections and understand their bands and properties.
* Exports are asynchronous tasks: start them, monitor their status, and retrieve files after completion.

See the [Colab notebook](https://colab.research.google.com/drive/1yH1dolerbJT7wXCtE1H9AZIJGcVvFZnJ?usp=sharing) to run the full comparison of Sentinel-2 composites over Sherbrooke.
