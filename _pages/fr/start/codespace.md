---
title: "Premiers pas avec GitHub Codespaces"
permalink: /fr/codespaces-intro/
lang: fr
date: 2025-04-26T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: start
---
> **_NOTE:_** La [documentation officielle de GitHub Codespaces](https://docs.github.com/fr/codespaces) fournit des instructions détaillées pour démarrer. Un exemple de github codespaces est disponible à cette adresse : [https://github.com/voirinprof/geolabspace](https://github.com/voirinprof/geolabspace)

## Introduction à GitHub Codespaces

**GitHub Codespaces** est un environnement de développement cloud intégré à GitHub. Il permet de coder directement dans un navigateur ou via Visual Studio Code, sans configurer d’environnement local. Préinstallé avec Python, Git, et d’autres outils, il est idéal pour débuter rapidement en programmation.

Ce guide vous montre comment créer un Codespace, écrire un programme Python simple, et gérer votre code avec Git.

Avant de commencer :
- **Définir l’objectif** : Par exemple, écrire un script Python qui calcule la somme de nombres.
- **Préparer les outils** : Aucun logiciel local n’est requis, juste un compte GitHub.
- **Planifier** : Écrire le code, tester, et sauvegarder les modifications dans un dépôt.

## Créer un Codespace

1. **Accéder à un dépôt** : Connectez-vous à GitHub, créez ou ouvrez un dépôt (par exemple, un projet vide ou un fork d’un dépôt existant).
2. **Lancer un Codespace** : Cliquez sur **Code** > **Codespaces** > **Créer un codespace sur la branche principale**. GitHub configure un environnement Ubuntu avec Python en quelques secondes.
3. **Ouvrir l’éditeur** : Le Codespace s’ouvre dans votre navigateur ou dans VS Code (si connecté). Vous disposez d’un terminal et d’un éditeur de code.

![Ouvrir Codespaces](/assets/img/codespace-step1.png)

> Conseil : Si vous créez un nouveau dépôt, ajoutez un fichier README pour initialiser la branche principale.

## Écrire un programme Python simple

Dans votre Codespace, créez un fichier nommé `somme.py` pour calculer la somme d’une liste de nombres.

Exemple de code :

```python
def calculer_somme(nombres):
    """Calcule la somme d'une liste de nombres."""
    return sum(nombres)

# Exemple d'utilisation
liste = [1, 2, 3, 4, 5]
resultat = calculer_somme(liste)
print(f"La somme de {liste} est {resultat}")
```

Pour exécuter le programme :
1. Ouvrez le terminal dans le Codespace (raccourci : `Ctrl + ~`).
2. Exécutez la commande :

```shell
python somme.py
```

Sortie attendue :
```
La somme de [1, 2, 3, 4, 5] est 15
```

> Conseil : Utilisez l’extension Python dans VS Code (préinstallée dans Codespaces) pour bénéficier de l’autocomplétion et du débogage.

## Installer des bibliothèques

Si votre projet nécessite des bibliothèques Python, utilisez `pip`. Par exemple, pour installer `numpy` :

```shell
pip install numpy
```

Exemple d’utilisation de `numpy` pour une somme :

```python
import numpy as np

nombres = np.array([1, 2, 3, 4, 5])
somme = np.sum(nombres)
print(f"La somme avec numpy est {somme}")
```

Exécutez avec :
```shell
python votre_fichier.py
```

## Gérer le code avec Git

Codespaces est intégré à Git, ce qui facilite la gestion des versions.

1. **Vérifier les modifications** :
```shell
git status
```

2. **Ajouter les fichiers** :
```shell
git add somme.py
```

3. **Créer un commit** :
```shell
git commit -m "Ajout du script de calcul de somme"
```

4. **Pousser vers GitHub** :
```shell
git push origin main
```

Vos modifications sont maintenant sauvegardées dans le dépôt GitHub.

> Conseil : Validez fréquemment vos changements pour éviter de perdre votre travail, surtout si le Codespace est arrêté.

## Bonnes pratiques

- **Organisez vos fichiers** : Gardez une structure claire (par exemple, un dossier `scripts/` pour les fichiers Python).
- **Testez votre code** : Ajoutez des cas de test simples pour vérifier les résultats (par exemple, tester la fonction avec `[]` ou `[-1, 1]`).
- **Documentez** : Utilisez des docstrings et des commentaires pour expliquer votre code.
- **Utilisez un fichier requirements.txt** : Listez les dépendances pour faciliter la réutilisation :
```text
numpy==1.26.4
```

- **Sauvegardez régulièrement** : Les Codespaces sont temporaires ; poussez votre code sur GitHub pour le conserver.

## Ressources utiles

- [Documentation GitHub Codespaces](https://docs.github.com/fr/codespaces) : Guide pour configurer et utiliser Codespaces.
