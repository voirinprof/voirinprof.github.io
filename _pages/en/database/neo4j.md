---
title: "Python with Neo4j"
permalink: /en/python-neo4j/
lang: en
date: 2025-04-25T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: database
---

> **_NOTE:_** The [official Neo4j documentation](https://neo4j.com/docs/) and the Python library [neo4j-driver](https://neo4j.com/docs/api/python-driver/current/) offer detailed resources to explore further. You can use this GitHub Codespaces to practice with Neo4j [https://github.com/voirinprof/gis_starter_neo4j_geolab](https://github.com/voirinprof/gis_starter_neo4j_geolab). For setting up a Codespaces environment, see [Getting Started with GitHub Codespaces](/en/codespaces-intro/).

## Introduction to Neo4j

**Neo4j** is a **graph-based** database. It stores data as **nodes** (entities) and **relationships** (links), which is perfect for:
- Modeling networks (roads, rivers, infrastructure)
- Analyzing proximity and optimal paths
- Representing spatial information or mobility flows

In geomatics, Neo4j can be used for:
- Analyzing transport networks (buses, trains, bicycles)
- Managing flow graphs (e.g., hydrographic networks)
- Optimizing routes and exploring spatial connections

## Installing the Python Driver

Install Neo4j Desktop or use Neo4j Aura (cloud service).

Installation of the Python `neo4j` library:

```shell
pip install neo4j
```

## Connecting to Neo4j in Python

```python
from neo4j import GraphDatabase

# Connect to local Neo4j server
uri = "bolt://localhost:7687"
driver = GraphDatabase.driver(uri, auth=("neo4j", "your_password"))

# Simple function to test the connection
def test_connection(tx):
    result = tx.run("RETURN 'Connection successful' AS message")
    for record in result:
        print(record["message"])

with driver.session() as session:
    session.read_transaction(test_connection)
```

> Tip: Protect your password with environment variables.

## Modeling Geospatial Entities

Neo4j allows storing geographic coordinates directly in the properties of nodes.

### Example: Adding cities with their positions

```python
def add_city(tx, name, lon, lat):
    tx.run(
        "CREATE (v:City {name: $name, location: point({longitude: $lon, latitude: $lat})})",
        name=name, lon=lon, lat=lat
    )

with driver.session() as session:
    session.write_transaction(add_city, "Paris", 2.3522, 48.8566)
    session.write_transaction(add_city, "Lyon", 4.8357, 45.7640)
```

## Creating Spatial Links Between Objects

You can link two cities with a "CONNECTED" relationship:

```python
def link_cities(tx, city1, city2):
    tx.run(
        """
        MATCH (a:City {name: $city1}), (b:City {name: $city2})
        CREATE (a)-[:CONNECTED]->(b)
        """,
        city1=city1, city2=city2
    )

with driver.session() as session:
    session.write_transaction(link_cities, "Paris", "Lyon")
```

> In geomatics, this could model a road segment, railway line, or a hydrological link.

## Searching for Nearby Entities

Neo4j allows spatial queries on the `point` properties.

Example: Find all cities within 300 km of Paris:

```python
def nearby_cities(tx, lon, lat, distance):
    result = tx.run(
        """
        MATCH (v:City)
        WHERE distance(v.location, point({longitude: $lon, latitude: $lat})) < $distance
        RETURN v.name AS name
        """,
        lon=lon, lat=lat, distance=distance
    )
    return [record["name"] for record in result]

with driver.session() as session:
    cities = session.read_transaction(nearby_cities, 2.3522, 48.8566, 300000)
    print(cities)
```

> `distance()` returns a distance in **meters**.

## Use Cases in Geomatics

### 1. Transport Network Optimization

- Model stops as nodes.
- Link neighboring stops with relationships with a distance or time.
- Use shortest path algorithms (`shortestPath`) to find the optimal route.

### 2. Spatial Accessibility Analysis

- Define influence zones around specific points.
- Explore relationships between places based on spatial distances.

### 3. Representing Natural Networks

- Rivers, water flows, or drainage networks as directed graphs.

### 4. Flask Web Service with Neo4j

By combining Flask and Neo4j, you can set up a web service that leverages the graph database.
Neo4j is very useful when you have a network with nodes and links.

You can add nodes to the graph.

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

> See the example of a Flask web service combined with Neo4j [https://github.com/voirinprof/neo4j_docker/](https://github.com/voirinprof/neo4j_docker/)

## Best Practices

- **Normalize your units**: longitude/latitude in WGS84, distances in meters.
- **Use explicit relationship types** (`CONNECTED`, `FLOW_TO`, etc.).
- **Index your properties** if you have many nodes (`CREATE INDEX ON :City(name)`).
- **Use native spatial capabilities** (e.g., `point()`, `distance()`) to optimize your queries.

## Useful Resources

- [Neo4j Spatial Data Types Documentation](https://neo4j.com/docs/cypher-manual/current/functions/spatial/)
- [Python Driver for Neo4j](https://neo4j.com/docs/api/python-driver/current/)
- [Cypher Query Language](https://neo4j.com/docs/cypher-manual/current/)