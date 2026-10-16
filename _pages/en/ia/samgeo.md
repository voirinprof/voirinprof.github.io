---
title: "Satellite Image Segmentation with SamGeo"
permalink: /en/samgeo/
lang: en
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: ai
---

## Overview

This page presents a satellite-image segmentation workflow with **SamGeo**, a library that makes it easier to use the **Segment Anything Model (SAM)** with geospatial data. It is based on the [SamGeo Colab notebook](https://colab.research.google.com/drive/1JAZv-snF_DgT0sdBgbKM7SCEsUvHqmS_?usp=sharing).

The notebook uses a pretrained SAM model in automatic mask-generation mode: it finds and outlines regions in the image without additional training. The workflow is:

**satellite imagery → GeoTIFF → SAM segmentation → georeferenced mask → GIS polygons**

> **Segmentation and classification are different tasks.** SAM can outline objects or regions, but the resulting segments are not automatically labelled as “building,” “tree,” “road,” or any other class.

## 1. Set the Area and Parameters

The notebook defines a small geographic extent, zoom level, SAM model, and segmentation options:

```python
bbox = [-95.3704, 29.6762, -95.3680, 29.6775]
zoom = 19
model_type = "vit_b"
min_size = 100
unique = True
```

The `bbox` order is `[min_longitude, min_latitude, max_longitude, max_latitude]`. `zoom` controls the detail level of the retrieved image; `vit_b` selects a relatively lightweight model variant; `min_size` removes small segments, and `unique=True` assigns a distinct value to each segmented object.

Starting with a small area helps control runtime and memory. Results depend on the source imagery, its resolution, the model, and these parameters.

## 2. Create and Inspect a GeoTIFF

The notebook uses SamGeo's `tms_to_geotiff` to retrieve imagery for the area as a GeoTIFF:

```python
from samgeo.common import tms_to_geotiff

image = "satellite.tif"
tms_to_geotiff(
    output=image,
    bbox=bbox,
    zoom=zoom,
    source="Satellite",
    overwrite=True,
)
```

Unlike an ordinary JPEG or PNG, a GeoTIFF preserves spatial information needed to place pixels on Earth. Before segmentation, the notebook uses Rasterio to inspect its coordinate reference system, extent, resolution, and bands:

```python
import rasterio

with rasterio.open(image) as src:
    print("CRS:", src.crs)
    print("Dimensions:", src.width, "x", src.height)
    print("Resolution:", src.res)
    print("Bounds:", src.bounds)
    rgb = src.read([1, 2, 3]).transpose(1, 2, 0)
```

**Benefit:** inspecting geospatial metadata helps verify that the input is a usable raster and that the result can align with other layers. Also check the usage and attribution terms of the chosen imagery source.

## 3. Initialize SAM and Segment the Image

SamGeo hides some of the complexity of using the model and provides an interface suited to geospatial rasters. The notebook selects a GPU when one is available, then generates masks automatically:

```python
import torch
from samgeo import SamGeo

device = "cuda" if torch.cuda.is_available() else "cpu"
sam = SamGeo(
    model_type="vit_b",
    automatic=True,
    device=device,
)

mask = "masks.tif"
sam.generate(
    source=image,
    output=mask,
    foreground=True,
    unique=True,
    min_size=100,
)
```

The `masks.tif` output is a georeferenced segmentation raster. With `unique=True`, pixels in each segment receive a value that distinguishes the regions. `min_size` controls the minimum retained size; adjust it to remove tiny fragments or keep finer details.

**Benefit:** automatic segmentation makes it possible to explore an image without first creating an annotation dataset or training a specialized model. Selecting the device also lets the workflow use a GPU when available.

## 4. Visualize and Inspect Masks

The notebook displays the segments over the image and reads the mask raster with Rasterio:

```python
sam.show_anns(figsize=(12, 10), axis="off", alpha=0.45)

with rasterio.open(mask) as src:
    objects = src.read(1)
    print("Mask CRS:", src.crs)
    print("Bounds:", src.bounds)
```

Display colours distinguish segments; they do not represent thematic classes. Visually inspect several parts of the image: depending on resolution, contrast, and landscape complexity, SAM may merge objects, split one object into several regions, or create boundaries that do not match the object of interest.

## 5. Convert the Raster to Polygons

The mask can be vectorized to a GeoPackage and opened with GeoPandas:

```python
import geopandas as gpd

vector = "segments.gpkg"
sam.tiff_to_vector(mask, vector)

segments = gpd.read_file(vector)
print("Polygon count:", len(segments))
print("CRS:", segments.crs)
```

The polygons can then be inspected, filtered, compared with other layers, or used in QGIS. The notebook visualizes them with GeoPandas:

```python
ax = segments.plot(figsize=(10, 8), edgecolor="black")
ax.set_title("Segmented objects")
ax.set_axis_off()
```

**Benefit:** converting the mask to vector data makes it easier to use in GIS tools and vector workflows. However, conversion may produce many polygons; check their geometry, size, and usefulness before further analysis.

## Benefits and Limitations

* **No training in the notebook:** SAM provides exploratory segmentation directly from a pretrained model.
* **Georeferenced data:** the source GeoTIFF, mask, and vectors retain spatial context for GIS use.
* **Parameters to explore:** extent, zoom, model variant, and minimum size affect result granularity.
* **No automatic semantic labels:** segmentation outlines regions but does not determine their thematic class.
* **Results need validation:** inspect and, if needed, correct boundaries for the intended task, imagery, and resolution.
* **Computing resources:** large images or heavier SAM variants need more memory and time; starting with a small extent is recommended.

The notebook lets you download the GeoTIFF mask and GeoPackage polygons for use in QGIS. See the [SamGeo Colab notebook](https://colab.research.google.com/drive/1JAZv-snF_DgT0sdBgbKM7SCEsUvHqmS_?usp=sharing) to run the steps and experiment with another area or different parameters.
