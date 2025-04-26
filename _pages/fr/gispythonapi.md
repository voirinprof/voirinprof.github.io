---
title: "Bases de Flask et FastAPI"
permalink: /fr/web-python-api/
lang: fr
date: 2025-04-25T14:30:00+00:00
toc: true
toc_label: "Sur cette page"
typenav: gisserver
---

## Introduction

**Flask** et **FastAPI** sont deux frameworks web populaires en Python. Ils permettent de créer rapidement des applications web et des API.

| Fonctionnalité | Flask                            | FastAPI                                  |
|----------------|----------------------------------|-------------------------------------------|
| Type           | Micro-framework web              | Framework web moderne et performant       |
| Vitesse        | Plus lent (synchronisé)          | Plus rapide (asynchrone par défaut)       |
| Typage         | Optionnel                        | Utilisation poussée des types (Pydantic)  |
| Documentation  | Manuelle ou via extensions       | Générée automatiquement (Swagger/OpenAPI) |

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

### Lancer l’application

```bash
python app.py
```

L’application tourne sur `http://127.0.0.1:5000`

### Ajouter des paramètres

```python
@app.route('/hello/<name>')
def hello(name):
    return f"Bonjour, {name} !"
```

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

### Lancer l’application

```bash
uvicorn app:app --reload
```

L’application tourne sur `http://127.0.0.1:8000`

### Documentation automatique

FastAPI génère automatiquement deux interfaces :

- Swagger UI : [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
- ReDoc : [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)

---

## Gérer les paramètres

### Paramètres d’URL et de requête (FastAPI)

```python
@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

### Corps de requête (FastAPI)

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

@app.post("/items/")
def create_item(item: Item):
    return {"item": item}
```

---

## Quand utiliser quoi ?

- Utilisez **Flask** si :
  - Vous cherchez la simplicité et un contrôle total
  - Vous êtes à l’aise avec WSGI
  - Vous développez une petite application ou un projet existant

- Utilisez **FastAPI** si :
  - Vous avez besoin de performance et scalabilité
  - Vous voulez du typage fort et de la validation automatique
  - Vous créez une API moderne (ex. frontend/backend séparés)

---

## Ressources utiles

- Flask : [https://flask.palletsprojects.com/fr/](https://flask.palletsprojects.com/fr/)
- FastAPI : [https://fastapi.tiangolo.com/fr/](https://fastapi.tiangolo.com/fr/)
- Vidéo : [FastAPI vs Flask - Une comparaison](https://youtu.be/LhGsy7sjKas)