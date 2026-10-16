---
title: "Détection d’objets orientés en télédétection avec YOLO-OBB"
permalink: /fr/yolo-obb/
lang: fr
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: ai
---

## Présentation

Cette page présente une démarche de détection d’objets orientés sur des tuiles d’images de télédétection avec **YOLO-OBB**. Elle s’appuie sur le [notebook Colab YOLO-OBB](https://colab.research.google.com/drive/1iHFP0_si2-fxvMCXlTnfWdxzSnTnGZEt?usp=sharing), qui applique un modèle Ultralytics préentraîné à des tuiles Bing, puis visualise, dénombre et géoréférence les objets détectés.

**OBB** signifie *oriented bounding box*, ou boîte englobante orientée. Contrairement à une boîte rectangulaire alignée avec les axes de l’image, une boîte orientée peut suivre la rotation de l’objet. C’est utile en imagerie aérienne ou satellitaire, où des objets comme des véhicules ou des bâtiments peuvent apparaître sous des angles variés.

## Différence avec Faster R-CNN

Le notebook YOLO-OBB utilise directement le modèle `yolo11m-obb.pt`, préentraîné sur des images aériennes de l’ensemble DOTA. Il effectue de l’**inférence** sans entraîner le modèle dans le notebook. Les détections sont donc limitées aux classes que ce modèle connaît.

Le notebook [Faster R-CNN](/fr/faster-rcnn/) suit une autre approche : il remplace la tête de classification d’un modèle préentraîné sur COCO et entraîne ce modèle sur les annotations fournies. YOLO-OBB est prêt à détecter ses classes préentraînées; pour détecter de nouvelles classes, il faudrait préparer des annotations adaptées et entraîner ou ajuster un modèle YOLO-OBB.

## 1. Charger les tuiles

Le notebook attend des tuiles JPEG de 256 × 256 pixels nommées selon le schéma Bing/OSM, par exemple `tile_157460_187752_z19.jpg`. Les nombres `X`, `Y` et `z` identifient la position de la tuile dans la grille Web Mercator et son niveau de zoom. Le notebook liste les images du répertoire fourni :

```python
import glob
import os

DOSSIER_TUILES = "/content/jpeg_tiles"
tuiles = sorted(glob.glob(os.path.join(DOSSIER_TUILES, "tile_*.jpg")))

print(f"{len(tuiles)} tuile(s) trouvée(s)")
```

Les noms de fichiers sont importants : ils permettent au notebook de retrouver l’emplacement géographique d’une détection sans fichier de géoréférencement séparé pour chaque image.

## 2. Charger le modèle et régler l’inférence

Ultralytics charge le modèle OBB et fournit les noms des classes disponibles. Le notebook propose trois paramètres principaux :

```python
from ultralytics import YOLO

modele = YOLO("yolo11m-obb.pt")

SEUIL_CONFIANCE = 0.25
SEUIL_IOU = 0.5
TAILLE_INFERENCE = 256

resultats = modele.predict(
    tuiles,
    conf=SEUIL_CONFIANCE,
    iou=SEUIL_IOU,
    imgsz=TAILLE_INFERENCE,
    verbose=False,
)
```

* `conf` élimine les prédictions dont le score de confiance est trop faible.
* `iou` règle le seuil d’intersection sur union employé lors de la suppression des détections redondantes (NMS).
* `imgsz` détermine la taille d’image utilisée pour l’inférence. Le notebook propose la taille native de 256 pixels ou un agrandissement, par exemple 640 pixels.

**Avantage :** un modèle préentraîné permet d’obtenir rapidement des prédictions sans préparer d’entraînement dans le notebook. Les paramètres d’inférence peuvent être ajustés et relancés sans réentraîner le modèle. Il faut toutefois vérifier que les classes du modèle et le type d’images correspondent aux objets recherchés.

## 3. Lire les boîtes orientées

Pour chaque résultat, Ultralytics fournit les boîtes dans le format centre, largeur, hauteur et angle (`xywhr`), avec l’identifiant de classe et le score de confiance. Le notebook rassemble ces informations dans des enregistrements :

```python
import os

detections = []

for chemin_tuile, resultat in zip(tuiles, resultats):
    if resultat.obb is None or len(resultat.obb) == 0:
        continue

    for boite in resultat.obb:
        cx, cy, largeur, hauteur, angle = boite.xywhr[0].tolist()
        classe_id = int(boite.cls[0])
        confiance = float(boite.conf[0])

        detections.append({
            "tuile": os.path.basename(chemin_tuile),
            "classe": modele.names[classe_id],
            "confiance": round(confiance, 3),
            "px_centre_x": round(cx, 1),
            "px_centre_y": round(cy, 1),
            "largeur_px": round(largeur, 1),
            "hauteur_px": round(hauteur, 1),
            "angle_rad": round(angle, 3),
        })
```

**Avantage :** la boîte orientée décrit mieux la forme et l’orientation d’objets obliques qu’une boîte horizontale ou verticale, et les scores permettent de filtrer les détections. L’angle fait partie des mesures retournées; il convient d’en vérifier l’unité et la convention avant toute conversion géométrique ou comparaison entre modèles.

## 4. Replacer les prédictions sur la carte

Le notebook décode le nom `tile_X_Y_zZ`, puis convertit le centre en pixels de la tuile vers la latitude et la longitude correspondantes. La transformation reprend la grille Web Mercator utilisée par les tuiles Bing :

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

Le centre géographique peut être ajouté à chaque détection et exporté dans un tableau CSV. Le notebook affiche également les tuiles comme images superposées dans Folium, puis ajoute des marqueurs colorés par classe.

Cette conversion suppose que les images sont bien des tuiles de 256 × 256 pixels et que le nom encode correctement leurs indices et leur zoom. Le notebook géolocalise le **centre** de chaque boîte; il ne convertit pas ici le contour complet de la boîte orientée en polygone géographique.

## 5. Examiner et résumer les résultats

Le notebook utilise les fonctions de visualisation d’Ultralytics pour afficher les boîtes sur les tuiles et composer une mosaïque de résultats :

```python
import matplotlib.pyplot as plt

image_annotee = resultats[0].plot()
plt.figure(figsize=(7, 7))
plt.imshow(image_annotee[:, :, ::-1])  # BGR vers RGB pour Matplotlib
plt.axis("off")
plt.show()
```

Les détections peuvent aussi être exportées dans un tableau puis résumées par classe :

```python
import matplotlib.pyplot as plt
import pandas as pd

df_detections = pd.DataFrame(detections)
df_detections.to_csv("detections.csv", index=False)

comptage = df_detections["classe"].value_counts()
comptage.plot(kind="bar", title="Détections par classe")
plt.tight_layout()
plt.show()
```

**Avantage :** la carte, les images annotées et le tableau donnent des vues complémentaires : contrôle visuel local, distribution géographique et statistiques agrégées.

## Avantages et limites

* **Boîtes orientées :** la rotation des objets est représentée, ce qui convient mieux à de nombreuses scènes aériennes.
* **Inférence immédiate :** le modèle préentraîné peut être appliqué aux tuiles sans étape d’entraînement dans le notebook.
* **Chaîne de résultats géospatiale :** les coordonnées du centre, la classe et le score sont réunis dans un CSV et peuvent être visualisés dans Folium.
* **Classes prédéfinies :** sans fine-tuning, le modèle ne peut pas apprendre de nouvelles catégories spécifiques au projet.
* **Évaluation nécessaire :** les seuils et les prédictions doivent être contrôlés sur des exemples représentatifs annotés; un aperçu visuel seul ne mesure pas la précision ou le rappel.
* **Pas de géométrie OBB complète sur la carte :** le notebook place un marqueur au centre de la boîte, plutôt que de projeter son polygone orienté complet.
* **Conditions d’utilisation des images :** vérifiez les conditions d’accès, d’attribution et de réutilisation des tuiles de fond utilisées pour l’analyse et la diffusion des résultats.

Consultez le [notebook Colab YOLO-OBB](https://colab.research.google.com/drive/1iHFP0_si2-fxvMCXlTnfWdxzSnTnGZEt?usp=sharing) pour exécuter les cellules et essayer d’autres seuils ou tailles d’inférence.
