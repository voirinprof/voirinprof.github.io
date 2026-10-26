---
title: "Bases de Flask et FastAPI"
permalink: /fr/web-python-api/
lang: fr
date: 2025-04-25T14:30:00+00:00
toc: true
toc_sticky: true
toc_label: "Sur cette page"
typenav: gisserver
---

## Introduction

**Flask** et **FastAPI** sont deux frameworks web populaires en Python. Ils permettent de créer rapidement des applications web et des API, par exemple pour servir des données géographiques (GeoJSON) à une carte Leaflet ou OpenLayers.

| Fonctionnalité | Flask                            | FastAPI                                   |
|----------------|----------------------------------|-------------------------------------------|
| Type           | Micro-framework web              | Framework web moderne et performant       |
| Vitesse        | Plus lent (synchrone)            | Plus rapide (asynchrone possible)         |
| Typage         | Optionnel                        | Utilisation poussée des types (Pydantic)  |
| Validation     | Manuelle                         | Automatique à partir des types            |
| Documentation  | Manuelle ou via extensions       | Générée automatiquement (Swagger/OpenAPI) |
| Serveur        | Intégré (dev), WSGI en prod      | Uvicorn (ASGI)                            |

---

## Qu'est-ce qu'une API REST ?

Une **API REST** est une façon d'organiser un service web autour de **ressources** (des villes, des couches, des capteurs...), chacune accessible par une **URL**. Le client (navigateur, application, script Python) envoie une requête HTTP et le serveur répond, généralement en **JSON**.

L'action à réaliser est indiquée par la **méthode HTTP** :

| Méthode  | Action                  | Exemple              |
|----------|-------------------------|----------------------|
| `GET`    | Lire une ressource      | `GET /villes/1`      |
| `POST`   | Créer une ressource     | `POST /villes`       |
| `PUT`    | Modifier une ressource  | `PUT /villes/1`      |
| `DELETE` | Supprimer une ressource | `DELETE /villes/1`   |

Le serveur indique le résultat avec un **code HTTP** : `200` (OK), `201` (créé), `400` (requête invalide), `404` (introuvable), `500` (erreur serveur).

Flask et FastAPI servent justement à définir ces routes (URL + méthode) et ce qu'elles retournent.

---

## Préparer le projet

### Créer un environnement virtuel

Il est recommandé d'isoler chaque projet dans son propre environnement :

```bash
mkdir mon-api
cd mon-api
python -m venv .venv

# Linux / macOS
source .venv/bin/activate
# Windows
.venv\Scripts\activate
```

### Structure de départ

Une structure simple suffit pour commencer :

```text
mon-api/
├── .venv/
├── app.py              # le code de l'application
├── data/
│   └── villes.geojson  # vos données
└── requirements.txt    # les dépendances
```

### Enregistrer les dépendances

Après l'installation des librairies, gardez une trace des versions :

```bash
pip freeze > requirements.txt
```

Pour réinstaller le projet sur une autre machine :

```bash
pip install -r requirements.txt
```

---

## Bases de Flask

### Installation

```bash
pip install flask
```

### Exemple minimal

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Bonjour Flask !"

if __name__ == '__main__':
    app.run(debug=True)
```

### Lancer l'application

```bash
python app.py
```

L'application tourne sur `http://127.0.0.1:5000`. Avec `debug=True`, le serveur redémarre automatiquement quand vous modifiez le code.

### Paramètres dans l'URL

```python
@app.route('/hello/<name>')
def hello(name):
    return f"Bonjour, {name} !"

# On peut préciser le type attendu
@app.route('/villes/<int:ville_id>')
def get_ville(ville_id):
    return {"id": ville_id}
```

### Paramètres de requête (`?cle=valeur`)

```python
from flask import request

# Exemple : /recherche?nom=Montreal&limite=5
@app.route('/recherche')
def recherche():
    nom = request.args.get('nom', '')
    limite = request.args.get('limite', 10, type=int)
    return {"nom": nom, "limite": limite}
```

### Retourner du JSON

Flask convertit automatiquement un dictionnaire en JSON. Pour une liste, utilisez `jsonify` :

```python
from flask import jsonify

@app.route('/villes')
def villes():
    return jsonify([
        {"nom": "Montréal", "population": 1762949},
        {"nom": "Québec", "population": 549459},
    ])
```

### Recevoir des données (POST)

```python
@app.route('/villes', methods=['POST'])
def ajouter_ville():
    data = request.get_json()
    if not data or 'nom' not in data:
        return {"erreur": "Le champ 'nom' est obligatoire"}, 400
    return {"message": f"Ville {data['nom']} ajoutée"}, 201
```

Le deuxième élément du `return` est le **code HTTP** (200 = OK, 201 = créé, 400 = requête invalide, 404 = introuvable).

---

## Bases de FastAPI

### Installation

```bash
pip install fastapi uvicorn
```

### Exemple minimal

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Bonjour FastAPI !"}
```

### Lancer l'application

```bash
uvicorn app:app --reload
```

- `app:app` signifie : le fichier `app.py`, et l'objet `app` qu'il contient
- `--reload` redémarre le serveur à chaque modification du code

L'application tourne sur `http://127.0.0.1:8000`

