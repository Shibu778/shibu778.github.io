---
title: Python Package Development
date: 2025-02-25
categories: [Software Development]
tags: [python, development, programming]     # TAG names should always be lowercase
author: Shibu
description: A comprehensive tutorial to develop and deploy a python package.
toc: true # default is true
pin: true
comments: true
---


# Tutorial on Python Package Development
This article is about developing python package focusing on designing, building and publishing. Creating a Python package can be a rewarding experience, whether you’re looking to share your code with others or simply organize your own projects better. Here, I am writing the process I have used to develop the `defectpl` python package. This is a package that deals with post processing of VASP calculation of point defects and generate the PL spectra and relevant diagrams for analysis of phonon modes, partial HR factor and localization etc. 

Here, I have written this article to track the python package development process which wil help me in developing future python projects.

# Stages of Python Package Development
There are three stages of python package development. This includes Designing, Building and Publishing. In the Design Phase the name of the package, its purpose, the problem it solves, requirements, structure of the code, testing framework etc are detailed. In this stage, the actual code is written with suitable functions and corresponding tests. The code is refactored and well commented and documented for easier use by users.

## Design
So, lets first define the following things

+ Name of the package: `defectpl`
+ Purpose of the package: Unified package for analyzing the optical properties of defect.
+ Problem the package is solving: Calculating and plotting PL spectra of point defects and relevant quantities.
+ Requirements : It requires different packages like `pymatgen`, `pyyaml`, `numpy`, `matplotlib` etc.
+ Code : The code can be accessed at [GitHub Repository](https://github.com/Shibu778/defectpl).
+ Unit Testing : Pytest will be used to test the functionality of `defectpl`.
+ Code Refactoring : Use black, radon, bandit, vulture, isort

The first step is to check whether the name you have chosen is available in `PyPI` and `conda-forge`. If it's not choose a different name. For my case, the `defectpl` name was available in both `PyPI` and `conda-forge`. For your package use a different name, don't use `defectpl`. 

Create a GitHub repository with name defectpl. Add a readme, `.gitignore`, and a liscence file. Clone the repository to your local system. Now we need the `poetry` 

If poetry is not installed in your system, install it. Check following : [install poetry](https://python-poetry.org/docs/#installing-with-pipx). Do not install poetry in environment in which you are going to do your development. Keep the poetry in a separate environment to keep it isolated. Do not activate the poetry environment to run it. So, create an environment named let say `poetry` to install `poetry` and other packages like `black`, `radon` etc. Create another environment for development. For mycase the environment in `defectpl_dev`.

Once you have installed poetry, go to the clone repository and run `poetry new defectpl` to create the necessary directory structures. This will create a `defectpl` directory inside `defectpl` cloned repository. The new directory will contain a `pyproject.toml` file, a `README.md` file, a test directory and another `defectpl` directory for storing the code. So, finally we have the cloned `defectpl` directory which is the root for GitHub repository, inside which there is another `defectpl` directory created by `poetry new` as the root of the project to be published in `PyPI`. The second `defectpl` directory consists of the `pyproject.toml` file. Next, there is another `defectpl` directory which contains the actual code. 

Now you can add a version of the package to the `__init__.py` file in the innermost `defectpl` directory. Also, you can add test to `test_defectpl.py` file in the `tests` directory to test the version. Provide a description of the package. Check the version. Add name and email address. 

Then you can start editing the `pyproject.toml` file. This file takes care of information used in `setup.py`, `requirements.py` etc.

Create required source files in the `defectpl` directory. For example for this package, I have created the `defectpl.py` and `utils.py` file containing required class and function for optical properties calculation of point defects.

Configure the poetry setting so that it can use the conda environment for development. 
1. In `base` environment, run the following
```
poetry config virtualenvs.path $CONDA_ENV_PATH 
poetry config virtualenvs.create false
```
 Where `$CONDA_ENV_PATH` is the path to the base `envs` folder (e.g. `/Users/myuser/anaconda3envs`)
 2. Create a development environment for developing this package, name : `defectpl`.  with latest python version.
```
conda create -n defectpl python=3.11
```
3. Activate the environment
```
conda activate defectpl
```
4. Go to the project directory containing `pyproject.toml` and run following to update and install new packages. Do not use `conda` or `pip` to install package in that environment, otherwise the local environment and poetry(and thus `pyproject.toml`) will be out of sync. 
```
poetry install
```

This will create the `poetry.lock` file that contains the dependencies installed via `poetry`.

### Write the Code and Tests
Write code in `defectpl` and write necessary test in `tests` directory with `files` and `functions` with `test_` initial. We will use `pytest` as testing framework.

### Testing
After writing the tests. Go to the `defectpl` directory containing `pyproject.toml` and run `pytest` to run the tests. Running `pytest -v` will increase the verbosity and print which tests are failing and which tests are passing.

### Code Refactoring

Create and activate new environment of for code refactoring. This can be the environment containing `poetry` package.

```
conda create -n poetry python=3.11
conda activate poetry
pip install black bandit radon vulture isort pandoc
```

```
black : code formatter
bandit : Security oriented static analyzer for python code
radon : Evaluates various metrics for the source code
vulture : Finds unused/dead code in python programs
isort : A python utility/library to sort python imports
```

### radon

Using the following command with any file will give the complexity of the functions, methods, class in that file.

```
radon cc some_python_file.py -s
```

`-s` option at the end will give also the score.

How maintainable is the code?
```
radon mi some_python_file.py -s
```

Number of lines of codes, vocabulary etc.
```
radon hal some_python_file.py
```

### vulture
Check the dead objects that are not used
```
vulture some_python_file.py
```

### black

To format the code use.

```
black some_python_file.py
```


Improve the `README.md` file.

### pandoc

Using pandoc to create `README.rst` from `README.md`.

```bash
pandoc README.md --from markdown --to rst -s -o README.rst
```


Add further details in `pyproject.toml` like the homepage, license, keywords, classifiers etc.
### Check
Do the following to check everything is good.

```bash
poetry check
```

## Build
Build the package
+ Package
+ Docs
Tools : Poetry (Twine, Wheel), Pytest, Tox, Coverage

### Build the package
Go to the directory containing `pyproject.toml` and run

```bash
poetry build
```

It will create a `whl` and `tar.gz` file in `dist` folder.

### Push it to Github
Go to the master directory of the github clone. Before commiting anything edit the `.gitignore` file to ignore committing unnecessary files to Github.

```bash
git status
git add .
git commit -m "First Commit"
git push
```


## Deploy and Publish

+ Publish Docs
	+ github-pages
	+ readthedocs
	+ custom site
+ Deploy on PyPI or Conda or Github Packages
Tools for Docs : mkdocs, sphinx
Tools for Deploy: Git, Poetry


### Publish it on PyPI

Execute the following command to publish it on PyPI

```bash
poetry publish
```

You might have to give credential for your PyPI account.

### Publishing an Update of the Package on PyPI

Do all the changes. Do `poetry build` and then do `poetry publish`.

### Publish it on Conda-forge

Make the conda-forge recipe. For this install grayskull first.
[Grayskull](https://github.com/conda-incubator/grayskull) is a community-developed, stand-alone conda recipe generator.  
You can install Grayskull simply with `conda install -c conda-forge grayskull`  
and then run `grayskull pypi defectpl=0.1.0` to generate the recipe in the current folder for package `defectpl=0.1.0`. Please note that the package should already be published in PyPI.

Now, modify the `meta.yaml` file as required. Most of the settting will be already well set by grayskull. Other things like `dev_url`, `doc_url` can be set. An example of `meta.yaml` file is given below.

<script src="https://gist.github.com/Shibu778/d3ea6eacbed7a0cc7fc9079c0b7693ba.js"></script>

Now, get a fork of the staged-recipes([conda-forge/staged-recipes: A place to submit conda recipes before they become fully fledged conda-forge feedstocks](https://github.com/conda-forge/staged-recipes)) to start the process.

Once you have a fork press `.` to go to the dev mode in Github. Now go to recipes and create a folder for your package e.g. `defectpl` . Now add the above `meta.yaml` file in that folder. Also add a `LICENSE` file. Commit the changes to the repository with the commit message `Adding recipe for defectpl` . Now Create a pull request to the main repository. This will trigger several checks. Make sure all the checks pass. Once the check is passed, check the instruction in the pull request first comment (enable edit to see it) to make a comment for review. For my case, the second comment is `@conda-forge/help-python The recipe has passed all the tests, and it is ready for review. Thanks. @conda-forge-admin`. The first `@conda-forge/help-python` alerts the python team to make the review and the second `@conda-forge-admin` helps the first time contributer to the `conda-forge` to mention the team associated with the repo for PR review.

Once the review is completed the package will be published in [Anaconda.org](https://anaconda.org/conda-forge/defectpl).

### Updating a cond-forge publish


### Publishing the Documentation

Install the `mkdocs` package in `poetry` environment.

```bash
conda activate poetry
pip install mkdocs
```


Create directory called `defectpl-docs` in the cloned repository to write the docs. Go inside the directory and run

```bash
mkdocs new defectpl
```

It will create a `defectpl` folder with `mkdocs.yml` file and a `docs` folder containing `index.md` file.

Edit the `mkdocs.yml` file

```yml
site_name: defectpl
site_url: https://github.com/Shibu778/defectpl
nav:
  - "Home": index.md
  - "Getting Started": getting_started.md
  - "Installation": installation.md
  - "Usage": usage.md
  - "Tutorials": tutorials.md
  - "API": api_reference.md

theme: readthedocs
```

Create each of the `.md` file in the docs directory. Then go the run the following command in the folder containing `mkdocs.yml` file.

```bash
mkdocs serve
```

This will start a local server and give a link to see the documentation.

To build the docs do the following

```bash
mkdocs build
```

It will create a `site` folder. 

Now copy the `docs` folder and `mkdocs.yml` file to the cloned folder. Now go there and execute the following to deploy it to github pages.

```bash
mkdocs gh-deploy
```

It will create a `gh-pages` branch and push the `docs` related file to that branch. It will also provide the link to the deployed page for my case it is `https://Shibu778.github.io/defectpl/`. To get this link and also to set a custom domain name you can go to the `gh-pages` branch in the GitHub repository, then go to settings and then to pages. There you will get this published link and also other options. 

Now copy this link and put it in different files where you want to give access to documentations. For example in `README.md` files and `pyproject.toml` file.

Now you can delete the `defectpl-docs` folder which was created just to start developing the documentation as the `docs` and `mkdocs.yml` are already copied. Now push the changes to the github. Go the cloned directory.

```bash
git status
git add .
git commit -m "Published the docs"
git push
```




# Resources
1. [(157) Python Package Development Tutorial ( Design,Build and Publish) - YouTube](https://www.youtube.com/watch?v=ueuLe4PipiI)
2. PyPI reserve package name : [How to Reserve (and...](https://philipkiely.com/code/pypi_name)
3. Using poetry with conda : [How to Use Poetry with Conda for Package Management on a Specific Python Version - Data Science <3 Machine Learning](https://michhar.github.io/2023-07-poetry-with-conda/#:~:text=Instructions%201%20Setup%20Poetry%20and%20Conda%20Install%20poetry,guide%20%28go%20to%20Installing%20with%20poetry.lock%29.%20References%20)
4. [Uploading and installing conda packages — Anaconda documentation](https://docs.anaconda.com/anacondaorg/user-guide/packages/conda-packages/)
5. [GeoSoft Lesson 24 - Publishing Python Packages on conda-forge](https://www.youtube.com/watch?v=NNWhz7pqwCw)
6. [How to publish a Python package on conda-forge](https://blog.gishub.org/how-to-publish-a-python-package-on-conda-forge)
7. [GeoSoft Lesson 25 - Updating a Python Package on conda-forge](https://www.youtube.com/watch?v=yVFDdI-0E1M&list=PLAxJ4-o7ZoPeUqGpMhvJoVk5G-TrvMAd-&index=25)
8. [Python Software Development](https://youtube.com/playlist?list=PLAxJ4-o7ZoPeUqGpMhvJoVk5G-TrvMAd-&si=ZvOWVhrax2J3j_VG)
