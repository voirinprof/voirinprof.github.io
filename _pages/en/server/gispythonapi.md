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

**Flask** and **FastAPI** are two popular web frameworks in Python. They allow you to quickly create web applications and APIs.

| Feature         | Flask                            | FastAPI                                  |
|-----------------|----------------------------------|------------------------------------------|
| Type            | Micro-framework web              | Modern and high-performance web framework |
| Speed           | Slower (synchronous)             | Faster (asynchronous by default)         |
| Typing          | Optional                         | Extensive use of types (Pydantic)       |
| Documentation   | Manual or via extensions        | Automatically generated (Swagger/OpenAPI) |

---

## Basics of Flask

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

The application runs on `http://127.0.0.1:5000`

### Adding Parameters

```python
@app.route('/hello/<name>')
def hello(name):
    return f"Hello, {name}!"
```

---

## Basics of FastAPI

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

The application runs on `http://127.0.0.1:8000`

### Automatic Documentation

FastAPI automatically generates two interfaces:

- Swagger UI: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
- ReDoc: [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)

---

## Handling Parameters

### URL and Query Parameters (FastAPI)

```python
@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

### Request Body (FastAPI)

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

## When to Use What?

- Use **Flask** if:
  - You want simplicity and full control
  - You are comfortable with WSGI
  - You are developing a small app or an existing project

- Use **FastAPI** if:
  - You need performance and scalability
  - You want strong typing and automatic validation
  - You are creating a modern API (e.g., frontend/backend separation)

---

## Useful Resources

- Flask: [https://flask.palletsprojects.com/en/stable/](https://flask.palletsprojects.com/en/stable/)
- FastAPI: [https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)
- Video: [FastAPI vs Flask - A Comparison (in English)](https://www.youtube.com/watch?v=itfI5W3pCHE)
