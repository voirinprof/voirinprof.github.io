---
title: "Introduction to QGIS Server and Python"
permalink: /en/qgis-server-intro/
lang: en
date: 2025-04-28T21:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: gisserver
---

> **_NOTE:_** For official information, check out the [QGIS Server documentation](https://docs.qgis.org/3.34/en/docs/server_manual/index.html). An example of a `Flask` web service with `QGIS Server` can be found in this repo: [https://github.com/voirinprof/qgis_docker](https://github.com/voirinprof/qgis_docker)

## Introduction

**QGIS Server** is the server version of QGIS that allows you to publish **WMS**, **WFS**, and **WCS** map services directly from `.qgs` projects without heavy development.  
By combining **QGIS Server** with **Python**, you can automate, customize, and create dynamic web GIS services.

---

## Why use QGIS Server?

- **Simplicity**: Publish a QGIS project as a web service as is
- **OGC Standards compliance**: WMS 1.3, WFS 2.0, etc.
- **Advanced customization** via server plugins in Python
- **Same rendering capabilities** as the QGIS Desktop
- **Direct integration with PostGIS**, GeoTIFF, and more

---

## Install QGIS Server

### On Ubuntu / Debian:

```shell
sudo apt-get install qgis-server
```

Necessary services:
- Web server (Apache, Nginx)
- FastCGI or uwsgi to interface QGIS Server with the web server

### Verify installation:

Test the local URL (e.g., with Apache):

```
http://localhost/qgisserver?SERVICE=WMS&REQUEST=GetCapabilities
```

If everything is correctly set up, you should get an XML response describing your server capabilities.

---

## Deploy a QGIS project as a WMS service

1. **Prepare your `.qgs` project**:
   - Set the CRS (coordinate reference system)
   - Configure project properties (layer names, min/max scales)

2. **Place the project file**:
   - Save your `.qgs` in a folder accessible by the server (e.g., `/var/www/qgis/myproject.qgs`).

3. **Call your project via the URL**:

```text
http://localhost/qgisserver?MAP=/var/www/qgis/myproject.qgs&SERVICE=WMS&REQUEST=GetCapabilities
```

---

## Use Python with QGIS Server

Python allows you to **create custom server extensions** or **interact with QGIS Server**. Two main approaches exist:

### 1. **Create plugins for QGIS Server**

- Plugins are structured the same way as QGIS Desktop plugins but with a specific entry point: `server` instead of `gui`.
- Allows:
  - Dynamic modification of layers
  - Adding new operations to the service
  - Custom data filtering

**Example of server plugin architecture**:

```
my_plugin/
├── __init__.py
├── server/
│   ├── __init__.py
│   ├── myplugin.py
├── metadata.txt
```

In `server/myplugin.py`:

```python
from qgis.server import QgsServerFilter

class MyPluginServer(QgsServerFilter):
    def requestReady(self):
        print("Request received:", self.serverInterface().requestHandler().parameterMap())
```

### 2. **Use PyQGIS to pre-process your projects**

With **PyQGIS** (the Python API for QGIS), you can dynamically generate `.qgs` files, prepare styles, PostGIS layers, before publishing on QGIS Server.

**Simple PyQGIS script example**:

```python
from qgis.core import QgsProject

# Load an existing project
project = QgsProject.instance()
project.read('/path/to/my_project.qgs')

# Dynamically change the title
project.title = "New server title"

# Save
project.write()
```

---

## Typical Use Cases

- **Automatic generation of thematic GIS services**
- **Applying data access rules via server plugins**
- **Customizing GetFeatureInfo** (enriched JSON response)
- **Conditional rendering based on users/sessions**

---

## Useful Resources

- [QGIS Server documentation](https://docs.qgis.org/3.34/en/docs/server_manual/)
- [QGIS Server Plugin Development (PyQGIS Cookbook)](https://docs.qgis.org/3.34/en/docs/pyqgis_developer_cookbook/server/index.html)
- [Official server plugin example](https://github.com/qgis/QGIS-Server-Plugin-Examples)

---

## Best Practices

- **Optimize your `.qgs` projects** (light styles, logical groups)
- **Use PostGIS** for large spatial databases
- **Cache your services** (with MapProxy or QGIS Cache) to speed up responses
- **Monitor server logs** for debugging Python plugins
- **Automate project updates with PyQGIS**