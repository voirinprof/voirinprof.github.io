---
title: "Scalabilité et haute disponibilité"
permalink: /fr/scalability/
lang: fr
date: 2025-04-24T03:02:20+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: dev
---

# Trois approches pour déployer une application web avec PostGIS, Flask et Nginx

Ce guide présente trois méthodes de plus en plus robustes pour déployer une application web utilisant Flask (backend), PostGIS (base de données spatiale), et Nginx (serveur HTTP).

---

## 1. Déploiement simple avec Docker Compose

### Objectif :
Lancer les services Flask, PostGIS, et Nginx dans des conteneurs Docker, sans équilibrage de charge.

### Structure :
- `docker-compose.yml`
- `app/` → code Flask
- `nginx/` → config Nginx
- `db/` → PostGIS (image officielle)

### Points clés :
- Facile à mettre en place.
- Adapté aux environnements de développement et aux petites applis.

### Exemple du docker-compose.yml :
```yaml
version: '3.8'

services:
  flask:
    build:
      context: ./app
      dockerfile: Dockerfile
    container_name: flask
    environment:
      - DB_HOST=postgis
      - DB_NAME=gisdb
      - DB_USER=gisuser
      - DB_PASSWORD=gispassword
      - DB_PORT=5432
    depends_on:
      - postgis
    networks:
      - app-network
    restart: unless-stopped
    deploy:
      replicas: 3  # 3 instances Flask pour la répartition de charge
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/points"]
      interval: 30s
      timeout: 10s
      retries: 3

  postgis:
    image: postgis/postgis:15-3.4
    container_name: postgis
    environment:
      - POSTGRES_DB=gisdb
      - POSTGRES_USER=gisuser
      - POSTGRES_PASSWORD=gispassword
    volumes:
      - postgis-data:/var/lib/postgresql/data
    networks:
    - app-network
    restart: unless-stopped

  nginx:
    build:
      context: ./nginx
      dockerfile: Dockerfile
    container_name: nginx
    ports:
      - "80:80"
    depends_on:
      - flask
    networks:
      - app-network
    restart: unless-stopped

networks:
  app-network:
    driver: bridge

volumes:
  postgis-data:
```

Cette approche assez simple à mettre en oeuvre fait tourner un service par **container**. On peut demander à **Docker** d'ajouter 3 services pour **flask** permettant d'absorber la demande grandissante. 

Si on teste l'approche (sans duplication des services, donc un seul **container flask** ) avec un outil de tests (comme Locust), on va voir une saturation des ressources du système et une apparition de défauts dans les requêtes.

On a simulé la charge de l'arrivée de 3 utilisateurs à la seconde sur une API avec flask permettant de faire des GET et des POST. On pouvait ainsi ajouter des points dans une base de données PostGIS ou alors faire la liste des points. Des requêtes assez simples mais qui peuvent amener des défis pour un serveur lorsque le nombre d'utilisateurs augmente. L'approche de base (sans réplication) va soutenir assez bien 200 utilisateurs permettant d'avoir 95 % des requêtes avec une réponse en dessous de 1 000 ms.

![Graphiques de Locust](/assets/img/locust/total_requests_per_second_1747318384.279.png)

Quand on ajoute l'option de dupliquer les services avec ```deploy.replicas```, on peut voir une légère amélioration de la solution. L'application va quand même saturer mais cela va se faire après 300 utilisateurs (95 % des requêtes en dessous de 1 000 ms de temps de réponse). De plus, il n'y aura pas d'apparition de défauts même lorsque l'application est saturée.

![Graphiques de Locust](/assets/img/locust/total_requests_per_second_1747321537.241.png)

Ce genre d'approche peut convenir pour les petites applications.

---

## 2. Ajout de HAProxy pour la répartition de charge

### Objectif :

On peut éviter d'utiliser nginx comme proxy mais plutôt utiliser **HAProxy** (High Availability Proxy). C'est un bon **load balancer** très performant et largement utilisé, notamment dans les environnements de production pour répartir la charge entre plusieurs instances d’un service web.

