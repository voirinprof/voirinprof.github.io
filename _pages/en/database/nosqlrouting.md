---
title: "Routing with Redis and Neo4j"  
permalink: /en/routing-redis-neo4j/  
lang: en  
date: 2025-04-26T18:00:00+00:00  
toc: true  
toc_sticky: true  
toc_label: "On this page"  
typenav: database  
---

> **_NOTE:_** This project combines **Redis** for fast address lookup and **Neo4j** for finding optimized paths in a road network graph. See the example of pgRouting with a custom road network and a Flask web service [https://github.com/voirinprof/gis_nosql_docker](https://github.com/voirinprof/gis_nosql_docker).

## Introduction

In a modern geospatial application, it is common to:  
- **Quickly search** for an address from a label or coordinates ➔ **Redis** (ultra-fast key-value store)  
- **Calculate a path** between two addresses ➔ **Neo4j** (graph-based representation of the road network)  

This guide demonstrates how to use Python to connect the two databases and perform **complete routing**.

## Application Architecture

- **Redis** stores the addresses (with associated ID)  
- **Neo4j** stores the road network graph (nodes = addresses/intersections, edges = roads)  
- **Python** bridges the fast address lookup with path calculation  

Diagram:  

```
User search ➔ Address via Redis ➔ Start/End node ➔ Path calculation in Neo4j
```

## Install Dependencies

```shell
pip install redis neo4j
```

## Prepare Redis for Addresses

Insert your addresses into Redis in the format **(lon, lat, address)**.

Example in Python:

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
        # Add the address to a sorted set for indexing
        redis_client.zadd("index_locations", {address: 1})
```

## Prepare Neo4j for the Road Network Graph

The graph will be built using a road network containing segments.

Graph structure:  
- Node (intersection): `:Node {id: 123, ...}`  
- Relationship (road between two nodes): `[:Link {length: 150}]`

Example of graph creation in Cypher:

```cypher
CREATE (a:Node {id:123, ...})
CREATE (b:Node {id:456, ...})
CREATE (a)-[:Link {distance: 250}]->(b)
```

> Tip: also create an index on `:Node(id)` for faster searches:  
```cypher
CREATE INDEX lieu_id IF NOT EXISTS FOR (n:Node) ON (n.id);
```

You can also create a graph from a CSV file, notably one for nodes and another for links.  
These CSV files can be created from spatial data.

The nodes file should be:  

```csv
id:ID,latitude:FLOAT,longitude:FLOAT
0,45.4710109296785,-71.94436225521166
1,45.470409779469946,-71.94493520202225
...
```

The links file should be:  

```csv
:START_ID,:END_ID,street,objectid:INT,length:FLOAT,speed:FLOAT
0,1,Rue Oliva-Turgeon,123399,114.77815102017024,50
1,0,Rue Oliva-Turgeon,123399,114.77815102017024,50
...
```

Later, you can create the road network graph with the following command:

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

You can then apply methods on the `myGraphRoad` graph. It is recommended to use `gds` (Graph Data Science) for newer path algorithms.

## Python Application for Search and Routing

Connection:

```python
from redis import Redis
from neo4j import GraphDatabase

# Connections
redis_conn = Redis(host='localhost', port=6379, db=0)
neo4j_driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "password"))
```

Search for an address with Redis. This function would return a list of suggested addresses in a form.

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
        address = match
        coordinates = redis_client.geopos("locations", address)
        if coordinates:
            lat = coordinates[0][1]
            lon = coordinates[0][0]
            suggestions.append({"display_name": address, "lat": lat, "lon": lon})

    return suggestions
```

Once you know the position of the address, you can then find the closest node in the Neo4j graph. Since address points are often not on roads, the search might return a location outside the graph.

Using `cypher`, you can find the closest node to a point.

```python
# function to search for nodes in Neo4j
def nodeSearch(latitude, longitude):
    
    # use the point function to create a point and find the closest nodes
    cypher_query = """
    MATCH (c:Node)
    WITH c, point({latitude: $lat, longitude: $lon}) AS refPoint
    WITH c, point.distance(point({latitude: c.latitude, longitude: c.longitude}), refPoint) AS distance
    WHERE distance <= $max_distance
    RETURN c.id, c.longitude, c.latitude, distance
    ORDER BY distance
    """
```

Once you have the start and end nodes, you can compute the shortest path.

Calculate the path in Neo4j:

```python
# we will find the path between the two nodes
cypher_path_query = """
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

## Best Practices

- **Normalize addresses** before storing them in Redis (no uppercase, no accents if possible).
- **Cache** frequent paths to avoid unnecessary recalculations.
- **Monitor Redis** to avoid memory saturation if the address volume is very large (use `EXPIRE` if necessary).
- **Optimize Neo4j graphs**: indexes, remove duplicates, simplify secondary roads.

## Geospatial Use Cases

- **Quick custom route calculation** between addresses in a city
- **Delivery route optimization** by combining Redis (fast location lookup) and Neo4j (optimal path)
- **Dynamic search for alternative routes** based on points of interest (POI)

## Useful Resources

- [Redis Documentation](https://redis.io/docs/)
- [Neo4j Documentation](https://neo4j.com/docs/)
- [Neo4j Graph Data Science Library](https://neo4j.com/docs/graph-data-science/current/)