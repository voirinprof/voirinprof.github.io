---
title: "Notions de base en PostgreSQL/PostGIS"
permalink: /fr/postgresql-postgis-basics/
lang: fr
date: 2025-04-25T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: database
---

> **_NOTE:_** Vous avez la possibilité d'utiliser ce GitHub Codespaces pour pratiquer les requêtes [https://github.com/voirinprof/gis_starter_postgis_geolab](https://github.com/voirinprof/gis_starter_postgis_geolab). **pgAdmin4** est disponible dans l'environnement. Pour mettre en place un environnement Codespaces allez voir [Premiers pas avec GitHub Codespaces](/fr/codespaces-intro/).

## Introduction à PostgreSQL et PostGIS

PostgreSQL est un système de gestion de bases de données relationnelles (SGBDR) qui dispose d'une extension PostGIS pour gérer des données géographiques et spatiales, permettant de stocker, interroger et analyser des géométries comme des points, lignes et polygones.

## Création et gestion des bases de données

Une base de données PostgreSQL est un conteneur pour les tables, schémas et autres objets. Pour créer une base de données, utilisez la commande suivante dans l'interface `psql` ou un client SQL :

```sql
CREATE DATABASE ma_base_de_donnees;
```

Pour activer PostGIS sur une base de données, connectez-vous à la base et exécutez :

```sql
CREATE EXTENSION postgis;
```

> Conseil : Vérifiez que PostGIS est correctement installé sur votre serveur PostgreSQL avant d'exécuter cette commande.

Pour lister toutes les bases de données :

```sql
\l
```

Pour se connecter à une base spécifique :

```sql
\c ma_base_de_donnees
```

Vidéo utile : [Introduction à PostgreSQL (en anglais)](https://www.youtube.com/watch?v=74IWNUja05w)

## Création de tables avec des données spatiales

Les tables PostgreSQL stockent les données sous forme de lignes et de colonnes. Avec PostGIS, vous pouvez ajouter des colonnes de type géométrique pour stocker des données spatiales.

Exemple de création d'une table pour stocker des points d'intérêt (POI) avec une géométrie de type POINT :

```sql
CREATE TABLE points_interet (
    id SERIAL PRIMARY KEY,
    nom VARCHAR(100),
    geom GEOMETRY(POINT, 4326)
);
```

- `SERIAL` : Un type qui génère automatiquement des identifiants uniques.
- `GEOMETRY(POINT, 4326)` : Une colonne géométrique pour stocker des points avec le système de référence spatiale (SRID) 4326 (WGS84, utilisé pour les coordonnées GPS).

Pour insérer un point géographique (par exemple, la Tour Eiffel à Paris) :

```sql
INSERT INTO points_interet (nom, geom)
VALUES ('Tour Eiffel', ST_GeomFromText('POINT(2.2945 48.8584)', 4326));
```

> Conseil : Utilisez des noms de tables et de colonnes explicites pour faciliter la maintenance.

## Requêtes SQL de base

Les requêtes SQL permettent de manipuler et d'extraire des données. Voici quelques exemples fondamentaux :

### Sélectionner des données

Pour récupérer tous les enregistrements de la table `points_interet` :

```sql
SELECT id, nom, ST_AsText(geom) AS coordonnees
FROM points_interet;
```

`ST_AsText` convertit la géométrie en un format lisible (par exemple, `POINT(2.2945 48.8584)`).

### Filtrer des données

Pour trouver un point d'intérêt spécifique par son nom :

```sql
SELECT nom, ST_AsText(geom)
FROM points_interet
WHERE nom = 'Tour Eiffel';
```

### Mettre à jour des données

Pour modifier le nom d'un point d'intérêt :

```sql
UPDATE points_interet
SET nom = 'Tour Eiffel - Paris'
WHERE id = 1;
```

### Supprimer des données

Pour supprimer un point d'intérêt :

```sql
DELETE FROM points_interet
WHERE id = 1;
```

Vidéo utile : [Les bases des requêtes SQL avec PostgreSQL (en anglais)](https://youtu.be/7S_tz1z_5bA)

## Requêtes spatiales avec PostGIS

PostGIS permet d'effectuer des analyses spatiales avancées. Voici quelques exemples courants :

### Trouver des points à proximité

Pour trouver tous les points d'intérêt à moins de 1 km de la Tour Eiffel :

```sql
SELECT nom, ST_Distance(
    geom,
    ST_GeomFromText('POINT(2.2945 48.8584)', 4326)::geography
) AS distance_m
FROM points_interet
WHERE ST_DWithin(
    geom,
    ST_GeomFromText('POINT(2.2945 48.8584)', 4326)::geography,
    1000
);
```

- `ST_DWithin` : Vérifie si deux géométries sont à une distance donnée (en mètres avec `::geography`).
- `ST_Distance` : Calcule la distance entre deux géométries.

### Intersection de géométries

Pour vérifier si un point se trouve dans un polygone (par exemple, une zone administrative) :

```sql
SELECT nom
FROM points_interet
WHERE ST_Within(
    geom,
    ST_GeomFromText('POLYGON((2.2 48.8, 2.4 48.8, 2.4 48.9, 2.2 48.9, 2.2 48.8))', 4326)
);
```

> Conseil : Assurez-vous que toutes les géométries utilisent le même SRID pour éviter des erreurs.

Vidéo utile : [Introduction aux requêtes spatiales avec PostGIS](https://www.youtube.com/watch?v=yP5-GVnoy5I)

## Indexation pour les performances

Pour améliorer les performances des requêtes spatiales, créez un index spatial sur les colonnes géométriques :

```sql
CREATE INDEX idx_points_interet_geom ON points_interet USING GIST (geom);
```

- `GIST` : Un type d'index optimisé pour les données spatiales.

> Conseil : Les index sont particulièrement utiles pour les grandes tables ou les requêtes fréquentes.

## Bonnes pratiques

- **Utilisez des SRID cohérents** : Assurez-vous que toutes les géométries utilisent le même système de référence (par exemple, 4326 pour les coordonnées GPS).
- **Validez les géométries** : Utilisez `ST_IsValid` pour vérifier que les géométries sont correctes avant de les insérer.
- **Testez vos requêtes** : Préparez des jeux de données de test pour valider les résultats.
- **Optimisez les performances** : Utilisez des index spatiaux et limitez les colonnes dans les requêtes (`SELECT`) pour réduire la charge.

Exemple de validation de géométrie :

```sql
SELECT nom, ST_IsValid(geom) AS est_valide
FROM points_interet;
```

Vidéo utile : [PostgreSQL Install with PostGIS Spatial Extension - Connecting to PostgreSQL on QGIS](https://www.youtube.com/watch?v=ZaFJQg29ZQ4)