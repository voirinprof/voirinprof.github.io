---
title: "Routing avec Redis et Neo4j"
permalink: /fr/routing-redis-neo4j/
lang: fr
date: 2025-04-26T18:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: database
---

> **_NOTE:_** Ce projet combine **Redis** pour la recherche rapide d'adresses et **Neo4j** pour la recherche de chemins optimisés dans un graphe de réseau routier. Voir l'exemple de pgRouting avec un réseau routier personnalisé et une service web flask [https://github.com/voirinprof/gis_nosql_docker](https://github.com/voirinprof/gis_nosql_docker).

## Introduction

Dans une application géomatique moderne, il est courant de :
- **Chercher rapidement** une adresse à partir d'un libellé ou d'une coordonnée ➔ **Redis** (clé-valeur ultra rapide)
- **Calculer un chemin** entre deux adresses ➔ **Neo4j** (représentation orientée en graphe du réseau routier)

Ce guide montre comment utiliser Python pour connecter les deux bases et effectuer un **routing complet**.

## Architecture de l'application

- **Redis** stocke les adresses (avec ID associé)
- **Neo4j** stocke le graphe du réseau routier (nœuds = adresses / intersections, arcs = routes)
- **Python** fait le lien entre une recherche d'adresse rapide et un calcul de chemin

Schéma :

```
Recherche utilisateur ➔ Adresse via Redis ➔ Noeud de départ / arrivée ➔ Calcul du chemin dans Neo4j
```

## Installer les dépendances

```shell
pip install redis neo4j
```

## Préparer Redis pour les adresses

Insérez vos adresses dans Redis sous forme **(lon, lat, adresse)**.

Exemple en Python :

```python
# Initialize some example data on startup
def init_redis_data():
    data_qc_address = gpd.read_file(qc_addresses_file)

    logger.debug('Initializing Redis data with addresses')
    for index, rows in data_qc_address.iterrows():
        address = rows['ADRESSE']
        geometry = rows['geometry']
        coords = geometry.coords

        redis_client.geoadd("locations", (coords[0][0], coords[0][1], address))
        # Ajoutez également l'adresse à un ensemble trié pour l'index
        redis_client.zadd("index_locations", {address: 1})
```

## Préparer Neo4j pour le graphe routier

Le graphe va se construire en utilisant un réseau routier qui contient des segments.

Structure du graphe :

- Nœud (intersection) : `:Node {id: 123, ...}`
- Relation (route entre deux nœuds) : `[:Link {length: 150}]`

Exemple de création de graphe en Cypher :

```cypher
CREATE (a:Node {id:123, ...})
CREATE (b:Node {id:456, ...})
CREATE (a)-[:Link {distance: 250}]->(b)
```

> Conseil : créez aussi un index sur `:Node(id)` pour des recherches plus rapides :
```cypher
CREATE INDEX lieu_id IF NOT EXISTS FOR (n:Node) ON (n.id);
```

On peut aussi créer un graphe à partir de fichier csv, notamment un fichier pour les noeuds et un fichier pour les liens.
Ces fichiers csv peuvent être créés à partir de données spatiales.

Le fichier des noeuds doit être :

```csv
id:ID,latitude:FLOAT,longitude:FLOAT
0,45.4710109296785,-71.94436225521166
1,45.470409779469946,-71.94493520202225
...
```

Le fichier des liens doit être :

```csv
:START_ID,:END_ID,street,objectid:INT,length:FLOAT,speed:FLOAT
0,1,Rue Oliva-Turgeon,123399,114.77815102017024,50
1,0,Rue Oliva-Turgeon,123399,114.77815102017024,50
...
```

Par la suite, on peut créer le graphe du réseau routier avec l'instruction suivante :

```cypher
CALL apoc.import.csv(
    [{fileName: 'file:/nodes.csv', labels: ['Node']}],
    [{fileName: 'file:/streets.csv', type: 'Link'}],
    {delimiter: ',', stringIds: false}
)

CALL gds.graph.project(
    'myGraphRoad',
    'Node',
    'Link',
    {
        relationshipProperties: ['length', 'objectid']
    }
)
```
On pourra ensuite appliquer des méthodes sur le graphe `myGraphRoad`. Il est préférable d'utiliser `gds` (Graph Data Science) pour les nouveaux algorithmes de chemin.


## Application Python de recherche et routing

Connexion :

```python
from redis import Redis
from neo4j import GraphDatabase

# Connexions
redis_conn = Redis(host='localhost', port=6379, db=0)
neo4j_driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "motdepasse"))
```

Chercher une adresse avec Redis. Cette fonction permettrait d'obtenir une liste de suggestion d'adresses dans un formulaire.

```python
# function to search for addresses in Redis
def addressSearch(query):

    # Use the zscan command to iterate over the sorted set index
    matches = []
    for match in redis_client.zscan_iter("index_locations"):
        match_str = match[0]
        if match_str.startswith(query):
            matches.append(match_str)
    suggestions = []
    for match in matches:
        adresse = match
        coordonnees = redis_client.geopos("locations", adresse)
        if coordonnees:
            lat = coordonnees[0][1]
            lon = coordonnees[0][0]
            suggestions.append({"display_name": adresse, "lat": lat, "lon": lon})

    return suggestions
```

Connaissant la position de l'adresse, on pourrait ensuite trouver le noeud le plus proche dans le graphe Neo4j. En général, les points d'adresse ne sont pas sur les rues, donc on risque de tomber en dehors du graphe.

Avec `cypher`, on peut trouver le noeud le plus proche d'un point.

```python
# function to search for nodes in Neo4j
def nodeSearch(latitude, longitude):
    
    # use the point function to create a point and find the closest nodes
    requete_cypher = """
    MATCH (c:Node)
    WITH c, point({latitude: $lat, longitude: $lon}) AS refPoint
    WITH c, point.distance(point({latitude: c.latitude, longitude: c.longitude}), refPoint) AS distance
    WHERE distance <= $max_distance
    RETURN c.id, c.longitude, c.latitude, distance
    ORDER BY distance

    """
```
Une fois que l'on a un noeud de départ et un noeud d'arrivée, on peut calculer le chemin le plus court.

Calculer le chemin dans Neo4j :

```python
# we will find the path between the two nodes
requete_cypher_chemin = """
MATCH
(a:Node {id: """+str(first_node['id'])+"""}),
(b:Node {id: """+str(second_node['id'])+"""})
CALL gds.shortestPath.dijkstra.stream('myGraphRoad', {
    sourceNode: a,
    targetNode: b,
    relationshipWeightProperty: 'length'
})
YIELD index, sourceNode, targetNode, totalCost, nodeIds, costs, path
RETURN
index,
gds.util.asNode(sourceNode).id AS sourceNodeName,
gds.util.asNode(targetNode).id AS targetNodeName,
totalCost,
[nodeId IN nodeIds | gds.util.asNode(nodeId).id] AS nodeNames,
costs, nodes(path) as path

"""
```

## Bonnes pratiques

- **Normalisez les adresses** avant de les stocker dans Redis (sans majuscules, sans accents si possible).
- **Mettez en cache** les chemins fréquents pour éviter des recalculs inutiles.
- **Surveillez Redis** pour éviter qu’il ne sature la mémoire si le volume d’adresses est très grand (utilisez `EXPIRE` si nécessaire).
- **Optimisez les graphes Neo4j** : index, suppression des doublons, simplification des routes secondaires.

## Cas d'usage géomatiques

- **Calcul rapide d'itinéraires personnalisés** entre adresses dans une ville
- **Optimisation de tournées de livraison** en associant Redis (localisation rapide) et Neo4j (chemin optimal)
- **Recherche dynamique d'itinéraires alternatifs** en fonction de points d'intérêts (POI)

## Ressources utiles

- [Documentation Redis](https://redis.io/docs/)
- [Documentation Neo4j](https://neo4j.com/docs/)
- [Neo4j Graph Data Science Library](https://neo4j.com/docs/graph-data-science/current/)
