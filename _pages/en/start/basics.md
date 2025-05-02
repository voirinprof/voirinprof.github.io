---
title: "Python Basics"
permalink: /en/python-basics/
lang: en
date: 2025-04-24T03:02:20+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: start
---

## Basic Rules

When tackling a (more or less complex) computer science problem, there are several steps to follow before you start coding:

* Carefully read the problem statement or description. **To write good code, you first need to understand the problem**. Remember, the machine won’t do this for you.
* Clearly identify the **input parameters**: what data needs to be read? what values should be entered? etc.
* Understand what the program should produce (**the output**): what information should be generated? what kind of results does the user expect?
* Identify the **steps to get from inputs to outputs**: be able to explain the sequence of operations simply. This is often called the algorithm. Like a recipe: you start with the ingredients (inputs) and end with the cake (outputs).
* If the **problem is complex**, **break it down into smaller problems**. This is the only way to approach the development of complex systems. Each smaller problem often has a relatively simple (and well-known) solution.
* Once the recipe is clear, you can implement it — this is the coding stage.
* Python is a fairly permissive language and will often return a result even if the code isn't perfect. You must ask yourself: is this result correct (even if there are no errors)? The only way to ensure correctness is to **plan for test cases**. This is essential for verifying the code works properly.

Using simple examples, we demonstrate the approach a good developer should follow. The key idea is to ask the right questions before starting to code.

## Variables

In Python, variables are memory locations that can store values. They are used to hold temporary or permanent data in a program. Variables are often linked to the program’s inputs and outputs.

A variable name in Python must start with a letter, followed by letters, digits, or underscores (_). Python is also case-sensitive, so variable names like `data` and `Data` are considered different.

To assign a value to a variable, you use the equal sign (=). If you want to declare a variable `x` with the value 10, write:

```python
x = 10
```

Python variables can hold different types, such as integers (`int`), floating-point numbers (`float`), strings (`str`), or booleans (`bool`). Python uses dynamic typing, which means the type of a variable can change during execution.

You can declare `x` as an integer like this:

```python
x = 10
```

Then change it to a float simply by assigning a new value:

```python
x = 10.0
```

> Tip: the variable type affects how some operations behave, so it’s important to check what a variable contains.

To use a variable in your program, just refer to its name. For example, to increment `x` by 1:

```python
x = x + 1
```

> Tip: use explicit names for variables — it makes your code easier to understand. For example, `nbStudent` clearly refers to the number of students.

Useful video: [Explanation of variables in Python 3](https://youtu.be/MTZBtMsQ58E)

## Operators

In Python, operators are special symbols that perform operations on values or variables. Python has several types of operators, including arithmetic, comparison, and logical operators.

Arithmetic operators perform mathematical operations. Common ones are: addition (+), subtraction (-), multiplication (*), division (/), integer division (//), modulo (%), and exponentiation (**).

To compute the remainder of a division (e.g., 10 divided by 2), use modulo. In this case, `left` will be 0:

```python
x = 10
left = 10 % 2
```

Comparison operators compare values or variables. Common ones include: equality (==), inequality (!=), less than (<), greater than (>), less than or equal (<=), and greater than or equal (>=).

For example, to compare two variables:

```python
x = 10
y = 20
x == y
```

This returns `False` because `x` and `y` have different values.

Useful video: [Explanation of operators in Python 3](https://youtu.be/3ZmvYwf5C6k)

## Lists

In Python, lists are versatile data structures used to store collections of elements. A list is an ordered collection where each element can be of any data type: numbers, strings, objects, etc.

This is a valid list in Python (though it might not be in other languages):

```python
data = [10, 2.0, 'my text', 1]
```

You use square brackets *[...]* to define a list.

List elements can be accessed by their index (starting at zero):

```python
data = [10, 2.0, 'my text', 1]
print(data[0]) # first element
print(data[2]) # element at index 2
```

You can also use negative indices to access elements from the end of the list:

```python
data = [10, 2.0, 'my text', 1]
print(data[-1]) # last element
print(data[-2]) # second-to-last element
```

Lists are mutable — you can add, remove, or modify elements:

```python
data = [10, 2.0, 'my text', 1]
data.append(10) # add value to the end
data[2] = 'hello' # change the value at index 2
del data[1] # remove the element at index 1
```

You can sort a list using the `sort()` method:

```python
data = [10, 2.0, 1]
data.sort() # ascending order
data.sort(reverse=True) # descending order
```

Other similar data structures include `set`, `tuple`, and `dict`. Each has its own use cases and benefits.

Useful video: [Explanation of lists in Python 3](https://youtu.be/aZOiT_RN8Lo)

## Loops

Loops are control structures that allow you to repeat a block of code a certain number of times or until a condition is met. Python supports two main loop types: *for* and *while* loops.

The *for* loop is used to repeat code a fixed number of times:

```python
for element in iterable:
    # code to repeat
```

Here, `iterable` is a collection (like a list or tuple), and `element` takes the value of each item during each iteration. For example:

```python
my_list = [1, 2, 3, 4, 5]
for num in my_list:
    print(num)
```

The *while* loop repeats code as long as a condition is true:

```python
while condition:
    # code to repeat
```

To print all even numbers from 0 to 10:

```python
num = 0
while num <= 10:
    print(num)
    num += 2
```

Useful video: [Explanation of loops in Python 3](https://youtu.be/SGpVlxQ5Vp0)

## Conditions

Conditional statements let you execute code based on whether an expression is true or false. In Python, we use the `if` structure.

Basic syntax:

```python
if condition:
    # code to execute if condition is true
```

Example: print "You are an adult" if age is over 18:

```python
age = 20

if age > 18:
    print("You are an adult")
```

You can combine `if`, `elif` (else if), and `else` to handle multiple conditions:

```python
if condition_1:
    # code for condition_1
elif condition_2:
    # code if condition_1 is false and condition_2 is true
else:
    # code if all previous conditions are false
```

Example:

```python
current_hour = 8

if current_hour < 12:
    print("Good morning")
elif current_hour < 18:
    print("Good afternoon")
else:
    print("Good evening")
```

Useful video: [Explanation of conditions in Python 3](https://youtu.be/CmMQU5wOLZs)

## Functions

A function is a block of code designed to perform a specific task. In Python, functions are defined using the `def` keyword, followed by the function name and parameters in parentheses.

Basic syntax:

```python
def function_name(parameter_1, parameter_2, ...):
    # code to execute
```

Example: a function that adds two numbers:

```python
def calculate_sum(a, b):
    total = a + b
    return total
```

To use a function, call its name with arguments:

```python
result = calculate_sum(2, 3)
print(result) # outputs 5
```

You can also define functions that accept a variable number of arguments using an asterisk (*). Example:

```python
def calculate_sum(*numbers):
    total = 0
    for number in numbers:
        total += number
    return total
```

To call this function:

```python
result = calculate_sum(2, 3, 4)
print(result) # outputs 9
```

> Tip: use meaningful function names. For example, a function named `ManhattanDistance` clearly indicates that it calculates the Manhattan distance.

Useful video: [Explanation of functions in Python 3](https://youtu.be/oIivn-RdZpE)