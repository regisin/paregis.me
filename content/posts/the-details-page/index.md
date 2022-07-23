---
title: "The Details Page"
date: 2021-11-17T00:00:01-05:00
draft: false
description: "We created a static website with the About, Teaching, and Research pages. The `/research` page lists all projects in a compact manner. In this post, we will create a dedicated page for each research project listed in the data store."
categories:
- Tutorial
---

## Table of content

- [Table of content](#table-of-content)
- [Recap](#recap)
- [The details page](#the-details-page)
- [New template](#new-template)
- [Moar data](#moar-data)
- [Adding the new page to the build flow](#adding-the-new-page-to-the-build-flow)
- [A word on the development workflow](#a-word-on-the-development-workflow)
- [Summary](#summary)

## Recap

I have created a static website with a few pages in it: the About page, the Teaching, and the Research pages. The `/research` page, which is kind of my professional portfolio, right now simply shows a list of my research projects, but does not give out any details just yet... It's time to change that. In this post, I will be creating a details page for each research project in the data store.

Like I said before, I have this `/research` page in the website, and if we check it out all it does is display all of the projects in a `<ul>` list. Each project has an image associated with it, the thumbnail, and a short description of the project, the excerpt. Both the thumbnail and the excerpt are fields in the datastore (or keys in a dictionary), so when we add a new project to the file, we need to make sure we follow the same structure.

The excerpt and thumbnail are both text strings, but in reality they are used differently. The excerpt text will be printed out as is, meaning the `build.py` script will read it and inject that string into the template where the placeholder is.

The thumbnail, on the other hand, even though it's just a piece of text, the text itself is not displayed directly. Instead, it is used to make up the source `src` property of an `<img>` tag in the html. The thumbnail must point to the location of the image file on the server, otherwise no image will show up in the rendered website when it is served.

Just to give you an overview of the files and a step-by-step of what happens, here are samples of the datastore `data.py`, the template `research.html`, the `build.py` script, and the generated `index.html`, and the folder structure of the project:

- A single entry in the `research` from `data.py`:
  
    ```python
    research = [
        # ... other items ommited
        {
            'name': 'My Awesome Research',
            'slug' : 'my-awesome-research',
            'excerpt' : 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed aliquam lectus eget nisl iaculis, at maximus elit convallis. Donec mattis, sem et imperdiet convallis, leo enim tincidunt massa, non aliquam ante libero sollicitudin quam. Maecenas facilisis urna urna, sed malesuada diam facilisis sit amet.',
            'thumbnail': '/images/my-awesome-research-thumb.png',
        }
        # ...
    ]
    ```

- The `research.html` template file:

    ```html
    {% extends 'base.html' %}
    {% block content %}
        <h1>{{ title }}</h1>
        {% for project in projects %}
            <h2><a href=/research/{{ project.slug }}>{{ project.name }}</a></h2>
            <table>
            <tr>
                <td><img src={{ BASE_URL }}{{ project.thumbnail }} width="200pt"></td>
                <td><p>{{ project.excerpt }}</p></td>
            </tr>
            </table>
        {% endfor %}
    {% endblock %}
    ```

- Relevant snippet of the `build.py` script:

    ```python
    # ... ommited code
    shutil.copytree(STATIC_PATH, BUILD_PATH)        # Copy the static folder to the build dir
    # ...
    """
    Research
    """
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

- The generated `index.html` that is placed in `/project/folder/_site/research/index.html`:

    ```html
    <html>
        <head>
            <title>Research</title>
        </head>
        <body>
        <h1>Research</h1>
            <h2><a href=/research/my-awesome-research>My Awesome Research</a></h2>
            <table>
            <tr>
                <td><img src=http://localhost:8000/images/my-awesome-research-thumb.png width="200pt"></td>
                <td><p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed aliquam lectus eget nisl iaculis, at maximus elit convallis. Donec mattis, sem et imperdiet convallis, leo enim tincidunt massa, non aliquam ante libero sollicitudin quam. Maecenas facilisis urna urna, sed malesuada diam facilisis sit amet.</p></td>
            </tr>
            </table>
            <!-- ommitted html: this is where other <h2>...</h2><table>...</table> combos will show up, one for each entry in the research datastore -->
        </body>
    </html>
    ```

- Folder structure, `/_site` is the build script, and we don't manipulate files in there directly:

```bash
/path/to/project/
    |-build.py
    |-data.py
    |-static/
        |-files/
        |-images/
            |-my-alright-research-thumb.png
            |-my-awesome-research-thumb.png
    |-templates/
        |-about.html
        |-base.html
        |-research.html
        |-teaching.html
    |-_site/
        |-about/
            |-index.html
        |-files/
        |-images/
            |-my-alright-research-thumb.png
            |-my-awesome-research-thumb.png
        |-research/
            |-index.html
        |-teaching/
            |-index.html
```

## The details page

All of this is working fine, so now what? Well, now I want to give some more information about each project. I want to showcase my work to the visitors by creating a new subpage, dedicated to a single project. To do this there are a few things I'll need to do.

## New template

First, I need to create a new template to show the details of a research project. So let's do that. Create a new template file in the `/templates` folder, I'm going to call it `research_detail.html`, and it will look like this:

```html
{% extends 'base.html' %}
{% block content %}
      <h1>{{ project.name }}</h1>
      <p>{{ project.description }}</p>

      <h2>People</h2>
      <ul>
      {% for person in project.people %}
            <li>
                  {% if 'url' in person %}
                  <a href ="{{ person.url }}">{{ person.name }}</a>
                  {% else %}
                  {{ person.name }}
                  {% endif %}
            </li>
      {% endfor %}
      </ul>

      <h2>Resources</h2>
      {% for res in project.embed %}
        {{ res }}
      {% endfor %}
{% endblock %}
```

Wow, ok, so what's going on here? Let's break it down:

1. We extend the base template as usual, and we define stuff to go in the `content` block
2. The project name in a big old `<h1>` tag.
3. The description of the project, which will be a comprehensive text so it can get quite big, in a paragraph `<p>`.
4. A section of the page to list all the people that worked with me in the project. *Side note: always give credit when credit is due!*
   1. For each person associated with this project, the template checks if they have a url field associated with the person. It checks if the dictionary has the key `url`, and if so, it will output an anchor `<a>` link, otherwise it simply prints out the name of the person.
5. Finally, there is a resource section. This section is where I'll embed some `html` components directly from the data store.

Naturally, I need to edit my data store and add these things to each research project next.

## Moar data

Let's make it look like this:

```python
research = [
    # ... ommitted the other project
    {
        'name': 'My Awesome Research',
        'slug' : 'my-awesome-research',
        'excerpt' : 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed aliquam lectus eget nisl iaculis, at maximus elit convallis. Donec mattis, sem et imperdiet convallis, leo enim tincidunt massa, non aliquam ante libero sollicitudin quam. Maecenas facilisis urna urna, sed malesuada diam facilisis sit amet.',
        'thumbnail': '/images/my-awesome-research-thumb.png',
        'description': 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nullam et pulvinar erat. Suspendisse ultricies placerat mi at gravida. Nulla tellus eros, lacinia suscipit sem in, dignissim interdum nulla. Sed tincidunt diam ac lectus scelerisque aliquet. Sed tincidunt lorem ut enim aliquam hendrerit at eget ante. Cras at interdum diam, sed iaculis nunc. Ut dictum sit amet quam non ornare. Phasellus interdum congue convallis.Suspendisse rutrum quis neque et pharetra. Pellentesque porttitor tortor sit amet risus volutpat maximus. Ut ultrices metus non vestibulum ullamcorper. Morbi eu lacus metus. Etiam ac scelerisque ante. Praesent ut erat dictum, tristique orci laoreet, ultrices diam. Cras luctus ex at massa convallis auctor. Suspendisse facilisis lobortis lectus quis molestie. Praesent velit odio, fringilla gravida dolor non, faucibus facilisis quam. Nunc imperdiet consectetur arcu sit amet fringilla. Aenean ipsum dolor, consequat luctus magna at, varius vestibulum lorem. Nullam malesuada magna eu massa rutrum, in vestibulum eros vestibulum. Maecenas volutpat nunc et nulla semper, quis viverra leo pellentesque. Ut pellentesque sem et commodo hendrerit. Cras vitae pharetra neque.Cras aliquam vel mauris egestas egestas. Vestibulum at orci a quam vehicula viverra. Sed finibus eget mi vel viverra. Phasellus quis ligula non ante consectetur pellentesque quis eget eros. Ut vestibulum laoreet iaculis. Nam fermentum lacus sed elit bibendum hendrerit. Sed rutrum viverra lacus, non consequat urna consequat at. Fusce vehicula tristique turpis ornare bibendum. Praesent quam sapien, porttitor vel tincidunt vel, feugiat et justo.Maecenas in lectus quis neque interdum bibendum. Cras dapibus commodo leo, quis eleifend velit. Vivamus vel erat lobortis diam ornare blandit eget eget nisi. Duis id mauris purus. Vestibulum sagittis felis libero, ac ullamcorper dolor convallis quis. Maecenas et vestibulum augue, non efficitur metus. Pellentesque efficitur libero ut nulla pretium, eu dapibus mi consequat. Sed non tortor volutpat, porta ligula et, consequat sapien. Cras quis metus vitae mi euismod vulputate a non mi.Praesent ornare pretium iaculis. Pellentesque mollis lorem id pretium viverra. Praesent ac nisi scelerisque, ornare arcu id, viverra augue. Maecenas congue, diam vel viverra suscipit, urna felis posuere felis, ac eleifend erat nulla a dui. Sed suscipit vel ex eget rhoncus. Class aptent taciti sociosqu ad litora torquent per conubia nostra, per inceptos himenaeos. Nunc facilisis interdum massa ut feugiat.',
        'people': [
            { 'name': 'John Doe', 'url': 'https://google.com' },
            { 'name': 'Jane Doe' },
            { 'name': 'Marcus Zucko', 'url': 'https://facebok.com' },
        ],
        'embed': [
            '<iframe width="560" height="315" src="https://www.youtube.com/embed/ACmmxe3vPjo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>',
            '<iframe width="560" height="315" src="https://www.youtube.com/embed/ACmmxe3vPjo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>',
        ],
    },
]
```

So what's happening in the data? We added a few new key/value pairs to the dictionary to represent a project:

- The `description`, which is just a giant text.
- The people, which is a list of dictionaries where each entry has at least a `name`, and may have a `url` field.
- Then we also have this `embed` field, which is a list of strings.

Each string in the `embed` list actually looks like some `html` code! They are `<iframe>` tags that I got from the Youtube videos that I made specifically for this project. I got this code directly from the Youtube's website using the share options. You can get the code from any video, just click on share, then the embed option, and Youtube will give you the exact `html` code.

What will happen is that that string will be injected into the final `html` file as is, and once loaded by the browser, it will show the youtube video instead of the `html` characters like `<` and `>`. So if you have demo videos made for a project, this is a way you can showcase it. Visitors to the website won't have to leave the website to watch the video, they can watch it right there.

Ok, ok. If you haven't realized yet, we did not touch the build script. All we did so far doesn't really reflect in the built website if we run the script yet. Let's deal with that next.

## Adding the new page to the build flow

In the `build.py` script we need to iterate over all the research projects from the data store, and then create individual subfolders for each one of them, and then feed the template with the data, and spit out an `index.html` into that subfolder. We will end up with something like this:

```python
"""
Research
"""
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
"""
Research project detail
"""
# /research/*
template = env.get_template('research_detail.html')
for r in research:
    output = template.render(
        general=gen,
            title='Project details',
        project=r
    )
    o = path / Path("research/"+r['slug'])
    o.mkdir(parents=True, exist_ok=True)
    o = o / Path('index.html')
    with o.open(mode='w') as fh:
        fh.write(output)
```

It looks pretty similar to the other pages we generated, the only difference is that we loop over each research project inside the `research` list and create a dedicated subpage (using the `slug` item to name the folder) for each one of those items.

Now you can build and serve and test it out by loading `localhost:8000/research/my-awesome-research/` in the browser (or go to `localhost:8000/research` and click the links!):

```bash
python build.py
python -m http.server --directory ./_site 
```

![The details page]images/Screen_Shot_2021_11_17_at_12_07_47_2a8cc2edd3.png)

## A word on the development workflow

What we did today should feel pretty similar to what we have done so far for each new page in the website. In fact, we're not doing anything different... We do the same thing over and over again: add information to the database, create a template with some placeholder areas, then wire it together in the build script.

The order of events is not mandatory, if you are more comfortable with creating the build script first, and that gives you more clarity on what to do later, great, do that first. Or If you prefer to enter information into the data file first, that's totally fine too. The order you use should make sense to you, and you can even try out different workflows while coding. And use the one that makes more sense to you.

## Summary

In this article we:

- Created a new template for the project details
- Added more data to the each research project in the data store
- Used embedded resources from an external website: Youtube
- Build detail pages for each project using the `slug` property