### Documentation automatique

FastAPI génère automatiquement deux interfaces :

- Swagger UI : [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
- ReDoc : [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)

Swagger UI permet de **tester vos routes directement dans le navigateur**, sans écrire de code client.

### Paramètres d'URL et de requête

```python
# Exemple : /items/5?q=test
@app.get("/items/{item_id}")
def read_item(item_id: int, q: str | None = None):
    return {"item_id": item_id, "q": q}
```

- `item_id` est dans le chemin : il est **obligatoire** et doit être un entier (sinon FastAPI renvoie une erreur 422)
- `q` n'est pas dans le chemin : c'est un paramètre de requête **optionnel**

### Corps de requête (POST)

Les données envoyées sont décrites avec un modèle **Pydantic**. FastAPI vérifie automatiquement les types :

```python
from pydantic import BaseModel

class Ville(BaseModel):
    nom: str
    population: int
    lat: float
    lon: float

@app.post("/villes", status_code=201)
def ajouter_ville(ville: Ville):
    return {"message": f"Ville {ville.nom} ajoutée", "ville": ville}
```

### Gérer les erreurs

```python
from fastapi import HTTPException

VILLES = {1: "Montréal", 2: "Québec"}

@app.get("/villes/{ville_id}")
def get_ville(ville_id: int):
    if ville_id not in VILLES:
        raise HTTPException(status_code=404, detail="Ville introuvable")
    return {"id": ville_id, "nom": VILLES[ville_id]}
```

---

## Exemple géographique : servir du GeoJSON

Un cas fréquent : exposer une couche vectorielle pour l'afficher sur une carte web. Ici avec FastAPI et GeoPandas (`pip install geopandas`).

```python
import json
import geopandas as gpd
from fastapi import FastAPI

app = FastAPI()

# Chargé une seule fois au démarrage
gdf = gpd.read_file("data/villes.geojson").to_crs(epsg=4326)

@app.get("/villes")
def villes(min_pop: int = 0):
    # Exemple : /villes?min_pop=100000
    resultat = gdf[gdf["population"] >= min_pop]
    return json.loads(resultat.to_json())
```

L'équivalent avec Flask :

```python
@app.route('/villes')
def villes():
    min_pop = request.args.get('min_pop', 0, type=int)
    resultat = gdf[gdf["population"] >= min_pop]
    return json.loads(resultat.to_json())
```

> Le GeoJSON pour le web doit être en **WGS84 (EPSG:4326)**, d'où le `to_crs(epsg=4326)`.

---

## Appeler l'API depuis une page web (CORS)

Si votre carte (HTML/JavaScript) n'est pas servie par la même adresse que l'API, le navigateur bloquera les requêtes. Il faut autoriser le **CORS**.

**Flask** (`pip install flask-cors`) :

```python
from flask_cors import CORS

app = Flask(__name__)
CORS(app)
```

**FastAPI** :

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # à restreindre en production
    allow_methods=["*"],
    allow_headers=["*"],
)
```

Côté JavaScript, par exemple avec Leaflet :

```javascript
fetch("http://127.0.0.1:8000/villes?min_pop=100000")
  .then(response => response.json())
  .then(data => L.geoJSON(data).addTo(map));
```

---

## Tester son API

- **Navigateur** : pour les routes `GET`, il suffit d'ouvrir l'URL
- **Swagger UI** (FastAPI) : `http://127.0.0.1:8000/docs`
- **curl** dans un terminal :

```bash
# GET
curl http://127.0.0.1:8000/villes/1

# POST avec du JSON
curl -X POST http://127.0.0.1:8000/villes \
     -H "Content-Type: application/json" \
     -d '{"nom": "Laval", "population": 438366, "lat": 45.57, "lon": -73.69}'
```

- **Python** avec `requests` :

```python
import requests

r = requests.get("http://127.0.0.1:8000/villes", params={"min_pop": 100000})
print(r.status_code)
print(r.json())
```

---

## Quand utiliser quoi ?

- Utilisez **Flask** si :
  - Vous cherchez la simplicité et un contrôle total
  - Vous voulez aussi générer des pages HTML (templates Jinja2)
  - Vous développez une petite application ou un projet existant

- Utilisez **FastAPI** si :
  - Vous avez besoin de performance et scalabilité
  - Vous voulez du typage fort et de la validation automatique
  - Vous créez une API moderne (ex. frontend/backend séparés)
  - Vous voulez une documentation générée automatiquement

---

## Ressources utiles

- Flask : [https://flask.palletsprojects.com/en/stable/](https://flask.palletsprojects.com/en/stable/)
- FastAPI : [https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)
- Tutoriel FastAPI (pas à pas) : [https://fastapi.tiangolo.com/tutorial/](https://fastapi.tiangolo.com/tutorial/)
- Pydantic : [https://docs.pydantic.dev/](https://docs.pydantic.dev/)
- Vidéo : [FastAPI vs Flask - Une comparaison (en anglais)](https://www.youtube.com/watch?v=itfI5W3pCHE)
