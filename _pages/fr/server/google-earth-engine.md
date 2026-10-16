---
title: "Google Earth Engine avec Python"
permalink: /fr/google-earth-engine/
lang: fr
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: gisserver
---

## Présentation

**Google Earth Engine (GEE)** est une plateforme cloud pour explorer et analyser des données géospatiales à grande échelle. Elle donne accès à un vaste catalogue d’images satellites et de jeux de données géographiques, et exécute les traitements sur l’infrastructure de Google plutôt que de télécharger toutes les données sur votre ordinateur.

Cette page s’appuie sur le [notebook Colab « Trois façons de faire un composite Sentinel-2 »](https://colab.research.google.com/drive/1yH1dolerbJT7wXCtE1H9AZIJGcVvFZnJ?usp=sharing), qui compare plusieurs composites de l’été 2025 sur Sherbrooke.

## Le client Python et les calculs sur le serveur

La bibliothèque Python `ee` fournit une interface cliente à Earth Engine. Les appels comme `filterBounds`, `map`, `median` ou `clip` construisent des objets et des opérations qui décrivent le traitement. Le calcul des pixels est exécuté par les serveurs Earth Engine lorsque le résultat est demandé ou exporté; les images sources ne sont donc pas chargées dans la mémoire Python à chaque appel.

```python
import ee

ee.Authenticate()
ee.Initialize(project="VOTRE_PROJET_GOOGLE_CLOUD")

aoi = ee.Geometry.Rectangle([-72.05, 45.33, -71.80, 45.47])
collection = (
    ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED")
    .filterBounds(aoi)
    .filterDate("2025-06-01", "2025-09-01")
    .filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", 60))
)
composite = collection.median().clip(aoi)
```

Ici, `collection` et `composite` sont des objets Earth Engine côté client qui décrivent un calcul côté serveur. La ligne `median()` ne télécharge pas en soi toutes les scènes dans Python.

## Graphes de calcul et évaluation différée

Earth Engine construit un **graphe de calcul** à partir des opérations enchaînées. C’est le principe d’évaluation différée (*lazy evaluation*) : les étapes sont décrites avant d’être exécutées. Le serveur peut alors optimiser le traitement et ne calculer que les données nécessaires au résultat demandé.

Dans le notebook, les méthodes `map`, `updateMask`, `normalizedDifference`, `median`, `mosaic` et `qualityMosaic` s’enchaînent de cette manière. Par exemple, le masque de nuages et l’ajout d’un indice NDVI peuvent être décrits par une fonction appliquée à chaque image :

```python
classes_a_exclure = [0, 1, 3, 8, 9, 10]

def preparer_image(image):
    scl = image.select("SCL")
    masque_valide = (
        scl.remap(classes_a_exclure, [0] * len(classes_a_exclure), 1)
        .eq(1)
    )
    image = image.updateMask(masque_valide)
    ndvi = image.normalizedDifference(["B8", "B4"]).rename("NDVI")
    return image.addBands(ndvi)

collection_preparee = collection.map(preparer_image)
composite_median = collection_preparee.median().clip(aoi)
```

`map(preparer_image)` décrit l’opération à appliquer aux images de la collection; il ne crée pas une boucle Python qui télécharge les scènes une à une.

### Récupérer de petites valeurs côté client

`getInfo()` force l’évaluation de l’objet côté serveur et transfère le résultat vers Python de façon synchrone. Le notebook l’utilise notamment pour récupérer les dates et les pourcentages de nuages, qui sont de petites métadonnées. Évitez de l’utiliser pour rapatrier de grandes images ou des collections entières :

```python
dates = collection.aggregate_array("system:time_start").getInfo()
```

La règle pratique est de garder les images et les grands résultats sur le serveur, et de ne transférer vers Python que les petites informations nécessaires à l’affichage ou au contrôle.

## Découvrir le catalogue de données

Le **[Google Earth Engine Data Catalog](https://developers.google.com/earth-engine/datasets/catalog/)** est le catalogue des collections et produits disponibles dans GEE. Chaque fiche décrit la couverture, les bandes, les propriétés, la résolution, les dates disponibles et l’identifiant à utiliser dans le code.

Le notebook exploite la collection [Sentinel-2 MSI: MultiSpectral Instrument, Level-2A (SR) Harmonized](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED), identifiée dans l’API par `COPERNICUS/S2_SR_HARMONIZED`. On peut consulter les bandes, les classes de la bande `SCL` et les propriétés de scènes comme `CLOUDY_PIXEL_PERCENTAGE` avant de construire la requête.

Le pourcentage de nuages de cette propriété est calculé sur la scène entière, et non uniquement sur la zone d’étude. Le filtre est donc un premier tri; il ne remplace pas un masque de pixels nuageux.

## Trois façons de créer un composite

Après le masquage des pixels non désirés, le notebook compare trois opérations sur la collection :

```python
composite_median = collection_preparee.median().clip(aoi)

composite_mosaique = (
    collection_preparee
    .sort("CLOUDY_PIXEL_PERCENTAGE", False)
    .mosaic()
    .clip(aoi)
)

composite_plus_vert = (
    collection_preparee
    .qualityMosaic("NDVI")
    .clip(aoi)
)
```

* **Médiane par pixel :** calcule la médiane de chaque bande pour chaque pixel de la période; elle réduit l’influence des valeurs extrêmes.
* **Mosaïque :** assemble les images en remplissant les pixels masqués par les observations disponibles. L’ordre des images détermine la priorité des pixels qui se chevauchent.
* **`qualityMosaic("NDVI")` :** choisit, pixel par pixel, l’image où le NDVI est maximal et prend les bandes correspondantes de cette observation. Cela forme un composite du « pixel le plus vert », qui ne correspond pas nécessairement à une seule date.

Comparer les sorties avec le même étirement visuel, comme dans le notebook, aide à interpréter les différences entre méthodes.

## Exporter : une tâche asynchrone

Un export n’est pas un téléchargement immédiat dans la session Python. `ee.batch.Export` prépare une **tâche asynchrone** exécutée par Earth Engine. Il faut la démarrer, puis attendre qu’elle se termine; un export vers Google Drive apparaît ensuite dans le dossier Drive spécifié.

```python
task = ee.batch.Export.image.toDrive(
    image=composite_median,
    description="sentinel2_composite_sherbrooke",
    folder="earth_engine_exports",
    fileNamePrefix="sentinel2_composite",
    region=aoi,
    scale=10,
    maxPixels=1e13,
    fileFormat="GeoTIFF",
)
task.start()

print(task.status())
```

Le statut initial est généralement `READY` ou `RUNNING`. Pour suivre la tâche depuis Python, on peut vérifier son état à intervalles réguliers :

```python
import time

while task.active():
    print(task.status()["state"])
    time.sleep(30)

print(task.status())
```

Une tâche peut prendre du temps, échouer ou demander des paramètres adaptés à la taille et à la projection du résultat. Vérifiez son état final et les messages d’erreur; une fois terminée, récupérez le fichier dans Google Drive. Les paramètres d’export, notamment `region`, `scale`, `crs` et `maxPixels`, déterminent l’emprise et la résolution du raster généré.

## À retenir

* Les méthodes de la bibliothèque `ee` décrivent des opérations exécutées sur les serveurs Earth Engine; les objets image ne sont pas automatiquement chargés dans le client Python.
* Les opérations sont évaluées de façon différée au moyen d’un graphe de calcul, jusqu’à une requête de résultat ou une tâche d’export.
* `getInfo()` transfère les résultats vers le client et doit être réservé aux petites valeurs et métadonnées.
* Le catalogue Earth Engine permet de choisir les collections et de comprendre leurs bandes et propriétés.
* Les exports sont des tâches asynchrones : il faut les démarrer, suivre leur état et récupérer les fichiers une fois terminés.

Consultez le [notebook Colab](https://colab.research.google.com/drive/1yH1dolerbJT7wXCtE1H9AZIJGcVvFZnJ?usp=sharing) pour exécuter la comparaison complète des composites Sentinel-2 sur Sherbrooke.
