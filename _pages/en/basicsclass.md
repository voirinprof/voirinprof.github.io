---
title: "Introduction to Python Classes"
permalink: /en/python-classes-intro/
lang: en
date: 2025-04-26T10:00:00+00:00
toc: true
toc_label: "On this page"
typenav: start
---
> **_NOTE:_** The [Python Official Documentation](https://docs.python.org/3/tutorial/classes.html) provides comprehensive guides and examples for working with classes. The [Real Python YouTube channel](https://www.youtube.com/@realpython) also offers valuable tutorials for Python developers.

## Setting Up Your Environment

To work with Python classes, ensure you have Python installed. You can use a package manager like `pip` to install additional libraries if needed for your project:

```shell
pip install your_library
```

or with `conda`:

```shell
conda install your_library
```

For this introduction, no external libraries are required, as Python’s built-in features are sufficient for defining and using classes. However, you might find these libraries useful for advanced projects involving classes:

* **dataclasses**: Simplifies class creation for data storage with minimal boilerplate (built-in since Python 3.7).
* **pydantic**: Provides data validation and settings management using Python type hints.
* **attrs**: Offers a way to define classes with less code, similar to dataclasses.
* **numpy**: Useful for classes that handle numerical data or arrays.
* **pandas**: Handy for classes that manage tabular or structured data.

Ensure your development environment (e.g., VS Code, PyCharm, or Jupyter Notebook) is set up to run Python scripts or interactive sessions.

## Defining and Using Classes

### What Are Classes?

Classes in Python are blueprints for creating objects, which are instances of a class. They allow you to bundle data (attributes) and behavior (methods) into a single entity, promoting organized and reusable code.

To define a class, use the `class` keyword followed by the class name (typically in CamelCase) and a colon. The class body is indented.

Example of a simple class representing a point in 2D space:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

- `__init__`: The constructor method, called when an instance is created.
- `self`: Refers to the instance of the class, used to access attributes and methods.
- `x` and `y`: Instance attributes, storing the coordinates of the point.

To create an instance (object) of the `Point` class:

```python
# Create a point at (3, 4)
p1 = Point(3, 4)
print(p1.x)  # Output: 3
print(p1.y)  # Output: 4
```

### Adding Methods

Methods are functions defined inside a class that operate on its instances. They take `self` as the first parameter.

Example of a class with a method to calculate the distance from the origin (0, 0):

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def distance_from_origin(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5
```

Using the method:

```python
p1 = Point(3, 4)
distance = p1.distance_from_origin()
print(distance)  # Output: 5.0
```

> Tip: Use descriptive method names to make the purpose clear, e.g., `distance_from_origin` instead of `dist`.

## Working with Class Attributes and Methods

### Class Attributes

Class attributes are shared across all instances of a class, unlike instance attributes, which are unique to each object.

Example of a class with a class attribute to track the number of points created:

```python
class Point:
    # Class attribute
    count = 0
    
    def __init__(self, x, y):
        self.x = x
        self.y = y
        Point.count += 1  # Increment count when a new point is created
```

Using the class attribute:

```python
p1 = Point(1, 2)
p2 = Point(3, 4)
print(Point.count)  # Output: 2
```

### Class Methods and Static Methods

- **Class methods**: Operate on the class itself, not instances. Defined with the `@classmethod` decorator and take `cls` as the first parameter.
- **Static methods**: Don’t operate on the instance or class but are logically related to the class. Defined with the `@staticmethod` decorator.

Example:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    @classmethod
    def from_tuple(cls, coords):
        return cls(coords[0], coords[1])
    
    @staticmethod
    def is_valid(x, y):
        return isinstance(x, (int, float)) and isinstance(y, (int, float))
```

Using class and static methods:

```python
# Create a point from a tuple using a class method
p1 = Point.from_tuple((5, 6))
print(p1.x, p1.y)  # Output: 5 6

# Check if coordinates are valid using a static method
print(Point.is_valid(3, 4))  # Output: True
print(Point.is_valid("3", 4))  # Output: False
```

## Inheritance and Polymorphism

### Inheritance

Inheritance allows a class (child) to inherit attributes and methods from another class (parent), promoting code reuse.

Example of a child class `ColoredPoint` inheriting from `Point`:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def describe(self):
        return f"Point at ({self.x}, {self.y})"

class ColoredPoint(Point):
    def __init__(self, x, y, color):
        super().__init__(x, y)  # Call parent class constructor
        self.color = color
    
    def describe(self):  # Override parent method
        return f"ColoredPoint at ({self.x}, {self.y}) with color {self.color}"
```

Using inheritance:

```python
p1 = Point(1, 2)
cp1 = ColoredPoint(3, 4, "red")
print(p1.describe())   # Output: Point at (1, 2)
print(cp1.describe())  # Output: ColoredPoint at (3, 4) with color red
```

- `super()`: Calls the parent class’s method.
- Method overriding: The child class provides its own implementation of `describe`.

### Polymorphism

Polymorphism allows different classes to be treated as instances of a common parent class, with each class implementing shared methods differently.

Example using polymorphism:

```python
points = [Point(1, 2), ColoredPoint(3, 4, "blue")]
for point in points:
    print(point.describe())  # Calls the appropriate describe method
```

Output:

```
Point at (1, 2)
ColoredPoint at (3, 4) with color blue
```

## Best Practices

- **Use meaningful names**: Class names should be nouns (e.g., `Point`, `ColoredPoint`) and method names should be verbs or descriptive (e.g., `distance_from_origin`).
- **Keep classes focused**: Each class should have a single responsibility (Single Responsibility Principle).
- **Validate inputs**: Add checks in `__init__` or methods to ensure valid data (e.g., check if coordinates are numbers).
- **Use inheritance wisely**: Avoid deep inheritance hierarchies; prefer composition when possible.
- **Document your classes**: Use docstrings to explain the purpose of classes and methods.

Example with input validation and docstring:

```python
class Point:
    """A class representing a 2D point with x and y coordinates."""
    
    def __init__(self, x, y):
        if not all(isinstance(coord, (int, float)) for coord in (x, y)):
            raise ValueError("Coordinates must be numbers")
        self.x = x
        self.y = y
```

## Useful Resources

Here are some video tutorials covering the concepts in this guide:
* [Introduction to Python Classes and Objects](https://youtu.be/ZDa-Z5JzLYM)
* [Understanding Inheritance in Python](https://youtu.be/RSl87lq4yDE)
* [Class Methods vs Static Methods in Python](https://youtu.be/rq8cL2XMM5M)
* [Python OOP Tutorial: Real Python](https://youtu.be/JeznW_7DlB0)
* [Using Dataclasses for Cleaner Python Code](https://youtu.be/CvQ7e6yUtnw)