---
title: "Introduction à Celery"
permalink: /fr/celery-intro/
lang: fr
date: 2025-05-05T12:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: gisserver
---

> **_NOTE:_** Pour des détails avancés, consultez la [documentation officielle de Celery](https://docs.celeryq.dev/en/stable/). Un exemple de service web `Flask` avec `Celery` peut être trouvé dans ce dépôt : [https://github.com/voirinprof/gis_celery_automation_docker](https://github.com/voirinprof/gis_celery_automation_docker)

## Introduction

**Celery** est un système de file d'attente de tâches distribuées open-source pour Python, conçu pour gérer le traitement asynchrone des tâches et la planification. Il est largement utilisé dans les applications web pour déléguer des tâches chronophages, comme l'envoi d'emails, le traitement de fichiers ou l'exécution de calculs en arrière-plan, afin d'améliorer les performances et l'évolutivité. **Celery** est utilisé dans **GeoNode**, donc il peut être utile dans les SIG. Effectivement certaines tâches peuvent prendre du temps, on peut planifier des tâches férquentes, notamment dans la maintenance d'un site web.

Ce guide présente les concepts de base, les étapes d'installation et un exemple simple d'utilisation en Python.

---

## Pourquoi utiliser Celery ?

- **Exécution asynchrone des tâches** : Exécutez des tâches en arrière-plan pour éviter de bloquer l'application principale.
- **Évolutivité** : Distribuez les tâches sur plusieurs workers et machines.
- **Planification des tâches** : Programmez des tâches pour s'exécuter à des moments précis ou à intervalles réguliers.
- **Intégration** : Fonctionne de manière transparente avec des frameworks web comme Flask, Django et FastAPI.
- **Fiabilité** : Prend en charge les nouvelles tentatives de tâches, la gestion des erreurs et la surveillance.

---

## Installer Celery

Celery nécessite un courtier de messages (par exemple, RabbitMQ, Redis) pour gérer les files d'attente des tâches. Voici les étapes d'installation :

### Installer Celery via pip :

```shell
pip install celery
```

### Installer un courtier de messages :

#### RabbitMQ (recommandé) :
- Sur Ubuntu/Debian :
  ```shell
  sudo apt-get install rabbitmq-server
  ```
- Sur macOS (avec Homebrew) :
  ```shell
  brew install rabbitmq
  ```

#### Redis (alternative) :
- Sur Ubuntu/Debian :
  ```shell
  sudo apt-get install redis-server
  ```
- Sur macOS :
  ```shell
  brew install redis
  ```

### Optionnel : Installer avec le support Redis :
```shell
pip install celery[redis]
```

---

## Concepts clés

- **Tâche** : Une fonction Python décorée avec `@celery.task` pour être exécutée de manière asynchrone.
- **Courtier** : Une file d'attente de messages (par exemple, RabbitMQ, Redis) qui stocke les tâches jusqu'à leur traitement.
- **Worker** : Un processus qui exécute les tâches de la file d'attente.
- **Backend** : Un système de stockage (par exemple, Redis, base de données) pour conserver les résultats des tâches.
- **Application Celery** : L'instance principale de Celery qui configure le courtier, le backend et les tâches.

---

## Premier script Python avec Celery

Voici un exemple d'une configuration simple de Celery avec Redis comme courtier et backend :

### Fichier : `tasks.py`
```python
from celery import Celery

# Configurer Celery avec Redis
app = Celery('tasks', 
             broker='redis://localhost:6379/0', 
             backend='redis://localhost:6379/0')

# Configuration optionnelle
app.conf.update(
    task_serializer='json',
    accept_content=['json'],
    result_serializer='json',
    timezone='UTC',
    enable_utc=True,
)

# Définir une tâche
@app.task
def add(x, y):
    return x + y
```

### Fichier : `run_task.py`
```python
from tasks import add

# Appeler la tâche de manière asynchrone
result = add.delay(4, 6)

# Vérifier si la tâche est terminée
print(f"ID de la tâche : {result.id}")
print(f"Tâche terminée : {result.ready()}")

# Obtenir le résultat (bloque jusqu'à ce que la tâche soit terminée)
if result.ready():
    print(f"Résultat : {result.get()}")
```

### Lancer le Worker
Démarrez le serveur Redis (s'il n'est pas déjà en cours d'exécution) :
```shell
redis-server
```

Démarrez le worker Celery :
```shell
celery -A tasks worker --loglevel=info
```

Exécutez le script de tâche :
```shell
python run_task.py
```

Cela placera la tâche `add` dans la file d'attente, qu'un worker traitera, et stockera le résultat dans Redis.

---

## Utiliser un fichier de configuration

Pour les applications plus complexes, il est préférable de définir les configurations de Celery dans un fichier séparé pour gérer des paramètres comme les URL du courtier, les délais d'exécution des tâches et la concurrence des workers.

### Fichier : `celeryconfig.py`
```python
broker_url = 'redis://localhost:6379/0'
result_backend = 'redis://localhost:6379/0'

task_serializer = 'json'
accept_content = ['json']
result_serializer = 'json'
timezone = 'UTC'
enable_utc = True

# Exemple : Limiter les nouvelles tentatives de tâches
task_default_retry_delay = 300  # 5 minutes
task_max_retries = 3
```

### `tasks.py` mis à jour avec la configuration
```python
from celery import Celery

app = Celery('tasks')
app.config_from_object('celeryconfig')

@app.task
def add(x, y):
    return x + y
```

Démarrez le worker avec la même commande :
```shell
celery -A tasks worker --loglevel=info
```

Cette approche rend la configuration modulaire et réutilisable.

---

## Où utiliser Celery ?

- **Applications web** : Déléguez des tâches comme l'envoi d'emails, la génération de rapports ou le traitement de téléchargements dans des applications Flask/Django.
- **Traitement de données** : Exécutez des tâches par lots pour la transformation de données, l'entraînement de modèles d'apprentissage automatique ou des pipelines ETL.
- **Tâches planifiées** : Automatisez des tâches récurrentes, comme des sauvegardes quotidiennes ou des collectes de données périodiques.
- **Microservices** : Coordonnez les tâches dans des systèmes distribués avec des files d'attente de messages.
- **Traitement en temps réel** : Gérez des tâches comme le redimensionnement d'images ou l'envoi de notifications en quasi temps réel.

---

## Ressources utiles

- [Documentation officielle de Celery](https://docs.celeryq.dev/en/stable/)
- [Dépôt GitHub de Celery](https://github.com/celery/celery)
- [Tutoriel Real Python sur Celery](https://realpython.com/asynchronous-tasks-with-django-and-celery/)

---

## Bonnes pratiques

- **Choisissez le bon courtier** : Utilisez RabbitMQ pour une fiabilité en production ou Redis pour des configurations plus simples.
- **Surveillez les tâches** : Utilisez des outils comme Flower (`pip install flower`, exécutez `celery -A tasks flower`) pour surveiller les files d'attente et les workers.
- **Gérez les erreurs** : Implémentez des nouvelles tentatives de tâches et un journalisation des erreurs pour gérer les échecs de manière élégante.
- **Optimisez la concurrence des workers** : Ajustez le nombre de processus workers (option `-c`) en fonction des cœurs CPU et de l'intensité des tâches.
- **Sécurisez votre courtier** : Configurez l'authentification et le chiffrement pour les courtiers en production (par exemple, identifiants utilisateur RabbitMQ).
- **Testez les tâches localement** : Utilisez un courtier léger comme Redis pour le développement et les tests.