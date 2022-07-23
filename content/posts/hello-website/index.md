---
title: "Hello Website"
date: 2021-11-04T00:00:01-05:00
draft: false
description: "In this post, we will build our first static page using `python`, an `html` template, and serve it locally."
categories:
- Tutorial
---

## Table of content

- [Table of content](#table-of-content)
- [Recap](#recap)
- [Create a template](#create-a-template)
- [Updating the build script](#updating-the-build-script)
- [Serve the website with a proper webserver](#serve-the-website-with-a-proper-webserver)
- [Summary](#summary)

## Recap

We have set up our development environment on our own computer, now it's time to build something useful. In this post, we will build our first static page using `python`, an `html` template, and we will also serve the website locally.

## Create a template

To start, let's create a file in the `/templates` folder and call it `hello.html`. Now let's put some very basic html:

```html
<html>
    <body>
        Hello world!
    </body>
</html>
```

That's it. Nothing fancy here!

## Updating the build script

*At this point, it's worth mentioning that the workflow that we are about to create will not change. So even though these first examples will look super simplistic, the way that you will make your pages will feel very similar.*

Now let's go to the `build.py` script. In the build script, the first thing we need to do is import the jinja engine so add the import to the top. We will also use the `pathlib` library that should already be available to you with `python 3`:

```python
from jinja2 import Environment, FileSystemLoader
from pathlib import Path
```

We will use the `FileSystemLoader` to point to a folder that contains all the template files. It makes our life easier so we don't have to open the files and parse them ourselves. The `Environment` class will let us load the templates by passing the filename of the template, that of course is inside the `/templates` folder:

```python
file_loader = FileSystemLoader('templates')
env = Environment(loader=file_loader)
template = env.get_template('hello.html')   # tell jinja to load the template "hello.html"
output = template.render()                  # generates the html code as a string
o = Path.cwd()                              # get the current directory of the build script
o = o / Path('index.html')                  # append the output filename to the path
with o.open(mode='w') as fh:                # open de output file represented by the Path object 'o'
    fh.write(output)                        # write the generated html code into the file
```

Save the build script. Now go to the terminal and run it (don't forget you have to CD to the folder of your project):

```bash
cd /path/to/project/folder
python build.py
```

If everything goes smoothly, you should see a new file in your project folder, named `**index**.html`! If you open the file, you will notice that it looks exactly like the template `**hello**.html`, and that's not fun at all. Let's change that!

Edit the template hello.html to look like this:

```html
<html>
    <body>
        Hello **{{ name }}**!
    </body>
</html>
```

And in the build script you only edit one line, the `render()` method that Jinja uses to generate the `html` markup:

```python
output = template.render(name="Harry Potter")
```

Now run the build command again: `python build.py`

Wow! Now if you open the `index.html` file you will see Harry's name there! That's pretty cool if you ask me.

But it's a little annoying that the built `index.html` is mixed in the project folder, let's instead, write to our build directory `/_site`. You can name it whatever you want, but I'm used to this underscore site pattern so I'll stick to it.

In the build script edit so the `o` variable looks like this:

```python
o = o / Path('_site/index.html')
```

This will create a path the points to the build dir `/_site`. Run the build command. Now the file is generated in the subfolder. Awesome, this way we don't have to worry trying to figure out which `html` files are used as a template and which are generated pages. All files in the `/_site` subfolder will be considered the final product!

## Serve the website with a proper webserver

As of now we haven't really **SERVED** our website, we were simply opening the `html` files with whatever program we wanted like a text editor or a browser. Now that our build files are all centralized in a single folder, we can spin up an HTTP server that points to the `/_site` folder.

Luckily, `python` comes with a very simple server that gets the job done, very useful in development mode. In your terminal, after generating the `html` files in the build directory, run this command:

```bash
python -m http.server --directory ./_site/
```

*Note python 2 has a similar command: `python -m SimpleHTTPServer` but you will need to `cd /_site/` first.*

This command will start python's built-in http server pointing to the `/_site` folder. Now move to your browser and enter `127.0.0.1:8000` in the address bar, that's your localhost address. And the server by default uses port `8000`, so we need that too.

Ta-Da! Now we have a fully functional website. How exciting! Even though the end product is very minimalist, to be polite, it helps to figure out the development workflow of building and serving. We will build upon this process to personalize the website further.

## Summary

In this post, we saw how to create a simple jinja2 template. We also use the built-in python http server to serve the website locally.

- Summarizing, in this video we:
- Created a template file
- Loaded the template engine in the build script
- Used the template file to create a static page
- Served the website using an http server

