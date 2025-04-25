---
title: "First Concepts in GeoInfo"
permalink: /en/gis-level1/
lang: en
date: 2025-04-24T03:02:20+00:00
toc: true
toc_label: "On this page"
---

> **_NOTE:_** The [Yves’ Cookbook on GitHub](https://github.com/yvoirin/cookbook_python3) provides many useful code examples for geomatics. This [YouTube channel](https://www.youtube.com/@YvesVoirin) also offers plenty of resources.

## Installing Your Libraries

The commands to install libraries are always the same. You can either use PIP:

```shell
pip install your_lib
```

or CONDA:

```shell
conda install your_lib
```

To get started with geomatics development, there are a few essential libraries:

* **OGR/GDAL**: A long-standing C++ library used in geomatics. It allows manipulation of both vector and raster data. Installation can be tricky. It’s often easier to use `rasterio` and `fiona`, which are Python interfaces to GDAL/OGR.
* **rasterio**: A simpler interface to GDAL for manipulating raster data.
* **fiona**: The interface to OGR for vector data. Its consistent conventions with `rasterio` make it very user-friendly.
* **pandas/geopandas**: For handling formatted data (e.g., CSVs). These libraries allow fast and simple data manipulation and `geopandas` can also read vector data formats, potentially replacing `fiona`.
* **matplotlib**: Essential for producing charts and plots, a complete replacement for Excel charting.

Your development environment should include these libraries. Depending on your project, additional more specific ones may be needed.

## Reading Spatial Data

### Vector Data

Common formats Python supports include Shapefile, GeoJSON, GPKG, CSV, etc.

Using `fiona`, you can read spatial data like a Shapefile:

```python
import fiona

# Open Shapefile
with fiona.open('file.shp', 'r') as src:
    for feature in src:
        print(feature)
```

Or a GeoJSON file:

```python
import fiona

# Open GeoJSON file
with fiona.open('path/to/your/file.geojson', 'r') as src:
    for feature in src:
        print(feature)
```

An alternative to `fiona` is `geopandas`:

```python
import geopandas as gpd

# Read the Shapefile
gdf = gpd.read_file('path/to/your/file.shp')

# Display first 5 rows
print(gdf.head())
```

### Raster Data

To read raster files in Python, use the `rasterio` library:

```python
import rasterio
import matplotlib.pyplot as plt

# Open raster file
with rasterio.open('my_file.tif') as src:
    raster = src.read(1)

# Display the raster
plt.imshow(raster, cmap='gray')
plt.show()
```

Raster files may contain multiple bands, each representing a different variable. `src.read()` can read multiple bands at once. The loaded images are NumPy arrays, enabling full use of NumPy operations.

## Writing Spatial Data

### Vector Data

Suppose you have a list of features called `features` that you want to write to a Shapefile named `my_file.shp`:

```python
import fiona

schema = {
    'geometry': 'Point',
    'properties': {
        'name': 'str',
        'value': 'int'
    }
}

with fiona.open('my_file.shp', 'w', 'ESRI Shapefile', schema) as dst:
    for feature in features:
        dst.write(feature)
```

The `schema` defines the geometry type and attributes. Here, it's a Point geometry with two attributes: a string (`name`) and an integer (`value`).

### Raster Data

Suppose you have a NumPy matrix called `matrix` to write to a GeoTIFF file named `my_file.tif`:

```python
import rasterio

height, width = matrix.shape

metadata = {
    'count': 1,
    'crs': 'EPSG:4326',
    'dtype': 'float32',
    'driver': 'GTiff',
    'height': height,
    'width': width,
    'transform': rasterio.transform.from_bounds(0, 0, width, height, width, height),
    'nodata': -9999
}

with rasterio.open('my_file.tif', 'w', **metadata) as dst:
    dst.write(matrix, 1)
```

The `metadata` dictionary defines the raster’s properties such as number of bands, spatial reference, data type, and geotransform. `from_bounds()` sets the spatial extent and resolution.

## Database Interactions

To interact with a spatial database in Python, use `pandas` or `geopandas` for importing, manipulating, and exporting tabular data. You’ll also need a connector library like `psycopg2` (PostgreSQL) or `pyodbc` (SQL Server).

To load spatial data from a PostgreSQL table `my_spatial_data` into a `DataFrame`:

```python
import pandas as pd
import psycopg2

conn = psycopg2.connect("dbname=my_database user=my_user password=my_password")

df = pd.read_sql_query("SELECT * FROM my_spatial_data", conn)

conn.close()
```

For spatial databases, use `geopandas`:

```python
import geopandas as gpd
import psycopg2

conn = psycopg2.connect("dbname=my_database user=my_user password=my_password")

gdf = gpd.read_postgis("SELECT * FROM my_spatial_data", conn, geom_col="geom")

conn.close()
```

This returns a `GeoDataFrame` that includes both attributes and geometries, allowing for full geospatial analysis and export.

## Useful Videos

Here is a series of videos covering the concepts in this page:

* [Various ways to read vector and raster data with Python 3](https://youtu.be/QsnectEfqTo)
* [Reading and processing images with Numpy](https://youtu.be/YxnHey-V2Eg)
* [Basic operations with Numpy](https://youtu.be/V8Yk6j_gpxs)
* [NDVI calculation using Numpy, GDAL, and Matplotlib](https://youtu.be/-mJ0WnMlwQc)
* [Read CSV data and create a matrix with Numpy, then display with Matplotlib](https://youtu.be/cnITO_D9A8I)
* [Using Kaggle to analyze data (Pandas, Matplotlib, and folium)](https://youtu.be/XHOG8GqT15Y)
* [Getting started with Pandas and Anaconda (Spyder)](https://youtu.be/8j4vRloPyzU)
* [Analyzing multiple CSV files with Pandas (Bixi data example)](https://youtu.be/Qn6lGCDslhw)
* [Consolidating CSV files with Pandas](https://youtu.be/NKKu_-8-kGY)
* [Using Geopandas](https://youtu.be/hONrwu8Kbx4)
* [How to write a GeoTIFF or Shapefile with Rasterio and Fiona in a few lines](https://youtu.be/dJYH0jnydHI)