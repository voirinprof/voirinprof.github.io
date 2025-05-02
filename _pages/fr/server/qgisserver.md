---
title: "Introduction à QGIS Server et Python"
permalink: /fr/qgis-server-intro/
lang: fr
date: 2025-04-28T21:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: gisserver
---

> **_NOTE:_** Pour des informations officielles, consultez la [documentation de QGIS Server](https://docs.qgis.org/3.34/fr/docs/server_manual/index.html). Un exemple de service web `Flask` avec `Qgis Server` peut être trouvé dans ce repo : [https://github.com/voirinprof/qgis_docker](https://github.com/voirinprof/qgis_docker)

## Introduction

**QGIS Server** est la déclinaison serveur de QGIS qui permet de publier des services cartographiques **WMS**, **WFS**, **WCS** directement à partir de projets `.qgs` sans développement lourd.  
En combinant **QGIS Server** avec **Python**, on peut automatiser, personnaliser et créer des services SIG web dynamiques.

---

## Pourquoi utiliser QGIS Server ?

- **Simplicité** : Publier un projet QGIS tel quel en tant que service web
- **Normes OGC** respectées : WMS 1.3, WFS 2.0, etc.
- **Personnalisation avancée** via plugins serveur en Python
- **Mêmes capacités de rendu** que le bureau QGIS
- **Intégration directe avec PostGIS**, GeoTIFF, et bien plus

---

## Installer QGIS Server

### Sous Ubuntu / Debian :

```shell
sudo apt-get install qgis-server
```

Services nécessaires :
- Serveur web (Apache, Nginx)
- FastCGI ou uwsgi pour interfacer QGIS Server et le webserver

### Vérification de l'installation :

Tester l'URL locale (par exemple avec Apache) :

```
http://localhost/qgisserver?SERVICE=WMS&REQUEST=GetCapabilities
```

Si tout est bien configuré, vous obtiendrez une réponse XML décrivant vos capacités serveur.

---

## Déployer un projet QGIS en tant que service WMS

1. **Préparer votre projet `.qgs`** :
   - Définissez le SCR (système de coordonnées de référence)
   - Configurez les propriétés du projet (noms de couche, échelles min/max)

2. **Placer le fichier projet** :
   - Sauvegardez votre `.qgs` dans un dossier accessible par le serveur (ex : `/var/www/qgis/monprojet.qgs`).

3. **Appeler votre projet via l'URL** :

```text
http://localhost/qgisserver?MAP=/var/www/qgis/monprojet.qgs&SERVICE=WMS&REQUEST=GetCapabilities
```

---

## Utiliser Python avec QGIS Server

Python permet de **créer des extensions serveur personnalisées** ou **interagir avec QGIS Server**. Deux grandes approches existent :

### 1. **Créer des plugins pour QGIS Server**

- Plugins identiques en structure aux plugins QGIS Desktop, mais avec un point d'entrée spécifique : `server` au lieu de `gui`.
- Permet :
  - Modification dynamique des couches
  - Ajout de nouvelles opérations au service
  - Filtrage personnalisé des données

**Exemple d'architecture de plugin serveur** :

```
mon_plugin/
├── __init__.py
├── server/
│   ├── __init__.py
│   ├── monplugin.py
├── metadata.txt
```

Dans `server/monplugin.py` :

```python
from qgis.server import QgsServerFilter

class MonPluginServer(QgsServerFilter):
    def requestReady(self):
        print("Requête reçue :", self.serverInterface().requestHandler().parameterMap())
```

### 2. **Utiliser PyQGIS pour pré-traiter vos projets**

Avec **PyQGIS** (l'API Python de QGIS), vous pouvez générer dynamiquement des `.qgs`, préparer les styles, les couches PostGIS, avant la publication sur QGIS Server.

**Exemple simple de script PyQGIS** :

```python
from qgis.core import QgsProject

# Charger un projet existant
projet = QgsProject.instance()
projet.read('/path/to/mon_projet.qgs')

# Modifier dynamiquement un titre
projet.title = "Nouveau titre serveur"

# Sauvegarder
projet.write()
```

---

## Cas d'usage typiques

- **Génération automatique de services SIG thématiques**
- **Application de règles d'accès aux données via des plugins serveur**
- **Personnalisation de GetFeatureInfo** (réponse JSON enrichie)
- **Rendu conditionnel selon les utilisateurs / sessions**

---

## Ressources utiles

- [Documentation QGIS Server](https://docs.qgis.org/3.34/fr/docs/server_manual/)
- [Développement de plugins QGIS Server (PyQGIS Cookbook)](https://docs.qgis.org/3.34/en/docs/pyqgis_developer_cookbook/server/index.html)
- [Exemple de plugin serveur officiel](https://github.com/qgis/QGIS-Server-Plugin-Examples)

---

## Bonnes pratiques

- **Optimisez vos projets `.qgs`** (styles légers, groupes logiques)
- **Utilisez PostGIS** pour les grandes bases de données spatiales
- **Cachez vos services** (avec MapProxy ou QGIS Cache) pour accélérer les réponses
- **Surveillez les logs serveur** pour le débogage de plugins Python
- **Automatisez la mise à jour des projets avec PyQGIS**