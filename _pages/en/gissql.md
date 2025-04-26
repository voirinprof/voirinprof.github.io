---
title: "PostgreSQL/PostGIS Basics"
permalink: /en/postgresql-postgis-basics/
lang: en
date: 2025-04-25T10:00:00+00:00
toc: true
toc_label: "On this page"
typenav: database
---

## Introduction to PostgreSQL and PostGIS

PostgreSQL is a powerful, open-source relational database management system (RDBMS) known for its extensibility. PostGIS is an extension for PostgreSQL that adds support for geographic and spatial data, enabling the storage, querying, and analysis of geometries such as points, lines, and polygons.

Before working with PostgreSQL/PostGIS, it’s crucial to follow these steps:

* **Understand the requirements**: Identify the data to be stored (attributes, geometries) and the types of queries needed (spatial searches, distance calculations, etc.).
* **Define inputs**: Determine the tables, columns, and data types required, such as geographic coordinates or alphanumeric attributes.
* **Define outputs**: Specify the expected results, such as a list of nearby points or an intersection of polygons.
* **Plan queries**: Outline the SQL/PostGIS operations needed to transform raw data into the desired results.
* **Break down complex problems**: Divide complex queries into smaller, manageable sub-queries or intermediate steps.
* **Test results**: Verify query accuracy using well-defined test cases.

This approach ensures efficient use of PostgreSQL/PostGIS, minimizing errors and optimizing performance.

## Creating and Managing Databases

A PostgreSQL database is a container for tables, schemas, and other objects. To create a database, use the following command in the `psql` interface or a SQL client:

```sql
CREATE DATABASE my_database;
```

To enable PostGIS on a database, connect to the database and run:

```sql
CREATE EXTENSION postgis;
```

> Tip: Ensure PostGIS is installed on your PostgreSQL server before executing this command.

To list all databases:

```sql
\l
```

To connect to a specific database:

```sql
\c my_database
```

Video resource: [Introduction to PostgreSQL](https://youtu.be/4vJy2N7tQ0Q)

## Creating Tables with Spatial Data

PostgreSQL tables store data in rows and columns. With PostGIS, you can add geometry columns to store spatial data.

Example of creating a table to store points of interest (POI) with a POINT geometry:

```sql
CREATE TABLE points_of_interest (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    geom GEOMETRY(POINT, 4326)
);
```

- `SERIAL`: A type that automatically generates unique identifiers.
- `GEOMETRY(POINT, 4326)`: A geometry column for storing points with the spatial reference system (SRID) 4326 (WGS84, used for GPS coordinates).

To insert a geographic point (e.g., the Eiffel Tower in Paris):

```sql
INSERT INTO points_of_interest (name, geom)
VALUES ('Eiffel Tower', ST_GeomFromText('POINT(2.2945 48.8584)', 4326));
```

> Tip: Use clear table and column names to simplify maintenance.

## Basic SQL Queries

SQL queries allow you to manipulate and retrieve data. Here are some fundamental examples:

### Selecting Data

To retrieve all records from the `points_of_interest` table:

```sql
SELECT id, name, ST_AsText(geom) AS coordinates
FROM points_of_interest;
```

`ST_AsText` converts the geometry to a readable format (e.g., `POINT(2.2945 48.8584)`).

### Filtering Data

To find a specific point of interest by name:

```sql
SELECT name, ST_AsText(geom)
FROM points_of_interest
WHERE name = 'Eiffel Tower';
```

### Updating Data

To modify the name of a point of interest:

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

Video resource: [SQL Query Basics with PostgreSQL](https://youtu.be/7S_tz1z_5bA)

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

- `ST_DWithin`: Checks if two geometries are within a specified distance (in meters with `::geography`).
- `ST_Distance`: Calculates the distance between two geometries.

### Geometry Intersection

To check if a point lies within a polygon (e.g., an administrative area):

```sql
SELECT name
FROM points_of_interest
WHERE ST_Within(
    geom,
    ST_GeomFromText('POLYGON((2.2 48.8, 2.4 48.8, 2.4 48.9, 2.2 48.9, 2.2 48.8))', 4326)
);
```

> Tip: Ensure all geometries use the same SRID to avoid errors.

Video resource: [Introduction to Spatial Queries with PostGIS](https://youtu.be/8BveB1N0b2U)

## Indexing for Performance

To improve the performance of spatial queries, create a spatial index on geometry columns:

```sql
CREATE INDEX idx_points_of_interest_geom ON points_of_interest USING GIST (geom);
```

- `GIST`: An index type optimized for spatial data.

> Tip: Indexes are particularly useful for large tables or frequent queries.

## Best Practices

- **Use consistent SRIDs**: Ensure all geometries use the same spatial reference system (e.g., 4326 for GPS coordinates).
- **Validate geometries**: Use `ST_IsValid` to check geometry correctness before insertion.
- **Test queries**: Prepare test datasets to validate results.
- **Optimize performance**: Use spatial indexes and limit columns in `SELECT` statements to reduce load.

Example of geometry validation:

```sql
SELECT name, ST_IsValid(geom) AS is_valid
FROM points_of_interest;
```

Video resource: [Best Practices with PostgreSQL and PostGIS](https://youtu.be/5p3y2G5z5eY)