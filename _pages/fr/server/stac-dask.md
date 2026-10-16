---
title: "STAC, odc-stac, Dask et xarray"
permalink: /fr/stac-dask/
lang: fr
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: gisserver
---

## Analyser des images satellites à grande échelle

Pour traiter une série d'images satellites, il faut d'abord trouver les données, les charger et les aligner, puis effectuer les calculs sans nécessairement tout conserver en mémoire. **STAC**, **odc-stac**, **Dask** et **xarray** répondent chacun à une partie de ce problème et peuvent être combinés dans un même pipeline.

Les exemples de cette page sont inspirés du dépôt [stac_dask_samples](https://github.com/voirinprof/stac_dask_samples). Ils utilisent des scènes Sentinel-2 de la région de Sherbrooke et montrent le parcours allant de la recherche au calcul d'un composite NDVI.

## STAC : rechercher et décrire les données

**STAC** (*SpatioTemporal Asset Catalog*) est un standard qui décrit et rend découvrables des données géospatiales par leur étendue spatiale, leur période et leurs propriétés. Un catalogue expose des **Collections** (ensembles de données) et des **Items** (observations ou scènes individuelles). Un Item contient des métadonnées ainsi que des **assets**, c'est-à-dire des liens vers les fichiers associés, par exemple les bandes d'une image.

Avec `pystac-client`, on peut interroger un catalogue STAC sans parcourir manuellement ses fichiers :

```python
from pystac_client import Client

catalog = Client.open("https://earth-search.aws.element84.com/v1")
items = catalog.search(
    collections=["sentinel-2-l2a"],
    bbox=[-72.15, 45.30, -71.65, 45.65],
    datetime="2024-06-01/2024-08-31",
    query={"eo:cloud_cover": {"lt": 20}},
).item_collection()

print(f"{len(items)} scènes trouvées")
```

**Avantage :** on recherche les scènes qui couvrent une zone et une période données avant de transférer les rasters. Les métadonnées permettent aussi de filtrer ou de classer les scènes, par exemple selon leur couverture nuageuse. Le dépôt présente la recherche et l'inspection d'Items dans `01_stac/`.

## odc-stac : charger les Items comme données alignées

**odc-stac** relie les Items STAC au chargement de données raster. Il sélectionne les assets demandés et peut reprojeter les données, les aligner sur une grille commune et les découper à la zone d'étude. Il retourne un Dataset xarray; avec les paramètres de *chunks*, les données sont chargées paresseusement à l'aide de Dask.

```python
import odc.stac

ds = odc.stac.load(
    items,
    bands=["red", "nir"],
    crs="EPSG:32198",
    resolution=10,
    bbox=[-72.15, 45.30, -71.65, 45.65],
    chunks={"x": 512, "y": 512, "time": 1},
    groupby="solar_day",
)
```

**Avantage :** on passe d'une collection de scènes et de fichiers à des données multidimensionnelles alignées, sans avoir à gérer soi-même le téléchargement et le reprojetage de chaque bande. La création du Dataset ne signifie pas nécessairement que tous les pixels ont été téléchargés : les calculs sont souvent différés jusqu'à `.compute()`.

## Dask : calculer par blocs, au bon moment

**Dask** fournit des tableaux découpés en blocs (*chunks*) et construit un graphe de tâches à exécuter. Au lieu de lancer chaque opération immédiatement, il peut combiner plusieurs opérations, puis les exécuter avec un ordonnanceur local ou distribué. `.compute()` déclenche le calcul et retourne un résultat concret.

```python
import dask.array as da

red = da.ones((10_000, 10_000), chunks=(512, 512))
nir = red * 1.5
ndvi = (nir - red) / (nir + red)

# Le calcul est exécuté ici, sur les blocs nécessaires.
mean_ndvi = ndvi.mean().compute()
```

**Avantage :** les données peuvent être traitées bloc par bloc au lieu de devoir charger un grand tableau entier en mémoire. Pour tirer parti du calcul différé, regroupez les résultats à calculer dans un même appel à `dask.compute(...)` lorsque c'est possible. La taille des blocs est importante : des blocs trop petits ajoutent de la surcharge, tandis que des blocs trop gros augmentent la pression mémoire.

## xarray + Dask : exploiter les dimensions et coordonnées

**xarray** représente des données multidimensionnelles avec des dimensions nommées, des coordonnées et des attributs. Un raster par date peut par exemple avoir les dimensions `time`, `y` et `x`. Lorsque les données sous-jacentes sont des tableaux Dask, les opérations xarray sont elles aussi différées et s'exécutent par blocs.

```python
# ds est le Dataset chargé avec odc-stac.
red = ds["red"].astype("float32") / 10_000
nir = ds["nir"].astype("float32") / 10_000
ndvi = (nir - red) / (nir + red + 1e-10)

# Composite du pixel le plus vert pour la période.
greenest_pixel = ndvi.max(dim="time")

# L'opération devient concrète au moment du compute.
result = greenest_pixel.compute()
```

**Avantage :** on exprime les opérations par leur sens et leurs dimensions, par exemple une moyenne temporelle avec `.mean(dim="time")`, plutôt qu'en manipulant des indices de tableaux anonymes. La combinaison xarray-Dask permet d'écrire des analyses temporelles tout en conservant le calcul par blocs.

## Assembler les technologies

Le flux général est le suivant :

1. **STAC** découvre les Items correspondant à la zone, à la période et aux filtres.
2. **odc-stac** charge les bandes et les organise dans un Dataset xarray aligné.
3. **xarray** décrit et compose les transformations, par exemple le calcul du NDVI.
4. **Dask** construit et exécute les calculs sur les blocs, au moment choisi.
5. Les résultats calculés peuvent être visualisés ou sauvegardés, par exemple en Zarr.

Le dépôt [stac_dask_samples](https://github.com/voirinprof/stac_dask_samples) contient les scripts détaillés : recherche et inspection STAC, chargement avec odc-stac, masquage des nuages, ordonnanceurs Dask, séries temporelles xarray et sauvegarde Zarr. Le script `05_pipeline/01_pipeline_ndvi.py` assemble ces étapes dans un pipeline complet. Pour créer l'environnement et exécuter les exemples, suivez les instructions du README du dépôt.
