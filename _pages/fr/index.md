---
layout: home
title: Accueil
permalink: /fr/
lang: fr
author_profile: true
sidebar: false
full: true
---
# Bienvenue sur ce site dédié à la géomatique en Python !

Si vous êtes un développeur Python intéressé par la géomatique, ce site est fait pour vous. Une variété de ressources et d'outils sont disponibles pour vous aider à mieux comprendre et appliquer les concepts de la géomatique dans votre travail en Python.

Le site comprend des tutoriels et des exemples de code Python pour la manipulation de données géospatiales, la visualisation de cartes, l'analyse de données géospatiales, et bien plus encore. Vous pouvez également trouver des liens vers des bibliothèques Python populaires pour la géomatique telles que Geopandas, Shapely et Fiona.

La figure ci-dessous illustre ce que devrait connaître un développeur en Géomatique. Le site vous donnera des bases pour aborder tous ces points.
<pre class="mermaid">
      mindmap
  root((Développeur))
    :::firstblock
    Base de données
    :::secondblock
      SQL
      :::thirdblock
        PostGIS
      NoSQL
      :::thirdblock
        Redis
        MongoDB
        Neo4j
    Serveur
    :::secondblock
      Docker
      :::thirdblock
      WebAPI
      :::thirdblock
        Flask
        FastAPI
        Django
      Cartographique
      :::thirdblock
    Client
    :::secondblock
      JS, CSS, HTML
      :::thirdblock
      Frameworks
      :::thirdblock
        Vue
    Divers
    :::secondblock
      IA
      :::thirdblock
      QGis
      :::thirdblock
      Tests
      :::thirdblock
    </pre>
    

J'espère que vous apprécierez le site et que vous le trouverez utile dans vos projets de géomatique en Python. N'hésitez pas à me contacter si vous avez des commentaires ou des suggestions pour améliorer le contenu du site.

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

