---
title: "Python avec PostgreSQL/PostGIS"
permalink: /fr/python-postgresql-postgis/
lang: fr
date: 2025-04-25T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: database
---

> **_NOTE:_** La [documentation officielle de PostGIS](https://postgis.net/documentation/) fournit des instructions complètes sur son utilisation. Pour l'interaction avec Python, consultez également les ressources des bibliothèques [psycopg](https://www.psycopg.org/) et [GeoAlchemy](https://geoalchemy-2.readthedocs.io/en/latest/).

## Introduction à PostGIS et Python

**PostGIS** est une extension de PostgreSQL qui permet de gérer des données spatiales (géométrie et géographie). Combiné avec **Python**, il devient un outil puissant pour analyser, traiter et visualiser des données géospatiales.

Ce guide présente plusieurs méthodes pour interagir avec une base de données PostGIS en Python :
- Connexion directe avec `psycopg`
- Utilisation d'un ORM spatial avec `GeoAlchemy`
- Lecture des données dans `GeoPandas` via `read_postgis`

Avant de commencer :
- **Installer PostgreSQL et PostGIS**.
- **Créer une base de données PostGIS** ou utiliser une base existante.
- **Installer les bibliothèques Python nécessaires** (`psycopg`, `sqlalchemy`, `geopandas`, etc.).

## Approche 1 : Connexion directe avec `psycopg`

**`psycopg`** est la bibliothèque standard pour interagir avec PostgreSQL en Python.

### Exemple de connexion simple

```python
import psycopg

# Connexion à la base
conn = psycopg.connect(
    dbname="votre_base",
    user="votre_utilisateur",
    password="votre_motdepasse",
    host="localhost",
    port="5432"
)

# Création d'un curseur
cur = conn.cursor()

# Exécution d'une requête PostGIS
cur.execute("SELECT id, ST_AsText(geom) FROM votre_table;")

# Récupération des résultats
rows = cur.fetchall()
for row in rows:
    print(row)

# Fermeture de la connexion
cur.close()
conn.close()
```

> Conseil : utilisez `with psycopg.connect(...) as conn:` pour assurer la fermeture automatique de la connexion.

## Approche 2 : Utiliser un ORM spatial avec `GeoAlchemy`

**`GeoAlchemy 2`** étend SQLAlchemy pour supporter les types spatiaux.

### Exemple de modèle ORM

```python
from sqlalchemy import create_engine, Column, Integer
from sqlalchemy.ext.declarative import declarative_base
from geoalchemy2 import Geometry

# Connexion via SQLAlchemy
engine = create_engine("postgresql+psycopg://utilisateur:motdepasse@localhost:5432/votre_base")
Base = declarative_base()

# Définition du modèle
class Point(Base):
    __tablename__ = 'points'
    id = Column(Integer, primary_key=True)
    geom = Column(Geometry('POINT'))

# Création de la table (si nécessaire)
Base.metadata.create_all(engine)
```

> Conseil : Combinez `GeoAlchemy` avec `Shapely` pour manipuler des géométries en Python.

## Approche 3 : Lecture avec `GeoPandas`

**`GeoPandas`** facilite la manipulation de données géospatiales sous forme de DataFrame. Si vous n'avez pas besoin du volet spatial, **`Pandas`** sera suffisant et la syntaxe sera sensiblement la même.

### Lire une table PostGIS directement

```python
import geopandas as gpd
import psycopg

# Paramètres de connexion
conn_str = "postgresql+psycopg://utilisateur:motdepasse@localhost:5432/votre_base"

# Lecture avec GeoPandas
gdf = gpd.read_postgis(
    sql="SELECT * FROM votre_table;",
    con=conn_str,
    geom_col='geom'
)

print(gdf.head())
```

> Conseil : utilisez des filtres SQL (`WHERE`, `LIMIT`) pour éviter de charger des jeux de données trop volumineux.


## Bonnes pratiques

- **Utilisez des transactions** : lors de modifications importantes, travaillez dans des transactions pour assurer l'intégrité des données.
- **Créez des index spatiaux** : utilisez `CREATE INDEX ON table USING GIST(geom);` pour accélérer les requêtes spatiales.
- **Manipulez correctement les SRID** : soyez vigilant sur le système de coordonnées utilisé (`ST_Transform` pour changer le SRID si besoin).
- **Sécurisez vos connexions** : évitez de hardcoder vos mots de passe dans le code ; préférez l’utilisation de variables d'environnement.
- **Optimisez vos requêtes** : ne sélectionnez que les colonnes nécessaires et limitez les résultats si possible.

## Ressources utiles

- [Documentation officielle PostGIS](https://postgis.net/documentation/)
- [Psycopg 3 documentation](https://www.psycopg.org/psycopg3/docs/)
- [GeoAlchemy 2 documentation](https://geoalchemy-2.readthedocs.io/en/latest/)
- [GeoPandas documentation](https://geopandas.org/en/stable/)
