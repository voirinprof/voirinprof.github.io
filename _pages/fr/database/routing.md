---
title: "Calcul d'itinéraires avec pgRouting"
permalink: /fr/python-routing/
lang: fr
date: 2025-04-26T15:30:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: database
---

> **_NOTE:_** Il existe plusieurs approches pour calculer un itinéraire à partir d'un réseau routier. On peut utiliser pgRouting de PostGIS, on peut utiliser une simple librairie networkx, ou encore utiliser Neo4j. Voir l'exemple de pgRouting avec un réseau routier personnalisé et une service web flask [https://github.com/voirinprof/gis_pgrouting_docker](https://github.com/voirinprof/gis_pgrouting_docker). Si vous voulez voir comment utiliser networkx pour faire un calcul d'itinéraires, utilisez ce repo : [https://github.com/voirinprof/gis_nx_docker](https://github.com/voirinprof/gis_nx_docker)

## Introduction à pgRouting

**pgRouting** est une extension de PostGIS qui permet d'ajouter des capacités de calcul d'itinéraires et d'analyse de réseaux dans une base de données PostgreSQL/PostGIS.

En géomatique, pgRouting permet notamment :
- Le calcul d’itinéraires les plus courts (plus rapide, plus court, multimodal)
- L’analyse de réseaux (accessibilité, graphes, itinéraires multiples)
- Le traitement de grands réseaux routiers directement dans la base de données

> **_NOTE:_** La [documentation officielle de pgRouting](https://pgrouting.org/documentation.html) est très utile pour explorer toutes les fonctionnalités disponibles.

## Installation de pgRouting

Sur un serveur avec PostgreSQL et PostGIS, installez pgRouting :

```shell
sudo apt install postgresql-15-pgrouting
```

Puis activez l'extension dans votre base :

```sql
CREATE EXTENSION pgrouting;
```

> Conseil : pgRouting fonctionne sur des **graphes** stockés sous forme de **tables d'arcs** (edges).

## Préparer un réseau routier

Votre réseau routier doit être structuré sous forme d’une table d’arêtes (edges) :

```sql
CREATE TABLE reseau_routier (
    id SERIAL PRIMARY KEY,
    source BIGINT,
    target BIGINT,
    geom GEOMETRY(LineString, 4326),
    cout DOUBLE PRECISION
);
```

- `source` : identifiant du nœud de départ
- `target` : identifiant du nœud d’arrivée
- `cout` : coût pour parcourir l'arête (distance, temps, etc.)

> Vous pouvez utiliser `pgr_createTopology` pour générer automatiquement `source` et `target` à partir de `geom`.

```sql
SELECT pgr_createTopology('reseau_routier', 0.0001, 'geom', 'id');
```

## Connexion à pgRouting en Python

Installez les dépendances :

```shell
pip install psycopg sqlalchemy shapely
```

Connexion simple :

```python
import psycopg

conn = psycopg.connect("dbname=geomatique user=postgres password=motdepasse host=localhost")
cur = conn.cursor()
```

## Calculer un chemin le plus court

Supposons que vous connaissez les `source_id` et `target_id` dans votre réseau :

```python
depart = 1001
arrivee = 1050

cur.execute("""
    SELECT seq, id1 AS node, id2 AS edge, cost
    FROM pgr_dijkstra(
        'SELECT id, source, target, cout AS cost FROM reseau_routier',
        %s, %s,
        directed := true
    )
""", (depart, arrivee))

chemin = cur.fetchall()

for ligne in chemin:
    print(f"Étape {ligne[0]}: passer par le noeud {ligne[1]} via l'arc {ligne[2]} (coût {ligne[3]})")
```

> `pgr_dijkstra` est l'algorithme classique du plus court chemin.

## Autres fonctionnalités pgRouting utiles

- **Itinéraire vers plusieurs destinations** :  
  `pgr_dijkstraVia`
- **Calcul de matrices de coûts entre plusieurs points** :  
  `pgr_dijkstraCostMatrix`
- **Recherche isochrone (zones atteignables en distance ou temps)** :  
  `pgr_drivingDistance`

Exemple de distance de conduite :

```sql
SELECT * FROM pgr_drivingDistance(
    'SELECT id, source, target, cout FROM reseau_routier',
    1001,
    2000  -- distance maximum
);
```

## Bonnes pratiques

- **Créez des index sur `source` et `target`** pour accélérer les calculs :
  ```sql
  CREATE INDEX ON reseau_routier(source);
  CREATE INDEX ON reseau_routier(target);
  ```
- **Optimisez votre réseau** : limitez les arcs inutiles, vérifiez la topologie.
- **Utilisez un système de projection métrique** (ex : EPSG:2154 pour la France) pour des distances précises.
- **Séparez les réseaux dirigés et non-dirigés** selon vos besoins.

## Cas d'usage en géomatique

- **Calcul d’itinéraires de véhicules** (livraisons, bus, services d'urgence)
- **Analyse de l’accessibilité** (distance aux hôpitaux, écoles)
- **Construction de matrices Origine-Destination pour les flux de mobilité**

## Si on rend le problème plus complexe

Si votre problème nécessite de gérer plusieurs véhicules, plusieurs livraisons, une capacité pour chaque véhicules et des quantités différentes à livrer pour chaque client, on se retrouve dans un problème un peu plus complexe (avec de nombreuses conrtaintes).

On peut utiliser `or-tools`pour aborder ce genre d'optimisation. Voici un exemple d'application totalement fonctionnelle : [https://github.com/voirinprof/gis_advanced_routing_docker](https://github.com/voirinprof/gis_advanced_routing_docker)

## Ressources utiles

- [Documentation officielle pgRouting](https://pgrouting.org/documentation.html)
- [Exemples pratiques pgRouting](https://docs.pgrouting.org/3.4/en/pgr_dijkstra.html)
- [pgRouting Workshop (PostGIS & pgRouting)](https://workshop.pgrouting.org/)
