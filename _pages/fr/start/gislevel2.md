---
title: "Notions avancées en GéoInfo"
permalink: /fr/gis-level2/
lang: fr
date: 2025-04-24T03:02:20+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: start
---
> **_NOTE:_**  Le site [CookBook de Yves sur GitHub](https://github.com/yvoirin/cookbook_python3) fournit de nombreux exemples de codes utiles aux géomaticiens. Cette [chaîne Youtube](https://www.youtube.com/@YvesVoirin) offre aussi de nombreuses ressources.

## Shapely

Shapely est souvent utilisé en combinaison avec d'autres bibliothèques géospatiales telles que `geopandas` et `PyQGIS` pour traiter des données géospatiales. Voici quelques-unes des principales caractéristiques de Shapely :

* Shapely prend en charge une grande variété de géométries, y compris les points, les lignes, les polygones, les cercles, les ellipses, les multipoints, les multilignes et les multipolygones.
* Les géométries Shapely sont immuables, ce qui signifie qu'elles ne peuvent pas être modifiées une fois qu'elles ont été créées. Cela garantit que les géométries sont stables et cohérentes dans le temps.
* Shapely fournit des fonctions pour effectuer des opérations géométriques courantes telles que l'intersection, l'union, la différence et la distance entre les géométries.
* Les géométries Shapely peuvent être utilisées pour calculer des mesures géométriques telles que l'aire, la longueur, le périmètre et le centre de gravité d'une géométrie.
* Shapely est facile à utiliser et à intégrer dans des scripts Python. La documentation est bien écrite et fournit des exemples de code pour la plupart des fonctions.

Création d'un point et d'un polygone en utilisant Shapely :

```python
from shapely.geometry import Point, Polygon

# Création d'un point
point = Point(0, 0)

# Création d'un polygone
polygone = Polygon([(0, 0), (0, 1), (1, 1), (1, 0)])
```

Quelques bouts de codes montrant la manipulation avec `shapely`.

```python
import shapely.wkt as wkt
import shapely.geometry as geometry
from shapely.ops import transform
import pyproj

# quelques opérations de base avec Shapely

# on déclare 2 points
# Montréal
lon_mtl, lat_mtl = -73.561668,45.508888
# Sherbrooke
lon_she, lat_she = -71.888351,45.404476

# créer une géométrie à partir d'un WKT
pt_mtl = wkt.loads(f'POINT({lon_mtl} {lat_mtl})')

# créer une géométrie à partir d'un couple x, y
pt_she = geometry.Point((lon_she,lat_she))

# créer une géométrie à partir d'un json
pt_json = {'type': 'Point', 'coordinates': (lon_mtl, lat_mtl)}
pt_mtl2 = geometry.shape(pt_json)

# créer une ligne à partir d'une liste de valeurs
line = [(lon_mtl, lat_mtl), (lon_she, lat_she)]
line_mtl_she = geometry.LineString(line)

# créer une ligne à partir d'une liste de points
line2 = [pt_mtl, pt_she]
line_mtl_she2 = geometry.LineString(line2)

# récupérer le type de la géométrie
print(line_mtl_she2.geom_type)

# parcourir les points
for pt in line_mtl_she2.coords:
    print(pt)

# récupérer la boîte englobante
print(line_mtl_she.bounds)

# créer une géométrie à partir des bornes
poly = geometry.box(*line_mtl_she.bounds)
print(poly)

# vérifier si un point se trouve dans la géométrie
print(poly.contains(poly.centroid))

# reprojeter
crs1 = pyproj.CRS('EPSG:4326')
crs2 = pyproj.CRS('EPSG:6622')

trans = pyproj.Transformer.from_crs(crs1, crs2).transform
polyproj = transform(trans, poly)
print(polyproj)
```

Vidéo utile : [L'essentiel de Shapely en quelques minutes](https://youtu.be/vAvlzYwSca8)

## Rtree (index spatial)

L'utilisation de RTree permet de résoudre des problèmes de recherche spatiale, tels que la recherche de points dans un certain rayon, la recherche des intersections entre des polygones, et la recherche des polygones qui contiennent un point donné.

L'utilisation de RTree est relativement simple. Pour utiliser RTree, il faut d'abord créer un index, en spécifiant le type d'objet que l'on souhaite indexer, et en spécifiant les propriétés que l'on souhaite indexer. Ensuite, on peut ajouter des objets à l'index, et effectuer des requêtes pour rechercher des objets dans l'index.

Utilisation de RTree pour rechercher des points dans un rayon donné :

```python
from rtree import index

# Créer un index RTree pour les points
p_idx = index.Index(properties=['x', 'y'])

# Ajouter quelques points à l'index
p_idx.insert(1, (0, 0, 0, 0))
p_idx.insert(2, (1, 1, 1, 1))
p_idx.insert(3, (2, 2, 2, 2))
p_idx.insert(4, (3, 3, 3, 3))

# Rechercher les points dans un rayon de 1 autour du point (1, 1)
results = list(p_idx.nearest((1, 1, 1, 1), 1.0))
print(results)
```

RTree est une bibliothèque très utile pour l'indexation spatiale en Python, et est souvent utilisée en conjonction avec d'autres bibliothèques telles que Shapely et Geopandas.

Vidéo utile : [Comprendre l'intérêt de rtree en quelques minutes](https://youtu.be/drJkdRbSaBo)

## Passer d'une matrice à un vecteur (avec fiona et rasterio)

Cette opération est parfois nécessaire quand on souhaite obtenir un fichier vectoriel à partir d'une donnée matricielle. Par exemple, suite à une classification, on souhaite souvent obtenir les polygones des zones classifiées. Cette opération peut se faire avec fiona et rasetrio.

```python
import rasterio
from rasterio.features import shapes
import fiona
from shapely.geometry import shape, mapping
from fiona.crs import from_string
image_path = r'../data/mask_satellite.tiff'
# Ouvrez l'image raster en mode lecture
with rasterio.open(image_path) as src:
    
    image = src.read(1) # Lisez la première bande de l'image (ou la bande souhaitée)
    # Trouvez les régions (objets) dans l'image
    image_regions = list(shapes(image, mask=None, transform=src.transform))
    shapefile_path = r'../data/mask_satellite_convert.shp' # Spécifiez le chemin de sortie
    # obtenir le système de coordonnées de l'image en epsg
    crs_proj4 = src.crs.to_proj4() # Obtenez le système de coordonnées de l'image
    # Ouvrez le fichier Shapefile en mode écriture
    crs_vector = from_string(crs_proj4) # Convertissez le système de coordonnées
    with fiona.open(shapefile_path, 'w', 'ESRI Shapefile', crs=from_string(crs_proj4),
        schema={'geometry': 'Polygon', 'properties':{}}) as output:
        for geom, _ in image_regions:
            # Enregistrez la géométrie dans le fichier Shapefile
            feature = {'geometry': mapping(shape(geom)), 'properties': {}}
            output.write(feature)
```

## Passer d'un vecteur à une matrice (avec fiona et rasterio)

Cette opération est aussi assez courante en géomatique. Elle permet d'obtenir une grille à partir d'un fichier vectoriel. Par exemple, il arrive parfois que l'on souhaite appliquer un masque matriciel sur une image. La donnée à l'origine du masque est une donnée vectorielle.

```python
import fiona
import rasterio
from rasterio.features import geometry_mask
from shapely.geometry import shape
# Spécifiez le chemin vers votre fichier Shapefile
shapefile_path = r'../data/mask_satellite.shp'
# Ouvrez le fichier Shapefile en mode lecture
with fiona.open(shapefile_path, 'r') as shapefile:
    print(shapefile.meta)
    # Obtenez la première géométrie du Shapefile
    firstshape = shape(shapefile[0]["geometry"])
    # Obtenez les limites de la géométrie
    bbox = firstshape.bounds

    # la résolution souhaitée du raster en mètres par pixel
    resolution = 10 # 10 mètres
    # Transformation à partir de l'origine, de la résolution et des dimensions
    transform = rasterio.transform.from_origin(bbox[0], bbox[3], resolution, resolution)
    # Dimensions du raster à partir de la transformation
    width = int((bbox[2] - bbox[0]) / resolution)
    height = int((bbox[3] - bbox[1]) / resolution)
    # Masque raster à partir des géométries du Shapefile
    mask = geometry_mask([firstshape], transform=transform, invert=True,
    out_shape=(height, width))
    # Créez un nouveau fichier raster
    with rasterio.open(r'../data/mask_satellite.tiff', 'w', driver='GTiff', height=height,
        width=width, count=1, dtype='uint8', crs=shapefile.crs,
        transform=transform) as dst:
        dst.write(mask, 1)
```

## Découper une image par rapport à une géométrie (avec fiona et rasterio)

Cette opération est assez courante en géomatique car elle consiste à découper une image en fonction d'une zone d'intérêt. Cette opération se réalise simplement avec fiona et rasterio.

```python
import fiona
import rasterio
import rasterio.mask
# Spécifiez le chemin vers votre fichier Shapefile
shapefile_path = r'../data/mask_satellite.shp'
# Ouvrez le fichier Shapefile en mode lecture
with fiona.open(shapefile_path, 'r') as shapefile:
    # Accédez à la première entité géométrique
    geom = shapefile[0]['geometry']
    # Spécifiez le chemin vers votre image raster
    raster_path = r'../data/satellite.tiff'
    # Ouvrez l'image raster en mode lecture
    with rasterio.open(raster_path) as src:
        # Découpez l'image en utilisant la géométrie du Shapefile
        out_image, out_transform = rasterio.mask.mask(src, [geom], crop=True)
        out_meta = src.meta.copy()
        out_meta.update({"driver": "GTiff", "height": out_image.shape[1],
        "width": out_image.shape[2], "transform": out_transform})
        with rasterio.open(r"../data/satellite_clip.tiff", "w", **out_meta) as dest:
            dest.write(out_image)
```

## Reprojeter une image avec rasterio

On doit parfois reprojeter une image dans un autre système. Avec rasterio, on peut simplement réaliser cette opération.

```python
import numpy as np
import rasterio
from rasterio.warp import calculate_default_transform, reproject, Resampling
# image en entrée
source_image_path = r'../data/mask_satellite.tiff'
# crs de sortie
dst_crs = 'EPSG:4326'
# ouverture de l'image
with rasterio.open(source_image_path) as src:
    # calcul de la transformation
    transform, width, height = calculate_default_transform(
    src.crs, dst_crs, src.width, src.height, *src.bounds)
    kwargs = src.meta.copy()
    kwargs.update({
    'crs': dst_crs,
    'transform': transform,
    'width': width,
    'height': height
    })

    # création de l'image de sortie
    with rasterio.open(r'../data/mask_satellite_4326.tiff', 'w', **kwargs) as dst:
        for i in range(1, src.count + 1):
            reproject(
            source=rasterio.band(src, i),
            destination=rasterio.band(dst, i),
            src_transform=src.transform,
            src_crs=src.crs,
            dst_transform=transform,
            dst_crs=dst_crs,
            resampling=Resampling.nearest)
```

## Reprojeter une donnée vectorielle avec fiona

Il faut parfois reprojeter des données vectorielles dans un autre système. Pour ce faire on peut utiliser fiona.

```python
import fiona

import pyproj
from shapely.geometry import shape, mapping
from fiona.crs import from_epsg
from shapely.ops import transform
# Chemin vers le fichier Shapefile source
source_shapefile_path = r'../data/mask_satellite.shp'
# Spécifiez la projection cible
target_crs = from_epsg(4326) # WGS 84 (EPSG:4326)
# Chemin vers le fichier Shapefile cible
target_shapefile_path = r'../data/mask_satellite_4326.shp'

# Ouvrez le fichier source en mode lecture
with fiona.open(source_shapefile_path, 'r') as src:
    # Ouvrez le fichier cible en mode écriture et spécifiez la nouvelle projection
    with fiona.open(target_shapefile_path, 'w', 'ESRI Shapefile',
        schema=src.schema, crs=target_crs) as dst:
        # Utilisez pyproj pour effectuer la reprojection des géométries
        project = pyproj.Transformer.from_crs(src.crs, target_crs, always_xy=True).transform
        for feature in src:
            geom = shape(feature['geometry'])
            # Appliquez la transformation de projection
            reprojected_geom = transform(project, geom)
            # Écrivez l'entité dans le fichier cible
            feature['geometry'] = mapping(shape(reprojected_geom))

            dst.write(feature)
            # si cette dernière ligne pose problème (sur colab), essayez plutôt :
            # dst.write({'geometry': mapping(reprojected_geom), 'properties':feature['properties']})
```

<!-- Django fournit une grande variété d'outils et de modules pour faciliter le développement d'applications web (principe du Model-View-Controller (MVC)). Parmi ces outils, on trouve :

- Le système de routage : Django permet de définir facilement les routes URL et de les associer à des vues qui vont traiter les requêtes associées.

- Le système de modèles : Django fournit un ORM (Object-Relational Mapping) qui permet de travailler avec les bases de données de manière abstraite et de façon plus intuitive.

- Les templates : Django permet de créer des modèles HTML dynamiques qui peuvent être remplis avec des données générées par les vues.

- Les middlewares : permettent de traiter les requêtes et les réponses à différents stades du traitement, avant et après le passage par les vues.

- Les formulaires : Django permet de créer facilement des formulaires et de les valider côté serveur.

Django est également très extensible grâce à sa grande variété de modules tiers disponibles. Cela permet aux développeurs de construire rapidement des fonctionnalités avancées sans avoir à les coder eux-mêmes. 

En géomatique, on peut utiliser GeoDjango qui apporte un volet spatial au site web. Ce sont les mêmes principes que dans Django.

Vidéos utiles : 
* [Introduction à Django (Premiers pas)](https://youtu.be/PzQ9lfjdMv4)
* [Initiation à Django (ajouter une composante au site de base)](https://youtu.be/8aF0_pfxd4w)
* [Initiation à Django (ajout d'une vue mettant en relief les éléments de la base de données)](https://youtu.be/FW5gaWCl1HY)

## QGis

QGIS est un logiciel de SIG (Système d'Information Géographique) open-source et gratuit qui permet de visualiser, d'éditer et d'analyser des données spatiales. Il est écrit en C++ et dispose d'une API (Application Programming Interface) qui permet aux développeurs de créer des plugins en Python.

Le développement d'une extension Python dans QGIS nécessite des connaissances en Python et en SIG, ainsi qu'une compréhension de l'API de QGIS. Il faut aussi noter que le volet graphique est basé sur PyQt5. Toutefois il existe des outils permettant d'accélérer le développement, notamment le Plugin Builder. Cet outil vous donnera toute la structure minimale pour obtenir un plugin fonctionnel.

Il est important de noter que le développeur doit disposer de bonnes références vers la documentation technique touchant à QGis et PyQt5.

Vidéos utiles : 
* [Déployer la base d'un plugin sous QGis 3 (avec le plugin builder)](https://youtu.be/pnDr149JMWU)
* [Faire un plugin qui va créer une couche vectorielle (QGis 3.4)](https://youtu.be/hMbzWsq0-bI)
* [Comment modifier l'interface graphique d'un plugin (réalisé à partir du plugin Builder) dans QGis 3](https://youtu.be/1VDnon52M4E)
* [Comment bien développer un plugin dans QGis 3](https://youtu.be/KhPa1C1R4G4)
* [Création du plugin Super Csv Reader dans QGis 3](https://youtu.be/mefSqKODucA)
* [Débuter avec la 3d dans QGis](https://youtu.be/s86kIdrlUIo)
* [Ajouter une couche 3D (générée à partir de OSM)](https://youtu.be/eRAn_uZ3Yss)
* [Intégrer des éléments graphiques 3D venant d'autres logiciels](https://youtu.be/nt_XWLqbmVI)
* [QGis et ThreeJs pour la 3D](https://youtu.be/r-26nA05ztg)
* [Faire de la réalité augmentée avec QGis et Openspace3D](https://youtu.be/SnPn1oB8DcQ) -->