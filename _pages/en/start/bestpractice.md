---
title: "Programming Best Practices"
permalink: /en/best-practices/
lang: en
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: start
---

## Structuring a Geospatial Pipeline

A geospatial pipeline chains together several steps to transform source data into reproducible results: loading, processing, analysis, and export. To keep a project easy to understand and maintain, use a `main.py` file as the entry point and separate each step into Python modules in a `src/` directory.

The [geo_pipeline_sample repository on GitHub](https://github.com/voirinprof/geo_pipeline_sample) demonstrates this structure with an example using data about municipalities in Quebec:

```text
geo_pipeline/
├── data/
│   ├── raw/                 # source data
│   └── processed/           # transformed data
├── src/
│   ├── __init__.py
│   ├── ingest.py            # load data
│   ├── process.py           # clean, reproject, and filter data
│   ├── analyze.py           # calculations and statistics
│   └── export.py            # save results and create a map
├── tests/                   # module tests
├── outputs/                 # generated results
├── config.yaml              # pipeline settings
├── main.py                  # entry point and orchestration
└── requirements.txt         # Python dependencies
```

Each module groups the functions for one part of the pipeline. For example, `ingest.py` loads the data; `process.py` cleans, reprojects, and can filter it; `analyze.py` calculates statistics; and `export.py` saves the data and creates a map. This separation makes each step easier to reuse and test independently.

The role of `main.py` is to orchestrate these steps in the right order, rather than holding all the processing logic itself. In this simplified example, settings such as paths and the target projection are read from `config.yaml`:

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

In a real project, paths, the target projection, and other settings can be stored in `config.yaml` instead of being hard-coded. Run the pipeline from the project root with `python main.py`. Visit the [geo_pipeline_sample repository](https://github.com/voirinprof/geo_pipeline_sample) to explore the complete implementation, configuration, and tests.

## Centralizing Application Settings in `config.yaml`

Applications often have settings that vary by project or environment, such as data paths, a source URL, the target projection, filters, or export options. Keeping these in a `config.yaml` file avoids scattering them throughout the code and having to edit Python modules for each run. This keeps the code easier to read and lets you adjust settings without changing the application's logic.

For example, a geospatial pipeline might use a configuration like this:

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

Python can read the file with PyYAML and `safe_load`:

```python
from pathlib import Path

import yaml

with Path("config.yaml").open(encoding="utf-8") as config_file:
    config = yaml.safe_load(config_file)

target_epsg = config["processing"]["target_epsg"]
```

Keep settings in this file, not secrets: passwords, API keys, and other credentials should be provided securely, for example through environment variables, and should not be committed to the repository. Also validate required values and paths at startup so that incomplete or invalid configuration is reported clearly.

## Reproducible Development Environments

To make a project easier to set up consistently on different computers, document its environment and dependencies. If you use Conda, an `environment.yaml` file can define the project's base environment:

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

From the project root, create and activate the environment with:

```shell
conda env create -f environment.yaml
conda activate geo-quality-demo
```

In this example, `pip install -e ".[dev]"` installs the project in editable mode along with its development tools, declared in `pyproject.toml`. Keep dependencies and tool settings in this central file to make them easier to install and maintain. The [geo_quality_demo repository](https://github.com/voirinprof/geo_quality_demo) demonstrates this approach with `pyproject.toml` and also documents installation using `pip install -e ".[dev]"`.

## Naming, Comments, and Docstrings

Consistent conventions make code easier to read and maintain. In Python, follow the recommendations in [PEP 8](https://peps.python.org/pep-0008/) and choose descriptive names:

* Use `snake_case` for variables, functions, and methods: `target_epsg`, `load_data()`.
* Use `CapWords` for classes: `GeoDataProcessor`.
* Use uppercase letters with underscores for constants: `DEFAULT_EPSG`.
* Use short, lowercase names for modules and packages; separate words with underscores when needed: `data_loader.py`.
* A leading underscore, as in `_internal_helper()`, conventionally marks an element intended for internal use.

Avoid vague names (`x`, `data`, `process`) when a more precise name can explain the role of a value or function. Also avoid reusing built-in names such as `list`, `id`, or `str`.

Comments should add information that the code does not already express. In particular, explain why a decision or special handling is needed; avoid paraphrasing every line. Keep comments up to date when the code changes.

Docstrings describe the purpose and usage of a module, class, or function. Write them using triple double quotes, place them immediately after the definition, and document parameters, return values, and relevant exceptions when this clarifies the interface:

```python
def filter_by_region(gdf, region_column, region_name):
    """Filter features by administrative region.

    Args:
        gdf: GeoDataFrame containing the features to filter.
        region_column: Name of the column containing region values.
        region_name: Region to keep.

    Returns:
        A GeoDataFrame containing features in the requested region.
    """
    return gdf[gdf[region_column] == region_name].copy()
```

Document public elements and behavior that is not obvious. A good docstring complements the function's name and signature rather than simply repeating them.

## Checking Code Quality

Automated tools help keep a project readable, documented, and reliable:

* **Black** applies consistent formatting to Python code.
* **Ruff** detects common issues and checks, among other things, imports.
* **interrogate** measures docstring coverage.
* **pytest** runs unit tests and helps verify pipeline results.

Configure these tools in `pyproject.toml`, then run them from the project root:

```shell
black --check .
ruff check .
interrogate
pytest -v
```

`black --check .` checks formatting without changing files. To apply formatting, run `black .`. Ruff can also automatically fix some issues with `ruff check --fix .`; review the changes before keeping them.

You can automate these checks with `pre-commit` on every commit and in a continuous integration workflow such as GitHub Actions on each *push* or *pull request*. The [geo_quality_demo repository](https://github.com/voirinprof/geo_quality_demo) provides an example of configuring these tools, tests, and automated checks.
