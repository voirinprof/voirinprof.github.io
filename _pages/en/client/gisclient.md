---
title: "QGIS and Python"
permalink: /en/qgispython/  
lang: en  
date: 2026-10-16T08:54:00-04:00
toc: true  
toc_sticky: true  
toc_label: "In this page"  
typenav: client  
---

## Introduction

QGIS can be extended with Python plugins. A plugin can add a button or menu command, open a custom interface, or provide geoprocessing algorithms that work with the Processing Toolbox. This page introduces the main building blocks and helps you choose an approach.

You should be comfortable with basic Python and have QGIS Desktop installed. QGIS includes its own Python environment and the `qgis` modules; develop and test plugins with the Python interpreter provided by the same QGIS installation.

## Developing in QGIS

### Start small and use the Python console

The QGIS Python Console is a convenient place to explore the `qgis` API and test short snippets against the currently open project. The `iface` object gives access to the QGIS application interface, including the map canvas, menus, toolbars, and active layers.

```python
layer = iface.activeLayer()
if layer:
    print(layer.name(), layer.featureCount())
```

Once the idea works, move the code into a plugin. Keep the user interface, GIS operations, and data access separate where practical; this makes the code easier to test and maintain.

### Plugin structure and lifecycle

A Python plugin is a folder in the QGIS profile's `python/plugins` directory. A minimal plugin typically contains:

- `metadata.txt` for its name, description, version, QGIS compatibility, and other plugin information;
- `__init__.py` with a `classFactory(iface)` function, which QGIS calls to create the plugin;
- a main Python module containing the plugin class;
- optional modules, icons, translations, and Qt Designer `.ui` files.

The plugin class usually receives `iface` in its constructor. QGIS then calls `initGui()` to create and register actions or interface elements. When the plugin is disabled or QGIS closes, `unload()` should remove those elements and release resources. Clean shutdown matters: otherwise toolbar buttons or menu entries may be left behind after reloading during development.

For Qt imports, prefer `qgis.PyQt` (for example, `from qgis.PyQt.QtWidgets import QDialog`) so the plugin uses the Qt version bundled with QGIS.

The following abbreviated example shows the entry point and the plugin lifecycle:

```python
# __init__.py
def classFactory(iface):
    from .my_plugin import MyPlugin
    return MyPlugin(iface)
```

```python
# my_plugin.py
from qgis.PyQt.QtGui import QAction


class MyPlugin:
    def __init__(self, iface):
        self.iface = iface
        self.action = None

    def initGui(self):
        self.action = QAction("My tool", self.iface.mainWindow())
        self.action.triggered.connect(self.run)
        self.iface.addPluginToMenu("&My plugin", self.action)
        self.iface.addToolBarIcon(self.action)

    def run(self):
        print("The plugin action was triggered")

    def unload(self):
        self.iface.removePluginMenu("&My plugin", self.action)
        self.iface.removeToolBarIcon(self.action)
        self.action = None
```

This is only a starting point: a real plugin should provide translated labels, handle errors, and avoid using `print()` as its user-facing feedback.

### Create, install, and test a plugin

You can create the folder and metadata yourself or use the community **Plugin Builder** plugin to generate a starter structure. Install a development copy into the active QGIS profile's `python/plugins` folder, enable it in **Plugins → Manage and Install Plugins**, and reload it after changes. The community **Plugin Reloader** plugin can speed up this development cycle.

Test with different projects and layers, check both successful and invalid inputs, and watch the QGIS Python Console for errors. Before sharing a plugin, check its metadata and package the plugin folder as a ZIP using QGIS's plugin packaging tools.

## Plugins

Choose the plugin style based on how people will use the functionality. These patterns can also be combined: for example, a plugin can offer a dock panel for interaction and register one or more Processing algorithms.

### A command that opens a dialog

A **dialog-based plugin** adds an action to a menu or toolbar. Clicking it opens a `QDialog` containing controls such as input fields, buttons, and messages. It is a good fit for a focused, interactive task that users start explicitly and complete in one short session.

For example, a dialog could let the user choose a layer and a distance, then create a buffer. Qt Designer can be used to lay out the interface in a `.ui` file; the plugin's Python code handles the buttons and calls the QGIS API. Use `qgis.PyQt.QtWidgets` for Qt classes and QGIS classes such as `QgsVectorLayer` for GIS data.

### A dockable `QWidget`

A **widget-based plugin** provides a panel that remains available alongside the map canvas. The interface is commonly built from a `QWidget` placed inside a `QDockWidget`, which QGIS adds to a side of the main window. This works well for tools users revisit while navigating the map, such as a layer browser, a set of workflow controls, or a live status panel.

Unlike a modal dialog, a dock panel can stay open while users interact with the map and other QGIS panels. Keep long-running work responsive and update the widget when the current project or selection changes. In the plugin's `unload()` method, remove the dock widget and disconnect any signals that the plugin connected.

### A Processing algorithm or provider

A **Processing algorithm** is the right choice when the core task transforms data from defined inputs to outputs—for example, filtering features, calculating an attribute, or creating a raster. Implement the algorithm with `QgsProcessingAlgorithm` and declare its inputs and outputs with Processing parameters. The algorithm can then be run from the Processing Toolbox and, where appropriate, used in batch processing or the graphical modeler.

For a quick, personal tool, a Processing script may be enough. To distribute a group of algorithms, create a plugin with a Processing provider that registers them in the Processing Toolbox. A provider is not required for a single standalone script.

Processing algorithms should use the supplied processing context and feedback, report progress, and handle cancellation where relevant. This keeps algorithms usable in different QGIS Processing interfaces instead of tying them to a custom dialog.

## Official documentation and resources

- [Developing Python Plugins — PyQGIS Developer Cookbook](https://docs.qgis.org/latest/en/docs/pyqgis_developer_cookbook/plugins/index.html): plugin structure, metadata, and lifecycle.
- [Writing a Processing plugin — PyQGIS Developer Cookbook](https://docs.qgis.org/latest/en/docs/pyqgis_developer_cookbook/processing.html): algorithms and Processing providers.
- [PyQGIS Developer Cookbook](https://docs.qgis.org/latest/en/docs/pyqgis_developer_cookbook/): examples of the QGIS Python API.
- [PyQGIS API reference](https://qgis.org/pyqgis/): reference for QGIS Python classes.
- [`QgisInterface` API reference](https://qgis.org/pyqgis/master/gui/QgisInterface.html): methods plugins can use to interact with the QGIS application.
- [`QgsProcessingAlgorithm` API reference](https://qgis.org/pyqgis/master/core/QgsProcessingAlgorithm.html): base class for Processing algorithms.
- [QGIS Python Console — User Guide](https://docs.qgis.org/latest/en/docs/user_manual/plugins/python_console.html): using the built-in console.
- [QGIS Python Plugins repository](https://plugins.qgis.org/): find, install, and inspect plugins, including community development tools.

The PyQGIS API and plugin environment can change between QGIS versions. Use the documentation and API reference matching the QGIS version you intend to support, and test against that version.