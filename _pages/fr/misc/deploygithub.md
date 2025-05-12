---
title: "Déployer sur un serveur"
permalink: /fr/deploy-vps/
lang: fr
date: 2025-05-05T12:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: dev
---

> **_NOTE:_** L'approche proposée repose sur GitHub et Vultr, il serait possible d'ajuster pour un autre fournisseur de serveurs. Dans Vultr, on peut mettre en place un serveur avec Docker (2vCPUs / 4Gb / Shared CPU) pour les besoins d'un prototype. 

## Pré-requis

- Un dépôt GitHub avec un `docker-compose.yml`
- Un serveur Ubuntu (accessible par SSH)
- Docker et Docker Compose installés sur le serveur
- Un utilisateur non root avec les droits Docker sur le serveur (on utilise `deploy`)

---

## Quelques commandes à connaître

On doit se connecter avec `root`par terminal sur le serveur.

### Ajouter un utilisateur `deploy` avec accès à Docker

```bash
# Ajouter l'utilisateur 'deploy' au groupe docker
sudo usermod -aG docker deploy
```

### Créer un répertoire pour héberger le projet

```bash
# Créer le dossier du projet
sudo mkdir -p /home/deploy/myproject

# Donner les droits à l'utilisateur 'deploy'
sudo chown -R deploy:deploy /home/deploy/myproject
```

Vous pouvez choisir de déployer dans un autre répertoire, il faudra ajuster les scripts.

## Étapes pour créer un déploiement automatique

### 1. Générer une clé SSH

Sur une machine locale (ou un environnement sécurisé) :

```bash
ssh-keygen -t rsa -b 4096 -C "github-actions" -f github-actions-key
````

> ❗ Ne pas mettre de passphrase.

---

### 2. Copier la **clé publique** sur ton serveur

```bash
ssh-copy-id -i ~/.ssh/github-actions-key.pub deploy@your-server-ip
```

Vérifie ensuite que tu peux te connecter :

```bash
ssh -i ~/.ssh/github-actions-key deploy@your-server-ip
```

---

### 3. Ajouter la **clé privée** dans GitHub

1. Va dans ton dépôt GitHub → ⚙️ `Settings` → `Secrets and variables` → `Actions`
2. Crée un **secret** nommé `VPS_KEY`
3. Colle le contenu de `github-actions-key` (clé privée)
4. Crée un **secret** nommé `VPS_USER`
5. Écris le nom de l'utilisateur choisi (ex: `deploy`)
6. Crée un **secret** nommé `VPS_HOST`
7. Écris l'IP du serveur choisi (celui que l'on retrouve ci-dessus : `your-server-ip`)

---

### 4. Créer le workflow GitHub Actions

Dans ton dépôt GitHub, crée le fichier `.github/workflows/deploy.yml` :

{% raw %}
```yaml
name: Deploy to VPS

on:
  workflow_dispatch:
  # push:
  #   branches:
  #     - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Copy files to VPS via SSH
      uses: appleboy/scp-action@v0.1.4
      with:
        host: ${{ secrets.VPS_HOST }}
        username: ${{ secrets.VPS_USER }}
        key: ${{ secrets.VPS_KEY }}
        source: "."
        target: "/home/deploy/myproject"

    - name: Run Docker Compose on VPS
      uses: appleboy/ssh-action@v1.0.0
      with:
        host: ${{ secrets.VPS_HOST }}
        username: ${{ secrets.VPS_USER }}
        key: ${{ secrets.VPS_KEY }}
        script: |
          cd /home/deploy/myproject
          docker compose pull
          docker compose up -d --build
```
{% endraw %}

> Remplacez `/home/deploy/myproject` par celui de votre serveur.

---

## Résultat attendu

Je dispose d'une action manuelle qui va déclencher le `workflow`. Toutefois si je souhaite automatiser le déploiement suite à une mise à jour sur la branche `main`, je peux utiliser le déclencheur:

```yaml
name: Deploy to VPS

on:
  push:
    branches:
      - main
```

À chaque besoin, GitHub Actions :

1. Se connecte à ton serveur via SSH
2. Met à jour le code
3. Redémarre proprement les conteneurs avec Docker Compose