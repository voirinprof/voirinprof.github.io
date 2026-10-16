---
title: "Répertoire des codes utiles"
permalink: /fr/python-useful/
lang: fr
date: 2026-10-16T08:31:13-04:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: misc
---


> **_NOTE:_** Cette page recense mes principaux projets, exemples de code et ressources partagées autour de la **programmation**, de la **géomatique**, de **l'analyse spatiale** et des **bases de données**.

## Introduction

Bienvenue sur ma page de présentation de mes dépôts GitHub !  
Vous trouverez ici :
- des scripts utiles pour le traitement de données spatiales,
- des projets complets pour la création d'applications géomatiques,
- des exemples d'intégration Python avec différentes bases de données.

Chaque dépôt est accompagné d'une description rapide, d'un lien d'accès et parfois d'une recommandation d'usage.

---

## Dépôts principaux

### 1. **geolabspace** - Exemple de Codespace pour la géomatique

- **Lien** : [https://github.com/voirinprof/geolabspace](https://github.com/voirinprof/geolabspace)
- **Résumé** : Un environnement prêt à l'emploi (GitHub Codespaces) pour coder en Python dans un contexte géomatique.
- **Points clés** :
  - Python préinstallé
  - Bibliothèques spatiales (`geopandas`, `shapely`, `folium`)
  - Tutoriels d'introduction à la programmation géospatiale

---

### 2. **gis_nosql_docker** - Application de routing combinant Redis et Neo4j

- **Lien** : [https://github.com/voirinprof/gis_nosql_docker](https://github.com/voirinprof/gis_nosql_docker)
- **Résumé** : Démo d'une application combinant la recherche rapide d'adresse via Redis et le calcul d'itinéraire optimal avec Neo4j.
- **Points clés** :
  - Scripts Python pour interagir avec Redis et Neo4j
  - Algorithmes de recherche de chemin (`Dijkstra`)  

---

### 3. **gis_nx_docker** - Faire du routing avec networkx

- **Lien** : [https://github.com/voirinprof/gis_nx_docker](https://github.com/voirinprof/gis_nx_docker)
- **Résumé** : Application pour le calcul d'itinéraire sans utiliser de base de données.
- **Points clés** :
  - Utilisation de `networkx` 
  - Exemples de requêtes spatiales 
  - Chargement de fichiers GeoJSON et Shapefile

---

### 4. **gis_pgrouting_docker** - Utiliser pgRouting avec Python

- **Lien** : [https://github.com/voirinprof/gis_pgrouting_docker](https://github.com/voirinprof/gis_pgrouting_docker)
- **Résumé** : Scripts pratiques pour utiliser `pgRouting` depuis Python, sur un réseau routier spatial PostGIS.
- **Points clés** :
  - Préparation de graphes routiers
  - Exécution de `pgr_createTopology`, `pgr_dijkstra`

---

### 5. **neo4j_docker** - Débuter avec Neo4j

- **Lien** : [https://github.com/voirinprof/neo4j_docker](https://github.com/voirinprof/neo4j_docker)
- **Résumé** : Scripts pour débuter avec Neo4j.
- **Points clés** :
  - Usage de `Neo4j` avec un service web
  - Requêtes 

---

### 6. **neo4j_docker** - Débuter avec Neo4j

- **Lien** : [https://github.com/voirinprof/neo4j_docker](https://github.com/voirinprof/neo4j_docker)
- **Résumé** : Scripts pour débuter avec Neo4j.
- **Points clés** :
  - Usage de `Neo4j` avec un service web
  - Requêtes 

---

### 7. **redis_docker** - Débuter avec Redis

- **Lien** : [https://github.com/voirinprof/redis_docker](https://github.com/voirinprof/redis_docker)
- **Résumé** : Scripts pour débuter avec Redis.
- **Points clés** :
  - Usage de `Redis` avec un service web
  - Requêtes 

---

### 8. **mongodb_docker** - Débuter avec MongoDB

- **Lien** : [https://github.com/voirinprof/mongodb_docker](https://github.com/voirinprof/mongodb_docker)
- **Résumé** : Scripts pour débuter avec MongoDB.
- **Points clés** :
  - Usage de `MongoDB` avec un service web
  - Requêtes 

---

### 9. **python_webservices_docker** - Débuter avec Flask et FastAPI

- **Lien** : [https://github.com/voirinprof/python_webservices_docker](https://github.com/voirinprof/python_webservices_docker)
- **Résumé** : Scripts pour débuter avec Flask et FastAPI.
- **Points clés** :
  - Usage de `Flask` comme un service web
  - Usage de `FastAPI` comme un service web
  
---

### 10. **geoserver_docker** - Débuter avec GeoServer

- **Lien** : [https://github.com/voirinprof/geoserver_docker](https://github.com/voirinprof/geoserver_docker)
- **Résumé** : Application pour débuter avec GeoServer.
- **Points clés** :
  - Usage de `GeoServer` avec des services WMS, WFS
  - Petite application avec un client web 

---

### 11. **qgis_docker** - Débuter avec QGis Server

- **Lien** : [https://github.com/voirinprof/qgis_docker](https://github.com/voirinprof/qgis_docker)
- **Résumé** : Application pour débuter avec QGis Server.
- **Points clés** :
  - Usage de `QGIS Server` avec un service `Flask`
  - Petite application avec un client web 

---

### 12. **mapnik_docker** - Débuter avec Mapnik

- **Lien** : [https://github.com/voirinprof/mapnik_docker](https://github.com/voirinprof/mapnik_docker)
- **Résumé** : Application pour débuter avec Mapnik.
- **Points clés** :
  - Usage de `Mapnik` avec un service `Flask`
  - Petite application avec un client web 

---

### 13. **python-geo-colab** - Débuter avec Colab et la géomatique

- **Lien** : [https://github.com/voirinprof/python-geo-colab](https://github.com/voirinprof/python-geo-colab)
- **Résumé** : Ensemble de notebooks sur Colab qui permet d'utiliser GEE.
- **Points clés** :
  - Usage de `GEE` dans Colab
  - Démonstrations de `random forest` et de `deeplabv3`

---

### 14. **cookbook_python3** - Opérations de base en Python en géomatique

- **Lien** : [https://github.com/yvoirin/cookbook_python3](https://github.com/yvoirin/cookbook_python3)
- **Résumé** : Ensemble de scripts utiles en géomatique.
- **Points clés** :
  - Usage de `fiona`, `rasterio`, `pyproj`, `opencv`, `pandas`, `geopandas`, ...  

---


## Ressources complémentaires

- **Mes snippets utiles** : petits bouts de code réutilisables dans vos projets
- **Mes configurations Docker Compose** : pour déployer facilement des environnements géomatiques complets
- **Mes tutoriels et cours** : guides pas-à-pas pour découvrir Python appliqué aux SIG

> 📌 **À venir** : de nouveaux projets autour de QGIS Python plugins, ...

---

## Liens

### Langages et applications web

- [Documentation Python](https://docs.python.org/fr/3/)
- [Documentation JavaScript (MDN)](https://developer.mozilla.org/fr/docs/Web/JavaScript)
- [Documentation TypeScript](https://www.typescriptlang.org/docs/)
- [Documentation Phaser](https://docs.phaser.io/)
- [Documentation Vue.js](https://vuejs.org/guide/)
- [Documentation React](https://react.dev/learn)
- [Documentation Angular](https://angular.dev/overview)
- [Documentation Vite](https://vite.dev/guide/)
- [Documentation Pinia](https://pinia.vuejs.org/)
- [Documentation Leaflet](https://leafletjs.com/reference.html)
- [Documentation OpenLayers](https://openlayers.org/en/latest/apidoc/)
- [Documentation MapLibre GL JS](https://maplibre.org/maplibre-gl-js/docs/)

### Python et géomatique

- [Documentation NumPy](https://numpy.org/doc/stable/)
- [Documentation pandas](https://pandas.pydata.org/docs/)
- [Documentation GeoPandas](https://geopandas.org/en/stable/)
- [Documentation Shapely](https://shapely.readthedocs.io/en/stable/)
- [Documentation Fiona](https://fiona.readthedocs.io/en/stable/)
- [Documentation Rasterio](https://rasterio.readthedocs.io/en/stable/)
- [Documentation GDAL](https://gdal.org/en/stable/)
- [Documentation pyproj](https://pyproj4.github.io/pyproj/stable/)
- [Documentation Matplotlib](https://matplotlib.org/stable/)
- [Documentation Folium](https://python-visualization.github.io/folium/latest/)
- [Documentation NetworkX](https://networkx.org/documentation/stable/)
- [Documentation Rtree](https://rtree.readthedocs.io/en/latest/)
- [Documentation Pillow](https://pillow.readthedocs.io/en/stable/)
- [Documentation OpenCV](https://docs.opencv.org/4.x/)
- [Documentation Requests](https://requests.readthedocs.io/en/latest/)
- [Documentation PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation)

### Données géospatiales et traitement à grande échelle

- [Documentation PySTAC Client](https://pystac-client.readthedocs.io/en/stable/)
- [Documentation odc-stac](https://odc-stac.readthedocs.io/en/latest/)
- [Documentation Dask](https://docs.dask.org/en/stable/)
- [Documentation xarray](https://docs.xarray.dev/en/stable/)
- [Documentation de l’API Python Google Earth Engine](https://developers.google.com/earth-engine/guides/python_install)

### Services web, automatisation et tests

- [Documentation Flask](https://flask.palletsprojects.com/en/stable/)
- [Documentation FastAPI](https://fastapi.tiangolo.com/)
- [Documentation Uvicorn](https://www.uvicorn.org/)
- [Documentation Pydantic](https://docs.pydantic.dev/latest/)
- [Documentation PyWPS](https://pywps.readthedocs.io/)
- [Documentation Celery](https://docs.celeryq.dev/en/stable/)
- [Documentation Flower](https://flower.readthedocs.io/en/latest/)
- [Documentation Locust](https://docs.locust.io/)

### Bases de données et pilotes Python

- [Documentation PostgreSQL](https://www.postgresql.org/docs/current/)
- [Documentation PostGIS](https://postgis.net/documentation/)
- [Documentation pgRouting](https://pgrouting.org/documentation.html)
- [Documentation Psycopg 3](https://www.psycopg.org/psycopg3/docs/)
- [Documentation Psycopg 2](https://www.psycopg.org/docs/)
- [Documentation SQLAlchemy](https://docs.sqlalchemy.org/en/20/)
- [Documentation GeoAlchemy 2](https://geoalchemy-2.readthedocs.io/en/latest/)
- [Documentation Redis](https://redis.io/docs/latest/)
- [Documentation redis-py](https://redis-py.readthedocs.io/)
- [Documentation Neo4j](https://neo4j.com/docs/)
- [Documentation du pilote Python Neo4j](https://neo4j.com/docs/api/python-driver/current/)
- [Documentation Neo4j Graph Data Science](https://neo4j.com/docs/graph-data-science/current/)
- [Documentation MongoDB](https://www.mongodb.com/docs/)
- [Documentation PyMongo](https://pymongo.readthedocs.io/)

### Intelligence artificielle et vision par ordinateur

- [Documentation PyTorch](https://pytorch.org/docs/stable/index.html)
- [Documentation Torchvision](https://pytorch.org/vision/stable/index.html)
- [Documentation Ultralytics](https://docs.ultralytics.com/)
- [Documentation Segment Geospatial (SamGeo)](https://samgeo.gishub.org/)

### Serveurs et outils SIG

- [Documentation QGIS Server](https://docs.qgis.org/latest/fr/docs/server_manual/)
- [PyQGIS Developer Cookbook](https://docs.qgis.org/latest/en/docs/pyqgis_developer_cookbook/)
- [Documentation GeoServer](https://docs.geoserver.org/stable/en/user/)
- [Documentation MapServer](https://mapserver.org/documentation.html)
- [Documentation Mapnik](https://get-map.org/mapnik-lost-manual/book.html)

---

## Comment contribuer

- ⭐️ **Star** un dépôt si vous le trouvez utile
- 🐛 **Signalez** un bug ou proposez une amélioration via les Issues
- 🔥 **Forkez** et proposez des Pull Requests si vous souhaitez participer à l'amélioration de ces ressources