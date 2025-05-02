---
title: "Intro to GeoInfo Basics"
permalink: /en/gis-level1/
lang: en
date: 2025-04-24T03:02:20+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: start
---
> **_NOTE:_** The [Yves' Python CookBook on GitHub](https://github.com/yvoirin/cookbook_python3) contains many useful code examples for geomatics. This [YouTube channel](https://www.youtube.com/@YvesVoirin) also provides many resources.

## Installing Libraries

The commands to install libraries are always the same, either using PIP:

```shell
pip install your_lib
```

or using CONDA:

```shell
conda install your_lib
```

To get started with geomatics development, here are the essential libraries:

* **OGR/GDAL**: A long-standing C++ library used in geomatics. It handles both vector and raster data. Installation can be tricky, so it's often better to use `rasterio` and `fiona` which act as Python bridges to GDAL/OGR.
* **rasterio**: A Pythonic wrapper around GDAL for handling raster data. Very simple to use.
* **fiona**: A wrapper around OGR for handling vector data. Like rasterio, it follows a user-friendly API.
* **pandas/geopandas**: Essential for handling structured data like CSV files. Geopandas can also read vector formats and may replace fiona depending on the use case.
* **matplotlib**: Useful for generating plots and charts, fully replacing tools like Excel for this purpose.

Your development environment should include these libraries. Depending on your project, you may need other more specific packages.

## Reading Spatial Data

### Vector Data

Common formats supported in Python include Shapefile, GeoJSON, GPKG, CSV, and more.

With `fiona`, you can read spatial data such as a Shapefile:

```python
import fiona

with fiona.open('file.shp', 'r') as src:
    for feature in src:
        print(feature)
```

Reading a GeoJSON file works the same way:

```python
import fiona

with fiona.open('path/to/your/file.geojson', 'r') as src:
    for feature in src:
        print(feature)
```

An alternative to `fiona` is `geopandas`:

```python
import geopandas as gpd

gdf = gpd.read_file('path/to/your/file.shp')

print(gdf.head())
```

### Raster Data

To read raster data in Python, use the `rasterio` library:

```python
import rasterio
import matplotlib.pyplot as plt

with rasterio.open('my_file.tif') as src:
    raster = src.read(1)

plt.imshow(raster, cmap='gray')
plt.show()
```

Raster formats can have multiple bands (e.g., for NDVI or satellite images). The `src.read` method can read all bands as needed.

Since raster data is loaded into NumPy arrays, you can apply any NumPy operations.

## Writing Spatial Data

### Vector Data

If you have a list of vector features stored in a Python list named `features` and want to write them to a Shapefile:

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

The `schema` defines the geometry and attribute fields for the shapefile.

### Raster Data

If you have a matrix stored in a NumPy array called `matrix` and want to write it to a GeoTIFF:

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

The `metadata` dictionary specifies the characteristics of the GeoTIFF file, such as CRS, data type, size, and transform (position and resolution).

## Working with Databases

To interact with databases in geomatics projects, you can use `pandas` or `geopandas` along with a connector library like `psycopg2` (PostgreSQL) or `pyodbc` (SQL Server).

Example with a PostgreSQL database named `my_database`:

```python
import pandas as pd
import psycopg2

conn = psycopg2.connect("dbname=my_database user=my_user password=my_password")

df = pd.read_sql_query("SELECT * FROM my_spatial_data", conn)

conn.close()
```

If the table contains spatial geometries, use `geopandas`:

```python
import geopandas as gpd
import psycopg2

conn = psycopg2.connect("dbname=my_database user=my_user password=my_password")

gdf = gpd.read_postgis("SELECT * FROM my_spatial_data", conn, geom_col="geom")

conn.close()
```

This allows you to load both attributes and geometries into a `GeoDataFrame` and then perform spatial operations or export to various formats.

## Useful Videos

Here is a playlist covering the key concepts from this page:

* [Different ways to read vector and raster data in Python 3](https://youtu.be/QsnectEfqTo)
* [Read an image and process it with NumPy](https://youtu.be/YxnHey-V2Eg)
* [Basic operations with NumPy](https://youtu.be/V8Yk6j_gpxs)
* [Read images and compute NDVI with NumPy, GDAL, and Matplotlib](https://youtu.be/-mJ0WnMlwQc)
* [Read data from a CSV and create a matrix with NumPy, then plot with Matplotlib](https://youtu.be/cnITO_D9A8I)
* [Use Kaggle to analyze data (with Pandas, Matplotlib, and Folium)](https://youtu.be/XHOG8GqT15Y)
* [Getting started with Pandas and Anaconda (in Spyder)](https://youtu.be/8j4vRloPyzU)
* [Analyzing multiple CSV files with Pandas (Bixi data example)](https://youtu.be/Qn6lGCDslhw)
* [Merging CSV files with Pandas](https://youtu.be/NKKu_-8-kGY)
* [Using Geopandas](https://youtu.be/hONrwu8Kbx4)
* [How to write a GeoTIFF or a Shapefile with Rasterio and Fiona in just a few lines](https://youtu.be/dJYH0jnydHI)