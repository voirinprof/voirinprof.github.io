---
title: "Python with PostGIS"
permalink: /en/python-postgis/
lang: en
date: 2025-04-26T15:30:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: database
---

> **_NOTE:_** The [official PostGIS documentation](https://postgis.net/documentation/) as well as the Python library [psycopg](https://www.psycopg.org/) are essential resources for further learning. You may sometimes need to use [psycopg2](https://github.com/psycopg/psycopg2) with Python 3. You can practice queries using this GitHub Codespace [https://github.com/voirinprof/gis_starter_postgis_geolab](https://github.com/voirinprof/gis_starter_postgis_geolab). **pgAdmin4** is available in the environment. To set up a Codespace environment, check out [Getting Started with GitHub Codespaces](/en/codespaces-intro/).

## Introduction to PostGIS

**PostGIS** is a spatial extension for the relational database **PostgreSQL**.  
It allows you to:
- Store geometries (points, lines, polygons) in standard SQL format
- Perform complex spatial calculations (distances, intersections, buffers)
- Execute optimized spatial queries with indexes

In geomatics, PostGIS is used to:
- Manage large volumes of geospatial data (cadasters, networks, sensors)
- Perform advanced spatial analyses (zoning, accessibility)
- Serve as a backend for GIS and web mapping applications

## Installing PostGIS and Python Libraries

Install PostgreSQL and add the PostGIS extension.

For Python, install the following:

```shell
pip install psycopg[binary] geoalchemy2 sqlalchemy
```

- `psycopg`: PostgreSQL connection
- `SQLAlchemy` + `GeoAlchemy2`: ORM for spatial objects

> Tip: Use `Docker` to quickly launch a PostgreSQL + PostGIS instance locally.

## Connecting to PostGIS in Python

### Simple connection with `psycopg`

```python
import psycopg

conn = psycopg.connect("dbname=geomatics user=postgres password=password host=localhost")
cur = conn.cursor()
```

### Advanced connection with SQLAlchemy

```python
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg://postgres:password@localhost/geomatics")
```

## Creating a Spatial Table

Example: storing geographic points of interest.

```sql
CREATE TABLE points_of_interest (
    id SERIAL PRIMARY KEY,
    name TEXT,
    geom GEOMETRY(Point, 4326)
);
```

> `4326` is the EPSG code for WGS84 (standard GPS latitude/longitude).

## Inserting Spatial Data in Python

```python
cur.execute("""
    INSERT INTO points_of_interest (name, geom)
    VALUES (%s, ST_SetSRID(ST_MakePoint(%s, %s), 4326))
""", ("Eiffel Tower", 2.2945, 48.8584))

conn.commit()
```

> `ST_SetSRID` defines the spatial reference system (here, WGS84).

## Performing Spatial Queries

### Finding Points Near a Location

Example: all points within 500 meters of the Eiffel Tower.

```python
cur.execute("""
    SELECT name
    FROM points_of_interest
    WHERE ST_DWithin(
        geom::geography,
        ST_SetSRID(ST_MakePoint(%s, %s), 4326)::geography,
        %s
    )
""", (2.2945, 48.8584, 500))

results = cur.fetchall()

for row in results:
    print(row[0])
```

> `ST_DWithin` allows filtering by a **distance** in meters if you convert to `geography`.

## Use Cases in Geomatics

### 1. Storing Routes or Networks

You can use the `LINESTRING` type for roads or GPS routes:

```sql
CREATE TABLE routes (
    id SERIAL PRIMARY KEY,
    vehicle_id TEXT,
    geom GEOMETRY(LineString, 4326)
);
```

Python insertion:

```python
route_coords = [(2.2945, 48.8584), (2.2950, 48.8585), (2.2960, 48.8586)]

linestring = "LINESTRING({})".format(
    ", ".join(f"{lon} {lat}" for lon, lat in route_coords)
)

cur.execute("""
    INSERT INTO routes (vehicle_id, geom)
    VALUES (%s, ST_SetSRID(ST_GeomFromText(%s), 4326))
""", ("bus_42", linestring))

conn.commit()
```

### 2. Analyzing Intersections or Overlaps

Find all routes that intersect a given area:

```python
zone = "POLYGON((2.293 48.857, 2.296 48.857, 2.296 48.860, 2.293 48.860, 2.293 48.857))"

cur.execute("""
    SELECT vehicle_id
    FROM routes
    WHERE ST_Intersects(
        geom,
        ST_SetSRID(ST_GeomFromText(%s), 4326)
    )
""", (zone,))

results = cur.fetchall()

for row in results:
    print(row[0])
```

## Best Practices

- **Create spatial indexes** to speed up your queries:
  ```sql
  CREATE INDEX idx_points_geom ON points_of_interest USING GIST (geom);
  ```
- **Use `geography`** if working over large terrestrial distances (precise calculations on a sphere).
- **Always specify the SRID** (EPSG code) to avoid spatial reference errors.
- **Separate roles**: use PostgreSQL users with limited rights in production.

## Useful Resources

- [Official PostGIS Documentation](https://postgis.net/documentation/)
- [PostGIS Practical Guide](https://postgis.net/workshops/postgis-intro/)
- [Psycopg Documentation](https://www.psycopg.org/psycopg3/docs/)
- [GeoAlchemy2 for SQLAlchemy](https://geoalchemy-2.readthedocs.io/en/latest/)