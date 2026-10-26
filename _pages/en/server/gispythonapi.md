---
title: "Basics of Flask and FastAPI"
permalink: /en/web-python-api/
lang: en
date: 2025-04-25T14:30:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: gisserver
---

## Introduction

**Flask** and **FastAPI** are two popular web frameworks in Python. They allow you to quickly create web applications and APIs, for example to serve geographic data (GeoJSON) to a Leaflet or OpenLayers map.

| Feature         | Flask                            | FastAPI                                   |
|-----------------|----------------------------------|-------------------------------------------|
| Type            | Web micro-framework              | Modern and high-performance web framework |
| Speed           | Slower (synchronous)             | Faster (asynchronous support)             |
| Typing          | Optional                         | Extensive use of types (Pydantic)         |
| Validation      | Manual                           | Automatic, based on types                 |
| Documentation   | Manual or via extensions         | Automatically generated (Swagger/OpenAPI) |
| Server          | Built-in (dev), WSGI in prod     | Uvicorn (ASGI)                            |

---

## What Is a REST API?

A **REST API** is a way of organizing a web service around **resources** (cities, layers, sensors...), each one reachable through a **URL**. The client (browser, application, Python script) sends an HTTP request and the server responds, usually in **JSON**.

The action to perform is given by the **HTTP method**:

| Method   | Action              | Example              |
|----------|---------------------|----------------------|
| `GET`    | Read a resource     | `GET /cities/1`      |
| `POST`   | Create a resource   | `POST /cities`       |
| `PUT`    | Update a resource   | `PUT /cities/1`      |
| `DELETE` | Delete a resource   | `DELETE /cities/1`   |

The server reports the result with an **HTTP status code**: `200` (OK), `201` (created), `400` (bad request), `404` (not found), `500` (server error).

Flask and FastAPI are precisely the tools used to define these routes (URL + method) and what they return.

---

## Setting Up the Project

### Create a Virtual Environment

It is recommended to isolate each project in its own environment:

```bash
mkdir my-api
cd my-api
python -m venv .venv

# Linux / macOS
source .venv/bin/activate
# Windows
.venv\Scripts\activate
```

### Starting Structure

A simple structure is enough to get started:

```text
my-api/
├── .venv/
├── app.py              # the application code
├── data/
│   └── cities.geojson  # your data
└── requirements.txt    # the dependencies
```

### Save the Dependencies

After installing the libraries, keep track of their versions:

```bash
pip freeze > requirements.txt
```

To reinstall the project on another machine:

```bash
pip install -r requirements.txt
```

---

## Flask Basics

### Installation

```bash
pip install flask
```

### Minimal Example

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello Flask!"

if __name__ == '__main__':
    app.run(debug=True)
```

### Running the Application

```bash
python app.py
```

The application runs on `http://127.0.0.1:5000`. With `debug=True`, the server restarts automatically when you modify the code.

### URL Parameters

```python
@app.route('/hello/<name>')
def hello(name):
    return f"Hello, {name}!"

# You can specify the expected type
@app.route('/cities/<int:city_id>')
def get_city(city_id):
    return {"id": city_id}
```

### Query Parameters (`?key=value`)

```python
from flask import request

# Example: /search?name=Montreal&limit=5
@app.route('/search')
def search():
    name = request.args.get('name', '')
    limit = request.args.get('limit', 10, type=int)
    return {"name": name, "limit": limit}
```

### Returning JSON

Flask automatically converts a dictionary to JSON. For a list, use `jsonify`:

```python
from flask import jsonify

@app.route('/cities')
def cities():
    return jsonify([
        {"name": "Montreal", "population": 1762949},
        {"name": "Quebec City", "population": 549459},
    ])
```

### Receiving Data (POST)

```python
@app.route('/cities', methods=['POST'])
def add_city():
    data = request.get_json()
    if not data or 'name' not in data:
        return {"error": "The 'name' field is required"}, 400
    return {"message": f"City {data['name']} added"}, 201
```

The second element of the `return` is the **HTTP status code** (200 = OK, 201 = created, 400 = bad request, 404 = not found).

---

## FastAPI Basics

### Installation

```bash
pip install fastapi uvicorn
```

### Minimal Example

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello FastAPI!"}
```

### Running the Application

```bash
uvicorn app:app --reload
```

- `app:app` means: the file `app.py`, and the `app` object inside it
- `--reload` restarts the server every time the code changes

The application runs on `http://127.0.0.1:8000`

