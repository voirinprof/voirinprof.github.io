---
title: "Notions sur GeoServer et Mapserver"
permalink: /fr/gisserver/
lang: fr
date: 2025-04-25T10:00:00+00:00
toc: true
toc_label: "Dans cette page"
typenav: gisserver
---

## Introduction à GeoServer et MapServer

**GeoServer** et **MapServer** sont des serveurs open-source permettant de publier des données géospatiales sur le web. Ils sont largement utilisés pour partager des cartes interactives, des services web géographiques (WMS, WFS, WMTS), et des données spatiales dans des formats standards définis par l’**Open Geospatial Consortium (OGC)**. Ces outils sont essentiels pour créer des applications SIG (Systèmes d’Information Géographique) accessibles via des navigateurs ou des clients comme QGIS.

Avant de commencer à travailler avec GeoServer ou MapServer, il est essentiel de suivre ces étapes :
* **Comprendre les besoins** : Identifier les types de données (vecteurs, rasters) et les services à fournir (visualisation, interrogation, téléchargement).
* **Définir les entrées** : Quelles données seront publiées ? Par exemple, des shapefiles, des rasters GeoTIFF ou des tables PostGIS.
* **Définir les sorties** : Quel type de résultat attendre ? Par exemple, une carte interactive, un flux WFS ou une tuile WMTS.
* **Planifier la configuration** : Identifier les étapes pour connecter les données, configurer les styles et exposer les services.
* **Diviser les tâches complexes** : Séparer la configuration des données, des styles et des services en étapes distinctes.
* **Tester les résultats** : Vérifier que les services fonctionnent correctement avec des clients comme QGIS ou des bibliothèques JavaScript (Leaflet, OpenLayers).

Cette approche garantit une utilisation efficace de GeoServer et MapServer, en évitant les erreurs courantes et en optimisant les performances.

## Présentation de GeoServer et MapServer

- **GeoServer** : Un serveur Java open-source qui permet de publier des données géospatiales à partir de sources comme PostGIS, shapefiles ou rasters. Il offre une interface web intuitive pour configurer les couches, les styles (via SLD) et les services OGC (WMS, WFS, WCS, WMTS). GeoServer est idéal pour les projets nécessitant une gestion simplifiée et des fonctionnalités avancées comme le rendu dynamique.
- **MapServer** : Un serveur cartographique open-source écrit en C, conçu pour des performances élevées et une flexibilité maximale. Il utilise des fichiers de configuration texte (Mapfiles) pour définir les couches et les styles. MapServer est parfait pour les environnements nécessitant une personnalisation fine ou une intégration légère.

## Configuration et gestion des serveurs

### GeoServer
GeoServer peut être installé sur un serveur local ou déployé via un conteneur Docker. Téléchargez la version stable depuis le site officiel ou utilisez une image Docker.

**Démarrer GeoServer** :
1. Téléchargez et décompressez GeoServer ou utilisez Docker :
```bash
docker run -p 8080:8080 geoserver/geoserver
```
2. Accédez à l’interface web : `http://localhost:8080/geoserver`.
3. Connectez-vous avec les identifiants par défaut (`admin/geoserver`) et changez le mot de passe.

**Ajouter un magasin de données** :
1. Dans l’interface web, allez à **Magasins de données** > **Ajouter un nouveau magasin**.
2. Sélectionnez une source (par exemple, PostGIS ou shapefile).
3. Configurez les paramètres (par exemple, hôte, base de données, chemin du fichier).

> Conseil : Vérifiez que les dépendances Java (JDK) sont installées pour une installation manuelle.

### MapServer
MapServer est généralement installé sur un serveur Linux ou Windows. Il peut être déployé via des packages ou compilé depuis les sources.

**Installer MapServer** (sur Ubuntu) :
```bash
sudo apt-get update
sudo apt-get install mapserver
```

