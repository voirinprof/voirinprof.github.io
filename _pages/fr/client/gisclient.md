---
title: "QGIS et Python"
permalink: /fr/qgispython/
lang: fr
date: 2026-10-16T08:57:37-04:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: client
---

## Introduction

QGIS peut être étendu à l’aide de plugins Python. Un plugin peut ajouter une commande dans un menu ou une barre d’outils, ouvrir une interface personnalisée ou fournir des algorithmes géomatiques utilisables dans la boîte à outils de traitements. Cette page présente les principaux éléments d’un plugin et aide à choisir une approche.

Il est utile de connaître les bases de Python et d’avoir QGIS Desktop installé. QGIS fournit son propre environnement Python et les modules `qgis` : développez et testez les plugins avec l’interpréteur associé à la même installation de QGIS.

## Développer dans QGIS

### Commencer avec la console Python

La console Python intégrée à QGIS permet d’explorer l’API `qgis` et de tester rapidement du code dans le projet ouvert. L’objet `iface` donne accès à l’interface de l’application, notamment au canevas cartographique, aux menus, aux barres d’outils et aux couches actives.

```python
layer = iface.activeLayer()
if layer:
    print(layer.name(), layer.featureCount())
```

Quand le principe fonctionne, le code peut être déplacé dans un plugin. Dans la mesure du possible, séparez l’interface, les traitements SIG et l’accès aux données : le code sera plus facile à tester et à maintenir.

### Structure et cycle de vie d’un plugin

Un plugin Python est un dossier installé dans le répertoire `python/plugins` du profil QGIS. Un plugin minimal contient généralement :

- `metadata.txt`, qui définit notamment le nom, la description, la version et les versions de QGIS compatibles ;
- `__init__.py`, qui contient une fonction `classFactory(iface)` appelée par QGIS pour créer le plugin ;
- un module Python principal qui contient la classe du plugin ;
- au besoin, d’autres modules, des icônes, des traductions et des fichiers d’interface Qt Designer (`.ui`).

La classe du plugin reçoit généralement `iface` dans son constructeur. QGIS appelle ensuite `initGui()` pour créer et enregistrer les actions ou éléments d’interface. Lorsque le plugin est désactivé ou que QGIS se ferme, `unload()` doit retirer ces éléments et libérer les ressources. Un nettoyage incomplet peut laisser des boutons ou des entrées de menu après un rechargement du plugin pendant le développement.

Pour les imports Qt, privilégiez `qgis.PyQt` (par exemple `from qgis.PyQt.QtWidgets import QDialog`) afin d’utiliser la version de Qt fournie avec QGIS.

L’exemple simplifié suivant montre le point d’entrée et le cycle de vie du plugin :

```python
# __init__.py
def classFactory(iface):
    from .mon_plugin import MonPlugin
    return MonPlugin(iface)
```

```python
# mon_plugin.py
from qgis.PyQt.QtGui import QAction


class MonPlugin:
    def __init__(self, iface):
        self.iface = iface
        self.action = None

    def initGui(self):
        self.action = QAction("Mon outil", self.iface.mainWindow())
        self.action.triggered.connect(self.run)
        self.iface.addPluginToMenu("&Mon plugin", self.action)
        self.iface.addToolBarIcon(self.action)

    def run(self):
        print("L’action du plugin a été déclenchée")

    def unload(self):
        self.iface.removePluginMenu("&Mon plugin", self.action)
        self.iface.removeToolBarIcon(self.action)
        self.action = None
```

Ce code est un point de départ : un vrai plugin devrait traduire ses libellés, gérer les erreurs et éviter d’utiliser `print()` pour communiquer avec l’utilisateur.

### Créer, installer et tester un plugin

Vous pouvez créer vous-même le dossier et les fichiers de métadonnées, ou utiliser le plugin communautaire **Plugin Builder** pour générer une structure de départ. Installez une copie de développement dans le dossier `python/plugins` du profil QGIS actif, activez-la dans **Extensions → Installer/Gérer les extensions**, puis rechargez-la après vos modifications. Le plugin communautaire **Plugin Reloader** peut accélérer ce cycle.

Testez différents projets et différentes couches, y compris des entrées invalides, et surveillez les erreurs dans la console Python de QGIS. Avant de partager le plugin, vérifiez ses métadonnées et empaquetez son dossier en ZIP à l’aide des outils de création de paquet de plugins QGIS.

## Les plugins

