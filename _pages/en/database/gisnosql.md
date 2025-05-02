---
title: "Introduction to NoSQL Databases with Redis, MongoDB, and Neo4j"
permalink: /en/nosql-basics/
lang: en
date: 2025-04-26T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: database
---

> **_NOTE:_** You have the option to use this GitHub Codespaces to practice with Redis [https://github.com/voirinprof/gis_starter_redis_geolab](https://github.com/voirinprof/gis_starter_redis_geolab), MongoDB [https://github.com/voirinprof/gis_starter_mongodb_geolab](https://github.com/voirinprof/gis_starter_mongodb_geolab), or Neo4j [https://github.com/voirinprof/gis_starter_neo4j_geolab](https://github.com/voirinprof/gis_starter_neo4j_geolab). To set up a Codespaces environment, see [Getting Started with GitHub Codespaces](/en/codespaces-intro/).

## Introduction to NoSQL Databases

**NoSQL** (Not Only SQL) databases are designed to handle unstructured, semi-structured, or structured data. Unlike relational databases like PostgreSQL, they do not use rigid schemas and focus on performance for large data volumes or varied structures. This guide introduces three popular NoSQL databases: **Redis**, **MongoDB**, and **Neo4j**, with a focus on their use for **spatial data**.

## Overview of Redis, MongoDB, and Neo4j

- **Redis**: An in-memory key-value store optimized for speed. It's ideal for caching, user sessions, or simple spatial indexes through geospatial commands.
- **MongoDB**: A document-oriented database that stores data in JSON/BSON format. It supports geospatial indexes for proximity or intersection queries.
- **Neo4j**: A graph database designed to model complex relationships, including spatial networks (e.g., roads or infrastructure). Data is stored as nodes and edges.

## Redis

### Configuration

Redis is lightweight and fast. To get started, install Redis locally or use a cloud service like Redis Labs.

Command to start a local Redis server (after installation):
```bash
redis-server
```

Connect via the CLI client:
```bash
redis-cli
```

> Tip: Make sure Redis is installed (`redis-server --version`) before launching the server.

### Creating and Inserting Data

Redis stores key-value pairs and supports geospatial data through commands like `GEOADD`. Example of inserting a point of interest (e.g., the Eiffel Tower):

```bash
GEOADD places 2.2945 48.8584 "Eiffel Tower"
```

- `GEOADD`: Adds a position (longitude, latitude, name) to a geospatial set.

Example with multiple points:
```bash
GEOADD places 4.8357 45.7640 "Lyon" 5.3698 43.2965 "Marseille"
```

> Tip: Redis geospatial keys are ideal for fast proximity searches.

### Basic and Spatial Queries

Retrieve a key:
```bash
GET user:123
```

Get all items in a list:
```bash
LRANGE tasks 0 -1
```

Delete a key:
```bash
DEL user:123
```

Find places within 100 km of the Eiffel Tower:
```bash
GEORADIUS places 2.2945 48.8584 100 km
```

Calculate the distance between two places:
```bash
GEODIST places "Eiffel Tower" "Lyon" km
```

> Tip: `GEORADIUS` and `GEORADIUSBYMEMBER` are fast but limited to simple spherical calculations.

### Performance Optimization

- **Use appropriate structures**: Geospatial sets (`GEOADD`) are optimized for proximity searches.
- **Set expirations**: For temporary data (e.g., geospatial cache), use `EXPIRE`:
```bash
SETEX cache:places:paris "data" 3600  # Expires after 1 hour
```

## MongoDB

### Configuration

MongoDB can be installed locally or used via MongoDB Atlas (cloud). To install locally, download MongoDB Community Server.

Start the MongoDB server:
```bash
mongod
```

Connect via the MongoDB Shell client:
```bash
mongo
```

Create a database:
```javascript
use my_database
```

> Tip: MongoDB automatically creates the database on the first insert if it doesn't exist.

### Creating and Inserting Data

MongoDB organizes data into collections of documents and supports geospatial coordinates via the `GeoJSON` type. Example of inserting into a `points_of_interest` collection:

```javascript
db.points_of_interest.insertOne({
    name: "Eiffel Tower",
    location: {
        type: "Point",
        coordinates: [2.2945, 48.8584]  // [longitude, latitude]
    }
})
```

Insert multiple points:
```javascript
db.points_of_interest.insertMany([
    { name: "Lyon", location: { type: "Point", coordinates: [4.8357, 45.7640] } },
    { name: "Marseille", location: { type: "Point", coordinates: [5.3698, 43.2965] } }
])
```

> Tip: Ensure the `location` field follows the GeoJSON format for spatial queries.

### Basic and Spatial Queries

Select all documents from a collection:
```javascript
db.points_of_interest.find().pretty()
```

Filter by criteria:
```javascript
db.points_of_interest.find({ name: "Eiffel Tower" }).pretty()
```

Update a document:
```javascript
db.points_of_interest.updateOne(
    { name: "Eiffel Tower" },
    { $set: { name: "Eiffel Tower - Paris" } }
)
```

Delete a document:
```javascript
db.points_of_interest.deleteOne({ name: "Eiffel Tower" })
```

Create a geospatial index:
```javascript
db.points_of_interest.createIndex({ location: "2dsphere" })
```

Find points within 100 km of the Eiffel Tower:
```javascript
db.points_of_interest.find({
    location: {
        $near: {
            $geometry: { type: "Point", coordinates: [2.2945, 48.8584] },
            $maxDistance: 100000  // 100 km in meters
        }
    }
}).pretty()
```

Check if a point is within a polygon:
```javascript
db.points_of_interest.find({
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

> Tip: Use `$near` for proximity searches and `$geoWithin` for intersections.

### Performance Optimization

- **Create geospatial indexes**: The `2dsphere` index is crucial for `$near` and `$geoWithin` queries:
```javascript
db.points_of_interest.createIndex({ location: "2dsphere" })
```

- **Limit fields**: Only retrieve necessary fields to reduce load:
```javascript
db.points_of_interest.find({ name: "Eiffel Tower" }, { name: 1, location: 1, _id: 0 })
```

## Neo4j

### Configuration

Neo4j offers a desktop version (Neo4j Desktop) or server. Download Neo4j Community Edition to get started.

Start Neo4j (after installation):
1. Launch Neo4j Desktop or run `neo4j start` for a local server.
2. Access the web interface (default: `http://localhost:7474`).
3. Log in with default credentials (usually `neo4j/neo4j`) and change the password.

> Tip: Familiarize yourself with the Cypher language to query Neo4j, especially for spatial queries.

Useful video: [Introduction to NoSQL Databases](https://www.youtube.com/watch?v=l8ALv9q8i6Q)

### Creating and Inserting Data

Neo4j uses the **Cypher** language to create nodes and relationships. For spatial data, Neo4j supports the `Point` type (since version 3.4). Example of creating a place and relationship:

```cypher
CREATE (tour:PointOfInterest {name: "Eiffel Tower", location: point({longitude: 2.2945, latitude: 48.8584, srid: 4326})})
CREATE (lyon:PointOfInterest {name: "Lyon", location: point({longitude: 4.8357, latitude: 45.7640, srid: 4326})})
CREATE (tour)-[:NEAR]->(lyon)
```

- `point`: Spatial type for storing coordinates.
- `srid: 4326`: WGS84 reference system.

> Tip: Use relationships like `NEAR` to model spatial networks.

Useful video: [Getting Started with MongoDB (in English)](https://youtu.be/EE8ZTQxa0AM)

### Basic and Spatial Queries

Find all nodes with the `PointOfInterest` label:
```cypher
MATCH (p:PointOfInterest)
RETURN p
```

Find places connected to the Eiffel Tower:
```cypher
MATCH (tour:PointOfInterest {name: "Eiffel Tower"})-[:NEAR]->(place)
RETURN place.name
```

Delete a node and its relationships:
```cypher
MATCH (p:PointOfInterest {name: "Eiffel Tower"})
DETACH DELETE p
```

Find places within 100 km

 of the Eiffel Tower:
```cypher
MATCH (tour:PointOfInterest {name: "Eiffel Tower"})-[:NEAR]->(place)
WHERE distance(tour.location, place.location) < 100000  // 100 km in meters
RETURN place.name
```

### Performance Optimization

- **Use spatial indexes**: Create an index for faster spatial queries:
```cypher
CREATE INDEX FOR (p:PointOfInterest) ON (p.location)
```