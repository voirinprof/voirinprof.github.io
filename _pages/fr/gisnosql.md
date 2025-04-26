---
title: "Introduction aux bases de données NoSQL avec Redis, MongoDB et Neo4j"
permalink: /fr/nosql-basics/
lang: fr
date: 2025-04-26T10:00:00+00:00
toc: true
toc_label: "Dans cette page"
typenav: database
---

## Introduction aux bases de données NoSQL

Les bases de données **NoSQL** (Not Only SQL) sont conçues pour gérer des données non structurées, semi-structurées ou structurées, avec une flexibilité et une scalabilité adaptées aux applications modernes. Contrairement aux bases relationnelles comme PostgreSQL, elles n’utilisent pas de schémas rigides et privilégient la performance pour des volumes de données importants ou des structures variées. Ce guide introduit trois bases NoSQL populaires : **Redis**, **MongoDB** et **Neo4j**, avec un focus sur leur utilisation pour des **données spatiales**.

Avant de commencer, il est essentiel de suivre ces étapes :
* **Comprendre les besoins** : Identifier le type de données (clés-valeurs, documents, graphes) et les cas d’usage (cache, stockage de documents, relations complexes, ou analyses spatiales).
* **Définir les entrées** : Quelles données seront stockées ? Par exemple, des coordonnées géographiques, des profils JSON ou des réseaux spatiaux.
* **Définir les sorties** : Quels résultats attendre ? Par exemple, une récupération rapide de points à proximité ou une analyse de relations géographiques.
* **Planifier les opérations** : Identifier les commandes ou requêtes nécessaires pour manipuler les données, y compris les requêtes spatiales.
* **Diviser les problèmes complexes** : Découper les tâches complexes en étapes simples, comme la configuration, l’insertion et l’interrogation.
* **Tester les résultats** : Vérifier les opérations avec des données de test pour garantir leur exactitude.

Cette approche garantit une utilisation efficace des bases NoSQL, en évitant les erreurs courantes et en optimisant les performances pour les applications spatiales.

## Présentation de Redis, MongoDB et Neo4j

- **Redis** : Une base de données en mémoire de type clé-valeur, optimisée pour la rapidité. Elle est idéale pour le caching, les sessions utilisateur, ou les index spatiaux simples grâce à ses commandes géospatiales.
- **MongoDB** : Une base de données orientée documents, stockant des données sous forme de documents JSON/BSON. Elle prend en charge les index géospatiaux pour les requêtes de proximité ou d’intersection.
- **Neo4j** : Une base de données orientée graphes, conçue pour modéliser des relations complexes, y compris des réseaux spatiaux (par exemple, routes ou infrastructures). Les données sont stockées sous forme de nœuds et d’arêtes.

## Configuration et gestion des bases de données

### Redis
Redis est léger et rapide. Pour démarrer, installez Redis localement ou utilisez un service cloud comme Redis Labs.

Commande pour démarrer un serveur Redis local (après installation) :
```bash
redis-server
```

Connexion via le client CLI :
```bash
redis-cli
```

> Conseil : Vérifiez que Redis est installé (`redis-server --version`) avant de lancer le serveur.

### MongoDB
MongoDB peut être installé localement ou utilisé via MongoDB Atlas (cloud). Pour une installation locale, téléchargez MongoDB Community Server.

Démarrer le serveur MongoDB :
```bash
mongod
```

Connexion via le client MongoDB Shell :
```bash
mongo
```

Créer une base de données :
```javascript
use ma_base_de_donnees
```

> Conseil : MongoDB crée automatiquement la base lors de la première insertion si elle n’existe pas.

### Neo4j
Neo4j propose une version desktop (Neo4j Desktop) ou serveur. Téléchargez Neo4j Community Edition pour commencer.

Démarrer Neo4j (après installation) :
1. Lancez Neo4j Desktop ou exécutez `neo4j start` pour un serveur local.
2. Accédez à l’interface web (par défaut : `http://localhost:7474`).
3. Connectez-vous avec les identifiants par défaut (souvent `neo4j/neo4j`) et changez le mot de passe.

> Conseil : Familiarisez-vous avec le langage Cypher pour interroger Neo4j, en particulier pour les requêtes spatiales.

