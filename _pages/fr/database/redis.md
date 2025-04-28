---
title: "Python avec Redis"
permalink: /fr/python-redis/
lang: fr
date: 2025-04-25T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: database
---

> **_NOTE:_** La [documentation officielle de Redis](https://redis.io/docs/latest/) et celle de la bibliothèque Python [redis-py](https://redis-py.readthedocs.io/) fournissent des instructions détaillées pour aller plus loin.

## Introduction à Redis

**Redis** est une base de données NoSQL en mémoire, extrêmement rapide, souvent utilisée comme :
- Cache pour accélérer l'accès aux données
- File d'attente pour traiter des flux de données en temps réel
- Stockage temporaire de sessions ou de résultats intermédiaires

En géomatique, Redis peut servir à :
- Cacher des résultats de calculs géospatiaux lourds (ex. intersections, buffers)
- Stocker des flux de données GPS ou IoT en temps réel
- Partager rapidement des informations entre plusieurs services d'une application

## Installation des bibliothèques Python

Installez également la bibliothèque `redis` pour Python :

```shell
pip install redis
```

## Connexion à Redis en Python

Voici comment établir une connexion basique :

```python
import redis

# Connexion au serveur Redis local
r = redis.Redis(host='localhost', port=6379, db=0)

# Test de la connexion
r.set('cle_test', 'valeur_test')
print(r.get('cle_test'))
```

Sortie attendue :
```
b'valeur_test'
```

> Conseil : Redis utilise des **bytes** ; pensez à décoder (`.decode('utf-8')`) si besoin.

## Cas d'usage en géomatique

### 1. Mise en cache de résultats spatiaux

Supposons que vous effectuez un calcul d'intersection coûteux :

```python
import hashlib

def generer_cle(buffer_geom):
    """Génère une clé unique basée sur la géométrie."""
    return hashlib.sha256(buffer_geom.wkb).hexdigest()

def obtenir_ou_calculer_buffer(redis_conn, geom, distance):
    cle = generer_cle(geom)

    resultat = redis_conn.get(cle)
    if resultat:
        print("Récupéré depuis le cache")
        return shapely.wkb.loads(resultat)
    
    # Calcul du buffer
    buffer = geom.buffer(distance)
    
    # Stockage en cache
    redis_conn.set(cle, buffer.wkb)
    print("Calculé et stocké")
    return buffer
```

> Cette approche évite de recalculer plusieurs fois le même **buffer** pour une géométrie identique.

### 2. Stockage temporaire de flux GPS

Pour une flotte de véhicules équipés de GPS, Redis peut stocker les positions en temps réel :

```python
import json

def stocker_position(redis_conn, vehicule_id, latitude, longitude):
    position = {
        'latitude': latitude,
        'longitude': longitude,
        'timestamp': datetime.utcnow().isoformat()
    }
    redis_conn.set(f'vehicule:{vehicule_id}', json.dumps(position))

def recuperer_position(redis_conn, vehicule_id):
    donnees = redis_conn.get(f'vehicule:{vehicule_id}')
    if donnees:
        return json.loads(donnees)
    return None
```

> Conseil : utilisez `expire()` pour que les positions expirent automatiquement au bout de quelques minutes.

### 3. Gestion d'une file d'attente de traitements géospatiaux

Vous pouvez empiler des tâches spatiales :

```python
# Ajout d'une tâche dans la file
r.lpush('taches_spatiales', json.dumps({'type': 'buffer', 'id': 42, 'distance': 100}))

# Récupération et traitement
tache = r.rpop('taches_spatiales')
if tache:
    tache_data = json.loads(tache)
    print(f"Traitement de la tâche : {tache_data}")
```

> Redis offre aussi des structures de file d'attente plus évoluées comme les **streams** (`XADD`, `XREAD`).

### 4. Conservation des données spatiales et quelques analyses

Redis permet de conserver des données spatiales comme des points. 

```python
# Initialize some example data on startup
def init_data():
    redis_client.geoadd("locations", [
        -73.965355, 40.782865, "Central Park",
        -73.985130, 40.758896, "Times Square",
        -74.044500, 40.689247, "Statue of Liberty"
    ], nx=True)
```

Par la suite il est possible d'appeler des méthodes d'analyse spatiale comme trouver le point proche.

```python
lon = float(request.args.get('lon'))
lat = float(request.args.get('lat'))
radius = float(request.args.get('radius', 1000))  # meters
unit = request.args.get('unit', 'm')  # m, km, mi, ft
        
results = redis_client.georadius("locations", lon, lat, radius, unit=unit, withcoord=True)
        
```

Cette approche est très pratique pour mettre en place un outil de recherche de points d'intérêt à proximité.

> Voir l'exemple de service web flask combiné avec redis [https://github.com/voirinprof/redis_docker](https://github.com/voirinprof/redis_docker)

## Bonnes pratiques

- **Utilisez des clés organisées** : par exemple `vehicule:1234` ou `calculs:buffers:zone42`.
- **Définissez une expiration (`TTL`)** pour les données temporaires : `r.set('key', value, ex=3600)` expire au bout d'une heure.
- **Protégez les données critiques** : Redis est rapide mais n'est pas conçu comme base de données durable par défaut (sauvegarde manuelle ou configuration nécessaire).
- **Surveillez la mémoire** : Redis fonctionne principalement en mémoire RAM.

## Ressources utiles

- [Documentation Redis](https://redis.io/docs/latest/)
- [Documentation redis-py](https://redis-py.readthedocs.io/)
- [Article : Usage de Redis pour le stockage spatial temps réel](https://redis.com/blog/location-tracking-and-geo-spatial-data-with-redis/)
