---
title: "Advanced concepts of Geoinfo"
permalink: /en/gis-level2/
lang: en
date: 2025-04-24T03:02:20+00:00
toc: true
toc_label: "On this page"
typenav: start
---

> **_NOTE:_** The [CookBook by Yves on GitHub](https://github.com/yvoirin/cookbook_python3) provides many useful code examples for geomatics professionals. This [YouTube channel](https://www.youtube.com/@YvesVoirin) also offers a wealth of resources.

## Shapely: Geometric Operations in Python

**Shapely** is a powerful Python library for manipulating planar geometric objects. It is built on the GEOS library (used by PostGIS) and provides:

- Support for various geometry types: points, lines, polygons, multipoints, etc.
- Geometric operations: intersection, union, difference, distance.
- Geometry attributes: area, length, centroid.
- Seamless integration with libraries like GeoPandas and PyQGIS.

Example: Creating a point and a polygon:

```python
from shapely.geometry import Point, Polygon

# Create a point
point = Point(0, 0)

# Create a polygon
polygon = Polygon([(0, 0), (0, 1), (1, 1), (1, 0)])
```

📘 [Shapely Documentation](https://shapely.readthedocs.io/)  
🎥 [Shapely Essentials in a Few Minutes](https://youtu.be/vAvlzYwSca8)

---

## Rtree: Fast Spatial Indexing

**Rtree** provides spatial indexing and search capabilities using R-tree data structures. It is often used alongside Shapely and GeoPandas for spatial queries.

Key features:

- Nearest neighbor search
- Spatial containment and intersection queries
- Multi-dimensional indexing
- Disk-based storage and fast loading

Example: Finding the nearest object to a point:

```python
from rtree import index

# Create an Rtree index
idx = index.Index()

# Insert points
idx.insert(1, (0, 0, 0, 0))
idx.insert(2, (1, 1, 1, 1))

# Search for the closest point to (1, 1)
results = list(idx.nearest((1, 1, 1, 1), 1))
print(results)
```

📘 [Rtree Documentation](https://rtree.readthedocs.io/)  
🎥 [Why Use Rtree? (Short Explanation)](https://youtu.be/drJkdRbSaBo)

---

## Django & GeoDjango: Web Development with Spatial Support

**Django** is a high-level Python web framework based on the MVC (Model-View-Controller) pattern. **GeoDjango** extends it to support geographic data and spatial operations.

Core features:

- URL routing and view handling
- ORM for database access and manipulation
- Templating system for dynamic HTML rendering
- Middleware and form validation
- Integration with PostGIS for spatial queries via GeoDjango

📘 [Django Official Website](https://www.djangoproject.com/)  
🎥 Useful Videos:
- [Getting Started with Django](https://youtu.be/PzQ9lfjdMv4)
- [Adding Components to a Django Site](https://youtu.be/8aF0_pfxd4w)
- [Displaying Database Data in a Django View](https://youtu.be/FW5gaWCl1HY)

---

## QGIS & PyQGIS: Plugin Development for GIS

**QGIS** is a free and open-source desktop GIS application written in C++. It supports Python through the **PyQGIS** API, allowing developers to build plugins that interact with spatial data.

To build a QGIS plugin:

1. Use Qt Designer to create the plugin interface.
2. Use Plugin Builder to generate the plugin’s structure.
3. Write Python code using the QGIS API to implement functionality.

💡 Tip: Development is accelerated by tools like Plugin Builder, but familiarity with PyQt5 and the QGIS API is essential.

📘 [PyQGIS Developer Cookbook](https://docs.qgis.org/latest/en/docs/pyqgis_developer_cookbook/index.html)  
🎥 Helpful Videos:
- [Building a Basic QGIS Plugin](https://youtu.be/pnDr149JMWU)
- [Creating a Vector Layer Plugin (QGIS 3.4)](https://youtu.be/hMbzWsq0-bI)
- [Customizing Plugin Interfaces with Qt Designer](https://youtu.be/1VDnon52M4E)

---

## Additional Resources

- 📘 [Cookbook on GitHub (by Yves)](https://github.com/yvoirin/cookbook_python3) – Code examples for geospatial Python.
- 🎥 [Yves Voirin’s YouTube Channel](https://www.youtube.com/@YvesVoirin) – Tutorials on GIS and Python topics.