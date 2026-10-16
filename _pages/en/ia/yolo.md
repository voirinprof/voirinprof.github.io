---
title: "Detecting Cars in Bing Tiles with YOLO"
permalink: /en/yolo-detection/
lang: en
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: ai
---

## Overview

This page presents a car-detection workflow for remote-sensing imagery using **YOLO** and Ultralytics. It is based on the [YOLO Colab notebook](https://colab.research.google.com/drive/15DFRZD29aqEAIpSWFuIo1Qk0FZu2871d?usp=sharing), which prepares annotated Bing tiles, trains a model, evaluates its results, and runs the final model on test images.

Unlike [YOLO-OBB](/en/yolo-obb/), which uses oriented boxes and a pretrained model for inference, this notebook trains a YOLO detector with axis-aligned rectangular boxes. Unlike Faster R-CNN, it uses the Ultralytics training API instead of a custom PyTorch training loop in the notebook.

## 1. Prepare Images and Annotations

The notebook uses Bing tiles and car annotations created with the VIA annotation tool. Each VIA rectangle—defined by its top-left corner, width, and height in pixels—is converted to YOLO format:

```text
class_id normalized_x_center normalized_y_center normalized_width normalized_height
```

Coordinates are divided by image width or height so they fall between 0 and 1. The notebook illustrates the calculation as follows:

```python
x_center = (x_region + width_region / 2) / image_width
y_center = (y_region + height_region / 2) / image_height
width = width_region / image_width
height = height_region / image_height

label_line = f"0 {x_center} {y_center} {width} {height}"
```

Each text file shares its image's base name and uses the `.txt` extension. Class ID `0` represents the sole class here, `car`. Visually inspect some annotations after conversion: incorrect dimensions or image-to-label matching can undermine the entire training process.

## 2. Create Train, Validation, and Test Sets

The notebook splits image/annotation pairs into three subsets. The suggested split is 80% for training, 10% for validation, and the remainder for testing:

```python
train_ratio = 0.8
val_ratio = 0.1

train_count = int(len(label_files) * train_ratio)
val_count = math.ceil(len(label_files) * val_ratio)

train_files = shuffled_files[:train_count]
val_files = shuffled_files[train_count : train_count + val_count]
test_files = shuffled_files[train_count + val_count :]
```

* **train** is used to update model weights;
* **val** tracks performance during training and helps tune parameters;
* **test** is reserved for final evaluation after model choices have been made.

A fixed random seed makes the split reproducible. For geographically adjacent tiles, a spatial split by area is often more reliable than random sampling: it reduces the risk of near-identical tiles appearing in both train and test and producing an overly optimistic estimate.

## 3. Describe the Dataset for Ultralytics

The notebook creates a YAML file that tells Ultralytics where the images are and which classes the model should detect:

```yaml
path: /content/dataset
train: images/train
val: images/val
test: images/test
nc: 1
names:
  - car
```

`path` is the dataset root; `train`, `val`, and `test` point to their respective image folders. YOLO labels are stored in the corresponding `labels/` directories. `nc` is the number of classes, and `names` lists them in class-ID order.

## 4. Train the Detector

The notebook uses medium YOLOv8 (`yolov8m`) as its starting point and trains with the configured parameters:

```python
from ultralytics import YOLO

model = YOLO("yolov8m.pt")
model.train(
    data="/content/cars.yaml",
    epochs=100,
    imgsz=640,
    batch=10,
    name="car_yolo",
)
```

* `epochs` controls how many passes are made over the training data;
* `imgsz` sets the image size used by the model;
* `batch` is the number of images processed per batch and must fit available memory;
* model size (`yolov8n`, `yolov8s`, `yolov8m`, and so on) trades off resources, speed, and capacity.

**Benefit:** Ultralytics manages the training loop, periodic validation, and weight saving, letting you focus on annotations and experiment settings. Training can use a GPU when one is available.

## 5. Track and Evaluate Performance

The notebook examines training and validation losses, along with several detection metrics:

* **Box loss** measures bounding-box localization error;
* **precision** is the proportion of produced detections that are correct;
* **recall** is the proportion of real objects that were detected;
* **mAP@0.5** summarizes average precision at an IoU threshold of 0.5;
* **mAP@0.5:0.95** aggregates results across several IoU thresholds and is more demanding.

After tuning, the notebook evaluates the model on the separate test set:

```python
metrics = model.val(data="/content/cars.yaml", split="test")

precision = metrics.results_dict["metrics/precision(B)"]
recall = metrics.results_dict["metrics/recall(B)"]
map50 = metrics.results_dict["metrics/mAP50(B)"]
map50_95 = metrics.results_dict["metrics/mAP50-95(B)"]
```

Do not select a model only because its training loss decreases. Compare training and validation metrics, inspect false positives and missed objects, and reserve the test set for final evaluation to avoid indirectly tuning the model against it.

## 6. Predict and Review Detections

After training, the model can process a test image. Ultralytics returns boxes, classes, and scores, and can draw the predictions:

```python
results = model.predict(
    "/content/dataset/images/test/tile.jpg",
    conf=0.25,
)

annotated_image = results[0].plot()
results[0].show()
```

The `conf` threshold adjusts the trade-off between the number of retained detections and minimum confidence. The notebook displays annotated images and exports information for further analysis.

## 7. Georeference and Map Detections

Bing tile names, such as `tile_157460_187752_z19.jpg`, encode tile indices `X`, `Y`, and the zoom level. The notebook converts each box center from tile pixels to latitude/longitude using those indices and the Web Mercator grid. It records class, confidence, tile, and coordinates in a table, then displays them on an interactive map with the tiles as a background.

This georeferencing assumes the tile names, dimensions, and zoom levels are correct. It positions the center of each detected object; to create precise GIS geometries, transform the box corners as well and account for the projection.

## Benefits and Considerations

* **Training on a relevant class:** the model is fine-tuned using car annotations rather than applying a generic model directly.
* **Accessible workflow:** Ultralytics handles dataset preparation, training, evaluation, and prediction through its API.
* **Separate evaluation:** train, validation, and test have distinct roles; the test set measures performance on images not used for fitting.
* **Non-oriented boxes:** boxes are aligned with the image axes. For elongated objects where orientation matters, see [YOLO-OBB](/en/yolo-obb/).
* **Representative data:** annotation quality and diversity largely determine performance; include different areas, image conditions, and object types.
* **Geographic validation:** nearby tiles can be highly correlated; spatial splitting helps prevent leakage between train and test.
* **Background imagery:** check access and reuse terms before redistributing the dataset or results together with Bing tiles.

The notebook also includes cells for saving and transferring the trained model. Protect credentials and access keys, avoid embedding them directly in the notebook, and verify the security of the receiving service.

See the [YOLO Colab notebook](https://colab.research.google.com/drive/15DFRZD29aqEAIpSWFuIo1Qk0FZu2871d?usp=sharing) to run the workflow and experiment with training parameters.