On va reposer sur 3 **containers** flask comme expliqué plus haut. Au lieu que ce soit nginx qui répartit la charge, on va définir haproxy en mode ```leastconn```, autrement dit on envoie la requête vers le container avec le moins de trafic. Il faut savoir qu'il existe de nombreuses stratégies d'équilibrage, car certaines offrent la possibilité d'orienter un même utilisateur toujours vers le même **container**. Ainsi l'utilisateur pourrait bénéficier de la cache des opérations passées. Il y aussi des stratégies plus aléatoires ou simplement séquentielles, on envoie sur le prochain **container** à chaque fois.

### Modifications :

* Toujours plusieurs conteneurs Flask (`flask1`, `flask2`, …)
* HAProxy placé devant, pour dispatcher les requêtes.
* Toujours avec `docker-compose`.

### Nouveau service dans `docker-compose.yml` :

```yaml
 haproxy:
   build:
     context: ./haproxy
     dockerfile: Dockerfile
   container_name: haproxy
   ports:
     - "80:80"  # Bind port 80 to host
   depends_on:
     - flask
   networks:
     - app-network
   restart: unless-stopped
```

On peut définir un fichier Dockerfile

```bash
FROM haproxy:2.8
COPY haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg
EXPOSE 80
```

Ensuite au niveau de la config, on a :

```bash
global
    log /dev/log local0
    maxconn 10000
    tune.ssl.default-dh-param 2048
    nbthread 4

defaults
    log global
    mode http
    option httplog
    timeout connect 5000
    timeout client 30000
    timeout server 30000
    option forwardfor
    option http-server-close
    maxconn 5000
frontend http_front
    bind *:80
    default_backend flask_backend
    log /dev/log local0 debug

backend flask_backend
    balance leastconn
    option httpchk GET /status
    http-check expect status 200
    server flask1 flask:5000 check inter 5000 rise 3 fall 5
    server flask2 flask:5000 check inter 5000 rise 3 fall 5
    server flask3 flask:5000 check inter 5000 rise 3 fall 5    
```

**haproxy** va vérifier les **containers** pour chaque requête et orienter vers celui le moins occupé.

Cette approche donne un résultat intéressant car au lieu de saturer autour de 200 utilisateurs, l'application résiste bien jusqu'à 300 utilisateurs. On obtient encore 95 % des requêtes avec une réponse en dessous de 1 000 ms.

![Graphiques de Locust](/assets/img/locust/total_requests_per_second_1747336718.142.png)

Cette approche se greffe assez bien sur une application déjà définie avec un docker-compose.yml, on remplace uniquement nginx par haproxy. 

---

## 3. Déploiement avec MicroK8s (Kubernetes)

### Objectif :

Si on souhaite aller à un niveau supérieur, on peut alors envisager une solution comme **MicroK8s**, une version légère et simplifiée de Kubernetes.

On est dans un autre univers, donc l'application a ses propres configurations et commandes.

### Installation

Il faut passer par une étape d'installation. Je considère que vous disposez d'un système sous Debians avec Docker déjà installé.

#### Étape 1 : Mettre à jour le système et installer les dépendances

