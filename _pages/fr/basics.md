---
title: "Notions de base en Python"
permalink: /fr/python-basics/
lang: fr
date: 2025-04-24T03:02:20+00:00
toc: true
toc_label: "Dans cette page"
---

## Règles de base

Lorsque vous abordez un problème en informatique (plus ou moins complexe), il y a plusieurs étapes à suivre avant d'entamer le codage :

* Bien lire l'énoncé ou la description du problème. **Pour bien coder, il faut déjà comprendre le problème**. Il faut se dire que ce n'est pas la machine qui peut faire cela pour vous.
* Bien identifier **les paramètres en entrée** : quelles sont les données à lire ? quels sont les paramètres à saisir ? ...
* Bien comprendre ce que le programme doit fournir (**les sorties**) : quelles sont les infos à produire ? quels types de résultats souhaite l'utilisateur ? ...
* Identifier les **étapes pour passer des entrées aux sorties** : il faut pouvoir expliquer simplement la séquence d'opérations qu'il faut faire pour obtenir ce que je souhaite. Cette partie s'appelle souvent l'algorithme. C'est la recette de cuisine : on part des ingrédients (les entrées) pour arriver au gâteau (les sorties).
* Si un **problème est complexe**, on le **découpe en plus petits problèmes**. C'est la seule manière d'aborder le développement de systèmes complexes. À chaque petit problème, il y a une solution assez simple (souvent connue).
* Quand on dispose de la recette de cuisine, on peut la mettre en oeuvre. C'est le codage.
* Python est un langage assez simple et il accepte beaucoup de choses. Il est donc possible que l'interpréteur retourne tout le temps un résultat. Il faut se demander est-ce juste (même si le code ne comporte pas d'erreurs) ? Il faut s'assurer que les résultats sont justes, il faut donc **prévoir des cas de figures (des tests)**. Ce sera la seule manière de garantir que le code est bon.

En utilisant des exemples simples, on illustre le cheminement que devrait adopter un bon développeur. L'idée est de se poser les bonnes questions avant de débuter à coder sa solution.

## Les variables

En Python, les variables sont des emplacements de mémoire qui peuvent contenir des valeurs. Elles sont utilisées pour stocker des données temporaires ou permanentes qui peuvent être utilisées dans un programme. Les variables sont souvent liées aux entrées et sorties du programme.

Le nom d'une variable en Python doit commencer par une lettre, suivi de lettres, de chiffres ou du caractère souligné (_). Il est également important de noter que Python est un langage sensible à la casse, c'est-à-dire que les noms de variables "data" et "Data" sont considérés comme différents.

Pour affecter une valeur à une variable, on utilise le signe égal (=) en Python. Si on souhaite déclarer la variable `x` avec la valeur 10, il faudra écrire ceci :

```python
x = 10
```

Il est important de noter que les variables en Python peuvent être de différents types, tels que des entiers (int), des flottants (float), des chaînes de caractères (str) ou des booléens (bool). Python est également un langage à typage dynamique, ce qui signifie que le type d'une variable peut changer pendant l'exécution d'un programme.

Je peux déclarer x comme un entier comme ceci :

```python
x = 10
```

Il peut changer de type au cours du programme simplement en lui affectant une autre valeur. `x` peut devenir un float avec ceci :

```python
x = 10.0
```

> Conseil : le type de la variable peut affecter certaines opérations, donc il est important de bien vérifier ce que la variable contient.

Une variable pourra être utilisée dans le programme en utilisant son nom, par exemple si je veux ajouter la valeur 1 à `x`, il suffit de faire ceci :

```python
x = x + 1
```

> Conseil : utilisez des noms explicites pour les variables, ainsi il sera plus aisé de comprendre le programme. Ex: si j'utilise la variable `nbStudent`, on comprend assez bien qu'il s'agit du nombre des étudiants.

