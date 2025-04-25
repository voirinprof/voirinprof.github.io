---
title: "Docker Basics"
permalink: /en/gis-docker/
lang: en
date: 2025-04-25T03:02:20+00:00
toc: true
toc_label: "On this page"
---

## Introduction to Docker

Docker is a tool that allows you to **package an application and all its dependencies** into a container. A container is a lightweight, isolated environment that can run on any machine with Docker installed. This ensures your program works the same way everywhere, regardless of the system.

> Docker is widely used in development, deployment, automation, and continuous integration.

## Key concepts

Here are some basic concepts to understand before using Docker:

- **Image**: an image is like a template or blueprint. It includes your app and everything it needs to run.
- **Container**: a running instance of an image. You can run multiple containers from the same image.
- **Dockerfile**: a text file that contains the instructions to build an image.
- **Docker Hub**: an online registry where images are stored and shared. It's like GitHub, but for images.

## Installing Docker

Before using Docker, you need to install it on your computer. You can download Docker Desktop here:

[https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

To verify Docker is working, open a terminal and run:

```bash
docker --version
```

If it's correctly installed, it will display the version number.

## Running your first container

To test Docker, you can run a basic container using the `hello-world` image:

```bash
docker run hello-world
```

This command downloads an image from Docker Hub and runs a container that prints a welcome message.

## Searching and using images

You can search for existing images with:

```bash
docker search python
```

And run one like this:

```bash
docker run -it python
```

> Tip: use `-it` to interact with the container in terminal mode.

## Basic commands

Here are some useful Docker commands:

```bash
docker images            # List downloaded images
docker ps                # List running containers
docker ps -a             # List all containers (even stopped ones)
docker stop <id>         # Stop a running container
docker rm <id>           # Remove a container
docker rmi <image_id>    # Remove an image
```

> Tip: use meaningful names for your images and containers (with the `--name` option).

## Writing a Dockerfile

Here is a simple example of a Dockerfile that builds an image with Python:

```Dockerfile
# Use an official Python image
FROM python:3.10-slim

# Set the working directory
WORKDIR /app

# Copy local files into the container
COPY . .

# Default command
CMD ["python", "my_script.py"]
```

To build an image from this Dockerfile:

```bash
docker build -t my-python-image .
```

And to run it:

```bash
docker run my-python-image
```

## Using volumes

Volumes let you **share files between your machine and the container**:

```bash
docker run -v $(pwd):/app -it python
```

This mounts your current folder into the container at `/app`.

## Networking and ports

To expose an application (like a web server), you need to map ports:

```bash
docker run -p 8000:80 my-web-image
```

This connects port 80 inside the container to port 8000 on your machine.

## Cleaning up

Docker can take up disk space over time. Here are some helpful cleanup commands:

```bash
docker system prune      # Remove unused data
docker volume prune      # Remove unused volumes
```

## Helpful videos

- [Docker for Beginners - Getting Started](https://youtu.be/3c-iBn73dDE)