### Automatic Documentation

FastAPI automatically generates two interfaces:

- Swagger UI: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
- ReDoc: [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)

Swagger UI lets you **test your routes directly in the browser**, without writing any client code.

### Path and Query Parameters

```python
# Example: /items/5?q=test
@app.get("/items/{item_id}")
def read_item(item_id: int, q: str | None = None):
    return {"item_id": item_id, "q": q}
```

- `item_id` is in the path: it is **required** and must be an integer (otherwise FastAPI returns a 422 error)
- `q` is not in the path: it is an **optional** query parameter

### Request Body (POST)

The data sent is described with a **Pydantic** model. FastAPI checks the types automatically:

```python
from pydantic import BaseModel

class City(BaseModel):
    name: str
    population: int
    lat: float
    lon: float

@app.post("/cities", status_code=201)
def add_city(city: City):
    return {"message": f"City {city.name} added", "city": city}
```

### Handling Errors

```python
from fastapi import HTTPException

CITIES = {1: "Montreal", 2: "Quebec City"}

@app.get("/cities/{city_id}")
def get_city(city_id: int):
    if city_id not in CITIES:
        raise HTTPException(status_code=404, detail="City not found")
    return {"id": city_id, "name": CITIES[city_id]}
```

---

## Geographic Example: Serving GeoJSON

A common use case: exposing a vector layer to display it on a web map. Here with FastAPI and GeoPandas (`pip install geopandas`).

```python
import json
import geopandas as gpd
from fastapi import FastAPI

app = FastAPI()

# Loaded only once, at startup
gdf = gpd.read_file("data/cities.geojson").to_crs(epsg=4326)

@app.get("/cities")
def cities(min_pop: int = 0):
    # Example: /cities?min_pop=100000
    result = gdf[gdf["population"] >= min_pop]
    return json.loads(result.to_json())
```

The Flask equivalent:

```python
@app.route('/cities')
def cities():
    min_pop = request.args.get('min_pop', 0, type=int)
    result = gdf[gdf["population"] >= min_pop]
    return json.loads(result.to_json())
```

> GeoJSON for the web must be in **WGS84 (EPSG:4326)**, hence the `to_crs(epsg=4326)`.

---

## Calling the API from a Web Page (CORS)

If your map (HTML/JavaScript) is not served from the same address as the API, the browser will block the requests. You need to allow **CORS**.

**Flask** (`pip install flask-cors`):

```python
from flask_cors import CORS

app = Flask(__name__)
CORS(app)
```

**FastAPI**:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # restrict this in production
    allow_methods=["*"],
    allow_headers=["*"],
)
```

On the JavaScript side, for example with Leaflet:

```javascript
fetch("http://127.0.0.1:8000/cities?min_pop=100000")
  .then(response => response.json())
  .then(data => L.geoJSON(data).addTo(map));
```

---

## Testing Your API

- **Browser**: for `GET` routes, just open the URL
- **Swagger UI** (FastAPI): `http://127.0.0.1:8000/docs`
- **curl** in a terminal:

```bash
# GET
curl http://127.0.0.1:8000/cities/1

# POST with JSON
curl -X POST http://127.0.0.1:8000/cities \
     -H "Content-Type: application/json" \
     -d '{"name": "Laval", "population": 438366, "lat": 45.57, "lon": -73.69}'
```

- **Python** with `requests`:

```python
import requests

r = requests.get("http://127.0.0.1:8000/cities", params={"min_pop": 100000})
print(r.status_code)
print(r.json())
```

---

## When to Use What?

- Use **Flask** if:
  - You want simplicity and full control
  - You also want to generate HTML pages (Jinja2 templates)
  - You are developing a small app or an existing project

- Use **FastAPI** if:
  - You need performance and scalability
  - You want strong typing and automatic validation
  - You are creating a modern API (e.g., frontend/backend separation)
  - You want automatically generated documentation

---

## Useful Resources

- Flask: [https://flask.palletsprojects.com/en/stable/](https://flask.palletsprojects.com/en/stable/)
- FastAPI: [https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)
- FastAPI tutorial (step by step): [https://fastapi.tiangolo.com/tutorial/](https://fastapi.tiangolo.com/tutorial/)
- Pydantic: [https://docs.pydantic.dev/](https://docs.pydantic.dev/)
- Video: [FastAPI vs Flask - A Comparison](https://www.youtube.com/watch?v=itfI5W3pCHE)
