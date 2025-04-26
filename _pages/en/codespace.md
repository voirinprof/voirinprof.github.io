---
title: "Getting Started with GitHub Codespaces"
permalink: /en/codespaces-intro/
lang: en
date: 2025-04-26T10:00:00+00:00
toc: true
toc_label: "On this page"
typenav: start
---
> **_NOTE:_** The [official GitHub Codespaces documentation](https://docs.github.com/en/codespaces) provides detailed instructions for getting started. An example of a GitHub Codespace is available at: [https://github.com/voirinprof/geolabspace](https://github.com/voirinprof/geolabspace)

## Introduction to GitHub Codespaces

**GitHub Codespaces** is a cloud-based development environment integrated with GitHub. It allows you to code directly in a browser or via Visual Studio Code without setting up a local environment. Pre-installed with Python, Git, and other tools, it’s ideal for quickly starting programming projects.

This guide shows you how to create a Codespace, write a simple Python program, and manage your code with Git.

Before starting:
- **Define the goal**: For example, write a Python script that calculates the sum of numbers.
- **Prepare the tools**: No local software is required, just a GitHub account.
- **Plan**: Write the code, test it, and save changes to a repository.

## Creating a Codespace

1. **Access a repository**: Log in to GitHub, create or open a repository (e.g., an empty project or a fork of an existing repository).
2. **Launch a Codespace**: Click **Code** > **Codespaces** > **Create codespace on main**. GitHub sets up an Ubuntu environment with Python in seconds.
3. **Open the editor**: The Codespace opens in your browser or in VS Code (if connected). You have access to a terminal and a code editor.

![Open Codespaces](/assets/img/codespace-step1.png)

> Tip: If you create a new repository, add a README file to initialize the main branch.

## Writing a Simple Python Program

In your Codespace, create a file named `sum.py` to calculate the sum of a list of numbers.

Example code:

```python
def calculate_sum(numbers):
    """Calculates the sum of a list of numbers."""
    return sum(numbers)

# Example usage
numbers = [1, 2, 3, 4, 5]
result = calculate_sum(numbers)
print(f"The sum of {numbers} is {result}")
```

To run the program:
1. Open the terminal in the Codespace (shortcut: `Ctrl + ~`).
2. Run the command:

```shell
python sum.py
```

Expected output:
```
The sum of [1, 2, 3, 4, 5] is 15
```

> Tip: Use the Python extension in VS Code (pre-installed in Codespaces) for autocompletion and debugging.

## Installing Libraries

If your project requires Python libraries, use `pip`. For example, to install `numpy`:

```shell
pip install numpy
```

Example using `numpy` for a sum:

```python
import numpy as np

numbers = np.array([1, 2, 3, 4, 5])
total = np.sum(numbers)
print(f"The sum with numpy is {total}")
```

Run with:
```shell
python your_file.py
```

## Managing Code with Git

Codespaces is integrated with Git, making version control straightforward.

1. **Check changes**:
```shell
git status
```

2. **Stage files**:
```shell
git add sum.py
```

3. **Create a commit**:
```shell
git commit -m "Add sum calculation script"
```

4. **Push to GitHub**:
```shell
git push origin main
```

Your changes are now saved to the GitHub repository.

> Tip: Commit changes frequently to avoid losing work, especially if the Codespace is stopped.

## Best Practices

- **Organize files**: Maintain a clear structure (e.g., a `scripts/` folder for Python files).
- **Test your code**: Add simple test cases to verify results (e.g., test the function with `[]` or `[-1, 1]`).
- **Document**: Use docstrings and comments to explain your code.
- **Use a requirements.txt file**: List dependencies for easy reuse:
```text
numpy==1.26.4
```

- **Save regularly**: Codespaces are temporary; push your code to GitHub to preserve it.

## Useful Resources

- [GitHub Codespaces Documentation](https://docs.github.com/en/codespaces): Guide to setting up and using Codespaces.
