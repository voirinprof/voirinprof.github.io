---
title: "Remote Sensing Object Detection with Faster R-CNN"
permalink: /en/faster-rcnn/
lang: en
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: ai
---

## Overview

This page describes an object-detection workflow for remote-sensing tiles using **Faster R-CNN** and PyTorch. It is based on the [Faster R-CNN Colab notebook](https://colab.research.google.com/drive/1_ScSKRoTpQtDO0W3gT6nY97w1AIKr_KW?usp=sharing), which trains a model on annotated Bing tiles, then displays and georeferences the detections.

Faster R-CNN is a two-stage detector. A Region Proposal Network (RPN) proposes regions that may contain objects; the model then classifies those regions and refines their bounding boxes. The notebook uses the ResNet-50 with FPN implementation from `torchvision`.

## 1. Prepare Images and Annotations

The notebook uses **256 × 256 pixel** RGB Bing tiles. Each image named `tile_X_Y_zZ.jpg` has a corresponding annotation text file with the same base name. Each line describes a class and a box in YOLO format:

```text
class_id x_center y_center width height
```

Coordinates are normalized between 0 and 1. Faster R-CNN instead expects pixel coordinates in `[x_min, y_min, x_max, y_max]` form. For a tile of width `W` and height `H`, convert them as follows:

```python
x_center, y_center = x_center_norm * W, y_center_norm * H
box_width, box_height = width_norm * W, height_norm * H

box = [
    x_center - box_width / 2,
    y_center - box_height / 2,
    x_center + box_width / 2,
    y_center + box_height / 2,
]
```

Define class names in the same order as the IDs used during annotation. In the `torchvision` detection format, label `0` is reserved for background, so object class IDs start at `1`.

## 2. Build a PyTorch Dataset

A custom `Dataset` loads each image, reads its corresponding boxes, and returns the image as a tensor together with a target containing boxes and labels. This example follows the notebook’s approach:

```python
from pathlib import Path

from PIL import Image
import torch
from torch.utils.data import Dataset
import torchvision.transforms.functional as TF


class AnnotatedTileDataset(Dataset):
    def __init__(self, image_paths, labels_dir):
        self.image_paths = image_paths
        self.labels_dir = Path(labels_dir)

    def __len__(self):
        return len(self.image_paths)

    def __getitem__(self, idx):
        image_path = Path(self.image_paths[idx])
        image = Image.open(image_path).convert("RGB")
        width, height = image.size
        label_path = self.labels_dir / f"{image_path.stem}.txt"
        boxes, labels = [], []

        if label_path.exists():
            for line in label_path.read_text().splitlines():
                if not line.strip():
                    continue
                class_id, xc, yc, box_width, box_height = line.split()
                xc, yc = float(xc) * width, float(yc) * height
                box_width, box_height = float(box_width) * width, float(box_height) * height
                boxes.append([
                    xc - box_width / 2, yc - box_height / 2,
                    xc + box_width / 2, yc + box_height / 2,
                ])
                labels.append(int(class_id) + 1)

        target = {
            "boxes": torch.as_tensor(boxes, dtype=torch.float32).reshape(-1, 4),
            "labels": torch.as_tensor(labels, dtype=torch.int64),
        }
        return TF.to_tensor(image), target
```

In the notebook, a custom `collate_fn` lets the `DataLoader` return lists of images and targets, since each tile may contain a different number of objects.

The notebook shuffles tiles with a fixed seed and reserves about 20% for validation. To measure how well the model generalizes to new areas, a geographic block split is often preferable: neighboring tiles can be very similar and may otherwise appear in both sets after a random split.

## 3. Adapt the Pretrained Model

The notebook starts from weights pretrained on COCO, then replaces the classification head to match the number of classes in the application. This is **transfer learning** (fine-tuning):

```python
from torchvision.models.detection import (
    FasterRCNN_ResNet50_FPN_Weights,
    fasterrcnn_resnet50_fpn,
)
from torchvision.models.detection.faster_rcnn import FastRCNNPredictor

model = fasterrcnn_resnet50_fpn(
    weights=FasterRCNN_ResNet50_FPN_Weights.DEFAULT
)
num_classes = len(class_names) + 1  # object classes plus background
in_features = model.roi_heads.box_predictor.cls_score.in_features
model.roi_heads.box_predictor = FastRCNNPredictor(in_features, num_classes)
model.to(device)
```

COCO weights provide a starting point, but may not recognize objects of interest in aerial imagery. Fine-tuning on suitable annotations is therefore essential. Also check that images and annotations are correctly paired and that the boxes are valid.

## 4. Train and Track Learning

During training, Faster R-CNN receives a list of image tensors and their targets. The model returns several loss components; their sum is used to update the weights. The notebook uses SGD and tracks average loss per epoch:

```python
optimizer = torch.optim.SGD(
    (p for p in model.parameters() if p.requires_grad),
    lr=learning_rate,
    momentum=0.9,
    weight_decay=0.0005,
)

model.train()
for images, targets in train_loader:
    images = [image.to(device) for image in images]
    targets = [
        {key: value.to(device) for key, value in target.items()}
        for target in targets
    ]

    losses = model(images, targets)
    loss = sum(losses.values())
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```

The notebook’s suggested settings are 30 epochs, a `0.005` learning rate, and batches of 6 tiles. Treat these as starting points and adjust them to the dataset size and available GPU memory. The notebook plots training loss but does not calculate detection metrics on the validation set. A decreasing loss is encouraging but does not by itself measure detection quality: evaluate on independent validation data using detection metrics such as mean Average Precision (mAP), and inspect localization and classification errors.

## 5. Detect and Filter Objects

For prediction, switch the model to evaluation mode and disable gradient calculation. Faster R-CNN returns boxes, classes, and confidence scores. The notebook keeps detections above a configurable threshold:

```python
model.eval()
with torch.no_grad():
    prediction = model([image_tensor.to(device)])[0]

keep = prediction["scores"] >= confidence_threshold
boxes = prediction["boxes"][keep].cpu()
labels = prediction["labels"][keep].cpu()
scores = prediction["scores"][keep].cpu()
```

A higher threshold usually reduces false positives but may also remove real objects. Choose it based on the relative cost of false positives and false negatives, and validate it on held-out data rather than only on training tiles.

## 6. Put Detections in Geographic Context

Output coordinates are initially in tile pixels. The notebook decodes the tile indices `X`, `Y`, and zoom level `z` from `tile_X_Y_zZ` to convert a box center to latitude and longitude. Bing tiles use the Web Mercator grid, so this conversion depends on the tiling convention and file names:

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

Detections can then be recorded in a table with their class, score, tile, and geographic coordinates. The notebook visualizes boxes on the images and overlays the tiles and detections on a Folium map.

## Benefits and Considerations

* **Transfer learning:** pretrained weights provide a starting point, while replacing the head lets the model learn project-specific classes.
* **Object-level detections:** each result includes a box, class, and score, making it suitable for visual review and conversion to geospatial data.
* **GPU support:** PyTorch uses CUDA when available, which is useful for training; batch size must fit GPU memory.
* **Annotations matter:** incomplete, imprecise, or inconsistent annotations limit model quality even when the training code runs correctly.
* **Axis-aligned boxes:** Faster R-CNN produces boxes aligned to the image axes, not oriented boxes. For elongated or rotated objects, account for this limitation in the annotation and model choice.
* **Spatial validation:** a random tile split may overestimate performance if neighboring, similar tiles appear in both training and validation sets.

See the [Colab notebook](https://colab.research.google.com/drive/1_ScSKRoTpQtDO0W3gT6nY97w1AIKr_KW?usp=sharing) to run the full workflow and modify parameters, classes, and annotations.
