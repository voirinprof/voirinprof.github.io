---
title: "Votre environnement"
permalink: /quick-start/
date: 2023-04-24T03:02:20+00:00
toc: true
toc_label: "Dans cette page"
---

## L'environnement de développement

Lorsqu'on travaille en géomatique avec Python, il est important d'avoir un environnement de développement adapté à nos besoins. Tout d'abord, il est recommandé d'installer une version récente de Python et d'utiliser un gestionnaire de paquets comme Conda pour gérer les dépendances de nos projets.

> Conseil : ne pas forcément prendre la version la plus récente de Python. Les librairies en géomatique évoluent parfois lentement, donc il arrive des incompatibilités avec les dernières versions de Python.

Ensuite, il est conseillé d'utiliser un environnement de développement intégré (IDE) tel que PyCharm, Visual Studio Code ou Spyder. Ces IDE offrent des fonctionnalités avancées telles que la complétion automatique de code, la détection d'erreurs de syntaxe et la mise en évidence des variables non utilisées.

> Conseil : Visual Studio Code semble être plus léger et il s'intègre bien avec Conda.

Il est également recommandé d'installer des bibliothèques Python pour la géomatique telles que Geopandas, Shapely et Fiona. Ces bibliothèques permettent la manipulation de données géospatiales en Python et offrent des fonctionnalités avancées pour la visualisation et l'analyse de ces données.

> Conseil : La géomatique connaît un grand nombre de librairies en Python. Il n'est pas forcément utile de toutes les connaître. On peut s'en sortir avec des librairies comme Rasterio, Fiona, Pandas, GeoPandas. Le plus important est de développer une bonne expérience sur certaines librairies et d'éviter de multiplier les librairies.

## Les outils à connaître

Pour l'environnement Python, le mieux est de prendre Conda. Cet environnement vous permettra de gérer les versions des interpréteurs, les environnements virtuels, l'installation des libs. On peut installer [Anaconda](https://www.anaconda.com/download/), toutefois la version complète peut être très lourde. Évidemment Anaconda inclut de nombreux outils autour de Python, comme le gestionnaire graphique des librairies. Il serait préférable de partir plutôt de [Miniconda](https://docs.conda.io/en/latest/miniconda.html). Ceci signifie que la gestion de votre environnement se fera par commande en ligne. Il est donc important de connaître quelques commandes, voici un [lien intéressant](https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf).

> Conseil : prenez l'habitude des commandes en ligne pour gérer votre environnement de développement. Ceci vous aidera si vous avez à reproduire les commandes sur un serveur Linux. En général, on effectue toutes les commandes dans un terminal.

Pour votre IDE, Visual Studio Code devrait répondre à la majorité de vos besoins. Cet outil dispose de belles extensions pour Python qui vous permettra d'aller très vite dans le développement. De plus, si vous êtes un étudiant, vous pouvez bénéficier de l'assistant Co-pilot de Github directement dans VSCode.

> Conseil : utilisez un environnement dans lequel vous êtes à l'aise. Bien que de nombreuses vidéos montrent l'environnement VS Code, vous pouvez reproduire les mêmes démarches dans PyCharm ou Spyder. L'IDE ne doit pas vous bloquer dans votre travail. 

## Mettre en place son environnement

Télécharger [Miniconda](https://docs.conda.io/en/latest/miniconda.html) et l'installer sur sa machine.

Définir un environnement virtuel avec Conda pour votre développement. Par ex: définir un environnement nommé **spatialpy39** (ajoutez toujours une info sur la version de Python dans le nom de l'environnement, ce sera très utile)

La commande sera :

```shell
conda create --name spatialpy39 python=3.9
```

Si vous souhaitez ajouter des librairies lors de la création, il faudra écrire :

```shell
conda create --name spatialpy39 python=3.9 rasterio fiona
```


N'oubliez pas qu'il sera toujours possible d'ajouter les libs par la suite. 

> Conseil : faites la création d'un environnement selon vos besoins. Par exemple, développer un projet Django ne nécessite pas les mêmes librairies qu'un projet en analyse d'image. Évitez de trop charger un environnement.

Télécharger et installer votre IDE, comme [VS Code](https://code.visualstudio.com/).

Dans VS Code, activez l'extension Python ce qui permettra de travailler dans ce langage.

## Autres manières de développer : Jupyter Notebook

Jupyter Notebook est une application web open-source qui permet de créer et de partager des documents qui contiennent du code, des visualisations et du texte. Il est utilisé par de nombreux scientifiques et développeurs en Python pour explorer, analyser et communiquer des données.

Le principe de Jupyter Notebook est basé sur des cellules, qui peuvent contenir du code Python, du texte formaté en Markdown ou des visualisations. Les cellules de code peuvent être exécutées individuellement ou en groupe, ce qui permet une exploration interactive des données.

Lorsqu'on travaille avec Jupyter Notebook, on peut facilement intégrer des bibliothèques pour la géomatique telles que Geopandas et Matplotlib pour visualiser des données géospatiales. On peut également utiliser des widgets pour créer des interfaces interactives pour l'utilisateur.

Une des grandes forces de Jupyter Notebook est sa capacité à générer des rapports dynamiques et reproductibles. En effet, en combinant du code, des visualisations et du texte, on peut facilement communiquer nos résultats de manière claire et interactive. De plus, grâce à l'utilisation de notebooks, il est facile de reproduire des analyses précédentes ou de collaborer avec d'autres utilisateurs.

Des plateformes ouvertes permettent de bien débuter avec ce principe, on peut notamment citer :

* [Colab](https://colab.research.google.com/)
* [Kaggle](https://www.kaggle.com/)

> Conseil : disposez d'un compte sur une de ces plateformes, c'est très utile pour explorer les librairies en géomatique sans avoir trop de tracas pour l'installation.


## Vidéos utiles

* [Installer l'environnement de développement (Anaconda + Visual Studio Code)](https://youtu.be/D3NDo9Rw0SY)
* [Créer un environnement virtuel sous Anaconda (Python 3)](https://youtu.be/qq54aZmK3-Q)
* [Créer votre environnement spatial dans MiniConda](https://youtu.be/873XcjLRn6U)
* [Installer MiniConda et développer avec VSCode](https://youtu.be/P17GY1gSnFk)
* [Premier pas avec Kaggle](https://youtu.be/hU74_BG61sQ)