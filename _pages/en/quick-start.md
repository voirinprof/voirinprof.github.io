---
title: "Your Environment"
permalink: /en/quick-start/
lang: en
date: 2025-04-24T03:02:20+00:00
toc: true
toc_label: "On this page"
---

## Your Development Environment

When working in geomatics with Python, it’s important to have a development setup that suits your needs. First, it is recommended to install a recent version of Python and to use a package manager like Conda to manage project dependencies.

> Tip: You don’t necessarily need the latest Python version. Geomatics libraries tend to evolve slowly, and compatibility issues may arise with the most recent Python releases.

It’s also a good idea to work in an integrated development environment (IDE) such as PyCharm, Visual Studio Code, or Spyder. These IDEs offer features like code autocompletion, syntax checking, and highlighting of unused variables.

> Tip: Visual Studio Code is lightweight and integrates well with Conda.

You should also install some essential Python libraries for geomatics, such as Geopandas, Shapely, and Fiona. These libraries provide tools for manipulating and visualizing geospatial data with Python.

> Tip: There are many geomatics libraries in Python. You don’t need to know them all. You can do a lot with a few, like Rasterio, Fiona, Pandas, and GeoPandas. Focus on developing solid experience with a few libraries rather than trying to learn them all.

## Tools to Know

For Python environments, Conda is your best bet. It lets you manage Python versions, virtual environments, and library installations. You can install [Anaconda](https://www.anaconda.com/download/), but the full version can be quite large. A lighter alternative is [Miniconda](https://docs.conda.io/en/latest/miniconda.html), which means you'll be using the command line to manage your environment. It's useful to be familiar with a few basic commands—here’s a [cheat sheet](https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf).

> Tip: Get used to working in the terminal. This will help if you ever need to reproduce commands on a Linux server. Most tasks are handled from the command line.

Visual Studio Code should meet most of your IDE needs. It offers great Python extensions to speed up your development. Plus, if you're a student, you can take advantage of GitHub Copilot directly within VS Code.

> Tip: Use the environment you're most comfortable with. Even though many tutorials showcase VS Code, you can follow the same steps in PyCharm or Spyder. Your IDE shouldn’t get in the way of your work.

## Setting Up Your Environment

Download and install [Miniconda](https://docs.conda.io/en/latest/miniconda.html) on your machine.

Create a virtual environment using Conda. For example, to create an environment named **spatialpy39** (it’s useful to include the Python version in the name):

```shell
conda create --name spatialpy39 python=3.9
```

If you want to install some libraries right away:

```shell
conda create --name spatialpy39 python=3.9 rasterio fiona
```

You can always install additional libraries later.

> Tip: Tailor your environment to the needs of your project. A Django web project and a remote sensing project don’t require the same packages. Avoid bloating your environment unnecessarily.

Download and install an IDE such as [VS Code](https://code.visualstudio.com/).

Enable the Python extension in VS Code to start working in Python.

## Other Ways to Work: Jupyter Notebooks

Jupyter Notebook is an open-source web application that allows you to create and share documents containing code, visualizations, and text. It’s widely used by scientists and Python developers for exploring, analyzing, and communicating data.

Jupyter is based on the concept of cells, which can contain Python code, formatted Markdown, or visualizations. You can execute cells individually or as a group, making it ideal for interactive data exploration.

When working with Jupyter Notebooks, you can easily integrate geomatics libraries like GeoPandas and Matplotlib to visualize spatial data. You can also use widgets to build interactive user interfaces.

One of Jupyter’s strengths is the ability to create dynamic and reproducible reports. By combining code, text, and visualizations, you can communicate your results clearly and interactively. Plus, notebooks make it easy to reproduce analyses or collaborate with others.

Some great platforms to start with include:

* [Colab](https://colab.research.google.com/)
* [Kaggle](https://www.kaggle.com/)

> Tip: Create an account on one of these platforms—they’re handy for exploring geomatics libraries without worrying about installation.

## Useful Videos

* [Set up your development environment (Anaconda + Visual Studio Code)](https://youtu.be/D3NDo9Rw0SY)
* [Create a virtual environment with Anaconda (Python 3)](https://youtu.be/qq54aZmK3-Q)
* [Create your spatial environment with MiniConda](https://youtu.be/873XcjLRn6U)
* [Install MiniConda and develop with VSCode](https://youtu.be/P17GY1gSnFk)
* [Getting started with Kaggle](https://youtu.be/hU74_BG61sQ)