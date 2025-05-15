---
title: "Useful Tools for Testing"
permalink: /en/testing-useful/
lang: en
date: 2025-04-24T03:02:20+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: dev
---

# 🐛 Introduction to Locust

## 🔍 What is Locust?

**Locust** is an open-source Python tool used to perform **load testing** on web applications.  
It allows you to simulate virtual users making HTTP requests to measure the performance of your app or API.

> The goal of this tool is to reveal the limits of your application or infrastructure (how many requests can your geospatial API handle before slowing down or crashing?).

---

## Typical Use Cases in Geomatics

- Testing the performance of a **Flask endpoint** returning GeoJSON from PostGIS.
- Evaluating the load a geocoding service can handle.
- Simulating multiple clients querying a **WMS/WFS server**.

---

## Installation

```bash
pip install locust
```

---

## Basic `locustfile.py` Structure

The file simply describes the requests (tasks) a user might make on your system.

```python
from locust import HttpUser, task, between

class GeoApiUser(HttpUser):
    wait_time = between(1, 3)  # seconds between requests

    @task
    def get_geodata(self):
        self.client.get("/api/geojson?bbox=10,45,11,46")
```

You can define multiple possible tasks in the same file. By default, they are called with equal probability.
If you'd like to prioritize one action over the others, you can assign a higher weight to a task:
`@task(5)  -> called 5 times more often`

---

## Running a Test

From the folder containing `locustfile.py`:

```bash
locust
```

or

```bash
locust -f locustfile.py
```

**Locust** provides a simple but powerful web interface, and you can download a test report once the test completes.

Open in browser: [http://localhost:8089](http://localhost:8089)

> If you want to run the tests without a browser, use the `--headless` mode and set the number of users with `-u 1000`.

### Parameters to set:

* **Number of users**: number of virtual clients (e.g., 100)
* **Spawn rate**: how many new users per second (e.g., 10)

---

## Understanding the Results

* **Requests per second (RPS)**: how many requests are sent per second
* **Average response time (ms)**: average time your API takes to respond
* **Failures**: HTTP errors (404, 500, timeout…)

> If RPS drops as users increase → your system is likely saturating (server or database bottleneck).

**Locust** produces charts like the one below (showing API saturation at around 200 users).
As more users are added, error rates increase, indicating a bottleneck — either in the application or infrastructure.

![Locust Graphs](/assets/img/locust/total_requests_per_second_1747318384.279.png)

---

## Geomatics-Related Examples

### Example 1: Flask API returning GeoJSON

```python
@task
def get_feature_collection(self):
    self.client.get("/features?bbox=5,45,6,46&limit=100")
```

### Example 2: Simulating WMS requests

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

### Example 3: Simulating GET and POST requests

```python
@task(2)  # Weight of 2: GETs are more frequent
def get_points(self):
    response = self.client.get("/points")
    if response.status_code != 200:
        response.failure(f"GET /points failed with status {response.status_code}")

@task(1)  # Weight of 1: POSTs are less frequent
def add_point(self):
    # Generate random coordinates to simulate diverse input
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

## Tips for Effective Testing

* Use **realistic datasets** (realistic bbox sizes, query limits, etc.)
* Test **one service at a time**
* Add server-side logs or metrics to correlate slowdowns
* Compare performance between **simple spatial queries** and **complex joins**

---

## Going Further

* Use `on_start()` to simulate login sessions (authentication)
* Vary task weights to simulate real-world user behavior
* Run Locust in **distributed mode** to simulate more users.
  This prevents the local machine from becoming the bottleneck (too much traffic generation).

---

## Useful Links

* [Official Locust Documentation](https://docs.locust.io/)