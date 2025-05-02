---
title: "Python with Redis"
permalink: /en/python-redis/
lang: en
date: 2025-04-25T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: database
---

> **_NOTE:_** The [official Redis documentation](https://redis.io/docs/latest/) and the Python library documentation [redis-py](https://redis-py.readthedocs.io/) provide detailed instructions for further exploration. You can practice with Redis using this GitHub Codespace [https://github.com/voirinprof/gis_starter_redis_geolab](https://github.com/voirinprof/gis_starter_redis_geolab). To set up a Codespace environment, check out [Getting Started with GitHub Codespaces](/en/codespaces-intro/).

## Introduction to Redis

**Redis** is an in-memory NoSQL database, extremely fast, often used as:
- A cache to speed up data access
- A queue for processing real-time data streams
- Temporary storage for sessions or intermediate results

In geomatics, Redis can be used to:
- Cache the results of expensive geospatial calculations (e.g., intersections, buffers)
- Store real-time GPS or IoT data streams
- Quickly share information between multiple services in an application

## Installing Python Libraries

Also install the `redis` library for Python:

```shell
pip install redis
```

## Connecting to Redis in Python

Here’s how to establish a basic connection:

```python
import redis

# Connect to the local Redis server
r = redis.Redis(host='localhost', port=6379, db=0)

# Test the connection
r.set('test_key', 'test_value')
print(r.get('test_key'))
```

Expected output:
```
b'test_value'
```

> Tip: Redis uses **bytes**; remember to decode (`.decode('utf-8')`) if needed.

## Use Cases in Geomatics

### 1. Caching Spatial Results

Suppose you’re performing an expensive intersection calculation:

```python
import hashlib

def generate_key(buffer_geom):
    """Generates a unique key based on the geometry."""
    return hashlib.sha256(buffer_geom.wkb).hexdigest()

def get_or_compute_buffer(redis_conn, geom, distance):
    key = generate_key(geom)

    result = redis_conn.get(key)
    if result:
        print("Retrieved from cache")
        return shapely.wkb.loads(result)
    
    # Compute the buffer
    buffer = geom.buffer(distance)
    
    # Store in cache
    redis_conn.set(key, buffer.wkb)
    print("Computed and stored")
    return buffer
```

> This approach avoids recalculating the same **buffer** for identical geometries multiple times.

### 2. Temporary Storage of GPS Streams

For a fleet of GPS-equipped vehicles, Redis can store real-time positions:

```python
import json

def store_position(redis_conn, vehicle_id, latitude, longitude):
    position = {
        'latitude': latitude,
        'longitude': longitude,
        'timestamp': datetime.utcnow().isoformat()
    }
    redis_conn.set(f'vehicle:{vehicle_id}', json.dumps(position))

def get_position(redis_conn, vehicle_id):
    data = redis_conn.get(f'vehicle:{vehicle_id}')
    if data:
        return json.loads(data)
    return None
```

> Tip: Use `expire()` to make the positions automatically expire after a few minutes.

### 3. Managing a Geospatial Task Queue

You can enqueue geospatial tasks:

```python
# Add a task to the queue
r.lpush('spatial_tasks', json.dumps({'type': 'buffer', 'id': 42, 'distance': 100}))

# Retrieve and process the task
task = r.rpop('spatial_tasks')
if task:
    task_data = json.loads(task)
    print(f"Processing task: {task_data}")
```

> Redis also offers more advanced queue structures like **streams** (`XADD`, `XREAD`).

### 4. Storing Spatial Data and Performing Some Analysis

Redis allows storing spatial data like points.

```python
# Initialize some example data on startup
def init_data():
    redis_client.geoadd("locations", [
        -73.965355, 40.782865, "Central Park",
        -73.985130, 40.758896, "Times Square",
        -74.044500, 40.689247, "Statue of Liberty"
    ], nx=True)
```

Later, you can call spatial analysis methods like finding nearby points.

```python
lon = float(request.args.get('lon'))
lat = float(request.args.get('lat'))
radius = float(request.args.get('radius', 1000))  # meters
unit = request.args.get('unit', 'm')  # m, km, mi, ft
        
results = redis_client.georadius("locations", lon, lat, radius, unit=unit, withcoord=True)
```

This approach is very practical for building a tool to search for nearby points of interest.

> See the Flask web service example combined with Redis [https://github.com/voirinprof/redis_docker](https://github.com/voirinprof/redis_docker).

## Best Practices

- **Use organized keys**: for example, `vehicle:1234` or `calculations:buffers:zone42`.
- **Set an expiration (`TTL`)** for temporary data: `r.set('key', value, ex=3600)` will expire after an hour.
- **Protect critical data**: Redis is fast but not designed as a durable database by default (manual backups or configuration needed).
- **Monitor memory usage**: Redis works primarily in RAM.

## Useful Resources

- [Redis Documentation](https://redis.io/docs/latest/)
- [redis-py Documentation](https://redis-py.readthedocs.io/)