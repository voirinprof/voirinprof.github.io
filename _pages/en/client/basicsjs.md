---
title: "Basic Concepts in JavaScript"  
permalink: /en/js-basics/  
lang: en  
date: 2023-04-24T03:02:20+00:00  
toc: true  
toc_sticky: true  
toc_label: "In this page"  
typenav: client  
---

> **_NOTE:_** You have the option to use this GitHub Codespace to practice with JavaScript and Phaser [https://github.com/voirinprof/gis_starter_phaser_geolab](https://github.com/voirinprof/gis_starter_phaser_geolab). To set up a Codespace environment, check out [Getting Started with GitHub Codespaces](/en/codespaces-intro/).

## Basic Rules

To illustrate some concepts, we use the Phaser library, which allows you to make online video games. However, these principles apply regardless of the platform, the key point is that the programming language is JavaScript.

To test Phaser, simply create an HTML file (index.html) with the following content:

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

Next, create a `main.js` file with the following content:

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

        // asset loading
        preload: preload,

        // object creation
        create: create,

        // game loop
        update: update
    }
};

// game object
var game = new Phaser.Game(config);

// create preload function
function preload (){        
}

// create create function
function create (){    

}

// create update function
function update (){    
}
```

If you view the `index.html` file in a browser, you will see a black square. The code works.

Useful video: [Starting a project with Phaser (with VSCode and Copilot)](https://youtu.be/qu6V4Meun-g)

## Variables

In JavaScript, variables are containers that allow you to store values. They are used to store data and give it a name that can be used to access and manipulate it throughout the program. To declare a variable in JavaScript, you use the keyword "var", "let", or "const", followed by the variable name.

Here’s how to declare variables in JavaScript:

Using "var" (before ES6):

```javascript
var age = 30;
var name = "John";
```

Using "let" (after ES6):

```javascript
let age = 30;
let name = "John";
```

Using "const" (after ES6):

```javascript
const age = 30;
const name = "John";
```

In the examples above, we declared two variables: "age" and "name". "age" stores an integer (30), while "name" stores a string ("John"). Note that when using "const", the value of the variable cannot be changed once assigned. With "let", the value can be modified, while with "var" (used before the introduction of "let" and "const"), the scope of the variable can be different in some situations.

It’s also possible to declare variables without assigning a value immediately and assign a value to them later:

```javascript
let temperature; // Declaration of a variable without value
temperature = 25; // Assigning a value to the variable later
```

You can use variables to store values of any data type, such as numbers, strings, arrays, objects, etc. For example:

```javascript
let num = 42; // Integer
let message = "Hello!"; // String
let colors = ["red", "green", "blue"]; // Array
let person = { name: "Alice", age: 25 }; // Object
```

> Tip: The type of the variable can affect certain operations, so it’s important to verify what the variable contains.

A variable can be used in the program by using its name. For example, if I want to add 1 to `x`, simply do:

```javascript
x = x + 1
```

> Tip: Use descriptive names for variables, as it will make it easier to understand the program. For example, if I use the variable `nbStudent`, it's quite clear that it refers to the number of students.

Useful video: [The concept of variables with Phaser (in JavaScript)](https://youtu.be/qvSB0SofCqQ)

## Loops

In JavaScript, loops are control structures that allow you to execute a block of code repeatedly as long as a given condition is true. This allows you to repeat a task multiple times without writing the same code over and over again. There are mainly three types of loops in JavaScript: the "for" loop, the "while" loop, and the "do-while" loop.

1. **For loop:**
   The "for" loop is used when you know in advance how many iterations you want to perform.

   Syntax:
   ```javascript
   for (initialization; condition; increment) {
     // Block of code to execute on each iteration
   }
   ```

   Example:
   ```javascript
   for (let i = 0; i < 5; i++) {
     console.log("Iteration: " + i);
   }
   ```

2. **While loop:**
   The "while" loop is used when you don’t know in advance how many iterations there will be and the loop should run as long as a condition is true.

   Syntax:
   ```javascript
   while (condition) {
     // Block of code to execute as long as the condition is true
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
   The "do-while" loop is similar to the "while" loop, but it guarantees that the block of code will be executed at least once, even if the condition is false from the start.

   Syntax:
   ```javascript
   do {
     // Block of code to execute
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

In loops, make sure to include an exit condition to avoid infinite loops. For example, in a "for" loop, make sure the condition eventually evaluates to "false" to allow the loop to end.

Loops are a powerful tool to automate repetitive tasks and iterate over data collections, such as arrays, to perform operations on each element. However, be cautious with infinite loops, as they can lead to performance issues and block the execution of the code.

Useful video: [The principle of loops in JavaScript (with Phaser)](https://youtu.be/84MRHlxp4ec)

## Conditions

In JavaScript, conditions are control structures that allow you to execute different blocks of code based on the evaluation of a boolean expression (an expression that is either "true" or "false"). The most commonly used conditional structures are: "if", "else if", and "else".

1. **if:**
   The "if" structure allows you to execute a block of code if a condition is true.

   Syntax:
   ```javascript
   if (condition) {
     // Block of code to execute if the condition is true
   }
   ```

   Example:
   ```javascript
   let age = 18;
   if (age >= 18) {
     console.log("You are an adult.");
   }
   ```

2. **else if:**
   The "else if" structure allows you to test another condition if the previous condition is false.

   Syntax:
   ```javascript
   if (condition1) {
     // Block of code to execute if condition1 is true
   } else if (condition2) {
     // Block of code to execute if condition2 is true
   }
   ```

   Example:
   ```javascript
   let score = 85;
   if (score >= 90) {
     console.log("Very good!");
   } else if (score >= 80) {
     console.log("Good!");
   } else {
     console.log("Could do better!");
   }
   ```

3. **else:**
   The "else" structure allows you to execute a block of code if all previous conditions are false.

   Syntax:
   ```javascript
   if (condition) {
     // Block of code to execute if the condition is true
   } else {
     // Block of code to execute if all conditions are false
   }
   ```

   Example:
   ```javascript
   let weather = "rain";
   if (weather === "sun") {
     console.log("Go out and enjoy the sun!");
   } else {
     console.log("Stay inside, it's raining.");
   }
   ```

Conditional structures are essential for controlling the flow of a program based on different situations. You can also use logical operators like "&&" (and), "||" (or), and "!" (not) to create more complex conditional expressions.

Useful video: [The principle of conditions in JavaScript (with Phaser)](https://youtu.be/6EhQVrhZiHs)

## Functions

Functions in JavaScript are reusable blocks of code that perform a specific task when called. They allow you to group code logic, encapsulate it in one place, and give it a name, making it easier to reuse and maintain. Functions can accept parameters as input and return a value as output (optional).

Here’s how to declare and use a function in JavaScript:

1. **Declaring a function:**
   To declare a function, use the "function" keyword followed by the function name and parentheses containing parameters (if any).

   Syntax:
   ```javascript
   function functionName(parameter1, parameter2, ...) {
     // Block of code to execute
   }
   ```

   Example:
   ```javascript
   function sayHello() {
     console.log("Hello!");
   }
   ```

2. **Calling a function:**


   Once a function is declared, you can call it by using its name followed by parentheses.

   Syntax:
   ```javascript
   functionName();
   ```

   Example:
   ```javascript
   sayHello(); // Output: "Hello!"
   ```

3. **Functions with parameters:**
   You can define parameters inside the parentheses. These are variables that can be used inside the function.

   Example:
   ```javascript
   function greet(name) {
     console.log("Hello, " + name + "!");
   }

   greet("John"); // Output: "Hello, John!"
   ```

4. **Functions with return value:**
   Functions can also return a value. To return a value, use the "return" keyword inside the function.

   Example:
   ```javascript
   function add(a, b) {
     return a + b;
   }

   let sum = add(5, 3); // sum = 8
   console.log(sum);
   ```

Functions are a key part of structuring your code in a modular way and enhancing code reusability. You can also create anonymous functions and assign them to variables for more flexibility.

Useful video: [The principle of functions in JavaScript (with Phaser)](https://youtu.be/84MZYPK6uwk)

## TypeScript

TypeScript is a superset of JavaScript developed by Microsoft that adds powerful features to the language, including **static typing**. This means you can specify the types of variables and functions when writing the code, which helps catch many errors **before even running** the program.

**Why use TypeScript?**

- Better readability and code maintainability
- Fewer errors due to early detection
- Autocompletion and automatic documentation in modern editors (like VS Code)
- Compatible with any JavaScript project

**Example of a difference:**

```javascript
// In JavaScript
let score = 10;
score = "ten"; // No error at first, but may cause a bug later
```

```typescript
// In TypeScript
let score: number = 10;
score = "ten"; // ❌ Error detected immediately
```

**Key features of TypeScript**

| Feature                 | JavaScript           | TypeScript                           |
|-------------------------|----------------------|--------------------------------------|
| Typing                  | Dynamic              | Static (optional but recommended)    |
| Error detection         | At runtime           | At compile time                      |
| Autocompletion          | Basic                | Advanced with custom types           |
| Usage                   | Direct in the browser| Requires compilation (`tsc`)         |
| Learning curve          | Very easy            | Slightly more technical              |
