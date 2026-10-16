---
title: "Oriented Object Detection in Remote Sensing with YOLO-OBB"
permalink: /en/yolo-obb/
lang: en
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: ai
---

## Overview

This page presents an oriented-object detection workflow for remote-sensing tiles using **YOLO-OBB**. It is based on the [YOLO-OBB Colab notebook](https://colab.research.google.com/drive/1iHFP0_si2-fxvMCXlTnfWdxzSnTnGZEt?usp=sharing), which applies a pretrained Ultralytics model to Bing tiles, then visualizes, counts, and georeferences the detections.

**OBB** means *oriented bounding box*. Unlike a rectangle aligned with the image axes, an oriented box can follow an object's rotation. This is useful in aerial or satellite imagery, where objects such as vehicles or buildings may appear at different angles.

## Difference from Faster R-CNN

The YOLO-OBB notebook directly uses `yolo11m-obb.pt`, pretrained on aerial imagery from the DOTA dataset. It performs **inference** without training the model in the notebook, so detections are limited to classes the model already knows.

The [Faster R-CNN notebook](/en/faster-rcnn/) follows a different approach: it replaces the classification head of a model pretrained on COCO and trains it on supplied annotations. YOLO-OBB can detect its pretrained classes immediately; detecting new classes requires preparing appropriate annotations and training or fine-tuning a YOLO-OBB model.

## 1. Load Tiles

The notebook expects 256 × 256 pixel JPEG tiles named according to the Bing/OSM tile scheme, for example `tile_157460_187752_z19.jpg`. The `X`, `Y`, and `z` values identify a tile's position in the Web Mercator grid and its zoom level. The notebook lists images in the provided directory:

```python
import glob
import os

TILE_DIR = "/content/jpeg_tiles"
tiles = sorted(glob.glob(os.path.join(TILE_DIR, "tile_*.jpg")))

print(f"{len(tiles)} tile(s) found")
```

Tile names matter: they allow the notebook to recover each detection's geographic location without a separate georeferencing file for every image.

## 2. Load the Model and Set Inference Parameters

Ultralytics loads the OBB model and provides its available class names. The notebook exposes three main parameters:

```python
from ultralytics import YOLO

model = YOLO("yolo11m-obb.pt")

CONFIDENCE_THRESHOLD = 0.25
IOU_THRESHOLD = 0.5
INFERENCE_SIZE = 256

results = model.predict(
    tiles,
    conf=CONFIDENCE_THRESHOLD,
    iou=IOU_THRESHOLD,
    imgsz=INFERENCE_SIZE,
    verbose=False,
)
```

* `conf` discards predictions whose confidence score is too low.
* `iou` sets the intersection-over-union threshold used to suppress redundant detections (NMS).
* `imgsz` sets the image size used for inference. The notebook suggests the native 256 pixels or an enlarged size, such as 640 pixels.

**Benefit:** a pretrained model can produce predictions quickly without setting up training in the notebook. Inference parameters can be adjusted and rerun without retraining. Check that the model's classes and the imagery match the objects of interest.

## 3. Read Oriented Boxes

For each result, Ultralytics provides boxes as center, width, height, and angle (`xywhr`), along with class IDs and confidence scores. The notebook collects them into records:

```python
import os

detections = []

for tile_path, result in zip(tiles, results):
    if result.obb is None or len(result.obb) == 0:
        continue

    for box in result.obb:
        cx, cy, width, height, angle = box.xywhr[0].tolist()
        class_id = int(box.cls[0])
        confidence = float(box.conf[0])

        detections.append({
            "tile": os.path.basename(tile_path),
            "class": model.names[class_id],
            "confidence": round(confidence, 3),
            "pixel_center_x": round(cx, 1),
            "pixel_center_y": round(cy, 1),
            "width_px": round(width, 1),
            "height_px": round(height, 1),
            "angle_rad": round(angle, 3),
        })
```

**Benefit:** an oriented box describes the shape and orientation of oblique objects better than a horizontal or vertical box, while scores allow filtering. The angle is part of the returned measurements; verify its unit and convention before converting geometries or comparing models.

## 4. Put Predictions on a Map

The notebook decodes `tile_X_Y_zZ`, then converts the pixel center to the corresponding latitude and longitude. The transformation follows the Web Mercator grid used by Bing tiles:

```python
import math

def pixel_to_latlon(px, py, tile_x, tile_y, zoom, tile_size=256):
    x = tile_x + px / tile_size
    y = tile_y + py / tile_size
    n = 2**zoom
    lon = x / n * 360.0 - 180.0
    lat = math.degrees(math.atan(math.sinh(math.pi * (1 - 2 * y / n))))
    return lat, lon
```

The geographic center can be added to each detection and exported to CSV. The notebook also overlays tile images in Folium and places markers coloured by class.

This conversion assumes that images are 256 × 256 pixels and that tile indices and zoom levels in their names are correct. The notebook geolocates the **center** of each box; it does not convert the complete oriented box outline to a geographic polygon.

## 5. Review and Summarize Results

The notebook uses Ultralytics visualization helpers to draw boxes on tiles and create a mosaic of results:

```python
import matplotlib.pyplot as plt

annotated_image = results[0].plot()
plt.figure(figsize=(7, 7))
plt.imshow(annotated_image[:, :, ::-1])  # BGR to RGB for Matplotlib
plt.axis("off")
plt.show()
```

Detections can also be exported to a table and summarized by class:

```python
import pandas as pd

detections_df = pd.DataFrame(detections)
detections_df.to_csv("detections.csv", index=False)

counts = detections_df["class"].value_counts()
counts.plot(kind="bar", title="Detections by class")
plt.tight_layout()
plt.show()
```

**Benefit:** the map, annotated images, and table provide complementary views: local visual checks, geographic distribution, and aggregate statistics.

## Benefits and Limitations

* **Oriented boxes:** object rotation is represented, which suits many aerial scenes.
* **Immediate inference:** the pretrained model can be applied to tiles without a training step in the notebook.
* **Geospatial results:** center coordinates, class, and score are collected in a CSV and can be displayed in Folium.
* **Predefined classes:** without fine-tuning, the model cannot learn new project-specific categories.
* **Evaluation is still needed:** validate thresholds and predictions on representative annotated examples; visual review alone does not measure precision or recall.
* **No complete OBB geometry on the map:** the notebook places a marker at each box center rather than projecting the full oriented polygon.
* **Imagery terms:** check access, attribution, and reuse requirements for background tiles used in analysis and publication.

See the [YOLO-OBB Colab notebook](https://colab.research.google.com/drive/1iHFP0_si2-fxvMCXlTnfWdxzSnTnGZEt?usp=sharing) to run the cells and try different thresholds or inference sizes.