Vidéo utile : [Introduction aux bases de données NoSQL](https://youtu.be/xh4gy1lbL2k)

## Création et insertion de données

### Redis
Redis stocke des paires clé-valeur et prend en charge les données géospatiales via des commandes comme `GEOADD`. Exemple d’insertion d’un point d’intérêt (par exemple, la Tour Eiffel) :

```bash
GEOADD lieux 2.2945 48.8584 "Tour Eiffel"
```

- `GEOADD` : Ajoute une position (longitude, latitude, nom) à une clé de type ensemble géospatial.

Exemple avec plusieurs points :
```bash
GEOADD lieux 4.8357 45.7640 "Lyon" 5.3698 43.2965 "Marseille"
```

> Conseil : Les clés géospatiales Redis sont idéales pour des recherches rapides de proximité.

### MongoDB
MongoDB organise les données en collections de documents et prend en charge les coordonnées géospatiales via le type `GeoJSON`. Exemple d’insertion dans une collection `points_interet` :

```javascript
db.points_interet.insertOne({
    nom: "Tour Eiffel",
    location: {
        type: "Point",
        coordinates: [2.2945, 48.8584]  // [longitude, latitude]
    }
})
```

Insérer plusieurs points :
```javascript
db.points_interet.insertMany([
    { nom: "Lyon", location: { type: "Point", coordinates: [4.8357, 45.7640] } },
    { nom: "Marseille", location: { type: "Point", coordinates: [5.3698, 43.2965] } }
])
```

> Conseil : Assurez-vous que le champ `location` suit le format GeoJSON pour les requêtes spatiales.

### Neo4j
Neo4j utilise le langage **Cypher** pour créer des nœuds et des relations. Pour les données spatiales, Neo4j prend en charge le type `Point` (depuis la version 3.4). Exemple de création d’un lieu et d’une relation :

```cypher
CREATE (tour:PointInteret {nom: "Tour Eiffel", location: point({longitude: 2.2945, latitude: 48.8584, srid: 4326})})
CREATE (lyon:PointInteret {nom: "Lyon", location: point({longitude: 4.8357, latitude: 45.7640, srid: 4326})})
CREATE (tour)-[:PROCHE]->(lyon)
```

- `point` : Type spatial pour stocker des coordonnées.
- `srid: 4326` : Système de référence WGS84.

> Conseil : Utilisez des relations comme `PROCHE` pour modéliser des réseaux spatiaux.

Vidéo utile : [Premiers pas avec MongoDB](https://youtu.be/EE8ZTQxa0AM)

## Requêtes de base et spatiales

### Redis
#### Requêtes de base
Récupérer une clé :
```bash
GET utilisateur:123
```

Obtenir tous les éléments d’une liste :
```bash
LRANGE taches 0 -1
```

Supprimer une clé :
```bash
DEL utilisateur:123
```

#### Requêtes spatiales
Trouver des lieux dans un rayon de 100 km autour de la Tour Eiffel :
```bash
GEORADIUS lieux 2.2945 48.8584 100 km
```

Calculer la distance entre deux lieux :
```bash
GEODIST lieux "Tour Eiffel" "Lyon" km
```

> Conseil : Les commandes `GEORADIUS` et `GEORADIUSBYMEMBER` sont rapides mais limitées aux calculs sphériques simples.

### MongoDB
#### Requêtes de base
Sélectionner tous les documents d’une collection :
```javascript
db.points_interet.find().pretty()
```

Filtrer par critère :
```javascript
db.points_interet.find({ nom: "Tour Eiffel" }).pretty()
```

Mettre à jour un document :
```javascript
db.points_interet.updateOne(
    { nom: "Tour Eiffel" },
    { $set: { nom: "Tour Eiffel - Paris" } }
)
```

Supprimer un document :
```javascript
db.points_interet.deleteOne({ nom: "Tour Eiffel" })
```

#### Requêtes spatiales
Créer un index géospatial :
```javascript
db.points_interet.createIndex({ location: "2dsphere" })
```

Trouver les points dans un rayon de 100 km autour de la Tour Eiffel :
```javascript
db.points_interet.find({
    location: {
        $near: {
            $geometry: { type: "Point", coordinates: [2.2945, 48.8584] },
            $maxDistance: 100000  // 100 km en mètres
        }
    }
}).pretty()
```

Vérifier si un point est dans un polygone :
```javascript
db.points_interet.find({
    location: {
        $geoWithin: {
            $geometry: {
                type: "Polygon",
                coordinates: [[
                    [2.2, 48.8], [2.4, 48.8], [2.4, 48.9], [2.2, 48.9], [2.2, 48.8]
                ]]
            }
        }
    }
}).pretty()
```

> Conseil : Utilisez `$near` pour les recherches de proximité et `$geoWithin` pour les intersections.

### Neo4j
#### Requêtes de base
Trouver tous les nœuds avec l’étiquette `PointInteret` :
```cypher
MATCH (p:PointInteret)
RETURN p
```

Trouver les lieux reliés à la Tour Eiffel :
```cypher
MATCH (tour:PointInteret {nom: "Tour Eiffel"})-[:PROCHE]->(lieu)
RETURN lieu.nom
```

Supprimer un nœud et ses relations :
```cypher
MATCH (p:PointInteret {nom: "Tour Eiffel"})
DETACH DELETE p
```

#### Requêtes spatiales
Trouver les lieux dans un rayon de 100 km autour de la Tour Eiffel :
```cypher
WITH point({longitude: 2.2945, latitude: 48.8584, srid: 4326}) AS centre
MATCH (p:PointInteret)
WHERE distance(p.location, centre) < 100000  // 100 km en mètres
RETURN p.nom, distance(p.location, centre) / 1000 AS distance_km
```

Créer une relation basée sur la proximité (moins de 200 km) :
```cypher
MATCH (p1:PointInteret), (p2:PointInteret)
WHERE p1.nom < p2.nom AND distance(p1.location, p2.location) < 200000
CREATE (p1)-[:PROCHE {distance_km: distance(p1.location, p2.location) / 1000}]->(p2)
RETURN p1.nom, p2.nom
```

> Conseil : La fonction `distance` utilise des calculs sphériques pour les points WGS84.

Vidéo utile : [Introduction à Neo4j et Cypher](https://youtu.be/5jAa7UXdg4Y)

## Optimisation des performances

### Redis
- **Utilisez des structures adaptées** : Les ensembles géospatiaux (`GEOADD`) sont optimisés pour les recherches de proximité.
- **Définissez des expirations** : Pour les données temporaires (par exemple, cache géospatial), utilisez `EXPIRE` :
```bash
SETEX cache:lieux:paris "donnees" 3600  # Expire après 1 heure
```

### MongoDB
- **Créez des index géospatiaux** : L’index `2dsphere` est essentiel pour les requêtes `$near` et `$geoWithin` :
```javascript
db.points_interet.createIndex({ location: "2dsphere" })
```

- **Limitez les champs** : Récupérez uniquement les champs nécessaires pour réduire la charge :
```javascript
db.points_interet.find({ nom: "Tour Eiffel" }, { nom: 1, location: 1, _id: 0 })
```

### Neo4j
- **Indexez les propriétés clés** : Accélérez les recherches sur les propriétés comme `nom` :
```cypher
CREATE INDEX FOR (p:PointInteret) ON (p.nom)
```

- **Utilisez des index spatiaux** : Depuis Neo4j 4.0, les index spatiaux accélèrent les requêtes sur `point` :
```cypher
CREATE POINT INDEX FOR (p:PointInteret) ON (p.location)
```

- **Optimisez les motifs** : Limitez les nœuds parcourus dans les requêtes spatiales avec des conditions précises.

> Conseil : Testez les performances avec des volumes de données réalistes pour identifier les goulots d’étranglement.

## Bonnes pratiques

- **Choisissez le bon outil** : Redis pour des recherches géospatiales rapides (proximité), MongoDB pour des documents géospatiaux flexibles, et Neo4j pour des réseaux spatiaux complexes.
- **Validez les données** : Vérifiez les formats (GeoJSON pour MongoDB, `point` pour Neo4j) avant insertion.
- **Testez les requêtes spatiales** : Utilisez des coordonnées réelles (par exemple, villes françaises) pour valider les résultats.
- **Sauvegardez régulièrement** : Configurez des sauvegardes automatiques, surtout pour MongoDB et Neo4j.
- **Surveillez les performances** : Utilisez Redis Monitor, MongoDB Compass, ou Neo4j Browser pour analyser l’activité.

Exemple de contrainte dans Neo4j pour garantir l’unicité des noms :
```cypher
CREATE CONSTRAINT FOR (p:PointInteret) REQUIRE p.nom IS UNIQUE
```

Vidéo utile : [Redis pour les débutants](https://youtu.be/jgpVdJB2sKQ)