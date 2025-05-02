---
title: "Docker in Geomatics"
permalink: /en/gis-docker/
lang: en
date: 2025-04-24T03:02:20+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: gisserver
---

## Introduction to Docker

Docker is a tool that allows you to **package an application and all its dependencies** into a container. A container is a lightweight and isolated environment that can run on any machine with Docker installed. This ensures that the program runs the same way everywhere, regardless of the system.

> Docker is widely used in development, deployment, automation, and continuous integration.

## Key Concepts

Here are some basic concepts to understand before using Docker:

- **Image**: An image is a template (similar to a blueprint) from which a container is created. It contains the application and everything it needs.
- **Container**: A container is a running instance of an image. You can have multiple containers from the same image.
- **Dockerfile**: A text file that contains instructions to build an image.
- **Docker Hub**: An online registry where Docker images are stored. It's like GitHub, but for images.

## Installing Docker

Before you start using Docker, you need to install it on your machine. Here is the official link to download Docker Desktop:

[https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

Make sure Docker is working by opening a terminal and typing:

```bash
docker --version
```

If Docker is installed correctly, the version number will appear.

## Running Your First Container

To test Docker, you can run a simple container based on the `hello-world` image:

```bash
docker run hello-world
```

This will download an image from Docker Hub and run a container that displays a welcome message.

## Searching for and Using an Image

You can search for existing images with:

```bash
docker search python
```

And run one like this:

```bash
docker run -it python
```

> Tip: use `-it` to interact with the container (terminal mode).

## Basic Commands

Here are some useful commands to know:

```bash
docker images            # Lists downloaded images
docker ps                # Lists running containers
docker ps -a             # Lists all containers (including stopped ones)
docker stop <id>         # Stops a container
docker rm <id>           # Removes a container
docker rmi <image_id>    # Removes an image
```

> Tip: use explicit names for your images and containers (with the `--name` option).

## Writing a Dockerfile

Here’s a simple example of a Dockerfile that creates an image containing Python:

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

Then you can run it:

```bash
docker run my-python-image
```

## Managing Volumes

Volumes allow you to **share files between the host and the container**:

```bash
docker run -v $(pwd):/app -it python
```

In this example, the current folder on your machine is mounted into the container at `/app`.

## Networking and Ports

To expose an application (e.g., a web server), you need to expose a port:

```bash
docker run -p 8000:80 my-web-image
```

This links port 80 of the container to port 8000 of the machine.

## Cleaning Up

Docker can consume disk space if not managed properly. Here are some useful commands:

```bash
docker system prune      # Cleans up everything that is no longer in use
docker volume prune      # Removes unused volumes
```

## Helpful Videos

- [Docker for Beginners - Getting Started](https://youtu.be/J27puPcFFQo?si=_xC_W0bgLZSIVwjj)