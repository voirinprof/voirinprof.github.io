---
title: "Notions de base en JavaScript"
permalink: /js-basics/
date: 2023-04-24T03:02:20+00:00
toc: true
toc_label: "Dans cette page"
---

## Règles de base

Pour illustrer certains concepts, on utilise la librairie Phaser qui permet de faire des jeux vidéos en ligne. Toutefois ces principes s'appliquent quelque soit la plateforme, l'essentiel est que le langage de programmation est en Javascript.

Pour tester Phaser, il suffit de créer un fichier HTML (index.html) avec le contenu suivant :

```HTML
<!DOCTYPE html>
<html>
<head>
    <script src="//cdn.jsdelivr.net/npm/phaser@3.55.2/dist/phaser.min.js"></script>
    <script src="main.js"></script>
</head>
<body>
</body>
</html>
```

Ensuite il faut créer un fichier `main.js` avec le contenu suivant :

```javascript
// configuration du jeu
var config = {
    type: Phaser.AUTO,
    width: 800,
    height: 600,
    backgroundColor: '#000000',
    parent: 'game',
    
    // physique
    physics: {
        default: 'arcade',
        arcade: {
            gravity: { y: 300 },
            debug: false
        }
    },

    // scènes
    scene: {

        // chargement des assets
        preload: preload,

        // création des objets
        create: create,

        // boucle de jeu
        update: update
    }
};

// objet jeu
var game = new Phaser.Game(config);

// créer la fonction preload
function preload (){        
}

// créer la fonction create
function create (){    

}

// créer la fonction update
function update (){    
}
```

Si vous consultez le fichier index.html dans un navigateur, vous allez avoir un carré noir. Le code fonctionne.