Mettez à jour le système et installez les outils essentiels nécessaires pour MicroK8s.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git apt-transport-https ca-certificates gnupg lsb-release software-properties-common
```

#### Étape 2 : Installer Snapd
Installez et activez Snapd, nécessaire pour installer MicroK8s via Snap.

```bash
sudo apt install -y snapd
sudo systemctl enable --now snapd
```

#### Étape 3 : Installer MicroK8s
Installez MicroK8s avec Snap en utilisant l'option `--classic` pour un accès complet au système.

```bash
sudo snap install microk8s --classic
```

Attention il est possible que cette opération ne fonctionne à cause de la version désuète de ```snap```. On peut simplement demander à snap de mettre à jour snap.

```bash
sudo snap install snapd
```

#### Étape 4 : Configurer les permissions utilisateur
Ajoutez l'utilisateur actuel au groupe `microk8s` pour exécuter les commandes MicroK8s sans sudo, et activez l'appartenance au groupe.

```bash
sudo usermod -a -G microk8s $USER
newgrp microk8s
```

#### Étape 5 : Activer les add-ons MicroK8s
Activez les add-ons essentiels pour prendre en charge DNS, Ingress, le stockage, Helm, le tableau de bord et MetalLB pour l'équilibrage de charge.

```bash
microk8s enable dns ingress storage helm dashboard metallb
```

- **Intérêts** :
  - `dns` : Fournit une résolution DNS interne.
  - `ingress` : Active l'Ingress pour le routage HTTP externe.
  - `storage` : Configure le stockage persistant.
  - `helm` : Installe Helm pour gérer les applications Kubernetes.
  - `dashboard` : Fournit l'interface web de Kubernetes.
  - `metallb` : Offre un équilibrage de charge pour les services.

#### Étape 6 : Vérifier le cluster MicroK8s
Vérifiez l'état du cluster MicroK8s et assurez-vous que tous les composants fonctionnent.

```bash
microk8s status --wait-ready
microk8s kubectl get nodes
microk8s kubectl get pods -A
```

#### Étape 7 : Construire et importer l'image de l'application Flask
Naviguez vers le répertoire de l'application, construisez l'image Docker Flask et importez-la dans MicroK8s.

```bash
cd kubernetes-k8s/app/
docker build -t my-flask-app:latest .
docker save my-flask-app:latest | microk8s ctr image import -
```

#### Étape 8 : Déployer l'application avec Helm
On doit créer des fichiers de configuration (Chart.yaml, values.yaml, templates/deployment.yaml, templates/ingress.yaml, templates/service.yaml) pour chaque composant.

Par exemple pour le composant **flask**, ce sera :

Pour values.yaml

```yaml
replicaCount: 3

image:
  repository: my-flask-app
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 5000

ingress:
  enabled: true
  hosts:
    - host: flask.local
      paths:
        - path: /
          pathType: Prefix
```

Pour Chart.yaml

```yaml
apiVersion: v2
name: flask-app
description: A Helm chart for deploying a Flask application
version: 0.1.0
appVersion: "1.0"
```

Pour deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-flask
  labels:
    app: flask
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: flask
  template:
    metadata:
      labels:
        app: flask
    spec:
      containers:
        - name: flask
          image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: 5000
          env:
            - name: DB_HOST
              value: "postgis-service.default.svc.cluster.local"
            - name: DB_NAME
              value: "gisdb"
            - name: DB_USER
              value: "gisuser"
            - name: DB_PASS
              value: "gispassword"
            - name: DB_PORT
              value: "5432"
```
Pour ingress.yaml

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-flask
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  {{- range .Values.ingress.hosts }}
    - host: {{ .host }}
      http:
        paths:
        {{- range .paths }}
          - path: {{ .path }}
            pathType: {{ .pathType }}
            backend:
              service:
                name: {{ $.Release.Name }}-flask
                port:
                  number: {{ $.Values.service.port }}
        {{- end }}
  {{- end }}
{{- end }}
```

Pour service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-flask
spec:
  selector:
    app: flask
  ports:
    - protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: 5000
  type: {{ .Values.service.type }}
```

Ensuite naviguez vers le répertoire des charts Helm et déployez les composants PostGIS, Flask et Nginx avec Helm.

```bash
cd ../helm/
microk8s helm3 install postgis-release postgis/
microk8s helm3 install flask-release flask-app/
microk8s helm3 install nginx-release nginx/
```

#### Étape 9 : Initialiser la base de données PostGIS
Si on doit créer la base de données on peut accéder au pod PostGIS pour créer une table géospatiale pour stocker des points.

