---

title: "Advanced GeoInfo Concepts"
permalink: /en/gis-level2/
lang: en
date: 2025-04-24T03:02:20+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: start
---

> **_NOTE:_** The [CookBook by Yves on GitHub](https://github.com/yvoirin/cookbook_python3) provides many useful code examples for geospatial professionals. This [YouTube channel](https://www.youtube.com/@YvesVoirin) also offers numerous resources.

## Shapely

Shapely is often used in combination with other geospatial libraries such as `geopandas` and `PyQGIS` to process geospatial data. Here are some of the main features of Shapely:

* Shapely supports a wide variety of geometries, including points, lines, polygons, circles, ellipses, multipoints, multilines, and multipolygons.
* Shapely geometries are immutable, meaning they cannot be modified once created. This ensures that geometries remain stable and consistent over time.
* Shapely provides functions for performing common geometric operations such as intersection, union, difference, and distance between geometries.
* Shapely geometries can be used to compute geometric measurements such as area, length, perimeter, and centroid of a geometry.
* Shapely is easy to use and integrate into Python scripts. The documentation is well-written and provides code examples for most functions.

Creating a point and a polygon using Shapely:

```python
from shapely.geometry import Point, Polygon

# Create a point
point = Point(0, 0)

# Create a polygon
polygon = Polygon([(0, 0), (0, 1), (1, 1), (1, 0)])
```

Some code snippets showing manipulation with `shapely`.

```python
import shapely.wkt as wkt
import shapely.geometry as geometry
from shapely.ops import transform
import pyproj

# Some basic operations with Shapely

# Declare 2 points
# Montreal
lon_mtl, lat_mtl = -73.561668,45.508888
# Sherbrooke
lon_she, lat_she = -71.888351,45.404476

# Create a geometry from a WKT
pt_mtl = wkt.loads(f'POINT({lon_mtl} {lat_mtl})')

# Create a geometry from x, y coordinates
pt_she = geometry.Point((lon_she, lat_she))

# Create a geometry from a json
pt_json = {'type': 'Point', 'coordinates': (lon_mtl, lat_mtl)}
pt_mtl2 = geometry.shape(pt_json)

# Create a line from a list of values
line = [(lon_mtl, lat_mtl), (lon_she, lat_she)]
line_mtl_she = geometry.LineString(line)

# Create a line from a list of points
line2 = [pt_mtl, pt_she]
line_mtl_she2 = geometry.LineString(line2)

# Get the geometry type
print(line_mtl_she2.geom_type)

# Iterate over the points
for pt in line_mtl_she2.coords:
    print(pt)

# Get the bounding box
print(line_mtl_she.bounds)

# Create a geometry from the bounds
poly = geometry.box(*line_mtl_she.bounds)
print(poly)

# Check if a point is inside the geometry
print(poly.contains(poly.centroid))

# Reproject
crs1 = pyproj.CRS('EPSG:4326')
crs2 = pyproj.CRS('EPSG:6622')

trans = pyproj.Transformer.from_crs(crs1, crs2).transform
polyproj = transform(trans, poly)
print(polyproj)
```

Useful video: [The Essentials of Shapely in a Few Minutes](https://youtu.be/vAvlzYwSca8)

## Rtree (Spatial Index)

Using RTree helps solve spatial search problems, such as finding points within a certain radius, searching for intersections between polygons, and identifying polygons that contain a given point.

Using RTree is relatively simple. To use RTree, you first need to create an index, specifying the type of object you want to index and the properties you want to index. Then, you can add objects to the index and perform queries to search for objects within the index.

Using RTree to search for points within a given radius:

```python
from rtree import index

# Create an RTree index for points
p_idx = index.Index(properties=['x', 'y'])

# Add some points to the index
p_idx.insert(1, (0, 0, 0, 0))
p_idx.insert(2, (1, 1, 1, 1))
p_idx.insert(3, (2, 2, 2, 2))
p_idx.insert(4, (3, 3, 3, 3))

# Search for points within a radius of 1 around the point (1, 1)
results = list(p_idx.nearest((1, 1, 1, 1), 1.0))
print(results)
```

RTree is a very useful library for spatial indexing in Python, and is often used in conjunction with other libraries like Shapely and Geopandas.

Useful video: [Understanding the Importance of rtree in a Few Minutes](https://youtu.be/drJkdRbSaBo)

## Converting from a Matrix to a Vector (with Fiona and Rasterio)

This operation is sometimes necessary when you want to obtain a vector file from matrix data. For example, after classification, you often want to obtain polygons of the classified areas. This can be done with Fiona and Rasterio.

```python
import rasterio
from rasterio.features import shapes
import fiona
from shapely.geometry import shape, mapping
from fiona.crs import from_string
image_path = r'../data/mask_satellite.tiff'
# Open the raster image in read mode
with rasterio.open(image_path) as src:
    
    image = src.read(1) # Read the first band of the image (or the desired band)
    # Find the regions (objects) in the image
    image_regions = list(shapes(image, mask=None, transform=src.transform))
    shapefile_path = r'../data/mask_satellite_convert.shp' # Specify the output path
    # Get the image's coordinate system in EPSG
    crs_proj4 = src.crs.to_proj4() # Get the image's coordinate system
    # Open the Shapefile in write mode
    crs_vector = from_string(crs_proj4) # Convert the coordinate system
    with fiona.open(shapefile_path, 'w', 'ESRI Shapefile', crs=from_string(crs_proj4),
        schema={'geometry': 'Polygon', 'properties':{}}) as output:
        for geom, _ in image_regions:
            # Save the geometry in the Shapefile
            feature = {'geometry': mapping(shape(geom)), 'properties': {}}
            output.write(feature)
```

## Converting from a Vector to a Matrix (with Fiona and Rasterio)

This operation is also quite common in geospatial analysis. It allows you to obtain a grid from a vector file. For example, sometimes you may want to apply a matrix mask to an image. The data from which the mask originates is vector data.

```python
import fiona
import rasterio
from rasterio.features import geometry_mask
from shapely.geometry import shape
# Specify the path to your Shapefile
shapefile_path = r'../data/mask_satellite.shp'
# Open the Shapefile in read mode
with fiona.open(shapefile_path, 'r') as shapefile:
    print(shapefile.meta)
    # Get the first geometry from the Shapefile
    firstshape = shape(shapefile[0]["geometry"])
    # Get the bounds of the geometry
    bbox = firstshape.bounds

    # Desired resolution of the raster in meters per pixel
    resolution = 10 # 10 meters
    # Transformation from the origin, resolution, and dimensions
    transform = rasterio.transform.from_origin(bbox[0], bbox[3], resolution, resolution)
    # Raster dimensions from the transformation
    width = int((bbox[2] - bbox[0]) / resolution)
    height = int((bbox[3] - bbox[1]) / resolution)
    # Raster mask from the Shapefile geometries
    mask = geometry_mask([firstshape], transform=transform, invert=True,
    out_shape=(height, width))
    # Create a new raster file
    with rasterio.open(r'../data/mask_satellite.tiff', 'w', driver='GTiff', height=height,
        width=width, count=1, dtype='uint8', crs=shapefile.crs,
        transform=transform) as dst:
        dst.write(mask, 1)
```

## Cutting an Image Based on a Geometry (with Fiona and Rasterio)

This operation is quite common in geospatial analysis as it involves cutting an image based on an area of interest. This operation is simply done with Fiona and Rasterio.

```python
import fiona
import rasterio
import rasterio.mask
# Specify the path to your Shapefile
shapefile_path = r'../data/mask_satellite.shp'
# Open the Shapefile in read mode
with fiona.open(shapefile_path, 'r') as shapefile:
    # Access the first geometric feature
    geom = shapefile[0]['geometry']
    # Specify the path to your raster image
    raster_path = r'../data/satellite.tiff'
    # Open the raster image in read mode
    with rasterio.open(raster_path) as src:
        # Cut the image using the Shapefile geometry
        out_image, out_transform = rasterio.mask.mask(src, [geom], crop=True)
        out_meta = src.meta.copy()
        out_meta.update({"driver": "GTiff", "height": out_image.shape[1],
        "width": out_image.shape[2], "transform": out_transform})
        with rasterio.open(r"../data/satellite_clip.tiff", "w", **out_meta) as dest:
            dest.write(out_image)
```

## Reprojecting an Image with Rasterio

Sometimes, you need to reproject an image into another coordinate system. With Rasterio, you can easily perform this operation.

```python
import numpy as np
import rasterio
from rasterio.warp import calculate_default_transform, reproject, Resampling
# Input image
source_image_path = r'../data/mask_satellite.tiff'
# Output CRS
dst_crs = 'EPSG:4326'
# Open the image
with rasterio.open(source_image_path) as src:
    # Calculate the transformation
    transform, width, height = calculate_default_transform(
    src.crs, dst_crs, src.width, src.height, *src.bounds)
    kwargs = src.meta.copy()
    kwargs.update({
    'crs': dst_crs,
    'transform': transform,
    'width': width,
    'height': height
    })

    # Create the output image
    with rasterio.open(r'../data/mask_satellite_4326.tiff', 'w', **kwargs) as dst:
        for i in range(1, src.count + 1):
            reproject(
            source=rasterio.band(src, i),
            destination=rasterio.band(dst, i),
            src_transform=src.transform,
            src_crs=src.crs,
            dst_transform=transform,
            dst_crs=dst_crs,
            resampling=Resampling.nearest)
```

## Reprojecting a Vector Dataset with Fiona

Sometimes, you need to reproject vector data into another system. You can use Fiona for this.

```python
import fiona

import pyproj
from shapely.geometry import shape, mapping
from fiona.crs import from_epsg
from shapely.ops import transform
# Path to the source Shapefile
source_shapefile_path = r'../data/mask_satellite.shp'
# Specify the target projection
target_crs = from_epsg(4326) # WGS 84 (EPSG:4326)
# Path to the target Shapefile
target_shapefile_path = r'../data/mask_satellite_4326.shp'

# Open the source file in read mode
with fiona.open(source_shapefile_path, 'r') as src:
    # Open the target file in write mode and specify the new projection
    with fiona.open(target_shapefile_path, 'w', 'ESRI Shapefile',
        schema=src.schema, crs=target_crs) as dst:
        # Use pyproj to perform the reprojection of geometries
        project = pyproj.Transformer.from_crs(src.crs, target_crs,

 always_xy=True).transform
        for feat in src:
            geom = shape(feat['geometry'])
            # Perform the transformation
            transformed_geom = transform(project, geom)
            # Save the transformed geometry in the target file
            feat['geometry'] = mapping(transformed_geom)
            dst.write(feat)
```
