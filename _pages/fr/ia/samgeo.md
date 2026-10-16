---
title: "Segmenter des images satellitaires avec SamGeo"
permalink: /fr/samgeo/
lang: fr
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: ai
---

## Présentation

Cette page présente une démarche de segmentation d’image satellitaire avec **SamGeo**, une bibliothèque qui facilite l’utilisation de **Segment Anything Model (SAM)** sur des données géospatiales. Elle est basée sur le [notebook Colab SamGeo](https://colab.research.google.com/drive/1JAZv-snF_DgT0sdBgbKM7SCEsUvHqmS_?usp=sharing).

Le notebook utilise un modèle SAM préentraîné en mode de génération automatique : il recherche et délimite des régions dans l’image sans entraînement supplémentaire. La chaîne de traitement est :

**imagerie satellite → GeoTIFF → segmentation SAM → masque géoréférencé → polygones SIG**

> **Segmentation et classification sont deux tâches différentes.** SAM peut délimiter des objets ou des régions, mais les segments produits ne sont pas automatiquement étiquetés comme « bâtiment », « arbre », « route » ou toute autre classe.

## 1. Définir la zone et les paramètres

Le notebook définit une petite emprise géographique, un niveau de zoom, le modèle SAM et les options de segmentation :

```python
bbox = [-95.3704, 29.6762, -95.3680, 29.6775]
zoom = 19
model_type = "vit_b"
min_size = 100
unique = True
```

La boîte `bbox` est donnée dans l’ordre `[longitude_min, latitude_min, longitude_max, latitude_max]`. `zoom` détermine le niveau de détail de l’image récupérée; `vit_b` choisit une variante relativement légère du modèle; `min_size` élimine les petits segments, et `unique=True` attribue une valeur distincte à chaque objet segmenté.

Commencer par une zone limitée aide à maîtriser le temps de traitement et la mémoire. Les résultats dépendent de l’image source, de sa résolution, du modèle et de ces paramètres.

## 2. Créer et inspecter un GeoTIFF

Le notebook utilise `tms_to_geotiff` de SamGeo pour récupérer l’imagerie de la zone sous forme de GeoTIFF :

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

Contrairement à une image JPEG ou PNG ordinaire, un GeoTIFF conserve les informations spatiales nécessaires pour replacer les pixels sur la Terre. Avant la segmentation, le notebook utilise Rasterio pour contrôler son système de coordonnées, son emprise, sa résolution et ses bandes :

```python
import rasterio

with rasterio.open(image) as src:
    print("CRS :", src.crs)
    print("Dimensions :", src.width, "x", src.height)
    print("Résolution :", src.res)
    print("Emprise :", src.bounds)
    rgb = src.read([1, 2, 3]).transpose(1, 2, 0)
```

**Avantage :** inspecter les métadonnées géospatiales permet de vérifier que l’entrée est bien un raster exploitable et que le résultat pourra être aligné avec d’autres couches. Vérifiez aussi les conditions d’utilisation et d’attribution de la source d’imagerie choisie.

## 3. Initialiser SAM et segmenter l’image

SamGeo masque une partie de la complexité d’utilisation du modèle et expose une interface adaptée aux rasters géospatiaux. Le notebook choisit le GPU lorsqu’il est disponible, puis lance la génération automatique des masques :

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

Le résultat `masks.tif` est un raster de segmentation géoréférencé. Avec `unique=True`, les pixels de chaque segment reçoivent une valeur qui permet de distinguer les régions. `min_size` contrôle la taille minimale retenue; il peut être ajusté pour écarter des fragments trop petits ou, au contraire, conserver des détails plus fins.

**Avantage :** la segmentation automatique permet d’explorer rapidement une image sans créer au préalable un jeu d’annotations ni entraîner un modèle spécialisé. Le choix du périphérique permet de profiter d’un GPU lorsque l’environnement en propose un.

## 4. Visualiser et contrôler les masques

Le notebook affiche les segments superposés à l’image, puis lit le raster de masque avec Rasterio :

```python
sam.show_anns(figsize=(12, 10), axis="off", alpha=0.45)

with rasterio.open(mask) as src:
    objects = src.read(1)
    print("CRS du masque :", src.crs)
    print("Emprise :", src.bounds)
```

Les couleurs de visualisation servent à distinguer les segments; elles ne représentent pas des classes thématiques. Contrôlez visuellement plusieurs secteurs de l’image : selon la résolution, les contrastes et la complexité du paysage, SAM peut regrouper plusieurs objets, diviser un objet en plusieurs régions ou produire des limites qui ne correspondent pas à l’objet d’intérêt.

## 5. Convertir le raster en polygones

Le masque peut être vectorisé en GeoPackage, puis ouvert avec GeoPandas :

```python
import geopandas as gpd

vector = "segments.gpkg"
sam.tiff_to_vector(mask, vector)

segments = gpd.read_file(vector)
print("Nombre de polygones :", len(segments))
print("CRS :", segments.crs)
```

Les polygones peuvent ensuite être inspectés, filtrés, croisés avec d’autres couches ou utilisés dans QGIS. Le notebook les visualise avec GeoPandas :

```python
ax = segments.plot(figsize=(10, 8), edgecolor="black")
ax.set_title("Objets segmentés")
ax.set_axis_off()
```

**Avantage :** convertir le masque en données vectorielles facilite son utilisation dans les outils SIG et les traitements vecteur. La conversion peut toutefois générer un grand nombre de polygones; vérifiez leur géométrie, leur taille et leur utilité avant de poursuivre l’analyse.

## Points forts et limites

* **Pas d’entraînement dans le notebook :** SAM fournit une segmentation exploratoire directement à partir d’un modèle préentraîné.
* **Données géoréférencées :** le GeoTIFF source, le masque et les vecteurs conservent un contexte spatial exploitable en SIG.
* **Paramètres expérimentables :** l’emprise, le zoom, la variante de modèle et la taille minimale influencent la granularité des résultats.
* **Pas d’étiquettes sémantiques automatiques :** la segmentation délimite des régions, mais ne détermine pas à quelle classe thématique elles appartiennent.
* **Résultats à valider :** les contours doivent être examinés et éventuellement corrigés selon l’objectif, l’image et la résolution.
* **Ressources de calcul :** les grandes images ou les variantes plus lourdes de SAM demandent davantage de mémoire et de temps; commencer par une petite emprise est recommandé.

Le notebook permet de télécharger le masque GeoTIFF et les polygones GeoPackage pour les ouvrir dans QGIS. Consultez le [notebook Colab SamGeo](https://colab.research.google.com/drive/1JAZv-snF_DgT0sdBgbKM7SCEsUvHqmS_?usp=sharing) pour exécuter les étapes et expérimenter avec une autre zone ou d’autres paramètres.
