---
title: "PostgreSQL/PostGIS Basics"
permalink: /en/postgresql-postgis-basics/
lang: en
date: 2025-04-25T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: database
---

> **_NOTE:_** You can use this GitHub Codespaces to practice queries [https://github.com/voirinprof/gis_starter_postgis_geolab](https://github.com/voirinprof/gis_starter_postgis_geolab). **pgAdmin4** is available in the environment. To set up a Codespaces environment, see [Getting Started with GitHub Codespaces](/en/codespaces-intro/).

## Introduction to PostgreSQL and PostGIS

PostgreSQL is a relational database management system (RDBMS) that has a PostGIS extension for handling geographic and spatial data, allowing you to store, query, and analyze geometries such as points, lines, and polygons.

## Creating and Managing Databases

A PostgreSQL database is a container for tables, schemas, and other objects. To create a database, use the following command in the `psql` interface or a SQL client:

```sql
CREATE DATABASE my_database;
```

To enable PostGIS on a database, connect to the database and run:

```sql
CREATE EXTENSION postgis;
```

> Tip: Make sure PostGIS is properly installed on your PostgreSQL server before running this command.

To list all databases:

```sql
\l
```

To connect to a specific database:

```sql
\c my_database
```

Useful video: [Introduction to PostgreSQL (in English)](https://www.youtube.com/watch?v=74IWNUja05w)

## Creating Tables with Spatial Data

PostgreSQL tables store data in rows and columns. With PostGIS, you can add geometric columns to store spatial data.

Example of creating a table to store points of interest (POIs) with a POINT geometry:

```sql
CREATE TABLE points_of_interest (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    geom GEOMETRY(POINT, 4326)
);
```

- `SERIAL`: A type that automatically generates unique identifiers.
- `GEOMETRY(POINT, 4326)`: A geometric column to store points with the spatial reference system (SRID) 4326 (WGS84, used for GPS coordinates).

To insert a geographical point (for example, the Eiffel Tower in Paris):

```sql
INSERT INTO points_of_interest (name, geom)
VALUES ('Eiffel Tower', ST_GeomFromText('POINT(2.2945 48.8584)', 4326));
```

> Tip: Use explicit table and column names to ease maintenance.

## Basic SQL Queries

SQL queries allow you to manipulate and retrieve data. Here are some basic examples:

### Selecting Data

To retrieve all records from the `points_of_interest` table:

```sql
SELECT id, name, ST_AsText(geom) AS coordinates
FROM points_of_interest;
```

`ST_AsText` converts the geometry into a readable format (e.g., `POINT(2.2945 48.8584)`).

### Filtering Data

To find a specific point of interest by name:

```sql
SELECT name, ST_AsText(geom)
FROM points_of_interest
WHERE name = 'Eiffel Tower';
```

### Updating Data

To change the name of a point of interest:

```sql
UPDATE points_of_interest
SET name = 'Eiffel Tower - Paris'
WHERE id = 1;
```

### Deleting Data

To delete a point of interest:

```sql
DELETE FROM points_of_interest
WHERE id = 1;
```

Useful video: [SQL Query Basics with PostgreSQL (in English)](https://youtu.be/7S_tz1z_5bA)

## Spatial Queries with PostGIS

PostGIS enables advanced spatial analysis. Here are some common examples:

### Finding Nearby Points

To find all points of interest within 1 km of the Eiffel Tower:

```sql
SELECT name, ST_Distance(
    geom,
    ST_GeomFromText('POINT(2.2945 48.8584)', 4326)::geography
) AS distance_m
FROM points_of_interest
WHERE ST_DWithin(
    geom,
    ST_GeomFromText('POINT(2.2945 48.8584)', 4326)::geography,
    1000
);
```

- `ST_DWithin`: Checks if two geometries are within a given distance (in meters with `::geography`).
- `ST_Distance`: Calculates the distance between two geometries.

### Geometry Intersection

To check if a point is within a polygon (e.g., an administrative area):

```sql
SELECT name
FROM points_of_interest
WHERE ST_Within(
    geom,
    ST_GeomFromText('POLYGON((2.2 48.8, 2.4 48.8, 2.4 48.9, 2.2 48.9, 2.2 48.8))', 4326)
);
```

> Tip: Ensure all geometries use the same SRID to avoid errors.

Useful video: [Introduction to Spatial Queries with PostGIS](https://www.youtube.com/watch?v=yP5-GVnoy5I)

## Indexing for Performance

To improve the performance of spatial queries, create a spatial index on the geometric columns:

```sql
CREATE INDEX idx_points_of_interest_geom ON points_of_interest USING GIST (geom);
```

- `GIST`: An index type optimized for spatial data.

> Tip: Indexes are particularly useful for large tables or frequent queries.

## Best Practices

- **Use consistent SRIDs**: Ensure all geometries use the same reference system (e.g., 4326 for GPS coordinates).
- **Validate geometries**: Use `ST_IsValid` to check that geometries are correct before inserting them.
- **Test your queries**: Prepare test datasets to validate results.
- **Optimize performance**: Use spatial indexes and limit columns in queries (`SELECT`) to reduce load.

Example of geometry validation:

```sql
SELECT name, ST_IsValid(geom) AS is_valid
FROM points_of_interest;
```

Useful video: [PostgreSQL Install with PostGIS Spatial Extension - Connecting to PostgreSQL on QGIS](https://www.youtube.com/watch?v=ZaFJQg29ZQ4)