---
title: "Introduction aux classes en Python"
permalink: /fr/python-classes-intro/
lang: fr
date: 2025-04-26T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: start
---

> **_NOTE:_** La [documentation officielle de Python](https://docs.python.org/fr/3/tutorial/classes.html) propose des guides détaillés et des exemples sur l’utilisation des classes. Cette [chaîne Youtube](https://www.youtube.com/@YvesVoirin) offre aussi de nombreuses ressources.

## À savoir

Pour cette introduction, aucune bibliothèque externe n’est requise, car les fonctionnalités intégrées de Python suffisent pour définir et utiliser des classes. Cependant, certaines bibliothèques peuvent être utiles pour des projets avancés impliquant des classes :

* **dataclasses** : Simplifie la création de classes pour stocker des données avec un minimum de code (intégré depuis Python 3.7).
* **pydantic** : Fournit une validation de données et une gestion des paramètres avec les annotations de type.
* **attrs** : Permet de définir des classes avec moins de code, similaire à dataclasses.


## Définir et utiliser des classes

### Qu’est-ce qu’une classe ?

Les classes en Python sont des modèles pour créer des objets, qui sont des instances d’une classe (en mémoire de l'ordinateur). Elles permettent de regrouper des données (attributs) et des comportements (méthodes) dans une seule entité, favorisant un code organisé et réutilisable.

Pour définir une classe, utilisez le mot-clé `class` suivi du nom de la classe (généralement en CamelCase) et deux-points. Le corps de la classe est indenté.

Exemple d’une classe simple représentant un point dans un espace 2D :

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

- `__init__` : Le constructeur, appelé lors de la création d’une instance.
- `self` : Référence à l’instance de la classe, utilisée pour accéder aux attributs et méthodes.
- `x` et `y` : Attributs d’instance, stockant les coordonnées du point.

Pour créer une instance (objet) de la classe `Point` :

```python
# Créer un point à (3, 4)
p1 = Point(3, 4)
print(p1.x)  # Affiche : 3
print(p1.y)  # Affiche : 4
```

### Ajouter des méthodes

Les méthodes sont des fonctions définies à l’intérieur d’une classe qui agissent sur ses instances. Elles prennent `self` comme premier paramètre.

En Python, `self` est une convention utilisée pour représenter l'instance actuelle d'une classe dans les méthodes de cette classe. C'est un paramètre implicite qui permet à une méthode d'accéder aux attributs et autres méthodes de l'objet sur lequel elle est appelée.

* Rôle de self

`self` désigne l'instance spécifique de la classe sur laquelle une méthode est appelée. Chaque objet créé à partir d'une classe a ses propres données (attributs), et `self` permet d'accéder à ces données ou de les modifier.
C'est un moyen pour Python de savoir à quel objet une méthode fait référence.

* Pourquoi `self` est nécessaire ?

Quand vous appelez une méthode sur un objet (par exemple, mon_objet.ma_methode()), Python passe automatiquement une référence à l'objet comme premier argument à la méthode. Par convention, cet argument est nommé `self`. Cela permet à la méthode de travailler avec les données spécifiques de cet objet.

Exemple d’une classe avec une méthode pour calculer la distance à l’origine (0, 0) :

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def distance_depuis_origine(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5
```

Utilisation de la méthode :

```python
p1 = Point(3, 4)
distance = p1.distance_depuis_origine()
print(distance)  # Affiche : 5.0
```

> Conseil : Utilisez des noms de méthodes descriptifs pour clarifier leur rôle, par exemple `distance_depuis_origine` plutôt que `dist`.

## Travailler avec les attributs et méthodes de classe

### Attributs de classe

Les attributs de classe sont partagés par toutes les instances d’une classe, contrairement aux attributs d’instance, qui sont uniques à chaque objet.

Exemple d’une classe avec un attribut de classe pour suivre le nombre de points créés :

```python
class Point:
    # Attribut de classe
    compteur = 0
    
    def __init__(self, x, y):
        self.x = x
        self.y = y
        Point.compteur += 1  # Incrémenter le compteur à chaque nouveau point
```

Utilisation de l’attribut de classe :

```python
p1 = Point(1, 2)
p2 = Point(3, 4)
print(Point.compteur)  # Affiche : 2
```

### Méthodes de classe et méthodes statiques

- **Méthodes de classe** : Agissent sur la classe elle-même, pas sur les instances. Définies avec le décorateur `@classmethod` et prennent `cls` comme premier paramètre.
- **Méthodes statiques** : N’agissent ni sur l’instance ni sur la classe, mais sont logiquement liées à la classe. Définies avec le décorateur `@staticmethod`.

Exemple :

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    @classmethod
    def depuis_tuple(cls, coords):
        return cls(coords[0], coords[1])
    
    @staticmethod
    def est_valide(x, y):
        return isinstance(x, (int, float)) and isinstance(y, (int, float))
```

Utilisation des méthodes de classe et statiques :

```python
# Créer un point à partir d’un tuple avec une méthode de classe
p1 = Point.depuis_tuple((5, 6))
print(p1.x, p1.y)  # Affiche : 5 6

# Vérifier si les coordonnées sont valides avec une méthode statique
print(Point.est_valide(3, 4))  # Affiche : True
print(Point.est_valide("3", 4))  # Affiche : False
```

## Héritage et polymorphisme

### Héritage

L’héritage permet à une classe (enfant) d’hériter des attributs et méthodes d’une autre classe (parent), favorisant la réutilisation du code.

Exemple d’une classe enfant `PointColore` héritant de `Point` :

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def decrire(self):
        return f"Point à ({self.x}, {self.y})"

class PointColore(Point):
    def __init__(self, x, y, couleur):
        super().__init__(x, y)  # Appeler le constructeur de la classe parent
        self.couleur = couleur
    
    def decrire(self):  # Redéfinir la méthode parent
        return f"PointColore à ({self.x}, {self.y}) de couleur {self.couleur}"
```

Utilisation de l’héritage :

```python
p1 = Point(1, 2)
cp1 = PointColore(3, 4, "rouge")
print(p1.decrire())   # Affiche : Point à (1, 2)
print(cp1.decrire())  # Affiche : PointColore à (3, 4) de couleur rouge
```

- `super()` : Appelle la méthode de la classe parent.
- Redéfinition de méthode : La classe enfant fournit sa propre implémentation de `decrire`.

### Polymorphisme

Le polymorphisme permet de traiter différentes classes comme des instances d’une classe parent commune, chaque classe implémentant les méthodes partagées différemment.

Exemple utilisant le polymorphisme :

```python
points = [Point(1, 2), PointColore(3, 4, "bleu")]
for point in points:
    print(point.decrire())  # Appelle la méthode decrire appropriée
```

Résultat :

```
Point à (1, 2)
PointColore à (3, 4) de couleur bleu
```

## Bonnes pratiques

- **Utilisez des noms clairs** : Les noms de classes doivent être des noms (ex. : `Point`, `PointColore`) et les méthodes doivent être des verbes ou descriptives (ex. : `distance_depuis_origine`).
- **Gardez les classes focalisées** : Chaque classe doit avoir une seule responsabilité (principe de responsabilité unique).
- **Validez les entrées** : Ajoutez des vérifications dans `__init__` ou les méthodes pour garantir des données valides (ex. : vérifier si les coordonnées sont des nombres).
- **Utilisez l’héritage avec modération** : Évitez les hiérarchies d’héritage profondes ; préférez la composition si possible.
- **Documentez vos classes** : Utilisez des docstrings pour expliquer le rôle des classes et des méthodes.

Exemple avec validation d’entrée et docstring :

```python
class Point:
    """Classe représentant un point 2D avec des coordonnées x et y."""
    
    def __init__(self, x, y):
        if not all(isinstance(coord, (int, float)) for coord in (x, y)):
            raise ValueError("Les coordonnées doivent être des nombres")
        self.x = x
        self.y = y
```

## Ressources utiles

Voici une sélection de vidéos couvrant les concepts de cette page :
* [Introduction aux classes et objets en Python](https://youtu.be/t7KTxdn19-I)