Choisissez le type de plugin selon la manière dont les utilisateurs interagiront avec la fonctionnalité. Ces approches peuvent aussi être combinées : un plugin peut, par exemple, proposer un panneau interactif et enregistrer un ou plusieurs algorithmes Processing.

### Une commande qui ouvre une boîte de dialogue

Un **plugin avec boîte de dialogue** ajoute une action dans un menu ou une barre d’outils. Un clic ouvre une fenêtre `QDialog` contenant des contrôles, comme des champs, des boutons et des messages. Cette approche convient à une tâche interactive et ponctuelle que l’utilisateur lance et termine en une courte session.

Par exemple, la boîte de dialogue peut permettre de choisir une couche et une distance, puis de créer une zone tampon. Qt Designer permet de concevoir l’interface dans un fichier `.ui`; le code Python connecte les boutons et appelle l’API QGIS. Utilisez les classes Qt de `qgis.PyQt.QtWidgets` et les classes SIG de QGIS, comme `QgsVectorLayer`.

### Un panneau dockable basé sur `QWidget`

Un **plugin avec panneau** fournit une interface qui reste disponible à côté du canevas cartographique. Elle est généralement construite à partir d’un `QWidget` placé dans un `QDockWidget`, que QGIS ajoute à l’un des côtés de sa fenêtre. Cette approche convient aux outils que l’on consulte pendant que l’on navigue dans la carte, par exemple un explorateur de couches, des commandes pour un workflow ou un panneau d’état.

Contrairement à une boîte de dialogue modale, un panneau dockable peut rester ouvert pendant que l’utilisateur manipule la carte et les autres panneaux de QGIS. Veillez à ne pas bloquer l’interface pendant les traitements longs; actualisez le panneau lorsque le projet ou la sélection change. Dans `unload()`, retirez le panneau et déconnectez les signaux auxquels le plugin s’est abonné.

### Un algorithme ou un fournisseur Processing

Un **algorithme Processing** est adapté lorsque la tâche principale transforme des données avec des entrées et des sorties définies — par exemple, filtrer des entités, calculer un attribut ou créer un raster. Implémentez-le avec `QgsProcessingAlgorithm` et déclarez ses paramètres d’entrée et de sortie. L’algorithme est alors accessible dans la boîte à outils de traitements et peut, selon le cas, servir dans un traitement par lots ou dans le modeleur graphique.

Pour un outil personnel simple, un script Processing peut suffire. Pour distribuer un ensemble d’algorithmes, créez un plugin qui enregistre un fournisseur Processing dans la boîte à outils. Un fournisseur n’est pas nécessaire pour un script isolé.

Les algorithmes doivent utiliser le contexte et l’objet de suivi (`feedback`) fournis par Processing, signaler leur progression et tenir compte des demandes d’annulation. Ils resteront ainsi utilisables dans les différentes interfaces Processing de QGIS sans dépendre d’une boîte de dialogue personnalisée.

## Documentation officielle et ressources

- [Développer des extensions Python — PyQGIS Developer Cookbook](https://docs.qgis.org/latest/fr/docs/pyqgis_developer_cookbook/plugins/index.html) : structure, métadonnées et cycle de vie des plugins.
- [Écrire un plugin Processing — PyQGIS Developer Cookbook](https://docs.qgis.org/latest/fr/docs/pyqgis_developer_cookbook/processing.html) : algorithmes et fournisseurs Processing.
- [PyQGIS Developer Cookbook](https://docs.qgis.org/latest/fr/docs/pyqgis_developer_cookbook/) : exemples d’utilisation de l’API Python de QGIS.
- [Référence de l’API PyQGIS](https://qgis.org/pyqgis/) : référence des classes Python de QGIS.
- [Référence de `QgisInterface`](https://qgis.org/pyqgis/master/gui/QgisInterface.html) : méthodes permettant aux plugins d’interagir avec l’application QGIS.
- [Référence de `QgsProcessingAlgorithm`](https://qgis.org/pyqgis/master/core/QgsProcessingAlgorithm.html) : classe de base des algorithmes Processing.
- [Console Python de QGIS — Manuel utilisateur](https://docs.qgis.org/latest/fr/docs/user_manual/plugins/python_console.html) : utilisation de la console intégrée.
- [Dépôt des extensions Python de QGIS](https://plugins.qgis.org/) : rechercher, installer et examiner des plugins, y compris des outils communautaires de développement.

L’API PyQGIS et l’environnement des plugins évoluent selon les versions de QGIS. Consultez la documentation et la référence API correspondant à la version que vous souhaitez prendre en charge, puis testez le plugin avec cette version.