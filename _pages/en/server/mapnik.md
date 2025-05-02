---
title: "Introduction to Mapnik"
permalink: /en/mapnik-intro/
lang: en
date: 2025-04-28T20:30:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: gisserver
---

> **_NOTE:_** For advanced details, refer to the [official Mapnik documentation](https://mapnik.org/). An example of a `Flask` web service with `Mapnik` can be found in this repo: [https://github.com/voirinprof/mapnik_docker](https://github.com/voirinprof/mapnik_docker)

## Introduction

**Mapnik** is a powerful open-source library used to generate high-quality maps from geographic data.  
Developed in C++ with Python bindings, it is widely used in projects like **OpenStreetMap** for rendering map tiles.

This guide presents the basic concepts, installation, and a first example of use in Python.

---

## Why use Mapnik?

- **Professional rendering quality** (print, web, etc.)
- **Multi-format support**: shapefiles, PostGIS, GeoTIFF, etc.
- **Fine-grained map styling** customization
- **High performance** due to its C++ core
- **Interoperability** with Python and XML

---

## Installing Mapnik

Depending on your operating system:

### On Ubuntu / Debian:

```shell
sudo apt-get install python3-mapnik mapnik-utils
```

### On Windows / Mac:

- Use **Homebrew** (`brew install mapnik`) on Mac
- Or install via **conda**:

```shell
conda install -c conda-forge mapnik
```

### Install via pip (less common):

```shell
pip install mapnik
```

---

## Key Concepts

- **Datasource**: Data source (e.g., shapefile, PostGIS database)
- **Style**: Drawing rules for geographic entities
- **Layer**: Thematic layer
- **Map**: Object containing all layers and styles
- **Renderer**: Rendering engine (e.g., raster, vector, SVG)

---

## First Python Script with Mapnik

Here is how to generate a simple map from a shapefile:

```python
import mapnik

# Create an empty map
map = mapnik.Map(800, 600)
map.background = mapnik.Color('steelblue')

# Define a style
style = mapnik.Style()
rule = mapnik.Rule()

# Simple symbol for polygons
polygon_symbolizer = mapnik.PolygonSymbolizer(mapnik.Color('white'))
rule.symbols.append(polygon_symbolizer)

# Black border
line_symbolizer = mapnik.LineSymbolizer(mapnik.Color('black'), 0.5)
rule.symbols.append(line_symbolizer)

style.rules.append(rule)
map.append_style('MyStyle', style)

# Add a layer
layer = mapnik.Layer('MyLayer')
layer.datasource = mapnik.Shapefile(file='my_shapefile.shp')
layer.styles.append('MyStyle')
map.layers.append(layer)

# Zoom to all data
map.zoom_all()

# Render to image
mapnik.render_to_file(map, 'output_map.png', 'png')

print("Map generated: output_map.png")
```

> Tip: Start with simple shapefiles before adding PostGIS or raster layers.

---

## Using an XML File

Mapnik can also work with XML configuration files. Sometimes, it's preferable to create an XML file with the desired map definition.

```xml
<?xml version="1.0" encoding="utf-8"?>
<Map background-color="white" 
srs="+proj=merc +a=6378137 +b=6378137 +lat_ts=0.0 +lon_0=0.0 +x_0=0.0 +y_0=0 +k=1.0 +units=m +nadgrids=@null +no_defs">

    <Style name="Provinces">
        <Rule>
            <Filter>([NAME_EN]='Ontario')</Filter>
            <PolygonSymbolizer fill="rgb(250, 190, 183)"/>            
        </Rule>
        <Rule>
            <Filter>([NAME_EN]='Quebec')</Filter>
            <PolygonSymbolizer fill="rgb(217, 235, 203)"/>            
        </Rule>
        <Rule>
            <LineSymbolizer stroke="black" stroke-width="1" stroke-dasharray="8 4 2 2 2 2"/>
        </Rule>
    </Style>

    <Style name="drainage">
        <Rule>
            <Filter>([HYC] = 8)</Filter>
            <PolygonSymbolizer fill="rgb(153, 204, 255)"/>
            <LineSymbolizer stroke="black" stroke-width="1" stroke-dasharray="8 4 2 2 2 2"/>
        </Rule>
    </Style>

    <Style name="labels">
        <Rule>            
            <TextSymbolizer face-name="DejaVu Sans Book" allow-overlap="false" fill="white" placement="centroid" size="8 + ([Shape_Area] / 100000000)" >[NAME_EN]</TextSymbolizer>
        </Rule>
    </Style>

    <Style name="smallroads">
        <Rule>
            <Filter>([CLASS] = 3) or ([CLASS] = 4)</Filter>
            <LineSymbolizer stroke="rgb(171,158,137)" stroke-width="2" stroke-linecap="round" />
        </Rule>
    </Style>
    <Style name="road-border">
        <Rule>
            <Filter>[CLASS] = 2</Filter>
            <LineSymbolizer stroke="rgb(171,158,137)" stroke-width="4" stroke-linecap="round"/>
        </Rule>
    </Style>
    <Style name="road-fill">
        <Rule>
            <Filter>[CLASS] = 2</Filter>
            <LineSymbolizer stroke="rgb(255,250,115)" stroke-width="2" stroke-linecap="round"/>
        </Rule>
    </Style>
    <Style name="highway-border">
        <Rule>
            <Filter>[CLASS] = 1</Filter>
            <LineSymbolizer stroke="rgb(188,149,28)" stroke-width="4" stroke-linecap="round"/>
        </Rule>
    </Style>
    <Style name="highway-fill">
        <Rule>
            <Filter>[CLASS] = 2</Filter>
            <LineSymbolizer stroke="rgb(242,191,36)" stroke-width="2" stroke-linecap="round"/>
        </Rule>
    </Style>

    <Style name="popplaces">
        <Rule>
            <TextSymbolizer face-name="DejaVu Sans Book" allow-overlap="false" fill="black" halo-fill="white" halo-radius="1" placement="point" size="10" >[GEONAME]</TextSymbolizer>
        </Rule>
    </Style>

    <Layer name="Provinces">
        <StyleName>Provinces</StyleName>
        <Datasource>
            <Parameter name="type">shape</Parameter>
            <Parameter name="file">/data/boundaries.shp</Parameter>
            <Parameter name="srs">+proj=lcc +ellps=GRS80 +lat_0=49 +lon_0=-95 +lat+1=49 +lat_2=77 +datum=NAD83 +units=m +no_defs</Parameter>            
        </Datasource>
    </Layer>

    <Layer name="Quebec Hydrography">
        <StyleName>drainage</StyleName>
        <Datasource>
            <Parameter name="type">shape</Parameter>
            <Parameter name="file">/data/qcdrainage.shp</Parameter>
            <Parameter name="srs">+proj=lcc +ellps=GRS80 +lat_0=49 +lon_0=-95 +lat+1=49 +lat_2=77 +datum=NAD83 +units=m +no_defs</Parameter>            
        </Datasource>
    </Layer>

    <Layer name="Ontario Hydrography">
        <StyleName>drainage</StyleName>
        <Datasource>
            <Parameter name="type">shape</Parameter>
            <Parameter name="file">/data/ontdrainage.shp</Parameter>
            <Parameter name="srs">+proj=lcc +ellps=GRS80 +lat_0=49 +lon_0=-95 +lat+1=49 +lat_2=77 +datum=NAD83 +units=m +no_defs</Parameter>            
        </Datasource>
    </Layer>

    <Layer name="Roads">
        <StyleName>smallroads</StyleName>
        <StyleName>road-border</StyleName>
        <StyleName>road-fill</StyleName>
        <StyleName>highway-border</StyleName>
        <StyleName>highway-fill</StyleName>
        <Datasource>
            <Parameter name="type">shape</Parameter>
            <Parameter name="file">/data/roads.shp</Parameter>
            <Parameter name="srs">+proj=lcc +ellps=GRS80 +lat_0=49 +lon_0=-95 +lat+1=49 +lat_2=77 +datum=NAD83 +units=m +no_defs</Parameter>            
        </Datasource>
    </Layer>

    <Layer name="Populated Places">
        <StyleName>popplaces</StyleName>
        <Datasource>
            <Parameter name="type">shape</Parameter>
            <Parameter name="file">/data/popplaces.shp</Parameter>
            <Parameter name="srs">+proj=lcc +ellps=GRS80 +lat_0=49 +lon_0=-95 +lat+1=49 +lat_2=77 +datum=NAD83 +units=m +no_defs</Parameter>            
        </Datasource>
    </Layer>

</Map>
```

Afterwards, you can use the XML file in Python code.

```python
def generate_map_from_xml():
    # see the doc for : https://get-map.org/mapnik-lost-manual/book.html
    # create a mapnik map object
    m = mapnik.Map(800, 600)
    # load the mapnik XML file
    mapnik.load_map(m, 'map.xml')
```

## Where to Use Mapnik?

- **Tile servers**: Dynamic tile generation for the web (e.g., OpenStreetMap, MapProxy)
- **Desktop applications**: Export maps to PNG, SVG, PDF
- **Embedded systems**: Pre-render maps for offline devices
- **Automated GIS reports**: Produce cartographic illustrations in data processing pipelines

---

## Useful Resources

- [Official Mapnik site](https://mapnik.org/)
- [Mapnik Python tutorials](https://mapnik.github.io/doc/python/)
- [OpenStreetMap Carto Project (Mapnik style)](https://github.com/gravitystorm/openstreetmap-carto)

---

## Best Practices

- **Optimize your datasources** (e.g., PostGIS spatial indexes)
- **Work with XML styles** to separate data and styling
- **Generate your maps in batch** if you have many zoom levels
- **Experiment with advanced symbolizers** (gradients, shading, dynamic labels)