Vidéo utile : [Débuter un projet avec Phaser (avec VSCode et Copilot)](https://youtu.be/qu6V4Meun-g)

## Les variables

En JavaScript, les variables sont des conteneurs qui permettent de stocker des valeurs. Elles sont utilisées pour stocker des données et leur donner un nom qui peut être utilisé pour y accéder et les manipuler tout au long du programme. Pour déclarer une variable en JavaScript, on utilise le mot-clé "var", "let", ou "const", suivi du nom de la variable.

Voici comment déclarer des variables en JavaScript :

Utilisation de "var" (avant ES6) :

```javascript
var age = 30;
var name = "John";
```

Utilisation de "let" (après ES6) :

```javascript
let age = 30;
let name = "John";
```

Utilisation de "const" (après ES6) :

```javascript
const age = 30;
const name = "John";
```

Dans les exemples ci-dessus, nous avons déclaré deux variables : "age" et "name". "age" stocke un nombre entier (30), tandis que "name" stocke une chaîne de caractères ("John"). Notez que lorsque vous utilisez "const", la valeur de la variable ne peut pas être modifiée une fois qu'elle est attribuée. Avec "let", la valeur peut être modifiée, tandis qu'avec "var" (utilisé avant l'introduction de "let" et "const"), la portée de la variable peut être différente dans certaines situations.

Il est également possible de déclarer des variables sans leur attribuer une valeur immédiate, et de leur assigner une valeur plus tard :

```javascript
let temperature; // Déclaration d'une variable sans valeur
temperature = 25; // Assignation d'une valeur à la variable ultérieurement
```

Vous pouvez utiliser des variables pour stocker des valeurs de n'importe quel type de données, comme des nombres, des chaînes de caractères, des tableaux, des objets, etc. Par exemple :

```javascript
let num = 42; // Nombre entier
let message = "Bonjour !"; // Chaîne de caractères
let colors = ["rouge", "vert", "bleu"]; // Tableau
let person = { name: "Alice", age: 25 }; // Objet
```


> Conseil : le type de la variable peut affecter certaines opérations, donc il est important de bien vérifier ce que la variable contient.

Une variable pourra être utilisée dans le programme en utilisant son nom, par exemple si je veux ajouter la valeur 1 à `x`, il suffit de faire ceci :

```javascript
x = x + 1
```

> Conseil : utilisez des noms explicites pour les variables, ainsi il sera plus aisé de comprendre le programme. Ex: si j'utilise la variable `nbStudent`, on comprend assez bien qu'il s'agit du nombre des étudiants.

Vidéo utile : [Le concept des variables avec Phaser (en javascript)](https://youtu.be/qvSB0SofCqQ)

## Les boucles

En JavaScript, les boucles sont des structures de contrôle qui permettent d'exécuter un bloc de code de manière répétée tant qu'une condition donnée est vraie. Cela permet de répéter une tâche plusieurs fois sans avoir à écrire le même code à plusieurs reprises. Il existe principalement trois types de boucles en JavaScript : la boucle "for", la boucle "while" et la boucle "do-while".

1. **Boucle "for" :**
   La boucle "for" est utilisée lorsque vous connaissez à l'avance le nombre d'itérations que vous souhaitez effectuer.

   Syntaxe :
   ```javascript
   for (initialisation; condition; incrémentation) {
     // Bloc de code à exécuter à chaque itération
   }
   ```

   Exemple :
   ```javascript
   for (let i = 0; i < 5; i++) {
     console.log("Itération : " + i);
   }
   ```

2. **Boucle "while" :**
   La boucle "while" est utilisée lorsque vous ne connaissez pas à l'avance le nombre d'itérations et que la boucle doit s'exécuter tant qu'une condition est vraie.

   Syntaxe :
   ```javascript
   while (condition) {
     // Bloc de code à exécuter tant que la condition est vraie
   }
   ```

   Exemple :
   ```javascript
   let count = 0;
   while (count < 5) {
     console.log("Itération : " + count);
     count++;
   }
   ```

3. **Boucle "do-while" :**
   La boucle "do-while" est similaire à la boucle "while", mais elle garantit l'exécution du bloc de code au moins une fois, même si la condition est fausse dès le départ.

   Syntaxe :
   ```javascript
   do {
     // Bloc de code à exécuter
   } while (condition);
   ```

   Exemple :
   ```javascript
   let num = 5;
   do {
     console.log("Numéro : " + num);
     num--;
   } while (num > 0);
   ```

Dans les boucles, assurez-vous d'inclure une condition de sortie pour éviter les boucles infinies. Par exemple, en utilisant une boucle "for", assurez-vous que la condition sera évaluée à "false" à un moment donné pour permettre à la boucle de se terminer.

Les boucles sont un outil puissant pour automatiser des tâches répétitives et itérer sur des collections de données, comme des tableaux, pour effectuer des opérations sur chaque élément. Cependant, soyez prudent avec les boucles infinies, car elles peuvent entraîner des problèmes de performances et bloquer l'exécution du code.

Vidéo utile : [Le principe des boucles en Javascript (avec Phaser)](https://youtu.be/84MRHlxp4ec)

## Les conditions

En JavaScript, les conditions sont des structures de contrôle qui permettent d'exécuter différents blocs de code en fonction de l'évaluation d'une expression booléenne (une expression qui est soit "true" soit "false"). Les structures de conditions les plus couramment utilisées sont les suivantes : "if", "else if" et "else".

1. **if** :
   La structure "if" permet d'exécuter un bloc de code si une condition est vraie.

   Syntaxe :
   ```javascript
   if (condition) {
     // Bloc de code à exécuter si la condition est vraie
   }
   ```

   Exemple :
   ```javascript
   let age = 18;
   if (age >= 18) {
     console.log("Vous êtes majeur.");
   }
   ```

2. **else if** :
   La structure "else if" permet de tester une condition supplémentaire si la condition précédente est fausse.

   Syntaxe :
   ```javascript
   if (condition1) {
     // Bloc de code à exécuter si condition1 est vraie
   } else if (condition2) {
     // Bloc de code à exécuter si condition2 est vraie
   }
   ```

   Exemple :
   ```javascript
   let score = 85;
   if (score >= 90) {
     console.log("Très bien !");
   } else if (score >= 80) {
     console.log("Bien !");
   } else {
     console.log("Peut mieux faire !");
   }
   ```

3. **else** :
   La structure "else" permet d'exécuter un bloc de code si toutes les conditions précédentes sont fausses.

   Syntaxe :
   ```javascript
   if (condition) {
     // Bloc de code à exécuter si la condition est vraie
   } else {
     // Bloc de code à exécuter si toutes les conditions sont fausses
   }
   ```

   Exemple :
   ```javascript
   let weather = "pluie";
   if (weather === "soleil") {
     console.log("Sortez et profitez du soleil !");
   } else {
     console.log("Restez à l'intérieur, il pleut.");
   }
   ```

Les structures de conditions sont essentielles pour contrôler le flux d'exécution d'un programme en fonction des différentes situations rencontrées. Vous pouvez également utiliser des opérateurs logiques tels que "&&" (et), "||" (ou) et "!" (non) pour créer des expressions conditionnelles plus complexes.

Vidéo utile : [Le principe des conditions en Javascript (avec Phaser)](https://youtu.be/6EhQVrhZiHs)

## Les fonctions

Les fonctions en JavaScript sont des blocs de code réutilisables qui effectuent une tâche spécifique lorsqu'elles sont appelées. Elles permettent de regrouper du code logique, de l'encapsuler dans un seul endroit et de lui donner un nom, ce qui facilite sa réutilisation et sa maintenance. Les fonctions peuvent accepter des paramètres en entrée et retourner une valeur en sortie (optionnelle).

Voici comment déclarer et utiliser une fonction en JavaScript :

1. **Déclaration d'une fonction :**
   Pour déclarer une fonction, utilisez le mot-clé "function", suivi du nom de la fonction et des parenthèses contenant les paramètres (s'ils existent).

   Syntaxe :
   ```javascript
   function nomDeLaFonction(paramètre1, paramètre2, ...) {
     // Bloc de code à exécuter
   }
   ```

   Exemple :
   ```javascript
   function direBonjour() {
     console.log("Bonjour !");
   }
   ```

2. **Appel d'une fonction :**
   Pour appeler une fonction, utilisez simplement son nom suivi de parenthèses (si la fonction n'attend pas de paramètres).

   Exemple :
   ```javascript
   direBonjour(); // Appel de la fonction direBonjour()
   ```

3. **Fonction avec des paramètres :**
   Vous pouvez définir des paramètres dans une fonction pour lui permettre de prendre des valeurs en entrée. Les paramètres sont déclarés entre les parenthèses de la déclaration de la fonction.

   Exemple :
   ```javascript
   function direBonjourAvecNom(nom) {
     console.log("Bonjour, " + nom + " !");
   }
   ```

4. **Fonction avec une valeur de retour :**
   Une fonction peut retourner une valeur en utilisant le mot-clé "return". Cette valeur peut ensuite être utilisée dans le reste du programme.

   Exemple :
   ```javascript
   function addition(a, b) {
     return a + b;
   }

   let resultat = addition(3, 5); // resultat aura la valeur 8
   ```

5. **Fonctions anonymes (fonctions fléchées) :**
   Depuis ECMAScript 6 (ES6), vous pouvez utiliser les fonctions fléchées (fonctions anonymes) pour déclarer des fonctions de manière concise.

   Exemple :
   ```javascript
   const multiplier = (x, y) => x * y;
   console.log(multiplier(2, 3)); // Affiche 6
   ```

Les fonctions sont un concept fondamental en JavaScript, et elles jouent un rôle essentiel dans la programmation modulaire et la réutilisation du code. Vous pouvez les utiliser pour accomplir différentes tâches, des opérations mathématiques simples aux traitements complexes de données.

> Conseil : utilisez des noms explicites pour les fonctions, ceci facilitera la compréhension du code. Si je nomme une fonction `ManhattanDistance`, on comprend assez bien que la fonction va calculer la distance de Manhattan.

Vidéo utile : [Le principe des fonctions en Javascript (avec Phaser)](https://youtu.be/ElWWN8U43lU)