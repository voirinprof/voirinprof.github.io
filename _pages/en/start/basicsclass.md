---
title: "Introduction to Classes in Python"
permalink: /en/python-classes-intro/
lang: en
date: 2025-04-26T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: start
---

> **_NOTE:_** The [official Python documentation](https://docs.python.org/3/tutorial/classes.html) offers detailed guides and examples on how to use classes. This [YouTube channel](https://www.youtube.com/@YvesVoirin) also provides many resources.

## Key Information

For this introduction, no external libraries are required, as Python’s built-in features are sufficient to define and use classes. However, some libraries can be useful for more advanced projects involving classes:

* **dataclasses**: Simplifies the creation of data storage classes with minimal code (built-in since Python 3.7).
* **pydantic**: Provides data validation and parameter management using type annotations.
* **attrs**: Allows class definition with less boilerplate code, similar to `dataclasses`.

## Defining and Using Classes

### What is a Class?

Classes in Python are templates for creating objects, which are instances of a class (stored in memory). They allow grouping data (attributes) and behavior (methods) into a single entity, promoting organized and reusable code.

To define a class, use the `class` keyword followed by the class name (typically in CamelCase) and a colon. The body of the class is indented.

Example of a simple class representing a point in 2D space:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

- `__init__`: The constructor, called when an instance is created.
- `self`: Refers to the instance of the class, used to access attributes and methods.
- `x` and `y`: Instance attributes that store the point's coordinates.

To create an instance (object) of the `Point` class:

```python
# Create a point at (3, 4)
p1 = Point(3, 4)
print(p1.x)  # Outputs: 3
print(p1.y)  # Outputs: 4
```

### Adding Methods

Methods are functions defined inside a class that operate on its instances. They take `self` as their first parameter.

In Python, `self` is a convention used to represent the current instance of a class in its methods. It's an implicit parameter that allows a method to access the object's attributes and other methods.

* Role of `self`

`self` refers to the specific instance of the class on which a method is called. Each object created from a class has its own data (attributes), and `self` provides access to or modifies that data. It's how Python knows which object a method refers to.

* Why is `self` necessary?

When you call a method on an object (e.g., `my_object.my_method()`), Python automatically passes a reference to the object as the first argument to the method. By convention, this argument is named `self`, enabling the method to work with that object's specific data.

Example of a class with a method to compute the distance from the origin (0, 0):

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
print(distance)  # Outputs: 5.0
```

> Tip: Use descriptive method names to clarify their purpose, e.g., `distance_from_origin` instead of `dist`.

## Working with Class Attributes and Methods

### Class Attributes

Class attributes are shared across all instances of a class, unlike instance attributes which are unique to each object.

Example of a class with a class attribute to track the number of created points:

```python
class Point:
    # Class attribute
    counter = 0
    
    def __init__(self, x, y):
        self.x = x
        self.y = y
        Point.counter += 1  # Increment the counter for each new point
```

Using the class attribute:

```python
p1 = Point(1, 2)
p2 = Point(3, 4)
print(Point.counter)  # Outputs: 2
```

### Class Methods and Static Methods

- **Class methods**: Operate on the class itself, not instances. Defined with the `@classmethod` decorator and take `cls` as the first parameter.
- **Static methods**: Do not operate on either the class or the instance, but are logically related to the class. Defined with the `@staticmethod` decorator.

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
print(p1.x, p1.y)  # Outputs: 5 6

# Check if coordinates are valid using a static method
print(Point.is_valid(3, 4))  # Outputs: True
print(Point.is_valid("3", 4))  # Outputs: False
```

## Inheritance and Polymorphism

### Inheritance

Inheritance allows one class (child) to inherit the attributes and methods of another class (parent), promoting code reuse.

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
        super().__init__(x, y)  # Call the parent class constructor
        self.color = color
    
    def describe(self):  # Override the parent method
        return f"ColoredPoint at ({self.x}, {self.y}) with color {self.color}"
```

Using inheritance:

```python
p1 = Point(1, 2)
cp1 = ColoredPoint(3, 4, "red")
print(p1.describe())   # Outputs: Point at (1, 2)
print(cp1.describe())  # Outputs: ColoredPoint at (3, 4) with color red
```

- `super()`: Calls the parent class method.
- Method overriding: The child class provides its own implementation of `describe`.

### Polymorphism

Polymorphism allows treating different classes as instances of a common parent class, each implementing shared methods differently.

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

- **Use clear names**: Class names should be nouns (e.g., `Point`, `ColoredPoint`), and method names should be verbs or descriptive (e.g., `distance_from_origin`).
- **Keep classes focused**: Each class should have a single responsibility (Single Responsibility Principle).
- **Validate inputs**: Add checks in `__init__` or methods to ensure data is valid (e.g., check if coordinates are numbers).
- **Use inheritance moderately**: Avoid deep inheritance hierarchies; prefer composition when possible.
- **Document your classes**: Use docstrings to explain the purpose of classes and methods.

Example with input validation and a docstring:

```python
class Point:
    """Class representing a 2D point with x and y coordinates."""
    
    def __init__(self, x, y):
        if not all(isinstance(coord, (int, float)) for coord in (x, y)):
            raise ValueError("Coordinates must be numbers")
        self.x = x
        self.y = y
```

## Useful Resources

Here is a selection of videos covering the concepts in this page:
* [Introduction to classes and objects in Python](https://youtu.be/t7KTxdn19-I)