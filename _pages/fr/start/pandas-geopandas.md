---
title: "Fiche pratique pandas et GeoPandas"
permalink: /fr/pandas-geopandas/
lang: fr
date: 2026-10-16T08:45:34-04:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: start
---

Cette fiche rassemble les opérations les plus utiles de **pandas** pour manipuler des tableaux et de **GeoPandas** pour analyser des données vectorielles. Les exemples supposent que les bibliothèques sont installées (`pip install pandas geopandas`) et que les fichiers d’entrée existent.

```python
import pandas as pd
import geopandas as gpd
```

## pandas : manipuler des tableaux

### Lire, écrire et inspecter

```python
df = pd.read_csv("donnees.csv")
df = pd.read_excel("donnees.xlsx")

df.head()                    # premières lignes
df.info()                    # colonnes et types
df.describe()                # statistiques numériques
df.shape                     # nombre de lignes et de colonnes
df.columns                   # noms des colonnes
df.to_csv("resultat.csv", index=False)
```

Pour d’autres formats, pandas propose notamment `read_json()` et `read_parquet()`.

### Sélectionner et filtrer

```python
df["population"]                         # une colonne (Series)
df[["nom", "population"]]                # plusieurs colonnes (DataFrame)
df.loc[df["population"] > 1000]          # filtrer par condition
df.loc[df["population"] > 1000, ["nom", "population"]]
df.iloc[0:5, 0:3]                        # sélection par positions
df[df["nom"].str.contains("lac", case=False, na=False)]
```

Utilisez `.loc` pour sélectionner par étiquettes ou conditions et `.iloc` pour sélectionner par positions.

### Créer, modifier et nettoyer des colonnes

```python
df["densite"] = df["population"] / df["superficie_km2"]
df = df.rename(columns={"nom_zone": "nom"})
df = df.drop(columns=["colonne_inutile"])
df = df.dropna(subset=["population"])    # retirer les lignes sans population
df["population"] = df["population"].fillna(0)
df["date"] = pd.to_datetime(df["date"], errors="coerce")
```

### Trier, résumer et combiner

```python
df = df.sort_values("population", ascending=False)

resume = (
    df.groupby("region", as_index=False)
      .agg(population_totale=("population", "sum"),
           population_moyenne=("population", "mean"),
           nombre=("nom", "count"))
)

resultat = df.merge(table_regions, on="region", how="left")
```

`merge()` combine les tables à partir d’une clé. Vérifiez que cette clé a le même type et le même sens dans chaque table; précisez `validate=` si vous connaissez la cardinalité attendue.

## GeoPandas : manipuler des données vectorielles

Un `GeoDataFrame` est un tableau pandas qui contient aussi une colonne de géométries et un système de coordonnées (CRS).

### Lire, inspecter et écrire

```python
gdf = gpd.read_file("zones.gpkg")

gdf.head()
gdf.columns
gdf.crs                         # CRS des coordonnées
gdf.geometry                    # colonne de géométries active
gdf.geom_type.value_counts()    # types de géométrie
gdf.total_bounds                # xmin, ymin, xmax, ymax

gdf.to_file("resultat.gpkg", driver="GPKG")
```

GeoPandas lit et écrit notamment les fichiers GeoPackage, Shapefile et GeoJSON. Un CSV avec des coordonnées peut être converti en `GeoDataFrame` :

```python
points = gpd.GeoDataFrame(
    df,
    geometry=gpd.points_from_xy(df["longitude"], df["latitude"]),
    crs="EPSG:4326",
)
```

### Vérifier et reprojeter les coordonnées

```python
gdf = gdf.to_crs("EPSG:32198")  # reprojeter vers un CRS adapté à l’analyse
```

`set_crs()` déclare le CRS des coordonnées existantes; il ne transforme pas les coordonnées. `to_crs()` effectue la transformation et nécessite que le CRS source soit connu. Pour calculer des distances ou des surfaces, utilisez un CRS projeté adapté à votre région, pas des coordonnées longitude/latitude.

### Filtrer et calculer avec les géométries

```python
selection = gdf[gdf["population"] > 1000]
selection = gdf[gdf.geometry.notna() & ~gdf.geometry.is_empty]

gdf["aire_m2"] = gdf.geometry.area
gdf["longueur_m"] = gdf.geometry.length
gdf["centroide"] = gdf.geometry.centroid
gdf["zone_tampon"] = gdf.geometry.buffer(500)
```

Les surfaces, longueurs et distances sont exprimées dans les unités du CRS. Les géométries doivent donc être dans un CRS projeté approprié pour obtenir des résultats métriques fiables.

### Effectuer les opérations spatiales courantes

```python
zones = gpd.read_file("zones.gpkg")
zone_etude = gpd.read_file("zone_etude.gpkg").to_crs(zones.crs)

# Associer chaque point au polygone qui le contient.
associes = gpd.sjoin(
    points.to_crs(zones.crs),
    zones[["nom_zone", "geometry"]],
    how="left",
    predicate="within",
)

# Regrouper des polygones selon une catégorie.
zones_fusionnees = zones.dissolve(by="region", as_index=False)

# Garder les portions des entités situées dans une zone d'étude.
decoupees = gpd.clip(gdf, zone_etude)

# Combiner les attributs et géométries selon le chevauchement.
intersection = gpd.overlay(couche_a, couche_b, how="intersection")
```

Pour `sjoin()`, `clip()` et `overlay()`, les couches doivent utiliser le même CRS. Le prédicat de `sjoin()` (`within`, `intersects`, `contains`, etc.) définit la relation spatiale recherchée. `dissolve()` fusionne les géométries d’un même groupe; `clip()` découpe selon une limite; `overlay()` réalise une opération topologique entre deux couches.

### Visualiser rapidement

```python
gdf.plot(column="population", cmap="OrRd", legend=True)
```

`plot()` produit une carte statique avec Matplotlib. Pour explorer une carte interactive directement dans un notebook, essayez `gdf.explore()`.

## À retenir

- Examinez les colonnes et leurs types avant de lancer une analyse.
- Utilisez `merge()` pour les jointures par attribut et `sjoin()` pour les jointures spatiales.
- Contrôlez le CRS avant toute opération géométrique; reprojetez les couches au besoin.
- Vérifiez les géométries vides ou invalides ainsi que les valeurs manquantes.
- Sauvegardez les données géospatiales dans un format adapté, par exemple GeoPackage.

## Documentation officielle

- [Documentation pandas](https://pandas.pydata.org/docs/)
- [Documentation GeoPandas](https://geopandas.org/en/stable/)
