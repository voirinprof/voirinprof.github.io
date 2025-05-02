---
layout: home
title: Home
permalink: /en/
lang: en
author_profile: true
sidebar: false
full: true
---

# Welcome to this site dedicated to Geomatics with Python!

If you're a Python developer interested in geomatics, this site is for you. A variety of resources and tools are available to help you better understand and apply geomatics concepts in your Python work.

The site includes tutorials and Python code examples for handling geospatial data, visualizing maps, performing geospatial analysis, and much more. You will also find links to popular Python libraries for geomatics such as Geopandas, Shapely, and Fiona.

This site aims to introduce concepts across various aspects of programming that a geomatics developer should master, including:

- **Setting up a complete development environment**  
  (Visual Studio Code, Git, Docker, installation, ...)

- **Designing, administering, and querying relational databases**  
  (SQL modeling, query optimization, spatial integration, ...)

- **Exploring alternatives to traditional databases**  
  (implementing NoSQL databases suited for geospatial needs: MongoDB, Redis, Neo4j, ...)

- **Deploying and administering a map server**  
  (QGIS Server, GeoServer, MapServer, managing OGC services: WMS, WFS, WCS, WPS, ...)

- **Setting up custom geospatial web services**  
  (REST APIs, geocoding, routing, spatial analysis services, ...)

- **Developing traditional web clients**  
  (HTML5, JavaScript, CSS3, integrating mapping libraries like Leaflet, OpenLayers)

- **Using modern frameworks to develop rich interfaces**  
  (VUE.js, React, building interactive and high-performance applications)

- **Extending QGIS functionalities**  
  (Python plugin development, custom processing, integration into existing workflows)

The diagram below illustrates what a geomatics developer should ideally know. This site will give you a solid foundation to explore each of these areas.

<pre class="mermaid">
      mindmap
  root((Developer))
    :::firstblock
    Databases
    :::secondblock
      SQL
      :::thirdblock
        PostGIS
      NoSQL
      :::thirdblock
        Redis
        MongoDB
        Neo4j
    Server
    :::secondblock
      Docker
      :::thirdblock
      WebAPI
      :::thirdblock
        Flask
        FastAPI
        Django
      Mapping
      :::thirdblock
    Client
    :::secondblock
      QGIS
      :::thirdblock
      JS, CSS, HTML
      :::thirdblock
      Frameworks
      :::thirdblock
        Vue
    Miscellaneous
    :::secondblock
      AI
      :::thirdblock      
      Testing
      :::thirdblock
</pre>

I hope you enjoy the site and find it helpful for your Python geomatics projects. Feel free to contact me if you have any feedback or suggestions to improve the site’s content.

<style>

pre.mermaid {
  
  border: 2px dashed #eaeaea;   /* border */
  padding: 20px;                /* inner spacing */
  border-radius: 10px;          /* rounded corners */
  overflow: auto;               /* scroll if needed */
  /* font-family: "consolas" */
}

pre.mermaid .firstblock  {
  fill: #eaeaea; /* soft gray */
  
  stroke-width: 2px;
  /* font-weight: bold;
  font-size: 14px; */
  
}

pre.mermaid .firstblock .node-bkg {
  fill: #252a34!important;
  stroke: #eaeaea!important;
}

pre.mermaid .secondblock .node-bkg {
  fill:rgb(70, 80, 99)!important;
  
}

pre.mermaid .secondblock text {
  fill: #eaeaea!important;
}

pre.mermaid .thirdblock .node-bkg {
  fill:rgb(109, 125, 155)!important;
  
}

pre.mermaid .thirdblock text {
  fill: #eaeaea!important;
}

pre.mermaid .mindmap-edges .edge {
  stroke:rgb(248, 246, 246)!important;
}
</style>