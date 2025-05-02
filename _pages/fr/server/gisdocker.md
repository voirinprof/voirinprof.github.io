---
title: "Docker en géomatique"
permalink: /fr/gis-docker/
lang: fr
date: 2025-04-24T03:02:20+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: gisserver
---

## Introduction à Docker

Docker est un outil qui permet d’**emballer une application et toutes ses dépendances** dans un conteneur. Un conteneur est un environnement léger et isolé qui peut être exécuté sur n’importe quelle machine disposant de Docker. Cela garantit que le programme fonctionne de la même manière partout, peu importe le système.

> Docker est très utilisé en développement, déploiement, automatisation et intégration continue.

## Les concepts clés

Voici quelques concepts de base à comprendre avant d’utiliser Docker :

- **Image** : une image est un modèle (un peu comme un plan) à partir duquel on crée un conteneur. Elle contient l’application et tout ce dont elle a besoin.
- **Conteneur** : c’est une instance en cours d’exécution d’une image. On peut avoir plusieurs conteneurs à partir de la même image.
- **Dockerfile** : c’est un fichier texte qui contient les instructions pour construire une image.
- **Docker Hub** : un registre en ligne où sont stockées les images Docker. C’est comme GitHub, mais pour des images.

## Installer Docker

Avant de commencer à utiliser Docker, vous devez l’installer sur votre machine. Voici le lien officiel pour télécharger Docker Desktop :

[https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

Assurez-vous que Docker fonctionne en ouvrant un terminal et en tapant :

```bash
docker --version
```

Si Docker est bien installé, vous verrez le numéro de version s’afficher.

## Exécuter son premier conteneur

Pour tester Docker, vous pouvez lancer un conteneur simple basé sur l’image `hello-world` :

```bash
docker run hello-world
```

Cela télécharge une image depuis Docker Hub et exécute un conteneur qui affiche un message de bienvenue.

## Rechercher et utiliser une image

Vous pouvez chercher des images existantes avec :

```bash
docker search python
```

Et en lancer une comme ceci :

```bash
docker run -it python
```

> Conseil : utilisez `-it` pour interagir avec le conteneur (mode terminal).

## Les commandes de base

Voici quelques commandes utiles à connaître :

```bash
docker images            # Liste les images téléchargées
docker ps                # Liste les conteneurs en cours d'exécution
docker ps -a             # Liste tous les conteneurs (même arrêtés)
docker stop <id>         # Arrête un conteneur
docker rm <id>           # Supprime un conteneur
docker rmi <image_id>    # Supprime une image
```

> Conseil : utilisez des noms explicites pour vos images et conteneurs (avec l’option `--name`).

## Écrire un Dockerfile

Voici un exemple simple de Dockerfile qui crée une image contenant Python :

```Dockerfile
# Utilise une image officielle Python
FROM python:3.10-slim

# Définit le répertoire de travail
WORKDIR /app

# Copie les fichiers locaux dans le conteneur
COPY . .

# Commande par défaut
CMD ["python", "mon_script.py"]
```

Pour construire une image à partir de ce Dockerfile :

```bash
docker build -t mon-image-python .
```

Puis on peut l’exécuter :

```bash
docker run mon-image-python
```

## Gérer les volumes

Les volumes permettent de **partager des fichiers entre l’hôte et le conteneur** :

```bash
docker run -v $(pwd):/app -it python
```

Dans cet exemple, le dossier courant de votre machine est monté dans le conteneur à `/app`.

## Réseau et ports

Pour exposer une application (ex: un serveur web), il faut exposer un port :

```bash
docker run -p 8000:80 mon-image-web
```

Cela relie le port 80 du conteneur au port 8000 de la machine.

## Nettoyage

Docker peut consommer de l’espace disque si on n’y fait pas attention. Voici quelques commandes utiles :

```bash
docker system prune      # Nettoie tout ce qui n'est plus utilisé
docker volume prune      # Supprime les volumes inutilisés
```

## Vidéos utiles

- [Docker pour les débutants - mise en route](https://youtu.be/J27puPcFFQo?si=_xC_W0bgLZSIVwjj)