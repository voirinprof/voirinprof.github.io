---
title: "Python avec PostGIS"
permalink: /fr/python-postgis/
lang: fr
date: 2025-04-26T15:30:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: database
---

> **_NOTE:_** La [documentation officielle de PostGIS](https://postgis.net/documentation/) ainsi que la bibliothèque Python [psycopg](https://www.psycopg.org/) sont des ressources essentielles pour aller plus loin. Il faut parfois utiliser [psycopg2](https://github.com/psycopg/psycopg2) avec Python 3. Vous avez la possibilité d'utiliser ce GitHub Codespaces pour pratiquer les requêtes [https://github.com/voirinprof/gis_starter_postgis_geolab](https://github.com/voirinprof/gis_starter_postgis_geolab). **pgAdmin4** est disponible dans l'environnement. Pour mettre en place un environnement Codespaces allez voir [Premiers pas avec GitHub Codespaces](/fr/codespaces-intro/).

## Introduction à PostGIS

**PostGIS** est une extension spatiale pour la base de données relationnelle **PostgreSQL**.  
Elle permet de :
- Stocker des géométries (points, lignes, polygones) au format SQL standard
- Faire des calculs spatiaux complexes (distances, intersections, buffers)
- Réaliser des requêtes spatiales optimisées grâce aux index

En géomatique, PostGIS est utilisé pour :
- Gérer de grands volumes de données géospatiales (cadastres, réseaux, capteurs)
- Effectuer des analyses spatiales avancées (zonage, accessibilité)
- Servir de backend pour des applications SIG et web cartographiques

## Installation de PostGIS et bibliothèques Python

Installez PostgreSQL et ajoutez l’extension PostGIS.

Côté Python, installez :

```shell
pip install psycopg[binary] geoalchemy2 sqlalchemy
```

- `psycopg` : connexion PostgreSQL
- `SQLAlchemy` + `GeoAlchemy2` : ORM pour les objets spatiaux

> Conseil : Utilisez `Docker` pour lancer rapidement une instance PostgreSQL + PostGIS en local.

## Connexion à PostGIS en Python

### Connexion simple avec `psycopg`

```python
import psycopg

conn = psycopg.connect("dbname=geomatique user=postgres password=motdepasse host=localhost")
cur = conn.cursor()
```

### Connexion avancée avec SQLAlchemy

```python
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg://postgres:motdepasse@localhost/geomatique")
```

## Créer une table spatiale

Exemple : stocker des points d'intérêt géographiques.

```sql
CREATE TABLE points_interet (
    id SERIAL PRIMARY KEY,
    nom TEXT,
    geom GEOMETRY(Point, 4326)
);
```

> `4326` est le code EPSG pour WGS84 (latitude/longitude standard GPS).

## Insérer des données spatiales en Python

```python
cur.execute("""
    INSERT INTO points_interet (nom, geom)
    VALUES (%s, ST_SetSRID(ST_MakePoint(%s, %s), 4326))
""", ("Tour Eiffel", 2.2945, 48.8584))

conn.commit()
```

> `ST_SetSRID` définit le système de référence spatiale (ici WGS84).

## Effectuer des requêtes spatiales

### Trouver des points proches d'un lieu

Exemple : tous les points à moins de 500 mètres de la Tour Eiffel.

```python
cur.execute("""
    SELECT nom
    FROM points_interet
    WHERE ST_DWithin(
        geom::geography,
        ST_SetSRID(ST_MakePoint(%s, %s), 4326)::geography,
        %s
    )
""", (2.2945, 48.8584, 500))

resultats = cur.fetchall()

for row in resultats:
    print(row[0])
```

> `ST_DWithin` permet de filtrer selon une distance en **mètres** si vous convertissez en `geography`.

## Cas d'usage en géomatique

### 1. Stockage de trajets ou de réseaux

Vous pouvez utiliser le type `LINESTRING` pour des routes ou des trajets GPS :

```sql
CREATE TABLE trajets (
    id SERIAL PRIMARY KEY,
    vehicule_id TEXT,
    geom GEOMETRY(LineString, 4326)
);
```

Insertion Python :

```python
trajet_coords = [(2.2945, 48.8584), (2.2950, 48.8585), (2.2960, 48.8586)]

linestring = "LINESTRING({})".format(
    ", ".join(f"{lon} {lat}" for lon, lat in trajet_coords)
)

cur.execute("""
    INSERT INTO trajets (vehicule_id, geom)
    VALUES (%s, ST_SetSRID(ST_GeomFromText(%s), 4326))
""", ("bus_42", linestring))

conn.commit()
```

### 2. Analyse d'intersections ou de recoupements

Trouver tous les trajets qui croisent une zone donnée :

```python
zone = "POLYGON((2.293 48.857, 2.296 48.857, 2.296 48.860, 2.293 48.860, 2.293 48.857))"

cur.execute("""
    SELECT vehicule_id
    FROM trajets
    WHERE ST_Intersects(
        geom,
        ST_SetSRID(ST_GeomFromText(%s), 4326)
    )
""", (zone,))

resultats = cur.fetchall()

for row in resultats:
    print(row[0])
```

## Bonnes pratiques

- **Créez des index spatiaux** pour accélérer vos requêtes :
  ```sql
  CREATE INDEX idx_points_geom ON points_interet USING GIST (geom);
  ```
- **Utilisez `geography`** si vous travaillez sur de grandes distances terrestres (calculs précis sur une sphère).
- **Toujours préciser le SRID** (code EPSG) pour éviter des erreurs de référence spatiale.
- **Séparez les rôles** : utilisez des utilisateurs Postgres avec des droits limités en production.

## Ressources utiles

- [Documentation officielle PostGIS](https://postgis.net/documentation/)
- [Guide pratique de PostGIS](https://postgis.net/workshops/postgis-intro/)
- [Psycopg documentation](https://www.psycopg.org/psycopg3/docs/)
- [GeoAlchemy2 pour SQLAlchemy](https://geoalchemy-2.readthedocs.io/en/latest/)