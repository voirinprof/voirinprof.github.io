---
title: "Bonnes pratiques de programmation"
permalink: /fr/best-practices/
lang: fr
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: start
---

## Structurer un pipeline géomatique

Un pipeline géomatique enchaîne plusieurs étapes pour transformer des données sources en résultats reproductibles : chargement, traitement, analyse et export. Pour garder un projet facile à comprendre et à faire évoluer, on peut placer le point d'entrée dans un fichier `main.py` et séparer les étapes dans des modules Python dans le dossier `src/`.

Le dépôt [geo_pipeline_sample sur GitHub](https://github.com/voirinprof/geo_pipeline_sample) illustre cette organisation avec un exemple de données communales québécoises :

```text
geo_pipeline/
├── data/
│   ├── raw/                 # données sources
│   └── processed/           # données transformées
├── src/
│   ├── __init__.py
│   ├── ingest.py            # chargement des données
│   ├── process.py           # nettoyage, reprojection et filtrage
│   ├── analyze.py           # calculs et statistiques
│   └── export.py            # sauvegarde des résultats et de la carte
├── tests/                   # tests des modules
├── outputs/                 # résultats générés
├── config.yaml              # paramètres du pipeline
├── main.py                  # point d'entrée et orchestration
└── requirements.txt         # dépendances Python
```

Chaque module regroupe les fonctions d'une partie du pipeline. Par exemple, `ingest.py` charge les données, `process.py` les nettoie, les reprojette et peut les filtrer, `analyze.py` calcule des statistiques, puis `export.py` enregistre les données et produit une carte. Cette séparation permet de réutiliser et de tester les étapes indépendamment.

Le rôle de `main.py` est d'orchestrer ces étapes dans le bon ordre, sans y concentrer toute la logique de traitement. Dans cet extrait simplifié, les paramètres comme les chemins et la projection sont lus depuis `config.yaml` :

```python
from src.ingest import load_data
from src.process import clean_geometries, reproject
from src.analyze import compute_area_stats
from src.export import save_geodata, save_stats, save_map

gdf = load_data(url=data_url, cache_path=raw_path)
gdf = clean_geometries(gdf)
gdf = reproject(gdf, epsg=target_epsg)
gdf, stats = compute_area_stats(gdf, name_col=name_column)
save_geodata(gdf, path=processed_path)
save_stats(stats, path=stats_path)
save_map(gdf, column="aire_km2", path=map_path, title=map_title)
```

Dans un projet réel, les chemins, la projection et les autres paramètres peuvent être conservés dans `config.yaml` plutôt que codés en dur. On lance ensuite le pipeline depuis la racine du projet avec `python main.py`. Consultez le dépôt [geo_pipeline_sample](https://github.com/voirinprof/geo_pipeline_sample) pour voir l'implémentation complète, les paramètres et les tests.

## Centraliser les paramètres de l'application dans `config.yaml`

Une application a souvent des paramètres qui changent selon le projet ou l'environnement : chemins des données, URL d'une source, projection cible, filtres ou options d'export. Les conserver dans un fichier `config.yaml` évite de les disperser dans le code et de modifier les modules Python pour chaque nouvelle exécution. Le code reste ainsi plus lisible, et les paramètres peuvent être ajustés sans changer la logique de l'application.

Par exemple, un pipeline géomatique peut utiliser une configuration de ce type :

```yaml
data:
  url: "https://example.org/municipalities.geojson"
  raw_path: "data/raw/municipalities.geojson"
processing:
  target_epsg: 32198
  region_filter: null
export:
  processed_path: "data/processed/municipalities.geojson"
  stats_path: "outputs/area_stats.csv"
  map_path: "outputs/area_map.png"
```

Python peut lire le fichier avec PyYAML et `safe_load` :

```python
from pathlib import Path

import yaml

with Path("config.yaml").open(encoding="utf-8") as config_file:
    config = yaml.safe_load(config_file)

target_epsg = config["processing"]["target_epsg"]
```

Gardez dans cette configuration les paramètres, pas les secrets : mots de passe, clés d'API et autres identifiants doivent être fournis de façon sécurisée, par exemple au moyen de variables d'environnement, et ne doivent pas être ajoutés au dépôt. Validez également les valeurs obligatoires et les chemins au démarrage afin de signaler clairement une configuration incomplète ou incorrecte.

## Reproduire l'environnement de développement

Pour qu'un projet puisse être installé de façon cohérente sur plusieurs ordinateurs, documentez son environnement et ses dépendances. Si vous utilisez Conda, un fichier `environment.yaml` permet de recréer l'environnement de base du projet :

```yaml
name: geo-quality-demo
channels:
  - conda-forge
dependencies:
  - python=3.11
  - pip
  - pip:
      - "-e .[dev]"
```

À partir de la racine du projet, créez et activez l'environnement avec :

```shell
conda env create -f environment.yaml
conda activate geo-quality-demo
```

Dans cet exemple, `pip install -e ".[dev]"` installe le projet en mode développement ainsi que ses outils de développement, déclarés dans `pyproject.toml`. Gardez les dépendances et les paramètres des outils dans ce fichier central pour faciliter leur installation et leur maintenance. Le dépôt [geo_quality_demo](https://github.com/voirinprof/geo_quality_demo) illustre cette approche; il utilise `pyproject.toml` et propose également l'installation avec `pip install -e ".[dev]"`.

## Nommage, commentaires et docstrings

Des conventions cohérentes rendent le code plus facile à lire et à maintenir. En Python, suivez les recommandations de [PEP 8](https://peps.python.org/pep-0008/) et choisissez des noms explicites :

* Les variables, fonctions et méthodes utilisent `snake_case` : `target_epsg`, `load_data()`.
* Les classes utilisent `CapWords` : `GeoDataProcessor`.
* Les constantes utilisent des majuscules avec des tirets bas : `DEFAULT_EPSG`.
* Les modules et packages utilisent des noms courts en minuscules; les mots peuvent être séparés par des tirets bas : `data_loader.py`.
* Un tiret bas initial, comme dans `_internal_helper()`, indique par convention un élément destiné à un usage interne.

Évitez les noms trop vagues (`x`, `data`, `process`) lorsqu'un nom plus précis peut expliquer le rôle de la valeur ou de la fonction. Évitez aussi de réutiliser les noms de fonctions intégrées comme `list`, `id` ou `str`.

Les commentaires doivent apporter une information utile que le code n'exprime pas déjà. Expliquez surtout pourquoi une décision ou un traitement particulier est nécessaire; évitez de paraphraser chaque ligne. Gardez les commentaires à jour lorsque le code change.

Les docstrings décrivent le rôle et l'utilisation d'un module, d'une classe ou d'une fonction. Écrivez-les entre triples guillemets doubles, placez-les juste après la définition et documentez les paramètres, la valeur retournée et les exceptions utiles lorsque cela clarifie l'interface :

```python
def filter_by_region(gdf, region_column, region_name):
    """Filtre les entités selon leur région administrative.

    Args:
        gdf: GeoDataFrame contenant les entités à filtrer.
        region_column: Nom de la colonne qui contient les régions.
        region_name: Région à conserver.

    Returns:
        Un GeoDataFrame contenant les entités de la région demandée.
    """
    return gdf[gdf[region_column] == region_name].copy()
```

Documentez les éléments publics et les comportements qui ne sont pas évidents. Une bonne docstring complète le nom et la signature de la fonction, mais ne les répète pas simplement.

## Vérifier la qualité du code

Quelques outils automatisés aident à garder un projet lisible, documenté et fiable :

* **Black** applique un formatage uniforme au code Python.
* **Ruff** détecte des erreurs courantes et vérifie notamment les imports.
* **interrogate** mesure la couverture du code par des docstrings.
* **pytest** exécute les tests unitaires et aide à vérifier les résultats du pipeline.

Configurez ces outils dans `pyproject.toml`, puis exécutez-les depuis la racine du projet :

```shell
black --check .
ruff check .
interrogate
pytest -v
```

`black --check .` vérifie le formatage sans modifier les fichiers. Pour appliquer le formatage, utilisez `black .`. Ruff peut aussi corriger automatiquement certains problèmes avec `ruff check --fix .`; examinez les changements avant de les conserver.

Pour automatiser ces contrôles, vous pouvez les exécuter à chaque commit avec `pre-commit` et dans une intégration continue comme GitHub Actions, à chaque *push* ou *pull request*. Le dépôt [geo_quality_demo](https://github.com/voirinprof/geo_quality_demo) fournit un exemple de configuration de ces outils, de tests et de vérifications automatisées.