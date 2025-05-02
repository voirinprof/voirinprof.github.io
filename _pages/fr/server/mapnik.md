---
title: "Introduction à Mapnik"
permalink: /fr/mapnik-intro/
lang: fr
date: 2025-04-28T20:30:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: gisserver
---

> **_NOTE:_** Pour des détails avancés, consultez la [documentation officielle de Mapnik](https://mapnik.org/). Un exemple de service web `Flask` avec `Mapnik` peut être trouvé dans ce repo : [https://github.com/voirinprof/mapnik_docker](https://github.com/voirinprof/mapnik_docker)

## Introduction

**Mapnik** est une bibliothèque open-source puissante utilisée pour générer des cartes haute qualité à partir de données géographiques.  
Développée en C++ avec des liaisons en Python, elle est largement utilisée dans des projets comme **OpenStreetMap** pour le rendu de tuiles cartographiques.

Ce guide présente les concepts de base, l'installation, et un premier exemple d'utilisation en Python.

---

## Pourquoi utiliser Mapnik ?

- **Qualité de rendu professionnelle** (impression, web, etc.)
- **Support multi-formats** : shapefiles, PostGIS, GeoTIFF, etc.
- **Personnalisation fine** du style cartographique
- **Performance élevée** grâce à son cœur C++
- **Interopérabilité** avec Python et XML

---

## Installer Mapnik

Selon votre système d'exploitation :

### Sous Ubuntu / Debian :

```shell
sudo apt-get install python3-mapnik mapnik-utils
```

### Sous Windows / Mac :

- Utiliser **Homebrew** (`brew install mapnik`) sur Mac
- Ou installer via **conda** :
  
```shell
conda install -c conda-forge mapnik
```

### Installer via pip (moins courant) :

```shell
pip install mapnik
```

---

## Concepts clés

- **Datasource** : Source de données (ex : fichier shapefile, base PostGIS)
- **Style** : Règles de dessin des entités géographiques
- **Layer** : Couche thématique
- **Map** : Objet contenant l'ensemble des couches et du style
- **Renderer** : Moteur de rendu (ex : raster, vectoriel, SVG)

---

## Premier script Python avec Mapnik

Voici comment générer une carte simple à partir d'un shapefile :

```python
import mapnik

# Créer une carte vide
carte = mapnik.Map(800, 600)
carte.background = mapnik.Color('steelblue')

# Définir un style
style = mapnik.Style()
rule = mapnik.Rule()

# Symbole simple pour les polygones
polygon_symbolizer = mapnik.PolygonSymbolizer(mapnik.Color('white'))
rule.symbols.append(polygon_symbolizer)

# Bordure noire
line_symbolizer = mapnik.LineSymbolizer(mapnik.Color('black'), 0.5)
rule.symbols.append(line_symbolizer)

style.rules.append(rule)
carte.append_style('MonStyle', style)

# Ajouter une couche
layer = mapnik.Layer('MaCouche')
layer.datasource = mapnik.Shapefile(file='mon_shapefile.shp')
layer.styles.append('MonStyle')
carte.layers.append(layer)

# Zoom sur l'ensemble des données
carte.zoom_all()

# Rendu en image
mapnik.render_to_file(carte, 'carte_sortie.png', 'png')

print("Carte générée : carte_sortie.png")
```

> Conseil : commencez avec des shapefiles simples avant d'ajouter des couches PostGIS ou raster.

---

## Utiliser un fichier XML

Mapnik fonctionne aussi avec des fichiers XML de configuration. Parfois c'est préférable de créer un fichier XML avec la définition de la carte que l'on souhaite.

```xml
<?xml version="1.0" encoding="utf-8"?>
<Map background-color="white" 
srs="+proj=merc +a=6378137 +b=6378137 +lat_ts=0.0 +lon_0=0.0 +x_0=0.0 +y_0=0 +k=1.0 +units=m +nadgrids=@null +no_defs">

    
    <Style name="Provinces">
        <Rule >
            <Filter>([NAME_EN]='Ontario')</Filter>
            <PolygonSymbolizer fill="rgb(250, 190, 183)"/>            
        </Rule>
        <Rule >
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
        <!-- <StyleName>labels</StyleName> -->
        <Datasource>
            <Parameter name="type">shape</Parameter>
            <Parameter name="file">/data/boundaries.shp</Parameter>
            <Parameter name="srs">+proj=lcc +ellps=GRS80 +lat_0=49 +lon_0=-95 +lat+1=49 +lat_2=77 +datum=NAD83 +units=m +no_defs</Parameter>            
        </Datasource>
    </Layer>

    <Layer name="Quebec Hydrography">
        
        <StyleName>drainage</StyleName>
        <!-- <StyleName>labels</StyleName> -->
        <Datasource>
            <Parameter name="type">shape</Parameter>
            <Parameter name="file">/data/qcdrainage.shp</Parameter>
            <Parameter name="srs">+proj=lcc +ellps=GRS80 +lat_0=49 +lon_0=-95 +lat+1=49 +lat_2=77 +datum=NAD83 +units=m +no_defs</Parameter>            
        </Datasource>
    </Layer>

    <Layer name="Ontario Hydrography">
        
        <StyleName>drainage</StyleName>
        <!-- <StyleName>labels</StyleName> -->
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

Par la suite, on peut utiliser le fichier xml dans un code Python.

```python
def generate_map_from_xml():
    # see the doc for : https://get-map.org/mapnik-lost-manual/book.html
    # create a mapnik map object
    m = mapnik.Map(800, 600)
    # load the mapnik XML file
    mapnik.load_map(m, 'map.xml')
```

## Où utiliser Mapnik ?

- **Serveurs de tuiles** : génération dynamique de tuiles web (ex : OpenStreetMap, MapProxy)
- **Applications desktop** : export de cartes en PNG, SVG, PDF
- **Systèmes embarqués** : pré-rendu de cartes pour appareils offline
- **Rapports SIG automatisés** : production d'illustrations cartographiques dans des pipelines de traitement de données

---

## Ressources utiles

- [Site officiel Mapnik](https://mapnik.org/)
- [Tutoriels Mapnik en Python](https://mapnik.github.io/doc/python/)
- [Projet OpenStreetMap Carto (style Mapnik)](https://github.com/gravitystorm/openstreetmap-carto)

---

## Bonnes pratiques

- **Optimisez vos datasources** (ex : index spatial PostGIS)
- **Travaillez avec des styles XML** pour séparer données et habillage
- **Générez vos cartes en batch** si vous avez de nombreux niveaux de zoom
- **Expérimentez avec les symbolizers avancés** (dégradés, ombrages, étiquettes dynamiques)