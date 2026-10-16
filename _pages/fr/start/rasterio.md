---
title: "Fiche pratique Rasterio"
permalink: /fr/rasterio/
lang: fr
date: 2026-10-16T08:48:13-04:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: start
---

Cette fiche rassemble les opérations essentielles de **Rasterio** pour lire, inspecter, traiter et écrire des rasters géoréférencés. Les exemples supposent que Rasterio et NumPy sont installés (`pip install rasterio numpy`) et que le fichier d’entrée existe.

```python
import numpy as np
import rasterio
```

## Ouvrir et inspecter un raster

Utilisez un gestionnaire de contexte `with` pour que le fichier se ferme correctement après utilisation.

```python
with rasterio.open("image.tif") as src:
    print(src.crs)          # système de coordonnées
    print(src.bounds)       # emprise spatiale
    print(src.width, src.height)
    print(src.count)        # nombre de bandes
    print(src.dtypes)       # type de données par bande
    print(src.nodata)       # valeur nodata, si définie
    print(src.transform)    # transformation pixel vers coordonnées
    print(src.res)          # résolution en unités du CRS
    print(src.profile)      # paramètres pour créer un raster similaire
```

`rasterio.open()` ouvre un fichier en lecture par défaut. Les identifiants de bandes utilisés par Rasterio commencent à **1**.

## Lire les bandes et gérer nodata

```python
with rasterio.open("image.tif") as src:
    bande_1 = src.read(1)                  # tableau (lignes, colonnes)
    rgb = src.read([1, 2, 3])              # tableau (bandes, lignes, colonnes)
    bande_masquee = src.read(1, masked=True)

    hauteur, largeur = src.shape
    nombre_bandes = src.count
```

Avec `masked=True`, les pixels nodata sont masqués lorsque leur valeur est définie dans le fichier. Pour les statistiques, utilisez les fonctions NumPy adaptées aux tableaux masqués, comme `np.ma.mean(bande_masquee)`. Vérifiez aussi `src.nodata` et le masque de validité si les données source n’indiquent pas correctement les pixels manquants.

## Convertir entre lignes, colonnes et coordonnées

```python
with rasterio.open("image.tif") as src:
    row, col = src.index(x, y)     # coordonnées vers ligne et colonne
    x, y = src.xy(row, col)        # centre du pixel vers coordonnées
```

`x` et `y` doivent être exprimés dans le CRS du raster. `src.index()` renvoie la cellule correspondante; `src.xy()` renvoie par défaut le centre de la cellule.

## Lire une fenêtre d’un grand raster

Les fenêtres permettent de ne charger qu’une partie du raster à la fois.

```python
from rasterio.windows import Window

with rasterio.open("image.tif") as src:
    window = Window(col_off=100, row_off=200, width=256, height=256)
    extrait = src.read(1, window=window, masked=True)
    emprise_extrait = rasterio.windows.bounds(window, src.transform)
    transform_extrait = src.window_transform(window)
```

Pour parcourir tout un raster par blocs, `src.block_windows(1)` fournit des fenêtres adaptées à son organisation de stockage :

```python
with rasterio.open("image.tif") as src:
    for _, window in src.block_windows(1):
        bloc = src.read(1, window=window, masked=True)
        # Traiter le bloc avant de passer au suivant.
```

## Découper un raster selon une géométrie

`rasterio.mask.mask()` découpe le raster selon une géométrie exprimée dans le CRS du raster. `crop=True` ajuste l’emprise au découpage.

```python
import geopandas as gpd
from rasterio.mask import mask

zones = gpd.read_file("zone_etude.gpkg")

with rasterio.open("image.tif") as src:
    zones = zones.to_crs(src.crs)
    geometries = zones.geometry
    donnees, transform_decoupe = mask(
        src,
        [geom.__geo_interface__ for geom in geometries if geom is not None],
        crop=True,
        filled=False,
    )

    profil = src.profile.copy()
    profil.update(
        height=donnees.shape[1],
        width=donnees.shape[2],
        transform=transform_decoupe,
    )
```

Avec `filled=False`, le résultat est un tableau masqué, ce qui permet de distinguer les pixels hors géométrie sans remplacer les valeurs nodata. Vérifiez que la couche contient des géométries et qu’elles recouvrent le raster.

## Reprojeter un raster

La reprojection change le CRS et recalcule la grille du raster cible. Choisissez une méthode de rééchantillonnage adaptée : `nearest` pour les catégories, `bilinear` ou `cubic` pour des données continues.

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

    with rasterio.open("reprojete.tif", "w", **profile) as dst:
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

Le CRS source doit être connu. Pour des données continues, remplacez `Resampling.nearest` par une méthode appropriée et tenez compte de la propagation des valeurs nodata.

## Écrire un nouveau raster

Copiez le profil de la source pour conserver ses paramètres géospatiaux, puis mettez à jour les dimensions et le type de données si nécessaire.

```python
with rasterio.open("image.tif") as src:
    profil = src.profile.copy()
    profil.update(dtype="float32", nodata=-9999)

    donnees = src.read(1, masked=True).astype("float32")
    resultat = donnees.filled(-9999)

    with rasterio.open("resultat.tif", "w", **profil) as dst:
        dst.write(resultat, 1)
```

Le profil doit correspondre au nombre de bandes, aux dimensions et au type des données écrites. Pour écrire un tableau multibande, son ordre d’axes est `(bandes, lignes, colonnes)`.

## Calculer un indice sur deux bandes

Rasterio lit les pixels et NumPy applique le calcul aux tableaux. Cet exemple calcule le NDVI avec une bande proche infrarouge et une bande rouge :

```python
with rasterio.open("image.tif") as src:
    rouge = src.read(3, masked=True).astype("float32")
    proche_infrarouge = src.read(4, masked=True).astype("float32")

    denominateur = proche_infrarouge + rouge
    ndvi = np.ma.masked_where(
        np.ma.getmaskarray(rouge)
        | np.ma.getmaskarray(proche_infrarouge)
        | (denominateur == 0),
        (proche_infrarouge - rouge) / np.ma.masked_equal(denominateur, 0),
    )
```

Les numéros de bandes dépendent du produit satellite. Vérifiez les métadonnées et les conventions de la source avant de choisir les bandes.

## À retenir

- Ouvrez les rasters avec `with rasterio.open(...)` et fermez-les avant de réutiliser ou déplacer les fichiers.
- Vérifiez le CRS, la résolution, les dimensions, le nombre de bandes et la valeur nodata avant le traitement.
- Lisez les grands fichiers par fenêtres pour limiter l’utilisation mémoire.
- Conservez ou mettez à jour correctement le profil, la transformation et le CRS lors de l’écriture.
- Choisissez le rééchantillonnage selon la nature des données et gardez les couches vectorielles dans le CRS du raster pour les découper.

## Documentation officielle

- [Documentation Rasterio](https://rasterio.readthedocs.io/en/stable/)
- [Guide de démarrage Rasterio](https://rasterio.readthedocs.io/en/stable/quickstart.html)
