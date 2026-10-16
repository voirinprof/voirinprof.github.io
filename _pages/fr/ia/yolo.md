---
title: "Détection de voitures sur des tuiles Bing avec YOLO"
permalink: /fr/yolo-detection/
lang: fr
date: 2026-10-15T00:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: ai
---

## Présentation

Cette page présente une démarche de détection de voitures dans des images de télédétection avec **YOLO** et Ultralytics. Elle est basée sur le [notebook Colab YOLO](https://colab.research.google.com/drive/15DFRZD29aqEAIpSWFuIo1Qk0FZu2871d?usp=sharing), qui prépare des tuiles Bing annotées, entraîne un modèle, évalue ses résultats et applique le modèle final à des images de test.

Contrairement à [YOLO-OBB](/fr/yolo-obb/), qui utilise des boîtes orientées et un modèle préentraîné pour l’inférence, ce notebook entraîne un détecteur YOLO à boîtes rectangulaires alignées sur les axes de l’image. Contrairement à Faster R-CNN, il s’appuie sur l’API d’entraînement Ultralytics plutôt que sur une boucle d’entraînement PyTorch écrite dans le notebook.

## 1. Préparer les images et annotations

Le notebook travaille sur des tuiles Bing et des annotations de voitures créées avec l’outil d’annotation VIA. Il convertit chaque rectangle VIA — défini par son coin supérieur gauche, sa largeur et sa hauteur en pixels — au format YOLO :

```text
identifiant_classe x_centre_normalisé y_centre_normalisé largeur_normalisée hauteur_normalisée
```

Les coordonnées sont divisées par la largeur ou la hauteur de l’image pour obtenir des valeurs entre 0 et 1. Voici le calcul illustré dans le notebook :

```python
x_center = (x_region + width_region / 2) / image_width
y_center = (y_region + height_region / 2) / image_height
width = width_region / image_width
height = height_region / image_height

label_line = f"0 {x_center} {y_center} {width} {height}"
```

Chaque fichier texte porte le même nom que son image, avec l’extension `.txt`. L’identifiant de classe `0` correspond ici à l’unique classe, `car`. Il faut vérifier visuellement quelques annotations après conversion : une erreur de dimensions ou de correspondance image-fichier fausse tout l’entraînement.

## 2. Créer les ensembles train, validation et test

Le notebook sépare les paires image/annotation en trois sous-ensembles. La répartition proposée est de 80 % pour l’entraînement, 10 % pour la validation et le reste pour le test :

```python
train_ratio = 0.8
val_ratio = 0.1

train_count = int(len(label_files) * train_ratio)
val_count = math.ceil(len(label_files) * val_ratio)

train_files = shuffled_files[:train_count]
val_files = shuffled_files[train_count : train_count + val_count]
test_files = shuffled_files[train_count + val_count :]
```

* **train** sert à ajuster les poids du modèle ;
* **val** sert à suivre les performances pendant l’entraînement et à régler les paramètres ;
* **test** sert à l’évaluation finale, une fois les choix de modèle faits.

Une graine aléatoire fixe rend le partage reproductible. Pour des tuiles géographiques voisines, un partage spatial par secteur est souvent plus fiable qu’un tirage aléatoire : il limite le risque que des tuiles presque identiques se retrouvent dans train et test et donnent une estimation trop optimiste.

## 3. Décrire le jeu de données pour Ultralytics

Le notebook crée un fichier YAML qui indique où sont les images et quelles classes le modèle doit détecter :

```yaml
path: /content/dataset
train: images/train
val: images/val
test: images/test
nc: 1
names:
  - car
```

Le chemin `path` désigne la racine du jeu de données; `train`, `val` et `test` pointent vers leurs images respectives. Les annotations YOLO sont rangées dans les répertoires `labels/` correspondants. `nc` est le nombre de classes et `names` fournit leur nom dans l’ordre des identifiants.

## 4. Entraîner le détecteur

Le notebook utilise YOLOv8 moyen (`yolov8m`) comme point de départ et entraîne le modèle avec les paramètres configurés :

```python
from ultralytics import YOLO

model = YOLO("yolov8m.pt")
model.train(
    data="/content/cars.yaml",
    epochs=100,
    imgsz=640,
    batch=10,
    name="car_yolo",
)
```

* `epochs` contrôle le nombre de passages sur les données d’entraînement ;
* `imgsz` règle la taille des images utilisées par le modèle ;
* `batch` indique le nombre d’images traitées par lot et doit tenir dans la mémoire disponible ;
* la taille du modèle (`yolov8n`, `yolov8s`, `yolov8m`, etc.) représente un compromis entre ressources, vitesse et capacité.

**Avantage :** Ultralytics prend en charge la boucle d’entraînement, la validation périodique et la sauvegarde des poids, ce qui permet de se concentrer sur les annotations et les paramètres de l’expérience. L’entraînement peut utiliser un GPU lorsqu’il est disponible.

## 5. Suivre et évaluer les performances

Le notebook examine les pertes d’entraînement et de validation ainsi que plusieurs mesures de détection :

* **Box loss** mesure l’erreur de localisation des boîtes ;
* **précision** indique la proportion des détections produites qui sont correctes ;
* **rappel** indique la proportion des objets réels qui ont été détectés ;
* **mAP@0.5** résume la précision moyenne avec un seuil IoU de 0,5 ;
* **mAP@0.5:0.95** agrège les résultats pour plusieurs seuils IoU, et donne une évaluation plus exigeante.

Après les réglages, le notebook évalue le modèle sur le jeu de test séparé :

```python
metrics = model.val(data="/content/cars.yaml", split="test")

precision = metrics.results_dict["metrics/precision(B)"]
recall = metrics.results_dict["metrics/recall(B)"]
map50 = metrics.results_dict["metrics/mAP50(B)"]
map50_95 = metrics.results_dict["metrics/mAP50-95(B)"]
```

Ne choisissez pas un modèle uniquement parce que sa perte d’entraînement diminue. Comparez les métriques train/validation, inspectez les faux positifs et les objets manqués, puis réservez le test à l’évaluation finale pour éviter d’ajuster indirectement le modèle sur celui-ci.

## 6. Prédire et examiner les détections

Une fois entraîné, le modèle peut traiter une image de test. Ultralytics retourne notamment les boîtes, classes et scores, et peut dessiner les prédictions :

```python
results = model.predict(
    "/content/dataset/images/test/tuile.jpg",
    conf=0.25,
)

annotated_image = results[0].plot()
results[0].show()
```

Le seuil `conf` permet de modifier le compromis entre détections retenues et confiance minimale. Le notebook visualise les images annotées et exporte aussi les informations utiles pour analyser les résultats.

## 7. Géoréférencer et cartographier les détections

Les noms des tuiles Bing, par exemple `tile_157460_187752_z19.jpg`, encodent les indices de tuile `X`, `Y` et le niveau de zoom. Le notebook transforme le centre pixel de chaque boîte en latitude/longitude à partir de ces indices et de la grille Web Mercator. Il enregistre les détections dans un tableau comportant notamment la classe, la confiance, la tuile et les coordonnées, puis les représente sur une carte interactive avec les images de tuiles en fond.

Cette géolocalisation suppose que les noms, dimensions et niveau de zoom des tuiles sont exacts. Elle sert ici à positionner le centre des objets détectés; pour produire des géométries précises dans un SIG, il faudrait aussi transformer les coins des boîtes et tenir compte de la projection.

## Avantages et points d’attention

* **Entraînement sur une classe adaptée :** le modèle est ajusté à partir d’annotations de voitures, contrairement à une inférence directe avec un modèle générique.
* **Pipeline accessible :** la préparation des données, l’entraînement, l’évaluation et la prédiction sont pilotés par l’API Ultralytics.
* **Évaluation séparée :** train, validation et test ont des rôles distincts; le jeu de test permet de mesurer le modèle sur des images non utilisées pour l’ajustement.
* **Boîtes non orientées :** les rectangles sont alignés sur les axes de l’image. Pour les objets allongés dont l’orientation importe, voir la page [YOLO-OBB](/fr/yolo-obb/).
* **Données représentatives :** la qualité et la diversité des annotations déterminent en grande partie les performances; vérifiez plusieurs régions, conditions d’image et types d’objets.
* **Validation géographique :** des tuiles proches peuvent être très corrélées; un partage géographique évite une fuite spatiale entre train et test.
* **Images de fond :** avant de redistribuer le jeu de données ou les résultats avec les images, vérifiez les conditions d’accès et de réutilisation des tuiles Bing.

Le notebook inclut également des cellules de sauvegarde et de transfert du modèle entraîné. Pour les utiliser, protégez les identifiants et clés d’accès, évitez de les inscrire directement dans le notebook et vérifiez la sécurité du service destinataire.

Consultez le [notebook Colab YOLO](https://colab.research.google.com/drive/15DFRZD29aqEAIpSWFuIo1Qk0FZu2871d?usp=sharing) pour exécuter la démarche et expérimenter avec les paramètres d’entraînement.
