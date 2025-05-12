---
title: "Deploy to a Server"
permalink: /en/deploy-vps/
lang: en
date: 2025-05-05T12:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: dev
---

> **_NOTE:_** This approach is based on GitHub and Vultr, but can be adapted to other server providers. On Vultr, you can deploy a Docker server (2 vCPUs / 4 GB RAM / Shared CPU) suitable for prototyping needs.

## 🔧 Prerequisites

- A GitHub repository containing a `docker-compose.yml`
- An Ubuntu server (accessible via SSH)
- Docker and Docker Compose installed on the server
- A non-root user with Docker privileges on the server (we’ll use `deploy`)

---

## Useful Commands

Connect to your server as `root` using a terminal.

### Add a `deploy` user with Docker access

```bash
# Add 'deploy' user to the Docker group
sudo usermod -aG docker deploy
```

### Create a directory to host the project

```bash
# Create the project directory
sudo mkdir -p /home/deploy/myproject

# Grant permissions to the 'deploy' user
sudo chown -R deploy:deploy /home/deploy/myproject
```

You can choose a different directory name—just make sure to update your scripts accordingly.

---

## Steps to Set Up Automatic Deployment

### 1. Generate an SSH Key

On your local machine (or a secure environment):

```bash
ssh-keygen -t rsa -b 4096 -C "github-actions" -f github-actions-key
```

> ❗ Do not set a passphrase.

---

### 2. Copy the **public key** to your server

```bash
ssh-copy-id -i ~/.ssh/github-actions-key.pub deploy@your-server-ip
```

Then verify you can connect:

```bash
ssh -i ~/.ssh/github-actions-key deploy@your-server-ip
```

---

### 3. Add the **private key** to GitHub

1. Go to your GitHub repository → ⚙️ `Settings` → `Secrets and variables` → `Actions`
2. Create a **secret** named `VPS_KEY`
   Paste the content of `github-actions-key` (the private key)
3. Create a **secret** named `VPS_USER`
   Enter your chosen username (e.g., `deploy`)
4. Create a **secret** named `VPS_HOST`
   Enter the IP address of your server (`your-server-ip` as used above)

---

### 4. Create the GitHub Actions workflow

In your GitHub repo, create the file `.github/workflows/deploy.yml`:

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

> Replace `/home/deploy/myproject` with the actual path on your server.

---

## Expected Result

You now have a manual GitHub Action workflow that can deploy your application.
If you want to automate deployment after a push to the `main` branch, you can change the trigger to:

```yaml
name: Deploy to VPS

on:
  push:
    branches:
      - main
```

Each time this runs, GitHub Actions will:

1. Connect to your server via SSH
2. Pull the latest code
3. Restart the containers cleanly using Docker Compose
