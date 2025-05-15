---
title: "Des outils utiles aux tests"
permalink: /fr/testing-useful/
lang: fr
date: 2025-04-24T03:02:20+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: dev
---

# 🐛 Introduction à Locust

## 🔍 Qu'est-ce que Locust ?

**Locust** est un outil open-source écrit en Python pour réaliser des **tests de charge** sur des applications web.  
Il permet de simuler des utilisateurs virtuels qui effectuent des requêtes HTTP afin de mesurer la performance d'une application ou API.

> L'intérêt de cet outil est de mettre en relief la limite de votre application ou infrastructure (combien de requêtes votre API géospatiale peut supporter avant de ralentir ou planter ?).

---

## Cas d’usage typique en géomatique

- Tester la performance d’un **endpoint Flask** qui renvoie des données GeoJSON depuis PostGIS.
- Évaluer la charge supportée par un service de géocodage interne.
- Simuler plusieurs clients interrogeant un **serveur WMS/WFS**.

---

## Installation

```bash
pip install locust
```

---

## Structure de base d’un fichier `locustfile.py`

Le fichier décrit ni plus ni moins des requêtes (tâches) qu'un utilisateur pourrait faire sur votre système. 

```python
from locust import HttpUser, task, between

class GeoApiUser(HttpUser):
    wait_time = between(1, 3)  # secondes entre deux requêtes

    @task
    def get_geodata(self):
        self.client.get("/api/geojson?bbox=10,45,11,46")
```

On peut définir plusieurs tâches possibles dans un même fichier. Elles seront appellées à la même fréquence. Toutefois si vous souhaitez tester plus souvent une action, alors attribuez un poid plus fort à cette tâche ```@task(5)  -> appelée 5 fois plus souvent```.

---

## Lancer un test

Depuis le dossier contenant `locustfile.py` :

```bash
locust
```
ou
```bash
locust -f locustfile.py
```

**Locust** propose une interface web simple mais assez complète. On peut télécharger un rapport en fin de tests.

Puis, ouvrir : [http://localhost:8089](http://localhost:8089)

> Notez que si vous voulez lancer les tests sans navigateur alors utilisez le mode ```--headless``` et spécifiez le nombre d'utilisateurs avec ```-u 1000```

### Paramètres à entrer :

* **Nombre d’utilisateurs** : nb de clients virtuels (ex: 100)
* **Taux d’arrivée (spawn rate)** : nombre de nouveaux utilisateurs/seconde (ex: 10)

---

## Comprendre les résultats

* **Requests per second (RPS)** : nb de requêtes envoyées par seconde
* **Réponse moyenne (ms)** : temps de réponse moyen de l’API
* **Failures** : erreurs HTTP rencontrées (404, 500, timeout…)

> Si le RPS diminue alors que les utilisateurs augmentent → saturation probable (serveur ou DB).


**Locust** produit des graphiques comme ceux présentés ci-dessous (on voit la saturation de l'api à 200 utilisateurs). De plus, on voit que l'API produit des erreurs plus on poursuit avec le nombre d'utilisateurs. Cela indique clairement un problème, soit applicatif ou au niveau de l'infrastructure.

![Graphiques de Locust](/assets/img/locust/total_requests_per_second_1747318384.279.png)

---

## Exemples liés à la géomatique

### Exemple 1 : API Flask qui renvoie du GeoJSON

```python
@task
def get_feature_collection(self):
    self.client.get("/features?bbox=5,45,6,46&limit=100")
```

### Exemple 2 : Simulation de requêtes WMS

```python
@task
def get_wms_map(self):
    url = (
        "/geoserver/wms"
        "?service=WMS&version=1.1.1&request=GetMap"
        "&layers=topographie"
        "&bbox=5,45,6,46"
        "&width=256&height=256"
        "&srs=EPSG:4326&format=image/png"
    )
    self.client.get(url)
```

### Exemple 3 : Simulation des GET et des POST

```python
@task(2)  # Poids de 2 : les GET sont plus fréquents
def get_points(self):
    response = self.client.get("/points")
    if response.status_code != 200:
        response.failure(f"GET /points failed with status {response.status_code}")

@task(1)  # Poids de 1 : les POST sont moins fréquents
def add_point(self):
    # Générer des coordonnées aléatoires pour simuler des données variées
    longitude = random.uniform(-180, 180)
    latitude = random.uniform(-90, 90)
    name = f"Point_{random.randint(1, 1000)}"
    
    payload = {
        "longitude": longitude,
        "latitude": latitude,
        "name": name
    }
    
    response = self.client.post("/point", json=payload)
    if response.status_code != 200:
        response.failure(f"POST /point failed with status {response.status_code}")
```

---

## Conseils pour des tests utiles

* Testez avec **des jeux de données réalistes** (vraies tailles de bbox, limites, etc.)
* Ne testez **qu’un service à la fois**
* Ajoutez des logs ou métriques côté serveur pour corréler les lenteurs
* Comparez les performances entre **requêtes spatiales simples** et **jointures complexes**

---

## Aller plus loin

* Utilisez `on_start()` pour simuler des sessions (authentification)
* Faites varier les poids des tâches pour simuler des usages réels
* Exécutez Locust en **mode distribué** pour simuler plus d’utilisateurs. On évite le goulot d’étranglement d'une seule machine car cela fait beaucoup de trafic à générer.

---

## Liens utiles

* [Documentation officielle Locust](https://docs.locust.io/)
