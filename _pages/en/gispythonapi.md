---
title: "Flask and FastAPI Basics"
permalink: /en/web-python-api/
lang: en
date: 2025-04-25T14:30:00+00:00
toc: true
toc_label: "On this page"
---

## Introduction

**Flask** and **FastAPI** are two popular web frameworks for Python. They allow you to build web applications and APIs quickly and easily.

| Feature        | Flask                           | FastAPI                                 |
|---------------|----------------------------------|------------------------------------------|
| Type          | Micro web framework              | Modern, high-performance web framework   |
| Speed         | Slower (synchronous)             | Faster (asynchronous by default)         |
| Typing        | Optional                         | Strong use of type hints (Pydantic)      |
| Documentation | Manual or using extensions       | Auto-generated with Swagger/OpenAPI      |

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
    return "Hello, Flask!"

if __name__ == '__main__':
    app.run(debug=True)
```

### Running the App

```bash
python app.py
```

The app runs on `http://127.0.0.1:5000`

### Adding Parameters

```python
@app.route('/hello/<name>')
def hello(name):
    return f"Hello, {name}!"
```

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
    return {"message": "Hello, FastAPI!"}
```

### Running the App

```bash
uvicorn app:app --reload
```

The app runs on `http://127.0.0.1:8000`

### Automatic Docs

FastAPI automatically creates interactive documentation:

- Swagger UI: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
- ReDoc: [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)

---

## Handling Parameters

### Query and Path Parameters (FastAPI)

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
  - You're already familiar with WSGI
  - You're building small apps or legacy systems

- Use **FastAPI** if:
  - You need speed and scalability
  - You want strong typing and data validation
  - You're building modern APIs (especially for frontend/backend apps)

---

## Helpful Resources

- Flask: [https://flask.palletsprojects.com/](https://flask.palletsprojects.com/)
- FastAPI: [https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)
- Video: [FastAPI vs Flask - A Comparison](https://youtu.be/LhGsy7sjKas)