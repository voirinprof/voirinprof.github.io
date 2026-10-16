---
title: "GeoPandas, pandas, Matplotlib, Rasterio et NumPy avancés"
permalink: /fr/advanced-vector-raster/
lang: fr
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: start
---

## Aller plus loin avec les données vecteur et raster

Cette page présente des techniques avancées pour traiter et visualiser des données géospatiales en Python. Les extraits sont tirés des scripts du dépôt [gmq580_advanced_vector_raster](https://github.com/voirinprof/gmq580_advanced_vector_raster), qui contient des exemples autonomes dans `src_vector/` et `src_raster/`. Le jeu d'exemples porte notamment sur les données d'occupants et d'arrondissements de Sherbrooke ainsi que sur des images Landsat.

## GeoPandas : jointures et analyses spatiales

**GeoPandas** étend les fonctionnalités des tableaux pandas aux géométries. En plus des jointures attributaires, `gpd.sjoin()` associe des entités selon leur relation spatiale. Le choix du prédicat (`within`, `intersects`, `contains`, etc.) définit la règle d'association; `how` détermine quelles entités de la couche de gauche sont conservées.

Dans l'exemple du dépôt, les occupants sont représentés par des points et les arrondissements par des polygones. Après avoir mis les deux couches dans le même système de coordonnées, une jointure `within` associe chaque point à son arrondissement :

```python
import geopandas as gpd

# Les deux couches doivent avoir le même CRS pour la jointure.
districts = gpd.read_file("arrondissements.gpkg").to_crs("EPSG:32198")
occupants = gpd.GeoDataFrame(
    occupants_table,
    geometry=gpd.points_from_xy(occupants_table["lng"], occupants_table["lat"]),
    crs="EPSG:4326",
).to_crs("EPSG:32198")

joined = gpd.sjoin(
    occupants,
    districts[["NOM", "geometry"]],
    how="left",
    predicate="within",
)
counts = joined.groupby("NOM").size().reset_index(name="nb_occupants")
districts = districts.merge(counts, on="NOM", how="left")
districts["nb_occupants"] = districts["nb_occupants"].fillna(0)
```

**Avantage :** les associations entre points et polygones sont faites selon la géométrie plutôt qu'à partir d'une clé commune. Un `left` join permet aussi de repérer les points qui ne correspondent à aucun polygone. Un point exactement sur une frontière peut se comporter différemment selon `within` ou `intersects`; le choix du prédicat doit donc correspondre à la question d'analyse. L'exemple complet se trouve dans [`src_vector/05_sjoin_predicates.py`](https://github.com/voirinprof/gmq580_advanced_vector_raster/blob/master/src_vector/05_sjoin_predicates.py).

## pandas : optimiser les tables attributaires

**pandas** fournit les tableaux et opérations attributaires utilisés aussi par GeoPandas. Pour les tables volumineuses, le choix du type de données a une incidence sur la mémoire. Les colonnes textuelles ayant peu de valeurs distinctes peuvent être converties en `category`; les types numériques peuvent être réduits si leur précision le permet. Convertir les dates en `datetime` facilite leur filtrage et leur analyse.

Le dépôt applique ces optimisations aux colonnes de la table des occupants :

```python
import pandas as pd

before_mb = occupants.memory_usage(deep=True).sum() / 1e6

for column in ["type_etablissement", "secteur_nom", "classe_nom"]:
    if column in occupants.columns:
        occupants[column] = occupants[column].astype("category")

occupants["lng"] = occupants["lng"].astype("float32")
occupants["lat"] = occupants["lat"].astype("float32")
occupants["date_creation"] = pd.to_datetime(occupants["date_creation"])

after_mb = occupants.memory_usage(deep=True).sum() / 1e6
print(f"Réduction mémoire : {(1 - after_mb / before_mb) * 100:.1f}%")
```

**Avantage :** des types adaptés peuvent réduire l'empreinte mémoire et rendre les traitements plus efficaces. Vérifiez toutefois que la précision numérique reste suffisante et que les colonnes catégorielles ont réellement peu de valeurs uniques. Voir [`src_vector/02_memory_optimization.py`](https://github.com/voirinprof/gmq580_advanced_vector_raster/blob/master/src_vector/02_memory_optimization.py).

## Matplotlib : comparer plusieurs cartes

**Matplotlib** offre un contrôle fin sur les figures et les axes. Combiné à GeoPandas, il permet de présenter plusieurs indicateurs cartographiques côte à côte, avec leurs propres palettes, légendes et titres.

L'exemple du dépôt compare le nombre d'occupants, leur densité et le nombre de grandes chaînes par arrondissement :

```python
import matplotlib.pyplot as plt

fig, axes = plt.subplots(1, 3, figsize=(16, 6), layout="constrained")
variables = ["nb_occupants", "densite_occupants", "nb_grandes_chaines"]
titles = ["Nombre d'occupants", "Densité (occ./km²)", "Grandes chaînes"]
cmaps = ["YlOrRd", "Blues", "Purples"]

for ax, column, title, cmap in zip(axes, variables, titles, cmaps):
    districts.plot(
        column=column,
        ax=ax,
        cmap=cmap,
        legend=True,
        legend_kwds={"shrink": 0.6, "label": title},
        edgecolor="white",
        linewidth=0.5,
    )
    ax.set_title(title)
    ax.set_axis_off()

fig.suptitle("Analyse par arrondissement")
fig.savefig("comparaison.png", dpi=150, bbox_inches="tight")
```

**Avantage :** des cartes comparables dans une même figure facilitent l'interprétation de plusieurs variables et évitent de comparer des images produites avec des réglages différents. `layout="constrained"` aide à éviter le chevauchement entre les cartes, les titres et les légendes. Voir [`src_vector/10_matplotlib_subplots.py`](https://github.com/voirinprof/gmq580_advanced_vector_raster/blob/master/src_vector/10_matplotlib_subplots.py).

## Rasterio : lire et traiter un raster par fenêtres

**Rasterio** lit et écrit des rasters tout en donnant accès à leur géoréférencement, leurs bandes et leurs métadonnées. Pour les images trop grandes pour tenir en mémoire, une `Window` permet de lire une portion de l'image à la fois.

Le script du dépôt parcourt l'image Landsat par tuiles, lit seulement les bandes rouge et proche infrarouge, puis calcule le NDVI sur chaque tuile :

```python
import numpy as np
import rasterio
from rasterio.windows import Window

tile_size = 256
ndvi_sum = 0.0
pixel_count = 0

with rasterio.open("landsat.tif") as src:
    for row_off in range(0, src.height, tile_size):
        for col_off in range(0, src.width, tile_size):
            width = min(tile_size, src.width - col_off)
            height = min(tile_size, src.height - row_off)
            window = Window(col_off, row_off, width, height)

            red = src.read(1, window=window).astype("float32")
            nir = src.read(4, window=window).astype("float32")
            ndvi = (nir - red) / (nir + red + 1e-10)

            ndvi_sum += ndvi.sum()
            pixel_count += ndvi.size

mean_ndvi = ndvi_sum / pixel_count
```

**Avantage :** le traitement peut couvrir un raster complet sans charger toutes ses bandes simultanément en mémoire. La taille des fenêtres doit rester adaptée à la mémoire disponible; si l'image utilise une valeur `nodata`, celle-ci doit aussi être masquée ou exclue des statistiques. Voir [`src_raster/03_windowed_reading.py`](https://github.com/voirinprof/gmq580_advanced_vector_raster/blob/master/src_raster/03_windowed_reading.py).

## NumPy : calculs raster vectorisés

**NumPy** représente les bandes raster sous forme de tableaux et applique les opérations élément par élément, sans boucle Python sur chaque pixel. C'est utile pour calculer des indices spectraux tels que le NDVI, l'indice de végétation normalisé :

```python
import numpy as np

# red et nir sont des tableaux raster de mêmes dimensions.
red = np.clip(red.astype("float32"), 0, None)
nir = np.clip(nir.astype("float32"), 0, None)

ndvi = (nir - red) / (nir + red + 1e-10)
pct_vegetation = (ndvi > 0.3).mean() * 100
```

**Avantage :** une expression concise calcule l'indice pour tous les pixels à la fois, ce qui est généralement beaucoup plus rapide qu'une boucle explicite. Le `clip` reprend le traitement de valeurs négatives présenté dans le dépôt et le petit terme au dénominateur évite une division par zéro. En production, il faut également tenir compte des pixels `nodata` : le dépôt présente les tableaux masqués dans [`src_raster/08_nodata_masked_arrays.py`](https://github.com/voirinprof/gmq580_advanced_vector_raster/blob/master/src_raster/08_nodata_masked_arrays.py).

## Explorer les exemples

Le dépôt [gmq580_advanced_vector_raster](https://github.com/voirinprof/gmq580_advanced_vector_raster) regroupe les scripts par thème dans `src_vector/` et `src_raster/`. Chaque script est autonome et les données d'exemple doivent être préparées selon les indications du README. Les exemples avancés couvrent aussi la lecture partielle avec Pyogrio, la gestion des CRS, les géométries invalides, les statistiques zonales, la reprojection et l'écriture de Cloud Optimized GeoTIFF.