```bash
microk8s kubectl exec -it $(microk8s kubectl get pod -l app=postgis -o jsonpath="{.items[0].metadata.name}") -- psql -U gisuser -d gisdb
```

Dans l'interface psql, exécutez :
```sql
CREATE EXTENSION postgis;
CREATE TABLE points (id SERIAL PRIMARY KEY, name VARCHAR(100), geom GEOMETRY(POINT, 4326));
```

#### Étape 10 : Tester l'application
Vérifiez l'application en envoyant des requêtes HTTP au point de terminaison Flask via l'Ingress.

```bash
curl -X POST http://flask.local/point -H "Content-Type: application/json" -d '{"longitude": 2.3522, "latitude": 48.8566, "name": "Paris"}'
curl http://flask.local/points
```


#### Étape 11 : Surveiller et déboguer
Vérifiez l'état des pods, les journaux et les services pour s'assurer que tout fonctionne correctement.

```bash
microk8s kubectl get pods
microk8s kubectl get svc
microk8s kubectl logs <nom-du-pod-flask>
microk8s kubectl logs <nom-du-pod-nginx>
```

#### Étape 12 : Mettre à jour l'application Flask (facultatif)
Si des modifications sont nécessaires, mettez à jour les valeurs du chart Helm et redéployez.

```bash
nano helm/flask-app/values.yaml
microk8s helm3 upgrade flask-release helm/flask-app/
```

#### Étape 13 : Nettoyer MicroK8s
Désinstallez les releases Helm, désactivez les add-ons et arrêtez MicroK8s si vous passez à une autre configuration.

```bash
microk8s helm uninstall flask-release
microk8s helm uninstall postgis-release
microk8s helm uninstall nginx-release
microk8s disable metallb
microk8s disable ingress
microk8s stop
```
### Architecture :

La solution de Microk8s repose sur des images de containers Docker. Il suffit de construire l'image et de demander à Microk8s de l'utiliser.

* **Deployments** : Flask, PostGIS, Nginx
* **Services** : ClusterIP + Ingress
* **Replicas** : paramétrables
* **PersistentVolume** pour PostGIS

En utilisant cette approche, on arrive à des résultats intéressants. Le serveur sature autour de 300 utilisateurs, mais il n'y aura aucun défaut. Le serveur répond toujours mais effectivement la réponse devient de plus en plus lente avec le nombre d'utilisateurs qui s'ajoute.

![Graphiques de Locust](/assets/img/locust/total_requests_per_second_1747340627.868.png)

Les temps de réponse sont bien en dessous de ceux obtenus par les autres solutions, ce qui laisse sous-entendre que l'on pourrait trouver une configuration logicielle et matérielle qui donnerait de très bon résultat avec ce genre d'approche. L'inconvénient majeur est qu'il faut effectivement bien maîtriser Microk8s avec tous les fichiers config et les commandes.

---
### Configuration matérielle

Tous les tests ont été effectués sur le même serveur, donc les ressources ont toujours été les mêmes (16 Gb de ram et 6 vCPUs). Si on souhaitait obtenir une application performante, on pourrait augmenter le nombre de processeurs et la mémoire. 

De plus, le dimensionnement de l'application dépend grandement des besoins et des utilisateurs. Toutes les solutions présentées pourraient largement répondre aux besoins d'une application avec 50 utilisateurs faisant de multiples opérations en même temps. Afin d'être sûr que la solution répond bien aux attentes, on peut conseiller de simuler des tests de montée en charge avec Locust. Il suffirait d'identifier des opérations sur le système qui seraient faites souvent ou qui seraient gourmandes en ressources.

---

## Ressources utiles

* [MicroK8s](https://microk8s.io/)
* [HAProxy](http://www.haproxy.org/)
* [Docker Compose](https://docs.docker.com/compose/)
* [PostGIS Docker](https://hub.docker.com/r/postgis/postgis/)
* [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
