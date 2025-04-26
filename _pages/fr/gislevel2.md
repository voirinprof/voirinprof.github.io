---
title: "Notions avancées en GéoInfo"
permalink: /fr/gis-level2/
lang: fr
date: 2025-04-24T03:02:20+00:00
toc: true
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

## Django

Django fournit une grande variété d'outils et de modules pour faciliter le développement d'applications web (principe du Model-View-Controller (MVC)). Parmi ces outils, on trouve :

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
* [Faire de la réalité augmentée avec QGis et Openspace3D](https://youtu.be/SnPn1oB8DcQ)