**Créer un Mapfile** :
Un Mapfile est un fichier texte définissant les couches et les services. Exemple minimal (`mapfile.map`) :
```mapfile
MAP
  NAME "CarteSimple"
  EXTENT -180 -90 180 90
  SIZE 800 600
  IMAGETYPE PNG

  LAYER
    NAME "Pays"
    TYPE POLYGON
    DATA "pays.shp"
    STATUS ON
    CLASS
      STYLE
        COLOR 200 200 200
        OUTLINECOLOR 0 0 0
      END
    END
  END
END
```

**Tester le Mapfile** :
Utilisez l’outil `mapserv` pour vérifier :
```bash
mapserv -nh "QUERY_STRING=map=/chemin/vers/mapfile.map&mode=map"
```

> Conseil : Hébergez MapServer via un serveur web (Apache, Nginx) pour exposer les services OGC.

Vidéo utile : [Introduction à GeoServer](https://youtu.be/5m0aWn8VSFo)

## Publication de données spatiales

### GeoServer
**Créer une couche** :
1. Dans **Couches** > **Ajouter une nouvelle ressource**, sélectionnez un magasin de données configuré (par exemple, une table PostGIS avec une géométrie `POINT`).
2. Publiez la couche en définissant un nom et un style (par exemple, un style SLD pour des points rouges).

Exemple de style SLD pour des points :
```xml
<StyledLayerDescriptor version="1.0.0" xmlns="http://www.opengis.net/sld">
  <NamedLayer>
    <Name>PointsInteret</Name>
    <UserStyle>
      <FeatureTypeStyle>
        <Rule>
          <PointSymbolizer>
            <Graphic>
              <Mark>
                <WellKnownName>circle</WellKnownName>
                <Fill>
                  <CssParameter name="fill">#FF0000</CssParameter>
                </Fill>
              </Mark>
              <Size>10</Size>
            </Graphic>
          </PointSymbolizer>
        </Rule>
      </FeatureTypeStyle>
    </UserStyle>
  </NamedLayer>
</StyledLayerDescriptor>
```

**Accéder à la couche** :
Testez le service WMS via une URL comme :
```
http://localhost:8080/geoserver/wms?service=WMS&version=1.1.0&request=GetMap&layers=votre_couche&bbox=-180,-90,180,90&width=800&height=600&srs=EPSG:4326&format=image/png
```

### MapServer
**Ajouter une couche dans un Mapfile** :
Pour publier un shapefile de points d’intérêt :
```mapfile
LAYER
  NAME "PointsInteret"
  TYPE POINT
  DATA "/chemin/vers/points.shp"
  STATUS ON
  CLASS
    STYLE
      SYMBOL "circle"
      SIZE 10
      COLOR 255 0 0
    END
  END
END
```

**Publier via WMS** :
Configurez MapServer pour répondre aux requêtes WMS en ajoutant dans le Mapfile :
```mapfile
WEB
  METADATA
    "wms_title" "Carte de Points d'Intérêt"
    "wms_onlineresource" "http://localhost/cgi-bin/mapserv?map=/chemin/vers/mapfile.map"
    "wms_srs" "EPSG:4326"
  END
END
```

> Conseil : Validez le Mapfile avec l’outil `shp2img` pour générer une image de test.

Vidéo utile : [Premiers pas avec MapServer](https://youtu.be/2y8Z7Y5rI4U)

## Requêtes et services géospatiaux

### GeoServer
GeoServer prend en charge les services OGC comme WMS (visualisation), WFS (données vectorielles) et WMTS (tuiles).

**Exemple de requête WMS** :
Obtenir une carte d’une couche :
```
http://localhost:8080/geoserver/wms?service=WMS&version=1.1.0&request=GetMap&layers=points_interet&bbox=2.2,48.8,2.4,48.9&width=800&height=600&srs=EPSG:4326&format=image/png
```

**Exemple de requête WFS** :
Récupérer des données vectorielles en GeoJSON :
```
http://localhost:8080/geoserver/wfs?service=WFS&version=1.0.0&request=GetFeature&typeName=points_interet&outputFormat=application/json
```

**Requête spatiale (filtre CQL)** :
Filtrer les points dans un rectangle :
```
http://localhost:8080/geoserver/wms?service=WMS&version=1.1.0&request=GetMap&layers=points_interet&bbox=2.2,48.8,2.4,48.9&width=800&height=600&srs=EPSG:4326&format=image/png&CQL_FILTER=BBOX(geom,2.2,48.8,2.4,48.9)
```

### MapServer
MapServer supporte également WMS, WFS et WMTS, configurés via le Mapfile.

**Exemple de requête WMS** :
```
http://localhost/cgi-bin/mapserv?map=/chemin/vers/mapfile.map&service=WMS&version=1.1.0&request=GetMap&layers=PointsInteret&bbox=2.2,48.8,2.4,48.9&width=800&height=600&srs=EPSG:4326&format=image/png
```

**Exemple de requête WFS** :
```
http://localhost/cgi-bin/mapserv?map=/chemin/vers/mapfile.map&service=WFS&version=1.0.0&request=GetFeature&typeName=PointsInteret&outputFormat=GeoJSON
```

**Requête spatiale (filtre)** :
Utiliser un filtre spatial dans une requête WFS :
```
http://localhost/cgi-bin/mapserv?map=/chemin/vers/mapfile.map&service=WFS&version=1.0.0&request=GetFeature&typeName=PointsInteret&filter=<Filter><BBOX><PropertyName>geom</PropertyName><Box srsName='EPSG:4326'><coordinates>2.2,48.8 2.4,48.9</ coordinates></Box></BBOX></Filter>&outputFormat=GeoJSON
```

> Conseil : Testez les services avec un client comme QGIS pour valider les rendus et les données.

Vidéo utile : [Configurer GeoServer pour WMS et WFS](https://youtu.be/0y2X1v2gW8k)

## Optimisation des performances

### GeoServer
- **Cache des tuiles** : Activez le module GeoWebCache (intégré à GeoServer) pour générer des tuiles WMTS :
```bash
http://localhost:8080/geoserver/gwc/service/wmts?REQUEST=GetTile&VERSION=1.0.0&LAYER=points_interet&TILEMATRIX=EPSG:4326:0&TILEMATRIXSET=EPSG:4326&TILECOL=0&TILEROW=0&FORMAT=image/png
```
- **Indexez les données** : Si vous utilisez PostGIS, créez des index spatiaux sur les colonnes géométriques.
- **Optimisez les styles** : Simplifiez les règles SLD pour réduire le temps de rendu.

### MapServer
- **Utilisez des index spatiaux** : Pour les shapefiles, générez un index avec `shp2tile` ou utilisez des bases comme PostGIS avec des index GIST.
- **Activez le cache** : Configurez MapServer avec un système de tuiles (par exemple, TileCache) pour les services WMTS.
- **Optimisez les Mapfiles** : Réduisez les couches inutiles et limitez les projections complexes.

> Conseil : Surveillez les logs (GeoServer : `geoserver/data/logs`; MapServer : logs du serveur web) pour identifier les goulots d’étranglement.

## Bonnes pratiques

- **Utilisez des standards OGC** : Assurez la compatibilité avec les clients SIG en respectant les normes WMS, WFS, et WMTS.
- **Validez les données** : Vérifiez la géométrie et le système de référence (par exemple, EPSG:4326) avant publication.
- **Testez les services** : Utilisez des outils comme QGIS, OpenLayers ou Postman pour valider les réponses des services.
- **Sécurisez l’accès** : Configurez des utilisateurs et des rôles dans GeoServer ou des restrictions d’accès dans le serveur web pour MapServer.
- **Documentez les configurations** : Gardez une trace des Mapfiles ou des styles SLD pour faciliter la maintenance.

Exemple de validation des données dans PostGIS avant publication :
```sql
SELECT nom, ST_IsValid(geom) AS est_valide
FROM points_interet;
```

Vidéo utile : [MapServer vs GeoServer : Comparaison et configuration](https://youtu.be/9r8Z7b9x0Q4)