---
title: "Détection d’objets en télédétection avec Faster R-CNN"
permalink: /fr/faster-rcnn/
lang: fr
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: ai
---

## Présentation

Cette page explique une démarche de détection d’objets sur des tuiles d’images de télédétection avec **Faster R-CNN** et PyTorch. Elle est basée sur le [notebook Colab Faster R-CNN](https://colab.research.google.com/drive/1_ScSKRoTpQtDO0W3gT6nY97w1AIKr_KW?usp=sharing), qui entraîne le modèle sur des tuiles Bing annotées, puis affiche et géoréférence les détections.

Faster R-CNN est un détecteur à deux étapes. Un réseau de propositions de régions (*Region Proposal Network*, RPN) propose des zones susceptibles de contenir un objet; le modèle classe ensuite ces régions et affine leurs boîtes englobantes. Le notebook utilise l’implémentation Faster R-CNN avec backbone ResNet-50 et FPN dans `torchvision`.

## 1. Préparer les images et les annotations

Le notebook utilise des images RGB Bing découpées en tuiles de **256 × 256 pixels**. Chaque image `tile_X_Y_zZ.jpg` a un fichier texte d’annotations associé, de même nom. Une ligne décrit un objet avec la classe et une boîte au format YOLO :

```text
classe x_centre y_centre largeur hauteur
```

Les coordonnées sont normalisées entre 0 et 1. Faster R-CNN attend plutôt des coordonnées en pixels sous la forme `[x_min, y_min, x_max, y_max]`. Pour une tuile de largeur `W` et de hauteur `H`, la conversion est :

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

Il faut aussi définir les noms des classes dans le même ordre que les identifiants utilisés pendant l’annotation. Dans le format de détection de `torchvision`, l’étiquette `0` est réservée à l’arrière-plan : les identifiants des classes d’objets commencent donc à `1`.

## 2. Construire un Dataset PyTorch

Un `Dataset` personnalisé charge l’image, lit les boîtes correspondantes et renvoie l’image sous forme de tenseur ainsi qu’une cible contenant les boîtes et leurs classes. L’extrait reprend le principe du notebook :

```python
from pathlib import Path

from PIL import Image
import torch
from torch.utils.data import Dataset
import torchvision.transforms.functional as TF


class JeuTuilesAnnotees(Dataset):
    def __init__(self, chemins_images, dossier_labels):
        self.chemins_images = chemins_images
        self.dossier_labels = dossier_labels

    def __len__(self):
        return len(self.chemins_images)

    def __getitem__(self, idx):
        image_path = Path(self.chemins_images[idx])
        image = Image.open(image_path).convert("RGB")
        largeur, hauteur = image.size
        label_path = Path(self.dossier_labels) / f"{image_path.stem}.txt"
        boites, classes = [], []

        if label_path.exists():
            for line in label_path.read_text().splitlines():
                if not line.strip():
                    continue
                class_id, xc, yc, width, height = line.split()
                xc, yc = float(xc) * largeur, float(yc) * hauteur
                width, height = float(width) * largeur, float(height) * hauteur
                boites.append([
                    xc - width / 2, yc - height / 2,
                    xc + width / 2, yc + height / 2,
                ])
                classes.append(int(class_id) + 1)  # 0 correspond à l'arrière-plan

        cible = {
            "boxes": torch.as_tensor(boites, dtype=torch.float32).reshape(-1, 4),
            "labels": torch.as_tensor(classes, dtype=torch.int64),
        }
        return TF.to_tensor(image), cible
```

Dans le notebook, une fonction de regroupement personnalisée (`collate_fn`) permet au `DataLoader` de fournir des listes d’images et de cibles : chaque tuile peut contenir un nombre différent d’objets.

Le notebook mélange les tuiles avec une graine fixe, puis réserve environ 20 % à la validation. Pour mesurer la capacité du modèle à généraliser à de nouvelles zones, un partage par blocs géographiques est souvent préférable : deux tuiles voisines peuvent être très similaires et se retrouver dans les deux ensembles lors d’un partage aléatoire.

## 3. Adapter le modèle préentraîné

Le notebook part de poids préentraînés sur COCO, puis remplace la tête de classification pour le nombre de classes de l’application. C’est un exemple d’**apprentissage par transfert** (*fine-tuning*) :

```python
from torchvision.models.detection import (
    FasterRCNN_ResNet50_FPN_Weights,
    fasterrcnn_resnet50_fpn,
)
from torchvision.models.detection.faster_rcnn import FastRCNNPredictor

model = fasterrcnn_resnet50_fpn(
    weights=FasterRCNN_ResNet50_FPN_Weights.DEFAULT
)
num_classes = len(class_names) + 1  # les classes d'objets et l'arrière-plan
in_features = model.roi_heads.box_predictor.cls_score.in_features
model.roi_heads.box_predictor = FastRCNNPredictor(in_features, num_classes)
model.to(device)
```

Les poids COCO fournissent un point de départ, mais ne savent pas nécessairement reconnaître les objets d’intérêt dans les images aériennes. Le réentraînement sur des annotations adaptées au domaine est donc essentiel. Vérifiez également que les images et les annotations sont correctement appariées et que les boîtes sont valides.

## 4. Entraîner et suivre l’apprentissage

Pendant l’entraînement, Faster R-CNN reçoit une liste de tenseurs d’images et les cibles correspondantes. Le modèle renvoie plusieurs composantes de perte; leur somme sert à mettre à jour les poids. Le notebook utilise SGD et suit la perte moyenne par époque :

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

Les paramètres proposés dans le notebook sont 30 époques, un taux d’apprentissage de `0.005` et des lots de 6 tuiles. Ce sont des points de départ à ajuster selon la taille du jeu de données et la mémoire GPU disponible. Le notebook trace la perte d’entraînement, mais ne calcule pas de métrique de détection sur l’ensemble de validation. Une perte qui diminue est encourageante, mais ne suffit pas à mesurer la qualité des détections : évaluez aussi le modèle sur des données de validation indépendantes avec des mesures de détection, par exemple la précision moyenne (mAP), et inspectez les erreurs de localisation et de classification.

## 5. Détecter et filtrer les objets

En prédiction, on passe le modèle en mode évaluation et on désactive le calcul des gradients. Faster R-CNN retourne des boîtes, des classes et des scores de confiance. Le notebook conserve les détections dont le score dépasse un seuil réglable :

```python
model.eval()
with torch.no_grad():
    prediction = model([image_tensor.to(device)])[0]

keep = prediction["scores"] >= confidence_threshold
boxes = prediction["boxes"][keep].cpu()
labels = prediction["labels"][keep].cpu()
scores = prediction["scores"][keep].cpu()
```

Un seuil plus élevé réduit généralement les faux positifs, mais peut aussi éliminer des objets réels. Il faut le choisir en fonction du coût relatif des faux positifs et des faux négatifs, et le vérifier sur des données de validation plutôt que seulement sur les tuiles d’entraînement.

## 6. Replacer les détections dans leur contexte géographique

Les coordonnées de sortie sont d’abord exprimées en pixels dans la tuile. Le notebook décode les indices `X`, `Y` et le niveau de zoom `z` du nom `tile_X_Y_zZ` pour convertir le centre d’une boîte en latitude et longitude. Les tuiles Bing utilisent la grille Web Mercator; cette conversion dépend donc de la convention de tuilage et des noms de fichiers.

```python
import math

def pixel_vers_latlon(px, py, tile_x, tile_y, zoom, taille_tuile=256):
    x = tile_x + px / taille_tuile
    y = tile_y + py / taille_tuile
    n = 2**zoom
    lon = x / n * 360.0 - 180.0
    lat = math.degrees(math.atan(math.sinh(math.pi * (1 - 2 * y / n))))
    return lat, lon
```

Les détections peuvent ensuite être enregistrées dans un tableau avec la classe, le score, la tuile et les coordonnées géographiques. Le notebook visualise les boîtes sur les images et superpose les tuiles ainsi que les détections sur une carte Folium.

## Avantages et points d’attention

* **Apprentissage par transfert :** les poids préentraînés évitent de partir de paramètres aléatoires, tandis que le remplacement de la tête permet d’apprendre les classes propres au projet.
* **Détection par objet :** le modèle retourne une boîte, une classe et un score pour chaque détection, ce qui se prête à un examen visuel et à une conversion en données géographiques.
* **Calcul GPU :** PyTorch utilise CUDA si un GPU est disponible, ce qui est utile pour l’entraînement; la taille des lots doit alors être adaptée à la mémoire vidéo.
* **Annotations déterminantes :** des annotations incomplètes, imprécises ou incohérentes limitent la qualité du modèle, même si le code d’entraînement fonctionne.
* **Boîtes axis-aligned :** Faster R-CNN produit des rectangles alignés sur les axes de l’image, et non des boîtes orientées. Pour des objets allongés ou tournés, l’annotation et le modèle doivent tenir compte de cette limite.
* **Validation spatiale :** un partage aléatoire des tuiles peut surestimer les performances si des tuiles voisines très semblables apparaissent à la fois dans l’entraînement et la validation.

Consultez le [notebook Colab](https://colab.research.google.com/drive/1_ScSKRoTpQtDO0W3gT6nY97w1AIKr_KW?usp=sharing) pour exécuter la démarche complète et modifier les paramètres, les classes et les données d’annotation.
