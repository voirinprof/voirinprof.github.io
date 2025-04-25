---
title: "Python Basics"
permalink: /en/python-basics/
lang: en
date: 2023-04-24T03:02:20+00:00
toc: true
toc_label: "On this page"
---

## Basic Rules

When tackling a (more or less complex) computing problem, there are several steps to follow before starting to code:

* Carefully read the problem statement or description. **To code well, you must first understand the problem**. Remember, the machine can’t do that for you.
* Clearly identify the **input parameters**: what data needs to be read? what values need to be entered? ...
* Clearly understand what the program should output (**the results**): what information should be produced? what kind of output does the user expect? ...
* Identify the **steps to transform inputs into outputs**: you should be able to simply explain the sequence of operations needed to get the desired result. This part is often referred to as the algorithm. It's like a recipe: you start with the ingredients (inputs) and end up with the cake (outputs).
* If a **problem is complex**, **break it down into smaller problems**. This is the only way to tackle the development of complex systems. Each small problem usually has a simple (often known) solution.
* Once you have the recipe, you can implement it — that’s the coding part.
* Python is quite a permissive language. The interpreter may still return a result even if something is wrong. Ask yourself: is it correct (even if the code has no syntax errors)? You need to **design test cases** to make sure your results are accurate. This is the only way to ensure your code is correct.

Using simple examples, we illustrate the thought process a good developer should adopt. The idea is to ask the right questions before starting to code.

## Variables

In Python, variables are memory locations that can hold values. They are used to store temporary or permanent data to be used in a program. Variables are often linked to the inputs and outputs of a program.

A variable name in Python must start with a letter, followed by letters, digits, or underscores (_). Note that Python is case-sensitive, meaning the variable names "data" and "Data" are considered different.

To assign a value to a variable, use the equals sign (=) in Python. To declare a variable `x` with the value 10:

```python
x = 10
```

Variables in Python can be of different types, such as integers (int), floating-point numbers (float), strings (str), or booleans (bool). Python is also dynamically typed, which means the type of a variable can change during the program’s execution.

You can declare `x` as an integer like this:

```python
x = 10
```

Then change its type during the program by assigning a new value. `x` can become a float like this:

```python
x = 10.0
```

> Tip: The type of a variable can affect certain operations, so it's important to check what the variable holds.

A variable can be used in a program by referencing its name. For example, to add 1 to `x`:

```python
x = x + 1
```

> Tip: Use descriptive variable names to make your code easier to understand. For example, if you use the variable `nbStudent`, it’s quite clear that it refers to the number of students.

Useful video: [Explanation about Python 3 variables](https://youtu.be/MTZBtMsQ58E)

## Operators

In Python, operators are special symbols used to perform operations on values or variables. There are different types of operators, such as arithmetic, comparison, logical operators, etc.

Arithmetic operators allow you to perform math operations. Common ones include addition (+), subtraction (-), multiplication (*), division (/), integer division (//), modulo (%), and exponentiation (**).

To calculate the remainder of a division (e.g., 10 divided by 2), use modulo. Here, the variable `left` will be 0:

```python
x = 10
left = 10 % 2
```

Comparison operators let you compare values or variables. Common ones include equality (==), inequality (!=), less than (<), greater than (>), less than or equal (<=), and greater than or equal (>=).

To compare two variables:

```python
x = 10
y = 20
x == y
```

The result of `x == y` will be `False`, since the values are different.

Useful video: [Explanation of Python 3 operators](https://youtu.be/3ZmvYwf5C6k)

## Lists

In Python, lists are very useful data structures for storing collections of items. A list is an ordered collection where each element can be of any data type: numbers, strings, objects, etc.

This list is valid in Python (though not necessarily in other languages):

```python
data = [10, 2.0, 'my text', 1]
```

It’s a sequence of elements of different types. Lists are enclosed in square brackets *[...]*.

Elements in a list can be accessed using their index, starting from zero:

```python
data = [10, 2.0, 'my text', 1]
print(data[0])  # retrieves the first element
print(data[2])  # retrieves the element at index 2
```

You can also use negative indexing to access elements from the end of the list:

```python
data = [10, 2.0, 'my text', 1]
print(data[-1])  # retrieves the last element
print(data[-2])  # retrieves the second-to-last element
```

Python lists are mutable, meaning you can add, remove, or change elements:

```python
data = [10, 2.0, 'my text', 1]
data.append(10)         # adds 10 at the end
data[2] = 'hello'       # changes the element at index 2
del data[1]             # deletes the element at index 1
```

Lists can also be sorted using the `sort()` method:

```python
data = [10, 2.0, 'my text', 1]
data.sort()             # ascending order
data.sort(reverse=True) # descending order
```

Other similar structures include `set`, `tuple`, and `dict`. Each has its own benefits.

Useful video: [Explanation about Python 3 lists](https://youtu.be/aZOiT_RN8Lo)

## Loops

In programming, loops are control structures that allow repeating a set of instructions a certain number of times or until a condition is met. In Python, there are two main types of loops: *for* and *while*.

The *for* loop is used to repeat a block of instructions a fixed number of times. Its basic syntax:

```python
for element in iterable:
    # instructions to repeat
```

Here, `iterable` is a collection (like a list, tuple, or dictionary), and `element` takes the value of each item in the iterable at each iteration.

Example — printing all elements of a list:

```python
my_list = [1, 2, 3, 4, 5]
for num in my_list:
    print(num)
```

The *while* loop repeats instructions as long as a condition is true:

```python
while condition:
    # instructions to repeat
```

Example — printing even numbers from 0 to 10:

```python
num = 0
while num <= 10:
    print(num)
    num += 2
```

In this case, `num <= 10` is the condition, and the loop continues as long as it’s true.

Useful video: [Explanation about Python 3 loops](https://youtu.be/SGpVlxQ5Vp0)

## Conditions

Conditions are control structures that test whether an expression is true or false, and execute code accordingly. In Python, we use `if` statements.

Basic syntax:

```python
if condition:
    # instructions if the condition is true
```

Example — printing a message if age is over 18:

```python
age = 20

if age < 18:
    print("You are an adult")
```

You can also use `elif` (else if) and `else` to test multiple conditions:

```python
if condition_1:
    # if condition_1 is true
elif condition_2:
    # if condition_1 is false and condition_2 is true
else:
    # if all above conditions are false
```

Example — printing a greeting based on the time:

```python
current_hour = 8

if current_hour < 12:
    print("Good morning")
elif current_hour < 18:
    print("Good afternoon")
else:
    print("Good evening")
```

Useful video: [Explanation about Python 3 conditions](https://youtu.be/CmMQU5wOLZs)

## Functions

A function is a block of code that performs a specific task. In Python, functions are defined using the `def` keyword, followed by the function name and its parameters in parentheses.

Basic syntax:

```python
def function_name(param_1, param_2, ...):
    # instructions to execute
```

Example — function to compute the sum of two numbers:

```python
def calculate_sum(a, b):
    total = a + b
    return total
```

To use a function, call it with the desired arguments:

```python
result = calculate_sum(2, 3)
print(result) # prints 5
```

To define functions with a variable number of parameters, use an asterisk (*):

```python
def calculate_sum(*numbers):
    total = 0
    for number in numbers:
        total += number
    return total
```

Then call it with any number of arguments:

```python
result = calculate_sum(2, 3, 4)
print(result) # prints 9
```

> Tip: Use meaningful names for functions to make your code easier to understand. For example, naming a function `ManhattanDistance` clearly indicates what it does.

Useful video: [Explanation about Python 3 functions](https://youtu.be/oIivn-RdZpE)