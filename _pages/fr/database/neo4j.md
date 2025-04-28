---
title: "Python avec Neo4j"
permalink: /fr/python-neo4j/
lang: fr
date: 2025-04-25T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: database
---

> **_NOTE:_** La [documentation officielle de Neo4j](https://neo4j.com/docs/) ainsi que la bibliothèque Python [neo4j-driver](https://neo4j.com/docs/api/python-driver/current/) offrent des ressources détaillées pour explorer davantage.

## Introduction à Neo4j

**Neo4j** est une base de données orientée **graphes**. Elle stocke des données sous forme de **nœuds** (entités) et **relations** (liens), ce qui est parfait pour :
- Modéliser des réseaux (routes, rivières, infrastructures)
- Analyser les proximités et chemins optimaux
- Représenter des flux d'information ou de mobilité spatiaux

En géomatique, Neo4j peut être utilisé pour :
- Analyser les réseaux de transport (bus, trains, vélos)
- Gérer des graphes d'écoulement (ex. réseaux hydrographiques)
- Optimiser des trajets et explorer des connexions spatiales

## Installation du driver Python

Installez Neo4j Desktop ou utilisez Neo4j Aura (service cloud).

Installation de la bibliothèque Python `neo4j` :

```shell
pip install neo4j
```

## Connexion à Neo4j en Python

```python
from neo4j import GraphDatabase

# Connexion au serveur local Neo4j
uri = "bolt://localhost:7687"
driver = GraphDatabase.driver(uri, auth=("neo4j", "votre_mot_de_passe"))

# Fonction simple pour tester la connexion
def tester_connexion(tx):
    result = tx.run("RETURN 'Connexion réussie' AS message")
    for record in result:
        print(record["message"])

with driver.session() as session:
    session.read_transaction(tester_connexion)
```

> Conseil : Protégez votre mot de passe avec des variables d'environnement.

## Modéliser des entités géospatiales

Neo4j permet de stocker des coordonnées géographiques directement dans les propriétés des nœuds.

### Exemple : ajouter des villes avec leur position

```python
def ajouter_ville(tx, nom, lon, lat):
    tx.run(
        "CREATE (v:Ville {nom: $nom, localisation: point({longitude: $lon, latitude: $lat})})",
        nom=nom, lon=lon, lat=lat
    )

with driver.session() as session:
    session.write_transaction(ajouter_ville, "Paris", 2.3522, 48.8566)
    session.write_transaction(ajouter_ville, "Lyon", 4.8357, 45.7640)
```

## Créer des liens spatiaux entre objets

Vous pouvez relier deux villes par une relation "RELIE":

```python
def relier_villes(tx, ville1, ville2):
    tx.run(
        """
        MATCH (a:Ville {nom: $ville1}), (b:Ville {nom: $ville2})
        CREATE (a)-[:RELIE]->(b)
        """,
        ville1=ville1, ville2=ville2
    )

with driver.session() as session:
    session.write_transaction(relier_villes, "Paris", "Lyon")
```

> En géomatique, cela peut modéliser un tronçon routier, ferroviaire, ou un lien hydrologique.

## Rechercher des entités proches

Neo4j permet de faire des requêtes spatiales sur les propriétés `point`.

Exemple : trouver toutes les villes dans un rayon de 300 km autour de Paris :

```python
def villes_proches(tx, lon, lat, distance):
    result = tx.run(
        """
        MATCH (v:Ville)
        WHERE distance(v.localisation, point({longitude: $lon, latitude: $lat})) < $distance
        RETURN v.nom AS nom
        """,
        lon=lon, lat=lat, distance=distance
    )
    return [record["nom"] for record in result]

with driver.session() as session:
    villes = session.read_transaction(villes_proches, 2.3522, 48.8566, 300000)
    print(villes)
```

> `distance()` retourne une distance en **mètres**.

## Cas d'usage en géomatique

### 1. Optimisation de réseaux de transport

- Modéliser les arrêts comme des nœuds.
- Relier les arrêts voisins par des relations avec une distance ou un temps.
- Utiliser des algorithmes de plus court chemin (`shortestPath`) pour trouver l'itinéraire optimal.

### 2. Analyse d'accessibilité spatiale

- Définir des zones d'influence autour de certains points.
- Explorer les relations entre lieux en fonction de distances spatiales.

### 3. Représentation de réseaux naturels

- Rivières, flux d'eau ou réseaux de drainage comme graphes orientés.

### 4. Service web flask avec Neo4j

En associant flask et Neo4j, on peut mettre en place un service web qui tire profit de la base de données de type graphe.
Neo4j est très utile quand on dispose d'un réseau, avec des noeuds et des liens.

On peut ajouter des noeuds dans le graphe.

```python
@app.route('/locations/add', methods=['POST'])
def add_location():
    data = request.get_json()
    name = data.get('name')
    lon = float(data.get('longitude'))
    lat = float(data.get('latitude'))
    
    if not all([name, lon, lat]):
        return jsonify({"error": "Missing name, longitude, or latitude"}), 400
    
    with driver.session() as session:
        session.run(
            "CREATE (p:Place {name: $name, longitude: $lon, latitude: $lat})",
            name=name, lon=lon, lat=lat
        )
    return jsonify({"message": f"Added {name}"}), 201
```

> Voir l'exemple de service web flask combiné avec Neo4j [https://github.com/voirinprof/neo4j_docker/](https://github.com/voirinprof/neo4j_docker/)

## Bonnes pratiques

- **Normalisez vos unités** : longitude/latitude en WGS84, distances en mètres.
- **Utilisez des types de relations explicites** (`RELIE`, `FLUX_VERS`, etc.).
- **Indexez vos propriétés** si vous avez beaucoup de nœuds (`CREATE INDEX ON :Ville(nom)`).
- **Utilisez les capacités spatiales natives** (fonction `point()`, `distance()`, etc.) pour optimiser vos recherches.

## Ressources utiles

- [Documentation Neo4j Spatial Data Types](https://neo4j.com/docs/cypher-manual/current/functions/spatial/)
- [Python Driver pour Neo4j](https://neo4j.com/docs/api/python-driver/current/)
- [Cypher Query Language](https://neo4j.com/docs/cypher-manual/current/)