---
title: "Route Calculation with pgRouting"
permalink: /en/python-routing/
lang: en
date: 2025-04-26T15:30:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: database
---

> **_NOTE:_** There are several approaches to route calculation from a road network. You can use PostGIS’s pgRouting, a simple library like networkx, or even use Neo4j. See the example of pgRouting with a custom road network and a Flask web service [https://github.com/voirinprof/gis_pgrouting_docker](https://github.com/voirinprof/gis_pgrouting_docker). If you want to see how to use networkx for route calculation, use this repo: [https://github.com/voirinprof/gis_nx_docker](https://github.com/voirinprof/gis_nx_docker).

## Introduction to pgRouting

**pgRouting** is a PostGIS extension that adds route calculation and network analysis capabilities to a PostgreSQL/PostGIS database.

In geomatics, pgRouting allows:
- Calculation of the shortest routes (fastest, shortest, multimodal)
- Network analysis (accessibility, graphs, multiple routes)
- Processing of large road networks directly in the database

> **_NOTE:_** The [official pgRouting documentation](https://pgrouting.org/documentation.html) is very useful to explore all the available features.

## Installing pgRouting

On a server with PostgreSQL and PostGIS, install pgRouting:

```shell
sudo apt install postgresql-15-pgrouting
```

Then enable the extension in your database:

```sql
CREATE EXTENSION pgrouting;
```

> Tip: pgRouting works on **graphs** stored as **edge tables**.

## Preparing a Road Network

Your road network must be structured as an edge table:

```sql
CREATE TABLE reseau_routier (
    id SERIAL PRIMARY KEY,
    source BIGINT,
    target BIGINT,
    geom GEOMETRY(LineString, 4326),
    cost DOUBLE PRECISION
);
```

- `source`: ID of the start node
- `target`: ID of the end node
- `cost`: Cost to traverse the edge (distance, time, etc.)

> You can use `pgr_createTopology` to automatically generate `source` and `target` from `geom`.

```sql
SELECT pgr_createTopology('reseau_routier', 0.0001, 'geom', 'id');
```

## Connecting to pgRouting in Python

Install dependencies:

```shell
pip install psycopg sqlalchemy shapely
```

Simple connection:

```python
import psycopg

conn = psycopg.connect("dbname=geomatique user=postgres password=password host=localhost")
cur = conn.cursor()
```

## Calculating the Shortest Path

Assuming you know the `source_id` and `target_id` in your network:

```python
start = 1001
end = 1050

cur.execute("""
    SELECT seq, id1 AS node, id2 AS edge, cost
    FROM pgr_dijkstra(
        'SELECT id, source, target, cost AS cost FROM reseau_routier',
        %s, %s,
        directed := true
    )
""", (start, end))

path = cur.fetchall()

for line in path:
    print(f"Step {line[0]}: pass through node {line[1]} via edge {line[2]} (cost {line[3]})")
```

> `pgr_dijkstra` is the classic shortest path algorithm.

## Other Useful pgRouting Features

- **Route to multiple destinations**:  
  `pgr_dijkstraVia`
- **Calculate cost matrices between multiple points**:  
  `pgr_dijkstraCostMatrix`
- **Isochrone search (reachable areas by distance or time)**:  
  `pgr_drivingDistance`

Example of driving distance:

```sql
SELECT * FROM pgr_drivingDistance(
    'SELECT id, source, target, cost FROM reseau_routier',
    1001,
    2000  -- maximum distance
);
```

## Best Practices

- **Create indexes on `source` and `target`** to speed up calculations:
  ```sql
  CREATE INDEX ON reseau_routier(source);
  CREATE INDEX ON reseau_routier(target);
  ```
- **Optimize your network**: limit unnecessary edges, check topology.
- **Use a metric projection system** (e.g., EPSG:2154 for France) for accurate distances.
- **Separate directed and undirected networks** according to your needs.

## Use Cases in Geomatics

- **Vehicle route calculation** (deliveries, buses, emergency services)
- **Accessibility analysis** (distance to hospitals, schools)
- **Origin-Destination matrices for mobility flows**

## Useful Resources

- [Official pgRouting Documentation](https://pgrouting.org/documentation.html)
- [Practical pgRouting Examples](https://docs.pgrouting.org/3.4/en/pgr_dijkstra.html)
- [pgRouting Workshop (PostGIS & pgRouting)](https://workshop.pgrouting.org/)