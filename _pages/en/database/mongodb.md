---
title: "Python with MongoDB"
permalink: /en/python-mongodb/
lang: en
date: 2025-04-25T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: database
---

> **_NOTE:_** The [official MongoDB documentation](https://www.mongodb.com/docs/manual/) and the [PyMongo Python library](https://pymongo.readthedocs.io/) are essential resources for diving deeper. You can use this GitHub Codespaces to practice with MongoDB [https://github.com/voirinprof/gis_starter_mongodb_geolab](https://github.com/voirinprof/gis_starter_mongodb_geolab). For setting up a Codespaces environment, see [Getting Started with GitHub Codespaces](/en/codespaces-intro/).

## Introduction to MongoDB

**MongoDB** is a NoSQL document-oriented database. It stores data in **BSON** format (binary JSON), making it ideal for:
- Storing complex geographic objects
- Managing unstructured or semi-structured datasets
- Performing fast spatial queries via **geospatial indexes**

In geomatics, MongoDB is useful for:
- Storing large amounts of points, polygons, paths
- Performing proximity searches (e.g., finding all buses within a 1 km radius)
- Working with dynamic data (real-time streams)

## Installing PyMongo

Install `pymongo` in Python:

```shell
pip install pymongo
```

> Tip: Use a virtual environment (`venv`) to isolate your dependencies.

## Connecting to MongoDB in Python

```python
from pymongo import MongoClient

# Connect to local MongoDB server
client = MongoClient('mongodb://localhost:27017/')

# Access the database and collection
db = client['geomatics']
collection = db['geospatial_points']
```

> MongoDB automatically creates the database and collection if they do not exist.

## Storing Geospatial Data

MongoDB natively supports geospatial types in **GeoJSON** format.

### Example of inserting a point

```python
document = {
    "name": "Weather Station",
    "location": {
        "type": "Point",
        "coordinates": [2.2945, 48.8584]  # Longitude, Latitude
    }
}

collection.insert_one(document)
```

> Note: MongoDB expects coordinates in **(longitude, latitude)** order!

## Creating a Geospatial Index

To perform efficient spatial queries, create a geospatial index:

```python
collection.create_index([("location", "2dsphere")])
```

## Searching for Nearby Objects

Example: Find all weather stations within a 5 km radius of a point:

```python
center = [2.2945, 48.8584]  # Longitude, Latitude (e.g., Eiffel Tower)

results = collection.find({
    "location": {
        "$near": {
            "$geometry": {
                "type": "Point",
                "coordinates": center
            },
            "$maxDistance": 5000  # Distance in meters
        }
    }
})

for station in results:
    print(station)
```

> `$near` performs an optimized search using the 2dsphere index.

## Use Cases in Geomatics

### 1. Storing GPS Paths

A MongoDB document can contain an entire trajectory:

```python
trip = {
    "vehicle_id": "bus_42",
    "path": {
        "type": "LineString",
        "coordinates": [
            [2.2945, 48.8584],
            [2.2950, 48.8585],
            [2.2960, 48.8586]
        ]
    }
}

collection.insert_one(trip)
```

### 2. Querying Objects Inside a Polygon (Area)

```python
area = {
    "type": "Polygon",
    "coordinates": [[
        [2.2930, 48.8570],
        [2.2960, 48.8570],
        [2.2960, 48.8600],
        [2.2930, 48.8600],
        [2.2930, 48.8570]
    ]]
}

results = collection.find({
    "location": {
        "$geoWithin": {
            "$geometry": area
        }
    }
})

for obj in results:
    print(obj)
```

> `$geoWithin` allows you to find all objects within a polygon.

### 3. Integrating MongoDB with Flask

MongoDB can be used as a database in a web application. While PostGIS is typically used in web applications, MongoDB can fulfill some aspects of the application. This database can be dedicated to the most frequent or system-demanding queries.

By combining Flask and MongoDB, we can offer a web service that relies on a NoSQL database.

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

Queries performed on the MongoDB database translate to JSON filters.

> See the Flask web service example combined with MongoDB [https://github.com/voirinprof/mongodb_docker](https://github.com/voirinprof/mongodb_docker)

## Best Practices

- **Always index your geospatial fields** for optimal performance.
- **Follow the GeoJSON format**: type + coordinates.
- **Use additional filters** to reduce the number of documents explored (`vehicle_id`, `date`, etc.).
- **Consider schema validation** (e.g., MongoDB allows enforcing constraints for expected fields).

## Useful Resources

- [MongoDB Documentation on Geospatial Data](https://www.mongodb.com/docs/manual/geospatial-queries/)
- [PyMongo documentation](https://pymongo.readthedocs.io/)
- [Official GeoJSON format](https://geojson.org/)