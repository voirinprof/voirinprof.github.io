---
title: "Your Environment"
permalink: /en/quick-start/
lang: en
date: 2025-04-24T03:02:20+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: start
---

> **_NOTE:_** The [gis_starter_geolab site on GitHub](https://github.com/voirinprof/gis_starter_geolab) is a great environment to start programming in geospatial science. Use the repo and run **Codespaces**, you get 120 hours/month for free (beware of the processor choice, which will affect your hourly credit). It’s, however, the easiest way to get a free online VS Code Web environment.

## Development Environment

When working in geospatial science with Python, it is important to have a development environment suited to our needs. First, it is recommended to install a recent version of Python and use a package manager like Conda to manage the dependencies for our projects.

> Tip: Do not necessarily choose the latest version of Python. Geospatial libraries can evolve slowly, and incompatibilities may arise with the latest Python versions.

Next, it is advisable to use an Integrated Development Environment (IDE) such as PyCharm, Visual Studio Code, or Spyder. These IDEs offer advanced features such as code autocompletion, syntax error detection, and highlighting unused variables.

> Tip: Visual Studio Code seems to be lighter and integrates well with Conda. Also, `GitHub Codespaces` offers the same web environment.

It is also recommended to install Python libraries for geospatial science, such as Geopandas, Shapely, and Fiona. These libraries allow the manipulation of geospatial data in Python and provide advanced functionality for visualizing and analyzing such data.

> Tip: Geospatial science has many Python libraries. It’s not necessary to know all of them. You can manage with libraries like Rasterio, Fiona, Pandas, GeoPandas. The most important thing is to gain solid experience with certain libraries and avoid overloading your work with too many libraries.

## Tools to Know

For the Python environment, it’s best to use Conda. This environment will allow you to manage interpreter versions, virtual environments, and library installations. You can install [Anaconda](https://www.anaconda.com/download/), but the full version can be quite large. Of course, Anaconda includes many tools around Python, such as the graphical library manager. It’s better to start with [Miniconda](https://docs.conda.io/en/latest/miniconda.html). This means managing your environment through command line, so it’s important to know a few commands. Here’s a [useful link](https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf).

> Tip: Get used to using the command line to manage your development environment. This will help you if you need to reproduce the commands on a Linux server. Typically, all commands are executed in a terminal.

For your IDE, Visual Studio Code should meet most of your needs. This tool has excellent Python extensions that will help you move quickly in development. Also, if you’re a student, you can access GitHub’s Co-pilot assistant directly in VS Code.

> Tip: Use an environment where you feel comfortable. Although many videos show the VS Code environment, you can replicate the same steps in PyCharm or Spyder. The IDE should not block your work.

## Setting Up Your Environment

Download [Miniconda](https://docs.conda.io/en/latest/miniconda.html) and install it on your machine.

Define a virtual environment with Conda for your development. For example, define an environment named **spatialpy39** (always add Python version information in the environment name, it will be very useful).

The command will be:

```shell
conda create --name spatialpy39 python=3.9
```

If you want to add libraries when creating the environment, use:

```shell
conda create --name spatialpy39 python=3.9 rasterio fiona
```

Don’t forget that it will always be possible to add libraries later.

> Tip: Create your environment according to your needs. For example, developing a Django project doesn’t require the same libraries as an image analysis project. Avoid overloading the environment.

Download and install your IDE, such as [VS Code](https://code.visualstudio.com/).

In VS Code, enable the Python extension to work in this language.

## Other Ways to Develop: Jupyter Notebook

Jupyter Notebook is an open-source web application that allows you to create and share documents containing code, visualizations, and text. It is widely used by scientists and developers in Python to explore, analyze, and communicate data.

The principle of Jupyter Notebook is based on cells, which can contain Python code, formatted text in Markdown, or visualizations. Code cells can be executed individually or in groups, allowing interactive exploration of data.

When working with Jupyter Notebook, you can easily integrate geospatial libraries like Geopandas and Matplotlib to visualize geospatial data. You can also use widgets to create interactive user interfaces.

One of the major strengths of Jupyter Notebook is its ability to generate dynamic and reproducible reports. By combining code, visualizations, and text, you can communicate your results clearly and interactively. Additionally, thanks to notebooks, it’s easy to reproduce previous analyses or collaborate with other users.

Open platforms to get started with this concept include:

* [Colab](https://colab.research.google.com/)
* [Kaggle](https://www.kaggle.com/)

> Tip: Have an account on one of these platforms; it’s very useful for exploring geospatial libraries without worrying too much about installation.

## Useful Videos

* [Setting up the Development Environment (Anaconda + Visual Studio Code)](https://youtu.be/D3NDo9Rw0SY)
* [Create a Virtual Environment with Anaconda (Python 3)](https://youtu.be/qq54aZmK3-Q)
* [Create Your Spatial Environment in MiniConda](https://youtu.be/873XcjLRn6U)
* [Install MiniConda and Develop with VSCode](https://youtu.be/P17GY1gSnFk)
* [First Steps with Kaggle](https://youtu.be/hU74_BG61sQ)