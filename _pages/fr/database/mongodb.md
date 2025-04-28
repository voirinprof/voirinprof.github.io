---
title: "Python avec MongoDB"
permalink: /fr/python-mongodb/
lang: fr
date: 2025-04-25T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: database
---

> **_NOTE:_** La [documentation officielle de MongoDB](https://www.mongodb.com/docs/manual/) ainsi que la bibliothèque Python [PyMongo](https://pymongo.readthedocs.io/) sont des ressources essentielles pour aller plus loin.

## Introduction à MongoDB

**MongoDB** est une base de données NoSQL orientée documents. Elle stocke les données au format **BSON** (JSON binaire), ce qui la rend idéale pour :
- Stocker des objets géographiques complexes
- Gérer des jeux de données non structurés ou semi-structurés
- Effectuer des recherches spatiales rapides via des **indexes géospatiaux**

En géomatique, MongoDB est utile pour :
- Stocker de grandes quantités de points, polygones, trajets
- Faire des recherches de proximité (ex : trouver tous les bus dans un rayon de 1 km)
- Travailler avec des données dynamiques (flux temps réel)

## Installation de PyMongo

Installation de `pymongo` en Python :

```shell
pip install pymongo
```

> Conseil : Utilisez un environnement virtuel (`venv`) pour isoler vos dépendances.

## Connexion à MongoDB en Python

```python
from pymongo import MongoClient

# Connexion au serveur MongoDB local
client = MongoClient('mongodb://localhost:27017/')

# Accès à la base de données et collection
db = client['geomatique']
collection = db['points_geospatiaux']
```

> MongoDB crée automatiquement la base et la collection si elles n'existent pas.

## Stocker des données géospatiales

MongoDB supporte nativement les types géospatiaux avec le format **GeoJSON**.

### Exemple d'insertion d'un point

```python
document = {
    "nom": "Station météo",
    "localisation": {
        "type": "Point",
        "coordinates": [2.2945, 48.8584]  # Longitude, Latitude
    }
}

collection.insert_one(document)
```

> Attention : MongoDB attend les coordonnées **(longitude, latitude)**, dans cet ordre !

## Créer un index géospatial

Pour pouvoir faire des requêtes spatiales efficaces, créez un index géospatial :

```python
collection.create_index([("localisation", "2dsphere")])
```

## Rechercher des objets proches

Exemple : trouver toutes les stations météo dans un rayon de 5 km autour d'un point :

```python
centre = [2.2945, 48.8584]  # Longitude, Latitude (ex: Tour Eiffel)

resultats = collection.find({
    "localisation": {
        "$near": {
            "$geometry": {
                "type": "Point",
                "coordinates": centre
            },
            "$maxDistance": 5000  # Distance en mètres
        }
    }
})

for station in resultats:
    print(station)
```

> `$near` effectue une recherche optimisée à l'aide de l'index 2dsphere.

## Cas d'usage en géomatique

### 1. Stockage de trajets GPS

Un document MongoDB peut contenir toute une trajectoire :

```python
trajet = {
    "vehicule_id": "bus_42",
    "trajet": {
        "type": "LineString",
        "coordinates": [
            [2.2945, 48.8584],
            [2.2950, 48.8585],
            [2.2960, 48.8586]
        ]
    }
}

collection.insert_one(trajet)
```

### 2. Requêter les objets contenus dans une zone (polygone)

```python
zone = {
    "type": "Polygon",
    "coordinates": [[
        [2.2930, 48.8570],
        [2.2960, 48.8570],
        [2.2960, 48.8600],
        [2.2930, 48.8600],
        [2.2930, 48.8570]
    ]]
}

resultats = collection.find({
    "localisation": {
        "$geoWithin": {
            "$geometry": zone
        }
    }
})

for obj in resultats:
    print(obj)
```

> `$geoWithin` permet de trouver tous les objets contenus dans un polygone.


### 3. Associer MongoDB et flask

MongoDB peut être utiliser comme base de données dans une application web. Habituellement on utilise PostGIS dans une application web, toutefois MongoDB peut remplir certains aspects de l'application. Cette base de données peut être dédiée aux requêtes les plus fréquentes ou même exigentes au niveau du système.

En couplant flask et MongoDB, on peut proposer un service web qui repose sur une base de données NoSQL.

```python
from flask import Flask, jsonify, request
from pymongo import MongoClient
from bson import json_util
import os
import json

app = Flask(__name__)

client = MongoClient(os.environ.get('MONGO_URI'))
db = client.spatial_db

@app.route('/locations', methods=['GET'])
def get_locations():
    locations = list(db.locations.find())
    return jsonify(json.loads(json_util.dumps(locations)))

@app.route('/locations/near', methods=['GET'])
def get_nearby_locations():
    try:
        lon = float(request.args.get('lon'))
        lat = float(request.args.get('lat'))
        max_distance = float(request.args.get('max_distance', 1000))  # meters
        
        locations = list(db.locations.find({
            "location": {
                "$near": {
                    "$geometry": {
                        "type": "Point",
                        "coordinates": [lon, lat]
                    },
                    "$maxDistance": max_distance
                }
            }
        }))
        return jsonify(json.loads(json_util.dumps(locations)))
    except Exception as e:
        return jsonify({"error": str(e)}), 400

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

Les requêtes effectuées sur la base de données MongoDB se traduisent comme des filtres JSON.


> Voir l'exemple de service web flask combiné avec MongoDB [https://github.com/voirinprof/mongodb_docker](https://github.com/voirinprof/mongodb_docker)

## Bonnes pratiques

- **Toujours indexer vos champs géospatiaux** pour des performances optimales.
- **Respectez le format GeoJSON** : type + coordinates.
- **Utilisez des filtres supplémentaires** pour réduire le nombre de documents explorés (`vehicule_id`, `date`, etc.).
- **Pensez à la validation des schémas** (ex : MongoDB permet d'imposer des contraintes pour les champs attendus).

## Ressources utiles

- [Documentation MongoDB sur les données géospatiales](https://www.mongodb.com/docs/manual/geospatial-queries/)
- [PyMongo documentation](https://pymongo.readthedocs.io/)
- [GeoJSON format officiel](https://geojson.org/)
