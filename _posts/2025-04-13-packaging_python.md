---
title: A Beginner's Guide to Packaging Python Code
date: 2025-02-25
categories: [Software Development]
tags: [python, development, programming]     # TAG names should always be lowercase
author: Shibu
description: A step by step tutorial on packaging your Python Code
toc: true # default is true
pin: true
comments: true
---

## Level Up Your Python Code: A Beginner's Guide to Packaging

So, you've written some awesome Python code! Maybe it's a helpful utility script, a set of reusable functions, or even a small library. You can run it on your machine, which is great, but what if you want to:

*   Easily reuse it across different projects?
*   Share it with colleagues or friends?
*   Contribute it to the open-source community?
*   Install it reliably using `pip`?

The answer is **packaging**! Packaging bundles your code, metadata (like version, author, dependencies), and instructions so it can be easily distributed, installed, and managed.

This guide will walk you through the modern, standard way to package your Python code using `pyproject.toml` and standard tools like `build` and `twine`. Let's get started!

**Prerequisites:**

*   Python installed (including `pip`).
*   Basic familiarity with the command line/terminal.
*   Some Python code you want to package!

**Our Example Project: `mypackage`**

Let's imagine we have a simple project called `mypackage` with a single function that greets users.

---

**Step 1: Structure Your Project**

A standard project structure helps tools find your code and metadata. Here's a recommended layout:

```
mypackage_project/
├── src/
│   └── mypackage/
│       ├── __init__.py
│       └── greeter.py
├── pyproject.toml
├── README.md
└── LICENSE
```

