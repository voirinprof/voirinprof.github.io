---
title: "Getting Started with GitHub Codespaces"
permalink: /en/codespaces-intro/
lang: en
date: 2025-04-26T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: start
---

> **_NOTE:_** The [official GitHub Codespaces documentation](https://docs.github.com/en/codespaces) provides detailed instructions to get started. A sample GitHub Codespace is available here: [https://github.com/voirinprof/geolabspace](https://github.com/voirinprof/geolabspace)

## Introduction to GitHub Codespaces

**GitHub Codespaces** is a cloud-based development environment integrated with GitHub. It allows you to code directly in a browser or via VS Code without setting up a local environment. Preinstalled with Python, Git, and other tools, it's ideal for quickly starting programming projects.

This guide shows you how to create a Codespace, write a simple Python program, and manage your code with Git.

## Creating a Codespace

1. **Access a repository**: Log in to GitHub and create or open a repository (for example, an empty project or a fork of an existing one).
2. **Launch a Codespace**: Click on **Code** > **Codespaces** > **Create codespace on main branch**. GitHub will set up an Ubuntu environment with Python in seconds.
3. **Open the editor**: The Codespace opens in your browser or in VS Code (if connected). You get access to a terminal and a code editor.

![Open Codespaces](/assets/img/codespace-step1.png)

> Tip: If you're creating a new repository, add a README file to initialize the main branch.

## Writing a Simple Python Program

Inside your Codespace, create a file named `somme.py` to calculate the sum of a list of numbers.

Example code:

```python
def calculer_somme(nombres):
    """Calculates the sum of a list of numbers."""
    return sum(nombres)

# Example usage
liste = [1, 2, 3, 4, 5]
resultat = calculer_somme(liste)
print(f"The sum of {liste} is {resultat}")
```

To run the program:
1. Open the terminal in the Codespace (shortcut: `Ctrl + ~`).
2. Run the command:

```shell
python somme.py
```

Expected output:
```
The sum of [1, 2, 3, 4, 5] is 15
```

> Tip: Use the Python extension in VS Code (preinstalled in Codespaces) for autocomplete and debugging.

## Installing Libraries

If your project needs Python libraries, use `pip`. For example, to install `numpy`:

```shell
pip install numpy
```

Example using `numpy` to compute a sum:

```python
import numpy as np

nombres = np.array([1, 2, 3, 4, 5])
somme = np.sum(nombres)
print(f"The sum with numpy is {somme}")
```

Run with:
```shell
python your_file.py
```

## Managing Code with Git

Codespaces is integrated with Git, making version control easy.

1. **Check changes**:
```shell
git status
```

2. **Add files**:
```shell
git add somme.py
```

3. **Commit changes**:
```shell
git commit -m "Add sum calculation script"
```

4. **Push to GitHub**:
```shell
git push origin main
```

Your changes are now saved in the GitHub repository.

> Tip: Commit frequently to avoid losing work, especially if the Codespace is stopped.

## Best Practices

- **Organize your files**: Keep a clear structure (e.g., a `scripts/` folder for Python files).
- **Test your code**: Add basic test cases to check results (e.g., test the function with `[]` or `[-1, 1]`).
- **Document your code**: Use docstrings and comments to explain your code.
- **Use a requirements.txt file**: List dependencies to make reuse easier:
```text
numpy==1.26.4
```
- **Save regularly**: Codespaces are temporary; push your code to GitHub to keep it safe.
- **Stop the environment**: Check if the environment is running—stop it to reduce hourly credit usage. The environment is saved and consumes fewer resources when stopped.
- **Monitor your Codespaces**: You can view all your Codespaces here: [https://github.com/codespaces](https://github.com/codespaces)

## Useful Resources

- [GitHub Codespaces documentation](https://docs.github.com/en/codespaces): Guide for setting up and using Codespaces.