---
title: "Pretty Urls"
date: 2021-11-12T00:00:01-05:00
draft: false
description: "In this post, we will change the build script to make the URL that we type in the browser more convenient and intuitive for us humans to read, in other words \"prettier\"."
categories:
- Tutorial
---

## Table of content

- [Table of content](#table-of-content)
- [Previously](#previously)
- [The output folder and the URL](#the-output-folder-and-the-url)
- [Changing the output path](#changing-the-output-path)
- [Clean the house after playing](#clean-the-house-after-playing)
- [Summary](#summary)

## Previously

In the last post we separated the data into its own separate file, the `data.py`. We modified the build script to load the data from this new file, and combine it with the templates to generate html pages in the build folder. In this post, we will change the build script to make the url that we type in the browser more convenient and intuitive, or "prettier".

## The output folder and the URL

Right now, the build script is generating files directly into the `/_site` build directory. After building the pages, we can start the http server with the command:

```bash
python -m http.server --directory ./_site/
```

Then, with the server running, I can checkout the pages in my browser by typing in the address bar `localhost:8000`. And if I want to check out the `teaching.html` page specifically, I can go to `localhost:8000/teaching.html`. That's cool and all, but we can do better.

Most websites today will have pretty URLs, which are URL addresses that are easier for people in general to memorize. In our case, for example, instead of having that `html` dangling on the back of the url, we could make the URL look like `localhost:8000/teaching` which is much nicer to look at.

If you ever dealt with some fancy framework like ~~React~~, ~~Angular~~, ~~Vue~~, or Svelte... they require some substantial learning in order to make this pretty URLs happen. But we are going for fast and easy, right? All we need to know is that when you hit enter after typing an address in your browser, the webserver will look for specific files on the other side.

For example, when you enter `localhost:8000` your browser will ask the webserver for a webpage, which happens to be running on your own machine, you know, that "http server" from python. That webserver, because you configured it to point to the `/_site` folder,  will look for the pages to send back to the requesting browser. Now, here's the thing, if the address is a folder-like address (`localhost:8000` points to `/_site`), then the webserver will, by default, look for an `index.html` file in that folder, and return its contents to the browser. It's the same as entering `localhost:8000/index.html`.

Why is this so important and revolutionary? Well, it's not, but now that we know that, we can use that default behavior to our benefit. To have pretty URLs in our static website, all we have to do is create the paths as folders in our build directory, and create `index.html` files for each of those sub folders.

To give an example, in the `/_site` folder we will have an `index.html`, and that will be the front page of the website. Then we will have a subfolder `/_site/teaching/` and inside that subfolder we will have an `index.html`. Then when a request comes asking for the `localhost:8000/teaching/`, the server will be forced to return back the `index.html` file in that subfolder. All of this is to say, our output files will all be named `index.html`, but they'll each be placed inside a subfolder inside the `/_site` build directory.

The change will look something like this:

- From:

    ```bash
    /path/to/project/
        |-...other files
        |-_site/
            |-about.html
            |-teaching.html
    ```

- To:

    ```bash
    /path/to/project/
        |-...other files
        |-_site/
            |-about/
                |-index.html
            |-teaching/
                |index.html
    ```

## Changing the output path

That's a lot of talking, let's get our hands dirty. Let's work on the `build.py` script. More specifically the teaching part of the script. This is the current script:

```python
o = Path.cwd()
o = o / Path('_site/teaching.html')
```

We will need to change it to look like this:

```python
o = Path.cwd()
o = o / Path("_site/teaching")
o.mkdir(parents=True, exist_ok=True)
o = o / Path('index.html')
```

Notice how we break down the variable 'o' a little bit. If we simply used the entire path to the `index.html` (for example: `o = Path("_site/teaching/index.html")`), then python would have crashed/raised an exception, because the subfolder `/teaching` doesn't exist. That's why we have a call to `o.mkdir(parents=True, exist_ok=True)` in between, which creates any folder in the path that doesn't exist already.

*Note: if you call `o.mkdir(parents=True, exist_ok=True)`, and the path ends with `index.html`, it will treach that `index.html` as a folder, and not a file.*

Now when you run the build script (`python build.py`), it will create the subfolder, and write to a file in there. Then, if you start the http server, and go to the url localhost:8000/teaching you will see the `index.html` that's inside that folder, and not the teaching.html file from before.

## Clean the house after playing

Which brings me to the last part of this article. You see how the `teaching.html` file is still hanging out in the build directory? In the build script, all we do is create new files, or overwrite existing ones! That's not good enough... we don't want that old garbage files that we created for testing purposes from previous steps, right? What I'm going to do now is tell the build script to delete all files inside the build folder `/_site` before I even start generating the pages.

To do so, at the beginning of the build script, we will import another library that's already included with python, the `shutil`, it will help with deleting an entire folder and its content:

```python
import shutil
```

Then let's create a couple of constant variables, just because we use it all throughout the rest of the build script, so if I decide to change the build directory I can change it in one place and not all over the script:

```python
BUILD_DIR = '_site'
CURRENT_PATH = Path.cwd()
BUILD_PATH = CURRENT_PATH / Path(BUILD_DIR)
```

Then we can go ahead and delete the `/_site` from the project folder. Remember, this will be done everytime we run the build script, as part of the build process to clean up whatever the previous build was.

```python
# remove current build
try:
    shutil.rmtree(BUILD_PATH)
except FileNotFoundError:
    print("Build dir does not exist (yet).")
```

And of course, now we need to recreate the build dir:

```python
# make build dir
BUILD_PATH.mkdir(parents=True, exist_ok=True)
```

And now we will have a clean build every time we run the build script. The build script now will look like this:

```python
import shutil

from jinja2 import Environment, FileSystemLoader
from pathlib import Path

from data import teaching

BUILD_DIR = '_site'
CURRENT_PATH = Path.cwd()
BUILD_PATH = CURRENT_PATH / Path(BUILD_DIR)

file_loader = FileSystemLoader('templates')
env = Environment(loader=file_loader)

"""
About
"""
# /about.html
template = env.get_template('about.html')
output = template.render(
    title="About",
)
o = BUILD_PATH / Path("about")
o.mkdir(parents=True, exist_ok=True)
o = o / Path('index.html')
with o.open(mode='w') as fh:
    fh.write(output)
"""
Teaching
"""
# /teaching.html
template = env.get_template('teaching.html')
output = template.render(
    title="Teaching",
    universities=teaching
)
o = BUILD_PATH / Path("teaching")
o.mkdir(parents=True, exist_ok=True)
o = o / Path('index.html')
with o.open(mode='w') as fh:
    fh.write(output)
```

Now, if you build and serve, you can navigate to the pretty URLs `localhost:8000/teaching`.

## Summary

In this article we:

- Created subfolders inside the build folder, one for each web page
- Modified the build script to output each page in a different subfolder
- Renamed all output files to `index.html`, which the webserver look for by default if no file is specifically requested
- Ended up with prettier URLs while navigating the website