---
title: "Introduction to Celery"
permalink: /en/celery-intro/
lang: en
date: 2025-05-05T12:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: misc
---

> **_NOTE:_** For advanced details, consult the [official Celery documentation](https://docs.celeryq.dev/en/stable/). An example of a Flask web service with Celery can be found in this repository: [https://github.com/celery/celery-examples](https://github.com/celery/celery-examples)

## Introduction

**Celery** is a powerful open-source distributed task queue system for Python, designed to handle asynchronous task processing and scheduling. It is widely used in web applications to offload time-consuming tasks, such as sending emails, processing files, or running background computations, to improve performance and scalability. **Celery** is included in **GeoNode**, so it could be useful in GIS, because some tasks are time consuming.

This guide introduces the core concepts of Celery, installation steps, and a basic example of its use in Python.

---

## Why Use Celery?

- **Asynchronous Task Execution**: Run tasks in the background to avoid blocking the main application.
- **Scalability**: Distribute tasks across multiple workers and machines.
- **Task Scheduling**: Schedule tasks to run at specific times or intervals.
- **Integration**: Works seamlessly with web frameworks like Flask, Django, and FastAPI.
- **Reliability**: Supports task retries, error handling, and monitoring.

---

## Installing Celery

Celery requires a message broker (e.g., RabbitMQ, Redis) to manage task queues. Below are the installation steps:

### Install Celery via pip:

```shell
pip install celery
```

### Install a Message Broker:

#### RabbitMQ (recommended):
- On Ubuntu/Debian:
  ```shell
  sudo apt-get install rabbitmq-server
  ```
- On macOS (using Homebrew):
  ```shell
  brew install rabbitmq
  ```

#### Redis (alternative):
- On Ubuntu/Debian:
  ```shell
  sudo apt-get install redis-server
  ```
- On macOS:
  ```shell
  brew install redis
  ```

### Optional: Install with Redis support:
```shell
pip install celery[redis]
```

---

## Key Concepts

- **Task**: A Python function decorated with `@celery.task` to be executed asynchronously.
- **Broker**: A message queue (e.g., RabbitMQ, Redis) that stores tasks until they are processed.
- **Worker**: A process that executes tasks from the queue.
- **Backend**: A storage system (e.g., Redis, database) to store task results.
- **Celery App**: The main Celery instance that configures the broker, backend, and tasks.

---

## First Python Script with Celery

Here’s an example of a simple Celery setup with Redis as the broker and backend:

### File: `tasks.py`
```python
from celery import Celery

# Configure Celery with Redis
app = Celery('tasks', 
             broker='redis://localhost:6379/0', 
             backend='redis://localhost:6379/0')

# Optional configuration
app.conf.update(
    task_serializer='json',
    accept_content=['json'],
    result_serializer='json',
    timezone='UTC',
    enable_utc=True,
)

# Define a task
@app.task
def add(x, y):
    return x + y
```

### File: `run_task.py`
```python
from tasks import add

# Call the task asynchronously
result = add.delay(4, 6)

# Check if the task is complete
print(f"Task ID: {result.id}")
print(f"Task completed: {result.ready()}")

# Get the result (blocks until task is done)
if result.ready():
    print(f"Result: {result.get()}")
```

### Running the Worker
Start the Redis server (if not already running):
```shell
redis-server
```

Start the Celery worker:
```shell
celery -A tasks worker --loglevel=info
```

Run the task script:
```shell
python run_task.py
```

This will queue the `add` task, which a worker will process, and store the result in Redis.

---

## Using a Configuration File

For more complex applications, it’s preferable to define Celery configurations in a separate file to manage settings like broker URLs, task timeouts, and worker concurrency.

### File: `celeryconfig.py`
```python
broker_url = 'redis://localhost:6379/0'
result_backend = 'redis://localhost:6379/0'

task_serializer = 'json'
accept_content = ['json']
result_serializer = 'json'
timezone = 'UTC'
enable_utc = True

# Example: Limit task retries
task_default_retry_delay = 300  # 5 minutes
task_max_retries = 3
```

### Updated `tasks.py` with Configuration
```python
from celery import Celery

app = Celery('tasks')
app.config_from_object('celeryconfig')

@app.task
def add(x, y):
    return x + y
```

Start the worker with the same command:
```shell
celery -A tasks worker --loglevel=info
```

This approach keeps the configuration modular and reusable.

---

## Where to Use Celery?

- **Web Applications**: Offload tasks like sending emails, generating reports, or processing uploads in Flask/Django apps.
- **Data Processing**: Run batch jobs for data transformation, machine learning model training, or ETL pipelines.
- **Scheduled Tasks**: Automate recurring tasks, such as daily backups or periodic data scraping.
- **Microservices**: Coordinate tasks across distributed systems with message queues.
- **Real-Time Processing**: Handle tasks like image resizing or notification delivery in near real-time.

---

## Useful Resources

- [Official Celery Documentation](https://docs.celeryq.dev/en/stable/)
- [Celery GitHub Repository](https://github.com/celery/celery)
- [Real Python Celery Tutorial](https://realpython.com/asynchronous-tasks-with-django-and-celery/)

---

## Best Practices

- **Choose the Right Broker**: Use RabbitMQ for reliability in production or Redis for simpler setups.
- **Monitor Tasks**: Use tools like Flower (`pip install flower`, run `celery -A tasks flower`) to monitor task queues and workers.
- **Handle Errors**: Implement task retries and error logging to manage failures gracefully.
- **Optimize Worker Concurrency**: Adjust the number of worker processes (`-c` flag) based on CPU cores and task intensity.
- **Secure Your Broker**: Configure authentication and encryption for production brokers (e.g., RabbitMQ user credentials).
- **Test Tasks Locally**: Use a lightweight broker like Redis for development and testing.