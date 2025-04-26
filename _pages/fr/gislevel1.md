---
title: "Premières notions en GéoInfo"
permalink: /fr/gis-level1/
lang: fr
date: 2025-04-24T03:02:20+00:00
toc: true
toc_label: "Dans cette page"
typenav: start
---
> **_NOTE:_**  Le site [CookBook de Yves sur GitHub](https://github.com/yvoirin/cookbook_python3) fournit de nombreux exemples de codes utiles aux géomaticiens. Cette [chaîne Youtube](https://www.youtube.com/@YvesVoirin) offre aussi de nombreuses ressources.

## Installer vos librairies

Les commandes pour installer les librairies sont toujours les mêmes, soit on utilise PIP :

```shell
pip install votre_lib
```

ou avec CONDA :

```shell
conda install votre_lib
```

Pour débuter dans le développement en géomatique, il y a essentiellement quelques librairies qui sont utiles :

* OGR/GDAL : il s'agit d'une librairie C++ qui est utilisée depuis très longtemps en géomatique. Elle permet de manipuler des données vectorielles et matricielles. Toutefois l'installation peut nécessiter quelques astuces. Il est peut-être préférable de s'orienter vers rasterio et fiona qui sont des passerelles vers OGR/GDAL.
* rasterio : il s'agit d'une passerelle vers GDAL. Cette lib permet la manipulation des données matricielles. Elle est très simple d'usage.
* fiona : il s'agit de la passerelle vers OGR. Cette lib permet la manipulation des données vectorielles. Un peu comme rasterio, elle suit les mêmes conventions ce qui la rend très conviviale.
* pandas/geopandas : si votre projet nécessite la manipulation des données formattées, comme CSV. Ces librairies permettent la manipulation des fichiers (et même des bases de données) d'une manière assez rapide et simple. Il faut noter que geopandas est aussi capable de lire une données vectorielles. Elle pourrait remplacer fiona.
* matplotlib : cette librairie sera très utile si vous devez produire des graphiques. Elle remplace complètement les opérations que l'on pourrait faire sous Excel.

Votre environnement de développement devrait avoir ces librairies. Selon le projet, on peut ajouter d'autres libs plus spécifiques.

## Lire des données spatiales

### Données vectorielles

Les formats courants que Python prend en charge comprennent Shapefile, GeoJSON, GPKG, CSV, etc.

Avec `fiona`, on peut lire des données géospatiales, comme un fichier Shapefile :

```python
import fiona

# Ouverture du fichier Shapefile
with fiona.open('fichier.shp', 'r') as src:
    # Boucle à travers les entités du fichier Shapefile
    for feature in src:
        print(feature)
```

ou encore un fichier GeoJSON :

```python
import fiona

# Ouverture du fichier GeoJSON
with fiona.open('chemin/vers/mon/fichier.geojson', 'r') as src:
    # Boucle à travers les entités du fichier GeoJSON
    for feature in src:
        print(feature)
```

Une alternative à `fiona` est `geopandas`.

Voici un exemple de code pour lire un fichier Shapefile avec `geopandas` :

```python
import geopandas as gpd

# Lire le fichier Shapefile
gdf = gpd.read_file('chemin/vers/mon/fichier.shp')

# Afficher les 5 premières lignes du fichier
print(gdf.head())
```
### Données matricielles

Pour lire un fichier raster en Python, on peut utiliser la bibliothèque `rasterio`.

Nous pouvons utiliser les étapes suivantes :

```python
import rasterio
import matplotlib.pyplot as plt

# Ouverture du fichier raster
with rasterio.open('mon_fichier.tif') as src:
    # Lecture des données du raster
    raster = src.read(1)
    
# Affichage du raster
plt.imshow(raster, cmap='gray')
plt.show()
```

Il est important de noter que le format raster peut prendre en charge plusieurs bandes de données, chacune représentant une variable différente. Dans ce cas, `src.read` peut être utilisé pour lire plusieurs bandes de données en même temps.

Il faut noter que les images lues seront des matrices `numpy`. On peut donc appliquer toutes opérations possibles de la librairie `numpy`.


## Écrire des données spatiales

### Données vectorielles

Supposons que nous ayons un ensemble de données vectorielles stockées dans une liste Python nommée `features`, que nous voulons écrire dans un fichier shapefile nommé `mon_fichier.shp`. Nous pouvons utiliser les étapes suivantes :

```python
import fiona

# Création d'un schema pour les données vectorielles
schema = {
    'geometry': 'Point',
    'properties': {
        'nom': 'str',
        'valeur': 'int'
    }
}

# Création d'un fichier shapefile
with fiona.open('mon_fichier.shp', 'w', 'ESRI Shapefile', schema) as dst:
    # Écriture des données vectorielles dans le fichier
    for feature in features:
        dst.write(feature)
```

Le paramètre `schema` dans `fiona.open` définit le schéma pour les données vectorielles, y compris la géométrie et les attributs. Dans cet exemple, le schéma indique que les données vectorielles sont des points avec deux attributs : `nom` de type chaîne de caractères (`'str'`) et `valeur` de type entier (`'int'`).

La boucle `for` dans cet exemple itère sur chaque élément de la liste `features` et utilise la méthode `write` pour écrire les données vectorielles dans le fichier shapefile.

### Données matricielles

Supposons que nous ayons une matrice de données stockée dans une matrice `NumPy` nommé `matrice`, que nous voulons écrire dans un fichier GeoTIFF nommé `mon_fichier.tif`. Nous pouvons utiliser les étapes suivantes :

```python
import rasterio

# Obtention de la taille de la matrice
hauteur, largeur = matrice.shape

# Définition des métadonnées pour le fichier GeoTIFF
metadata = {
    'count': 1,
    'crs': 'EPSG:4326',
    'dtype': 'float32',
    'driver': 'GTiff',
    'height': hauteur,
    'width': largeur,
    'transform': rasterio.transform.from_bounds(0, 0, largeur, hauteur, largeur, hauteur),
    'nodata': -9999
}

# Écriture de la matrice dans le fichier GeoTIFF
with rasterio.open('mon_fichier.tif', 'w', **metadata) as dst:
    dst.write(matrice, 1)
```

Le dictionnaire `metadata` dans `rasterio.open` définit les métadonnées pour le fichier GeoTIFF, y compris le nombre de bandes (`'count'`), la référence spatiale (`'crs'`), le type de données (`'dtype'`), le format de fichier (`'driver'`), la hauteur (`'height'`) et la largeur (`'width'`) de la matrice, la transformation (`'transform'`) qui définit la position et la taille de chaque pixel, et la valeur de remplissage (`'nodata'`).

La méthode `from_bounds` dans `rasterio.transform` est utilisée pour définir la transformation. Dans cet exemple, la transformation est définie en utilisant les coordonnées x et y des coins inférieur gauche et supérieur droit de la matrice, ainsi que la largeur et la hauteur de la matrice.

## Échanges avec une BD

Pour échanger avec une base de données en géomatique en Python, nous pouvons utiliser la bibliothèque `pandas` ou `geopandas` qui permet d'importer, manipuler et exporter facilement des données tabulaires. Pour se connecter à une base de données, nous avons besoin d'une bibliothèque de connecteur appropriée telle que `psycopg2` pour PostgreSQL ou `pyodbc` pour Microsoft SQL Server.

Supposons que nous avons une base de données PostgreSQL nommée `ma_base_de_donnees` contenant une table `mes_donnees_spatiales` qui contient des données géospatiales, nous pouvons utiliser les étapes suivantes pour importer les données de la table en tant que `DataFrame` de `pandas` :

```python
import pandas as pd
import psycopg2

# Connexion à la base de données
conn = psycopg2.connect("dbname=ma_base_de_donnees user=mon_utilisateur password=mon_mot_de_passe")

# Lecture de la table en tant que DataFrame
df = pd.read_sql_query("SELECT * FROM mes_donnees_spatiales", conn)

# Fermeture de la connexion à la base de données
conn.close()
```

La table `mes_donnees_spatiales` de la base de données devient un DataFrame.

Si la base de données est à référence spatiale, on utilise `geopandas` :

```python
import geopandas as gpd
import psycopg2

# Connexion à la base de données
conn = psycopg2.connect("dbname=ma_base_de_donnees user=mon_utilisateur password=mon_mot_de_passe")

# Lecture de la table en tant que GeoDataFrame
gdf = gpd.read_postgis("SELECT * FROM mes_donnees_spatiales", conn, geom_col="geom")

# Fermeture de la connexion à la base de données
conn.close()
```

On remarque que l'on pourra récupérer aussi bien les attributs que les géométries. La table `mes_donnees_spatiales` devient ici un GeoDataFrame, nous pouvons effectuer des manipulations et des analyses de données géospatiales à l'aide de `geopandas` avant d'exporter les données dans un autre format de fichier géospatial.

## Vidéos utiles

Voici une série de vidéos qui couvrent les notions de cette page : 
* [Différentes manières de lire des données vectorielles et matricielles avec Python 3](https://youtu.be/QsnectEfqTo)
* [Lire une image et traiter avec Numpy](https://youtu.be/YxnHey-V2Eg)
* [Opérations de base avec Numpy](https://youtu.be/V8Yk6j_gpxs)
* [Lire des images et calculer un ndvi avec numpy, gdal et matplotlib](https://youtu.be/-mJ0WnMlwQc)
* [Lire une données d'un CSV et créer une matrice avec Numpy, ensuite afficher avec Matplotlib](https://youtu.be/cnITO_D9A8I)
* [Utiliser Kaggle pour analyser des données (avec Pandas, Matplotlib et folium)](https://youtu.be/XHOG8GqT15Y)
* [Premiers pas avec Pandas et Anaconda (dans Spyder)](https://youtu.be/8j4vRloPyzU)
* [Analyse de plusieurs fichiers CSV avec Pandas (exemple des données Bixi)](https://youtu.be/Qn6lGCDslhw)
* [Consolidation de fichiers CSV avec Pandas](https://youtu.be/NKKu_-8-kGY)
* [Utiliser Geopandas](https://youtu.be/hONrwu8Kbx4)
* [Comment écrire un GeoTiff ou un Shapefile avec Rasterio et Fiona en quelques lignes.](https://youtu.be/dJYH0jnydHI)
