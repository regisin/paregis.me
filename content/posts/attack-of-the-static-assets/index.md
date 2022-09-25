---
title: "Attack of the Static Assets"
date: 2021-11-13T00:00:01-05:00
draft: false
description: "A new force rises to the ranks of the website republic and threatens the peaceful galaxy of pages. In this chapter we tackle static assets and create a new sub page."
categories:
- Tutorial
---

## Table of content

- [Table of content](#table-of-content)
- [Recap](#recap)
- [Static assets and static website](#static-assets-and-static-website)
  - [Static pages](#static-pages)
  - [Static assets](#static-assets)
- [Adding static files to the project](#adding-static-files-to-the-project)
- [Adding static assets to the build script](#adding-static-assets-to-the-build-script)
- [A new page](#a-new-page)
- [The data manace](#the-data-manace)
- [The return of the build script](#the-return-of-the-build-script)
- [The template awakens](#the-template-awakens)
- [Summary](#summary)

## A blog post ago, in a sub page far, far away...

Up to this point we have a static website with a couple of pages, each of which can be accessed through conveniently named URLs. In this article, we will see how to deal with static files that make up our website.

Static assets are more commonly exemplified through JavaScript and CSS files that the browser loads with the webpage, and are used to customize the style, the aesthetics, of the website. Static assets can also be images, documents, or files that you upload with the site, or even the favicon, that little icon the browser shows on the tab of the website.

## The attack of the assets

I think I need to clarify some things here. So we are building a static website, and now I'm talking about static files. You might be wondering what's the difference between the pages and these so called "static assets"?

### Static pages

A static site is a website that doesn't generate its webpages dynamically, so there's no database that runs on a server, that each time someone's browser sends a request to load the site, would go and fetch the database to populate the pages.

What we are doing here is similar, but not dynamic. Yes, we look into a database, although it's not a database, it's just another python script. And yes, we generate pages based on the pseudo-database, but it's not generated each time someone tries to visit the website. So the script generates static webpages, when running `python build.py`, based on the given data.

### Static assets

Static assets are resources a website has saved on the webserver that may or may not be referenced in one of its webpages. CSS and JavaScript files are the most common examples of static files. Webpages can have a reference to these types of files, which will trigger the browser to fetch them. They allow the webdesigner to make the pages look prettier with CSS, and javascript can help with animations and alerts.

We will NOT be using separate files for CSS/JavaScript assets in this tutorial. Instead, what I want to show here is much simpler. I will use files such as images and pdfs. Things that you would normally upload to a content management system such as wordpress. But instead I will simply add those files to your project.

## Revenge of the assets

First, let's create another subfolder in our project folder and call it `/static`. It should be on the same level as the `/_site` and `/templates` folders.

Inside the static folder, create 2 subfolders: `/static/images` and `/static/files`. One will contain all images to be used in different places of the website, while the other will have other files. If you want to have other files in your pages, such as videos, source-codes, or whatever, you can use this `/static` folder to do so.

Alright, we have our static folder with some files inside. My full folder structure is looking like this:

```bash
/path/to/project/
    |-build.py
    |-data.py
    |-templates/
        |-about.html
        |-base.html
        |-teaching.html
    |-static/
        |-images/
        |-files/
```

## The return of the build script

Now we have to modify our build script to copy the static files into the build folder. Otherwise, the webserver won't be able to find them! Remember, when we serve the website, we point webserver to the `/_site` folder, but the assets are outside. You could workaround it and make the server work with the files from outside the `/_site` folder, but that's just bad practice and we don't need such a headache at this point.

In the build script, modify to copy the `/static` folder inside the `/_site` folder:

```python
BUILD_DIR = '_site'
STATIC_DIR = 'static'                           # Where the static assets are located
CURRENT_PATH = Path.cwd()
BUILD_PATH = CURRENT_PATH / Path(BUILD_DIR)
STATIC_PATH = CURRENT_PATH / Path(STATIC_DIR)
# clean-up build folder
try:
    shutil.rmtree(BUILD_PATH)
except FileNotFoundError:
    print("Build dir does not exist yet.")

# The following command will create the directories if they don't exist
# Thus, we don't need `BUILD_PATH.mkdir(parents=True, exist_ok=True)` anymore
shutil.copytree(STATIC_PATH, BUILD_PATH)        # Copy the static folder to the build dir
```

Now that's a bunch of weird stuff. Here's what it does: first we declare a couple of useful constants, the build and the static folders. Then we try to delete the build dir. If this is the first time you run the build script, or if you deleted the build folder by hand, the script will print the message that the folder does not exist. Then it will copy all the contents from the static into the build folder.

You could, and that's totally doable, simply never delete the static folders that are in the build folder at all! Then you wouldn't have to copy the static assets all the time. I just personally prefer it this way, always having a clean build of the website.

## A new page

Ok, so the build script now deletes and copies all static files to the build directory every time. One thing missing though, the files themselves! For my website, it makes sense to provide files related to my research. For a student, it could be files related to class projects. Any type of file that I will use in any page of my website will go in the `/static` folder.

I will add the PDF files of my publications, those will go into the `/static/files` folder. I will also add some images created over the years for these projects, those will go in the `/static/images` folder.

So... I now decided that to use these static files, I will create a new page for my website, the `/research/index.html` page. Naturally, I will create a template for this page in the `/templates` folder and call it `research.html`. In your case it could be a different name, like `project.html`. My template will be a list of my research projects. Something like this:

```html
{% extends 'base.html' %}
{% block content %}
      <h1>{{ title }}</h1>
      {% for project in projects %}
        <h2><a href=/research/{{ project.slug }}>{{ project.name }}</a></h2>
        <table>
          <tr>
            <td><img src={{ project.thumbnail }} width="200pt"></td>
            <td><p>{{ project.excerpt }}</p></td>
          </tr>
        </table>
      {% endfor %}
{% endblock %}
```

In a nutshell: I extend the base template so this page also includes the boilerplate stuff. This page will be expecting a list of `projects`, and for each item in that list (a `project`), it will output the `project.name`, `project.slug`, a short description with `project.excerpt`, and an image, the `project.thumbnail`. Just by looking at this template I can visualize how the research projects can be stored in my `data.py` file.

## The data manace

Each research project will be a dictionary with those key values (`name`, `slug`, `exerpt`, `thumbnail`), and strings values. The `name` will be the research proiject name (human readable), the `slug` is a version of the name that looks pretty in the URL address and be used when creating the detail page for each project, not yet though. The exerpt is a short description of the project, and the `thumbnail` which will be the path to an image that will be in the `/static/images` folder. And all project dictionaries will be contained inside of a `list` called `research` in the `data.py`.

Let's go ahead and add some stuff in our `data.py`:

```python
research = [
    {
        'name': 'My Awesome Research',
        'slug' : 'my-awesome-research',
        'excerpt' : 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed aliquam lectus eget nisl iaculis, at maximus elit convallis. Donec mattis, sem et imperdiet convallis, leo enim tincidunt massa, non aliquam ante libero sollicitudin quam. Maecenas facilisis urna urna, sed malesuada diam facilisis sit amet.',
        'thumbnail': '/images/my-awesome-research-thumb.png',
    },
    {
        'name': 'My Alright Research',
        'slug' : 'my-alright-research',
        'excerpt' : 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed aliquam lectus eget nisl iaculis, at maximus elit convallis. Donec mattis, sem et imperdiet convallis, leo enim tincidunt massa, non aliquam ante libero sollicitudin quam. Maecenas facilisis urna urna, sed malesuada diam facilisis sit amet.',
        'thumbnail': '/images/my-alright-research-thumb.png',
    },
]
```

Notice that the thumbnail value is not a full URL, it's the relative path that the image will be once copied to the build folder. For example, if the source file is in: `/path/to/project/static/images/my-alright-research-thumb.png`, after running the build script it will be copied to the build dir `/path/to/project/_site/images/my-alright-research-thumb.png`. So in the `data.py` I need to reflect that relative path to the build dir, not the original source folder.

We are almost there, there's one little detail that we need to take care of...

In the template, the address of the image file will be relative to the current URL. So when we visit the `/research` page, and the URL to a resource is relative (it's not a full address with `https://`, it's just a folder-like address), the resulting URL to the image, once the site is served, will be wrong (it will be `http://127.0.0.1:8000/research/images/my-alright-research-thumb.png` and not `http://127.0.0.1:8000/images/my-alright-research-thumb.png`), which naturally does not exist in the build folder. Two approaches to fix this:

1. In the build script I can copy those files to a new folder `/_site/research/images`
2. Or define a `BASE_URL` in the buildscript and use it in the templates.

I will be using 2.

## The rise of build script

First, don't forget to import your data to the build script:

```python
from store import teaching, research
```

In the build script, define a constant BASE_URL:

```python
BASE_URL = 'http://127.0.0.8000'
```

And in each call to a `render()` method, pass it to the template (I will add this to all other `render()` calls as well, not just the `research.html`, even if not used by template):

```python
# /research
template = env.get_template('research.html')
output = template.render(
    BASE_URL=BASE_URL,
    title="Research",
    projects=research
)
o = BUILD_PATH / Path("research")
o.mkdir(parents=True, exist_ok=True)
o = o / Path('index.html')
with o.open(mode='w') as fh:
    fh.write(output)
```

## The template awakens

Then in the template add `{{ BASE_URL }}` in front of any linked asset:

```html
<img src="{{ project.thumbnail }}" width="200pt">
```

Becomes:

```html
<img src="{{ BASE_URL }}{{ project.thumbnail }}" width="200pt">
```

## Your turn, Padawan

Go ahead and put images in the folder. Build and serve, see if it works for yourself:

```bash
python build.py
python -m http.server --directory ./_site/
```

## Summary

In this post we:

- Created static assets the website will use
- Added the assets to the build script flow
- Added references to the static assets in the data store
- Created a template that linked to the static assets