Vidéo utile : [Explication sur les variables en Python 3](https://youtu.be/MTZBtMsQ58E)

## Les opérateurs

En Python, les opérateurs sont des symboles spéciaux qui permettent d'effectuer des opérations sur des valeurs ou des variables. Il existe différents types d'opérateurs en Python, tels que les opérateurs arithmétiques, les opérateurs de comparaison, les opérateurs logiques, etc.

Les opérateurs arithmétiques en Python permettent d'effectuer des opérations mathématiques sur des valeurs ou des variables. Les opérateurs arithmétiques courants sont l'addition (+), la soustraction (-), la multiplication (*), la division (/), la division entière (//), le modulo (%) et l'exponentiation (**). 

Si on souhaite calculer le reste d'une division (ex: 10 divisé par 2), on utilise modulo. Dans ce cas, la variable `left` vaudra 0.
```python
x = 10
left = 10 % 2
```
Les opérateurs de comparaison en Python permettent de comparer des valeurs ou des variables. Les opérateurs de comparaison courants sont l'égalité (==), la différence (!=), l'infériorité (<), la supériorité (>), l'infériorité ou égalité (<=) et la supériorité ou égalité (>=).

Si on souhaite comparer deux variables, on peut faire ceci :

```python
x = 10
y = 20
x == y
```

Le résultat `x == y` donnera `False`, car le contenu des deux variables est différent.

Vidéo utile : [Explication des opérateurs en Python 3](https://youtu.be/3ZmvYwf5C6k)

## Les listes

En Python, les listes sont des structures de données très utiles pour stocker des collections d'éléments. Une liste est une collection ordonnée d'éléments, où chaque élément peut être de n'importe quel type de données : nombres, chaînes de caractères, objets, etc.

La liste suivante sera acceptée en Python (mais pas forcément dans un autre langage) :

```python
data = [10, 2.0, 'mon texte', 1]
```

Il s'agit d'une suite d'éléments de type différent. On utilise les crochets *[...]* pour encadrer les éléments de la liste.

Les éléments d'une liste peuvent être accédés en utilisant leur index, qui commence à zéro pour le premier élément de la liste. 

```python
data = [10, 2.0, 'mon texte', 1]
print(data[0]) # on récupère le premier élément
print(data[2]) # on récupère l'élément à la position 2
```

Un fait intéressant est que je peux utiliser des valeurs négatives d'index pour récupérer les éléments en partant plutôt de la fin de la liste. Le dernier élément de la liste est à la position -1, l'avant-dernier à la position -2, etc.

```python
data = [10, 2.0, 'mon texte', 1]
print(data[-1]) # on récupère le dernier élément
print(data[-2]) # on récupère l'avant-dernier élément
```

Les listes en Python sont également modifiables. Cela signifie que nous pouvons ajouter, supprimer ou modifier des éléments dans une liste.

```python
data = [10, 2.0, 'mon texte', 1]
data.append(10) # on ajoute la valeur 10 à la fin de la liste
data[2] = 'coucou' # on change le contenu de l'élément (index 2)
del data[1] # on efface l'élément à la position 1
```

Les listes en Python peuvent également être triées en utilisant la méthode `sort()`. Cette méthode trie les éléments de la liste dans l'ordre croissant par défaut. 

```python
data = [10, 2.0, 'mon texte', 1]
data.sort() # on trie en ordre croissant
data.sort(reverse=True) # on trie en ordre décroissant
```

Il existe d'autres structures identiques aux listes comme `set`, `tuple` et `dict`. Chaque type de structure a ses propres avantages.

Vidéo utile : [Explication sur les listes en Python 3](https://youtu.be/aZOiT_RN8Lo)

## Les boucles

En programmation, les boucles sont des structures de contrôle qui permettent de répéter une série d'instructions un certain nombre de fois ou jusqu'à ce qu'une condition donnée soit remplie. En Python, il existe deux types de boucles: la boucle *for* et la boucle *while*.

La boucle *for* est utilisée pour répéter un bloc d'instructions un nombre fixe de fois. La syntaxe de base de la boucle for en Python est la suivante:

```python
for element in iterable:
    # instructions à répéter
```

Dans cette syntaxe, `iterable` est une collection d'éléments, comme une liste, un tuple ou un dictionnaire, et `element` est une variable qui prend la valeur de chaque élément de l'itérable à chaque itération. Les instructions à répéter sont indentées sous la ligne de la boucle for. Par exemple, pour imprimer tous les éléments d'une liste, nous pouvons utiliser la boucle for comme suit :

```python
my_list = [1, 2, 3, 4, 5]
for num in my_list:
    print(num)
```

La boucle while est utilisée pour répéter un bloc d'instructions jusqu'à ce qu'une condition donnée soit remplie. La syntaxe de base de la boucle while en Python est la suivante :

```python
while condition:
    # instructions à répéter
```

Dans cette syntaxe, `condition` est une expression booléenne qui est évaluée à True ou False, et les instructions à répéter sont indentées sous la ligne de la boucle while. Par exemple, pour imprimer tous les nombres pairs de 0 à 10, nous pouvons utiliser la boucle while comme suit :

```python
num = 0
while num <= 10:
    print(num)
    num += 2
```

Dans cet exemple, la condition de la boucle while est `num <= 10`, ce qui signifie que les instructions seront répétées tant que `num` est inférieur ou égal à 10. À chaque itération, la valeur de `num` est augmentée de 2, de sorte que seuls les nombres pairs sont imprimés.

Vidéo utile : [Explication sur les boucles en Python 3](https://youtu.be/SGpVlxQ5Vp0)

## Les conditions

En programmation, les conditions sont des structures de contrôle qui permettent de tester si une expression est vraie ou fausse, et d'exécuter un bloc d'instructions en fonction du résultat du test. En Python, les conditions sont mises en œuvre à l'aide de la structure de contrôle `if`.

La syntaxe de base de la structure `if` en Python est la suivante :

```python
if condition:
    # instructions à exécuter si la condition est vraie
```

Dans cette syntaxe, `condition` est une expression booléenne qui est évaluée à True ou False, et les instructions à exécuter sont indentées sous la ligne de la structure `if`. Par exemple, pour imprimer "Tu es majeur" si l'âge de la personne est supérieur à 18 ans, nous pouvons utiliser la structure `if` comme suit :

```python
age = 20

if age < 18:
    print("Tu es majeur")
```

En Python, il est également possible d'utiliser la structure `if` en conjonction avec les structures `elif` (qui signifie "else if" en anglais) et `else` pour tester plusieurs conditions. La syntaxe de base de ces structures est la suivante :

```python
if condition_1:
    # instructions à exécuter si condition_1 est vraie
elif condition_2:
    # instructions à exécuter si condition_1 est fausse 
    # et condition_2 est vraie
else:
    # instructions à exécuter si toutes les conditions 
    # précédentes sont fausses
```

Dans cette syntaxe, `condition_1`, `condition_2`, etc. sont des expressions booléennes, et les instructions à exécuter sont indentées sous la ligne de la structure correspondante. Par exemple, pour imprimer "Bonjour" si l'heure est inférieure à 12h00, "Bon après-midi" si l'heure est comprise entre 12h00 et 18h00, et "Bonsoir" si l'heure est supérieure à 18h00, nous pouvons utiliser la structure `if`, `elif` et `else` comme suit :

```python
heure_actuelle = 8

if heure_actuelle < 12:
    print("Bonjour")
elif heure_actuelle < 18:
    print("Bon après-midi")
else:
    print("Bonsoir")
```

Vidéo utile : [Explications sur les conditions dans Python 3](https://youtu.be/CmMQU5wOLZs)

## Les fonctions

En programmation, une fonction est une portion de code qui effectue une tâche spécifique. En Python, les fonctions sont définies à l'aide du mot-clé `def`, suivi du nom de la fonction et de ses paramètres entre parenthèses. Les instructions à exécuter sont ensuite indentées sous la ligne de la définition de la fonction.

La syntaxe de base de la définition d'une fonction en Python est la suivante :

```python
def nom_de_la_fonction(paramètre_1, paramètre_2, ...):
    # instructions à exécuter
```

Dans cette syntaxe, `nom_de_la_fonction` est le nom de la fonction, `paramètre_1`, `paramètre_2`, etc. sont les paramètres que la fonction prend en entrée, et les instructions à exécuter sont indentées sous la ligne de la définition de la fonction.

Par exemple, voici une fonction simple qui calcule la somme de deux nombres :

```python
def calculer_somme(a, b):
    somme = a + b
    return somme
```

Dans cette fonction, `calculer_somme` est le nom de la fonction, `a` et `b` sont les paramètres que la fonction prend en entrée, et les instructions à exécuter consistent à calculer la somme des deux paramètres et à la retourner à l'aide du mot-clé `return`.

Pour utiliser une fonction, il suffit d'appeler son nom suivi des valeurs des paramètres entre parenthèses. Par exemple, pour calculer la somme de 2 et 3, nous pouvons utiliser la fonction `calculer_somme` comme suit :

```python
resultat = calculer_somme(2, 3)
print(resultat) # affiche 5
```

En Python, il est également possible de définir des fonctions qui prennent un nombre variable de paramètres en utilisant l'astérisque (*). Par exemple, voici une fonction qui calcule la somme de n nombres :

```python
def calculer_somme(*nombres):
    somme = 0
    for nombre in nombres:
        somme += nombre
    return somme
```

Dans cette fonction, le paramètre `*nombres` signifie que la fonction peut prendre un nombre variable de paramètres. Les instructions à exécuter consistent à initialiser une variable `somme` à 0, à parcourir tous les paramètres de la fonction à l'aide d'une boucle `for`, et à ajouter chaque nombre à la variable `somme`. Enfin, la somme est retournée à l'aide du mot-clé `return`.

Pour utiliser cette fonction, nous pouvons passer un nombre variable d'arguments :

```python
resultat = calculer_somme(2, 3, 4)
print(resultat) # affiche 9
```

> Conseil : utilisez des noms explicites pour les fonctions, ceci facilitera la compréhension du code. Si je nomme une fonction `ManhattanDistance`, on comprend assez bien que la fonction va calculer la distance de Manhattan.

Vidéo utile : [Explications sur les fonctions en Python 3](https://youtu.be/oIivn-RdZpE)