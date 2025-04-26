---
title: "JavaScript Basics"
permalink: /en/js-basics/
lang: en
date: 2023-04-24T03:02:20+00:00
toc: true
toc_label: "On this page"
typenav: start
---

## Basic Rules

To illustrate some concepts, we use the Phaser library, which allows for online video game development. However, these principles apply regardless of the platform, as long as the programming language is JavaScript.

To test Phaser, simply create an HTML file (`index.html`) with the following content:

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

Then, create a `main.js` file with the following content:

```javascript
// game configuration
var config = {
    type: Phaser.AUTO,
    width: 800,
    height: 600,
    backgroundColor: '#000000',
    parent: 'game',

    // physics
    physics: {
        default: 'arcade',
        arcade: {
            gravity: { y: 300 },
            debug: false
        }
    },

    // scenes
    scene: {
        preload: preload,   // load assets
        create: create,     // create objects
        update: update      // game loop
    }
};

// game object
var game = new Phaser.Game(config);

// define preload function
function preload (){        
}

// define create function
function create (){    
}

// define update function
function update (){    
}
```

If you open the `index.html` file in a browser, you will see a black square. The code works.

Helpful video: [Getting Started with Phaser (with VSCode and Copilot)](https://youtu.be/qu6V4Meun-g)

## Variables

In JavaScript, variables are containers used to store values. They are used to store data and assign a name to it that can be used to access and manipulate it throughout the program. To declare a variable in JavaScript, you use the keywords `var`, `let`, or `const`, followed by the variable name.

Here's how to declare variables in JavaScript:

Using `var` (before ES6):

```javascript
var age = 30;
var name = "John";
```

Using `let` (after ES6):

```javascript
let age = 30;
let name = "John";
```

Using `const` (after ES6):

```javascript
const age = 30;
const name = "John";
```

In the examples above, we declared two variables: `age` and `name`. `age` stores an integer (30), while `name` stores a string ("John"). Note that when you use `const`, the value cannot be changed once assigned. With `let`, the value can be changed, and with `var`, which was used before `let` and `const`, variable scope behaves differently in certain situations.

You can also declare variables without assigning them a value immediately:

```javascript
let temperature; // Declare a variable with no value
temperature = 25; // Assign a value later
```

Variables can store any type of data: numbers, strings, arrays, objects, etc. For example:

```javascript
let num = 42; // Integer
let message = "Hello!"; // String
let colors = ["red", "green", "blue"]; // Array
let person = { name: "Alice", age: 25 }; // Object
```

> Tip: The variable type can affect certain operations, so always make sure you know what your variable contains.

You can use a variable by referencing its name. For example, to increment `x` by 1:

```javascript
x = x + 1;
```

> Tip: Use explicit and meaningful names for your variables to make your code easier to understand. For example, `nbStudent` clearly represents the number of students.

Helpful video: [Understanding Variables with Phaser (in JavaScript)](https://youtu.be/qvSB0SofCqQ)

## Loops

In JavaScript, loops are control structures that allow you to execute a block of code repeatedly as long as a specified condition is true. This allows you to repeat tasks without duplicating code. The three main types of loops in JavaScript are: `for`, `while`, and `do-while`.

1. **For loop:**
   Used when you know in advance how many times the loop should run.

   Syntax:
   ```javascript
   for (initialization; condition; increment) {
     // Code block to be executed
   }
   ```

   Example:
   ```javascript
   for (let i = 0; i < 5; i++) {
     console.log("Iteration: " + i);
   }
   ```

2. **While loop:**
   Used when you don’t know how many times you need to loop and want to continue as long as a condition is true.

   Syntax:
   ```javascript
   while (condition) {
     // Code block to be executed
   }
   ```

   Example:
   ```javascript
   let count = 0;
   while (count < 5) {
     console.log("Iteration: " + count);
     count++;
   }
   ```

3. **Do-while loop:**
   Similar to `while`, but guarantees that the block of code is executed at least once.

   Syntax:
   ```javascript
   do {
     // Code block to be executed
   } while (condition);
   ```

   Example:
   ```javascript
   let num = 5;
   do {
     console.log("Number: " + num);
     num--;
   } while (num > 0);
   ```

Make sure your loops have an exit condition to avoid infinite loops, which can cause performance issues or crash your program.

Helpful video: [Looping Concepts in JavaScript (with Phaser)](https://youtu.be/84MRHlxp4ec)

## Conditions

In JavaScript, conditional statements are used to execute different blocks of code depending on whether a Boolean expression evaluates to `true` or `false`. The most common conditional structures are `if`, `else if`, and `else`.

1. **if**:
   Executes a block of code if a condition is true.

   Syntax:
   ```javascript
   if (condition) {
     // Code block
   }
   ```

   Example:
   ```javascript
   let age = 18;
   if (age >= 18) {
     console.log("You are an adult.");
   }
   ```

2. **else if**:
   Tests another condition if the previous one was false.

   Syntax:
   ```javascript
   if (condition1) {
     // Code block
   } else if (condition2) {
     // Another code block
   }
   ```

   Example:
   ```javascript
   let score = 85;
   if (score >= 90) {
     console.log("Excellent!");
   } else if (score >= 80) {
     console.log("Good!");
   } else {
     console.log("Needs improvement!");
   }
   ```

3. **else**:
   Executes a block if all previous conditions were false.

   Syntax:
   ```javascript
   if (condition) {
     // Code block
   } else {
     // Fallback code
   }
   ```

   Example:
   ```javascript
   let weather = "rain";
   if (weather === "sunny") {
     console.log("Go outside and enjoy the sun!");
   } else {
     console.log("Stay indoors, it's raining.");
   }
   ```

You can also use logical operators like `&&` (and), `||` (or), and `!` (not) to create more complex conditions.

Helpful video: [Understanding Conditions in JavaScript (with Phaser)](https://youtu.be/6EhQVrhZiHs)

## Functions

Functions in JavaScript are reusable blocks of code designed to perform a specific task. They allow you to encapsulate logic and reuse it throughout your program. Functions can take input parameters and return a value (optional).

Here’s how to declare and use functions in JavaScript:

1. **Function declaration:**

   Syntax:
   ```javascript
   function functionName(param1, param2, ...) {
     // Code block
   }
   ```

   Example:
   ```javascript
   function sayHello() {
     console.log("Hello!");
   }
   ```

2. **Calling a function:**

   Example:
   ```javascript
   sayHello(); // Calls the sayHello function
   ```

3. **Function with parameters:**

   Example:
   ```javascript
   function greetByName(name) {
     console.log("Hello, " + name + "!");
   }
   ```

4. **Function with a return value:**

   Example:
   ```javascript
   function add(a, b) {
     return a + b;
   }

   let result = add(3, 5); // result will be 8
   ```

5. **Arrow functions (anonymous functions):**

   Since ECMAScript 6 (ES6), you can use arrow functions for a shorter syntax.

   Example:
   ```javascript
   const multiply = (x, y) => x * y;
   console.log(multiply(2, 3)); // Outputs 6
   ```

Functions are fundamental in JavaScript, and they play a key role in modular programming and code reuse. You can use them for simple math or more complex tasks.

> Tip: Use descriptive names for functions to make your code easier to understand. For example, naming a function `ManhattanDistance` makes it clear that it calculates the Manhattan distance.

Helpful video: [Understanding Functions in JavaScript (with Phaser)](https://youtu.be/ElWWN8U43lU)