*   **`mypackage_project/`**: The root directory containing everything related to your package.
*   **`src/`**: A directory containing the actual Python source code. Using `src/` prevents common import issues.
*   **`mypackage/`**: This is your actual Python package directory. The name you import (e.g., `import mypackage`).
*   **`__init__.py`**: Makes Python treat the `mypackage` directory as a package. Can be empty or used to define package-level items.
*   **`greeter.py`**: A Python module within your package containing our example code.
*   **`pyproject.toml`**: **The most important file!** This defines your project metadata and build system requirements. (We'll create this next).
*   **`README.md`**: Describes your project (what it does, how to install, how to use). Written in Markdown.
*   **`LICENSE`**: Contains the license under which you distribute your code (e.g., MIT, Apache 2.0). Choose one using sites like [choosealicense.com](https://choosealicense.com/).

---

**Step 2: Write Your Code**

Let's put our simple greeting function into `src/mypackage/greeter.py`:

```python
# src/mypackage/greeter.py
def say_hello(name="World"):
  """Greets the given name."""
  return f"Hello, {name}!"
```

And make it easily accessible from the package level in `src/mypackage/__init__.py`:

```python
# src/mypackage/__init__.py
from .greeter import say_hello

__version__ = "0.1.0" # Define package version here
```

---

**Step 3: Create `pyproject.toml`**

This file is the heart of modern Python packaging. It tells build tools *how* to build your package and provides metadata for package indexes like PyPI (the Python Package Index).

Create `pyproject.toml` in your project root (`mypackage_project/`) with the following content:

```toml
# pyproject.toml

[build-system]
requires = ["setuptools>=61.0"] # Minimum version of the build backend
build-backend = "setuptools.build_meta"

[project]
name = "mypackage-tutorial" # Distribution name (must be unique on PyPI)
version = "0.1.0" # Your package version
authors = [
  { name="Your Name", email="your.email@example.com" },
]
description = "A simple example package for a tutorial"
readme = "README.md" # Path to your README file
requires-python = ">=3.8" # Minimum Python version required
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License", # Match your LICENSE file
    "Operating System :: OS Independent",
]
dependencies = [
    # List any external packages your code needs here, e.g.:
    # "requests>=2.20.0",
    # "numpy",
]

[project.urls] # Optional: Links for users
"Homepage" = "https://github.com/yourusername/mypackage_project"
"Bug Tracker" = "https://github.com/yourusername/mypackage_project/issues"
```

**Key `pyproject.toml` sections:**

*   **`[build-system]`**: Specifies the tools needed to build your package. `setuptools` is the most common build backend.
*   **`[project]`**: Contains metadata about your package:
    *   `name`: The distribution name used on PyPI (hyphens are common). **Must be unique!**
    *   `version`: Your package version (see [PEP 440](https://peps.python.org/pep-0440/) for versioning guidelines). Often synced with `__version__` in `__init__.py`.
    *   `authors`: Your name and email.
    *   `description`: A short summary.
    *   `readme`: Points to your README file for the long description on PyPI.
    *   `requires-python`: Specifies compatible Python versions.
    *   `classifiers`: Helps users find your package on PyPI (browse classifiers [here](https://pypi.org/classifiers/)).
    *   `dependencies`: Crucial! Lists other packages your code needs to run. Pip will automatically install these when your package is installed.
*   **`[project.urls]`**: Optional links displayed on your PyPI project page.

---

**Step 4: Write Your `README.md`**

Create a `README.md` file in the project root. Explain what your package does, how to install it (`pip install mypackage-tutorial`), and provide a basic usage example.

```markdown
# My Package Tutorial

A simple example package created for a tutorial.

## Installation

pip install mypackage-tutorial

## Usage

```python
import mypackage

print(mypackage.say_hello("Pythonista"))
```
```

---

**Step 5: Add a `LICENSE` File**

Choose an open-source license (MIT is common and permissive) from [choosealicense.com](https://choosealicense.com/), copy the text into a file named `LICENSE` in your project root, and update the year and copyright holder name. Make sure the `classifiers` list in `pyproject.toml` reflects your chosen license.

---

**Step 6: Set Up a Virtual Environment (Highly Recommended!)**

Always work within a virtual environment to isolate project dependencies.

```bash
# Navigate to your project root directory
cd mypackage_project

# Create a virtual environment (named .venv here)
python -m venv .venv

# Activate it
# On Windows (cmd.exe)
# .venv\Scripts\activate.bat
# On Windows (PowerShell)
# .venv\Scripts\Activate.ps1
# On macOS/Linux (bash/zsh)
source .venv/bin/activate

# Your terminal prompt should now show (.venv)
```

---

**Step 7: Install Build Tools**

You need tools to build your package distribution files and upload them.

```bash
pip install build twine
```

*   **`build`**: The tool that actually builds your package from your source code and `pyproject.toml`.
*   **`twine`**: The tool used to securely upload your built package to PyPI.

---

**Step 8: Build Your Package**

Now, run the `build` command in your project root directory:

```bash
python -m build
```

This command will:

1.  Read your `pyproject.toml`.
2.  Create a `dist/` directory if it doesn't exist.
3.  Generate two files inside `dist/`:
    *   `mypackage_tutorial-0.1.0-py3-none-any.whl`: A **wheel** file. This is the standard, pre-built distribution format. Pip prefers installing wheels as they are faster.
    *   `mypackage_tutorial-0.1.0.tar.gz`: A **source distribution** (sdist). This contains your source code and instructions on how to build it. It's a fallback if a wheel isn't available for the user's system.

---

**Step 9: (Optional but Recommended) Test Local Installation**

Before uploading, test if you can install your package locally from the built wheel file:

```bash
# Make sure you are in your project root with the virtual env active
pip install dist/mypackage_tutorial-0.1.0-py3-none-any.whl

# Now try importing and using it in a Python interpreter
python
>>> import mypackage
>>> mypackage.say_hello("Developer")
'Hello, Developer!'
>>> exit()

# Uninstall it before proceeding to upload
pip uninstall mypackage-tutorial
```

---

**Step 10: Upload to TestPyPI (Practice First!)**

It's crucial to test the upload process using **TestPyPI**, a separate instance of the package index for testing.

1.  **Register Accounts:** Create free accounts on both [TestPyPI](https://test.pypi.org/account/register/) and the real [PyPI](https://pypi.org/account/register/).
2.  **Create API Tokens:** For uploading, use API tokens instead of your username and password.
    *   Go to your Account Settings on TestPyPI.
    *   Scroll down to "API tokens" and click "Add API token".
    *   Give it a name (e.g., `mypackage-upload`) and set the Scope to "Entire account".
    *   **Copy the token immediately!** You won't see it again.
3.  **Upload to TestPyPI:** Use `twine` to upload the files from your `dist/` directory. When prompted for the username, enter `__token__`. When prompted for the password, paste your **TestPyPI API token**.

    ```bash
    twine upload --repository testpypi dist/*
    ```

---

**Step 11: Install from TestPyPI**

Verify your package works by installing it from TestPyPI:

```bash
pip install --index-url https://test.pypi.org/simple/ --no-deps mypackage-tutorial
# Use --no-deps to avoid pulling dependencies from TestPyPI if they exist on real PyPI

# Test it again in Python
python
>>> import mypackage
>>> mypackage.say_hello("Tester")
'Hello, Tester!'
>>> exit()

# Uninstall
pip uninstall mypackage-tutorial
```

---

**Step 12: Upload to Real PyPI**

Once you're confident everything works correctly on TestPyPI, you can upload to the real PyPI.

1.  **Get a Real PyPI API Token:** Repeat Step 10.2, but on the main [PyPI](https://pypi.org/) site.
2.  **Upload:** Run `twine upload`, this time without specifying the repository (it defaults to PyPI). Use `__token__` as the username and your **real PyPI API token** as the password.

    ```bash
    twine upload dist/*
    ```

**Congratulations!** Your package is now live on PyPI, and anyone can install it using `pip install mypackage-tutorial`.

---

**Next Steps & Maintenance**

*   **Versioning:** When you update your code, increment the `version` in `pyproject.toml` (and potentially `__init__.py`) following semantic versioning ([semver.org](https://semver.org/)) principles. Re-build and re-upload.
*   **Dependencies:** Keep your `dependencies` list in `pyproject.toml` up-to-date.
*   **Testing:** Add automated tests (using `pytest`, `unittest`) to ensure your code works correctly.
*   **Advanced Features:** Explore adding command-line scripts (`[project.scripts]`), including data files, or using different build backends if needed.

Packaging might seem daunting initially, but following these steps makes it manageable. It’s a fundamental skill for any serious Python developer, enabling code reuse, collaboration, and contribution to the vibrant Python ecosystem. Happy packaging!
