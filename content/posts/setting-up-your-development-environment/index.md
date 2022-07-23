---
title: "Setting Up Your Development Environment"
date: 2021-11-02T00:00:01-05:00
draft: false
description: "This is the first post of the static website development series in which I'll walk over the process of building a static portfolio website. In this post, we will set up our project on our local computer."
categories:
- Tutorial
---

## Table of content

- [Table of content](#table-of-content)
- [Recap](#recap)
- [First steps](#first-steps)
- [Project structure](#project-structure)
  - [Folder](#folder)
  - [`build.py`](#buildpy)
  - [`data.py`](#datapy)
  - [`_site/`](#_site)
  - [`templates/`](#templates)
  - [Jinja2](#jinja2)
- [Summary](#summary)

## Recap

This is the first post of the static website development series in which I'll walk over the process of building a static portfolio website. In this post, we will set up our project on our local computer. So let's get to it.

## First steps

First of all, we will be using `python` to create our website. It doesn't matter if you are on Linux, Windows, macOS. As long as you can install and run python scripts, you are good to go.

I will be using python version 3. I won't go over the process of installing it, but you can follow the instructions from the official [python website](https://www.python.org). There are numerous tutorials online about installing `python3` alongside `python2`, be mindful of that.

You will also need to have `pip`, a package manager for python. Since the goal of this series is to make the development workflow as simple as possible, we will not be covering python environments (pipenv, venv, etc).

## Project structure

The initial structure of the project looks like this:

```bash
/{project_source_folder}/
    |
    |-- _site/
    |-- templates/
    |-- build.py
    |-- data.py
```

### Folder

I want to create a folder on my computer where I want to put everything related to the project in. Everything about the website will be inside this folder, or its subfolders. For example, `C:/static_website`.

### `build.py`

The `build.py` script will be the one and only script that I will need to execute to generate the final static files.

### `data.py`

This file is where we will store our data in an organized way. You will have to put some thought into how you want to structure your data. But as you will see in other videos, it's not a big deal if you need to change it. This file will provide the data to be used in the build script, so not a lotl of logic stuff will go here, only data structures, or more specifically, lists and dictionaries.

### `_site/`

We will make the `build.py` script output, spit out, all the files into this `_site/` subfolder. The `_site/` folder is what we call the build folder. After running the script, the folder can be used to directly serve the website.

### `templates/`

The templates folder is where we will have all the `html` pages. The templates will be like a blank page, it will have some spaces where you will tell that there should be some information injected into it. Templates are generic, and do not depend on the content. So if you create a new blog post or portifolio project to your website, you do not need to even touch the template files.

These are the files you will modify in order to make your site prettier, they don't contain any data from the site. The build script will use the data that we will provide, and the template files, and combine both into the final files (save them in the build folder).

### Jinja2

We will use the `jinja2` template engine. It is a python package. In short, it reads a text file that looks very similar to an html file, and fills out some black spaces with the information we pass to it. To install it you can open the bash terminal and use pip:
`pip install jinja2`

## Summary

Now that you have the initial (NOT FINAL) setup, we can create the first page. In this post we:

- Created a project folder
- Created a build script
- Created a build directory
- Created a template folder
- Created a data store file
- Installed the Jinja2 template engine

