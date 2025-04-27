---
layout: home
title: Home
permalink: /en/
lang: en
author_profile: true
sidebar: false
full: true
---
# Welcome to this site dedicated to geomatics in Python!

If you are a Python developer interested in geomatics, this site is for you. A variety of resources and tools are available to help you better understand and apply geomatics concepts in your Python work.

The site includes tutorials and code examples for manipulating geospatial data, visualizing maps, analyzing geospatial data, and much more. You can also find links to popular Python libraries for geomatics such as Geopandas, Shapely, and Fiona.

The figure below illustrates what a developer in Geomatics should know. The website will provide you with the fundamentals to approach all these topics.

<pre class="mermaid">
      mindmap
  root((Developer))
    :::firstblock
    Database
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
      Cartographic
      :::thirdblock
    Client
    :::secondblock
      JS, CSS, HTML
      :::thirdblock
      Frameworks
      :::thirdblock
        Vue
    Miscellaneous
    :::secondblock
      AI
      :::thirdblock
      QGis
      :::thirdblock
      Tests
      :::thirdblock
</pre>

We hope you enjoy our site and find it useful in your geomatics projects in Python. Please feel free to contact us if you have any comments or suggestions for improving our site.


<style>

pre.mermaid {
  
  border: 2px dashed #eaeaea;   /* bordure */
  padding: 20px;             /* espace interne */
  border-radius: 10px;       /* coins arrondis */
  overflow: auto;            /* scroll si besoin */
  /* font-family: "consolas" */
}

pre.mermaid .firstblock  {
  fill: #eaeaea; /* vert doux */
  
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
