---
title: "Python with PostgreSQL/PostGIS"
permalink: /en/python-postgresql-postgis/
lang: en
date: 2025-04-25T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: database
---

> **_NOTE:_** The [official PostGIS documentation](https://postgis.net/documentation/) provides complete instructions on its usage. For Python interaction, also check out the resources for the libraries [psycopg](https://www.psycopg.org/) and [GeoAlchemy](https://geoalchemy-2.readthedocs.io/en/latest/).

## Introduction to PostGIS and Python

**PostGIS** is an extension of PostgreSQL that allows the management of spatial data (geometry and geography). Combined with **Python**, it becomes a powerful tool for analyzing, processing, and visualizing geospatial data.

This guide presents several methods to interact with a PostGIS database in Python:
- Direct connection with `psycopg`
- Using a spatial ORM with `GeoAlchemy`
- Reading data into `GeoPandas` via `read_postgis`

Before starting:
- **Install PostgreSQL and PostGIS**.
- **Create a PostGIS database** or use an existing one.
- **Install the necessary Python libraries** (`psycopg`, `sqlalchemy`, `geopandas`, etc.).

## Approach 1: Direct Connection with `psycopg`

**`psycopg`** is the standard library for interacting with PostgreSQL in Python.

### Simple Connection Example

```python
import psycopg

# Connect to the database
conn = psycopg.connect(
    dbname="your_database",
    user="your_user",
    password="your_password",
    host="localhost",
    port="5432"
)

# Create a cursor
cur = conn.cursor()

# Execute a PostGIS query
cur.execute("SELECT id, ST_AsText(geom) FROM your_table;")

# Fetch the results
rows = cur.fetchall()
for row in rows:
    print(row)

# Close the connection
cur.close()
conn.close()
```

> Tip: Use `with psycopg.connect(...) as conn:` to ensure the connection closes automatically.

## Approach 2: Using a Spatial ORM with `GeoAlchemy`

**`GeoAlchemy 2`** extends SQLAlchemy to support spatial types.

### ORM Model Example

```python
from sqlalchemy import create_engine, Column, Integer
from sqlalchemy.ext.declarative import declarative_base
from geoalchemy2 import Geometry

# Connect via SQLAlchemy
engine = create_engine("postgresql+psycopg://user:password@localhost:5432/your_database")
Base = declarative_base()

# Define the model
class Point(Base):
    __tablename__ = 'points'
    id = Column(Integer, primary_key=True)
    geom = Column(Geometry('POINT'))

# Create the table (if necessary)
Base.metadata.create_all(engine)
```

> Tip: Combine `GeoAlchemy` with `Shapely` to manipulate geometries in Python.

## Approach 3: Reading with `GeoPandas`

**`GeoPandas`** makes it easier to manipulate geospatial data as a DataFrame. If you don't need spatial features, **`Pandas`** will suffice, and the syntax is almost identical.

### Read a PostGIS Table Directly

```python
import geopandas as gpd
import psycopg

# Connection parameters
conn_str = "postgresql+psycopg://user:password@localhost:5432/your_database"

# Read with GeoPandas
gdf = gpd.read_postgis(
    sql="SELECT * FROM your_table;",
    con=conn_str,
    geom_col='geom'
)

print(gdf.head())
```

> Tip: Use SQL filters (`WHERE`, `LIMIT`) to avoid loading large datasets.

## Best Practices

- **Use transactions**: When making significant modifications, work within transactions to ensure data integrity.
- **Create spatial indexes**: Use `CREATE INDEX ON table USING GIST(geom);` to speed up spatial queries.
- **Handle SRID properly**: Be mindful of the coordinate system used (`ST_Transform` to change the SRID if needed).
- **Secure your connections**: Avoid hardcoding passwords in code; use environment variables instead.
- **Optimize your queries**: Select only the necessary columns and limit results when possible.

## Useful Resources

- [Official PostGIS Documentation](https://postgis.net/documentation/)
- [Psycopg 3 Documentation](https://www.psycopg.org/psycopg3/docs/)
- [GeoAlchemy 2 Documentation](https://geoalchemy-2.readthedocs.io/en/latest/)
- [GeoPandas Documentation](https://geopandas.org/en/stable/)