---
title: "FastAPI for frontend development with hot reload"
date: 2022-07-25T00:00:01-05:00
draft: false
description: "How to have hot reload when using Jinja templates with FastAPI."
categories:
- Tutorial
---

**TL;DR**: In this post, I will be implementing a hot reload mechanism do I can use FastAPI for both front and backend development. Example code [repo here](https://github.com/regisin/fastapi-hmr).

## Introduction

I'm a big fa of some frameworks out there. I like Svelte and [sveltekit](https://kit.svelte.dev/), I've used [Strapi](https://strapi.io/) for this website's backend for a couple of years, now I'm using Hugo to simplify my workflow. I have also been tinkering with [FastAPI](https://fastapi.tiangolo.com/), yet another framework, this time for backend. Each and every one of those have a purpose and try to solve some sort of problem.

While they do what they are meant to be, for me personally, the amount of different techs in a stack can get overwhelming. Yes I like using sveltekit for my frontend. Yes I like having a nice CMS in the backend. But sometimes all of these get in the way of being productive and prevent me from actually putting ideas to practice! That's what I'm trying to fix, little by little.

The web, the http protocol specifically, is basically the same as it was back i the day. It trasfers text files from a server (backend) to the client (browser). Those files are then interpreted by the client and displayed on the screen. All of that to say that I don't need a fancy frontend framework such as React/Vue/(Thor forbid)Angular/Svelte... I can just generate html files and send them back to the client! The client doesn't even need to do a single API call, just display the html. That's one of the reasons I will be using FastAPI. The main goal is to develop an entire website using a minimal stack. If I can do it all using python, awesome!

*Gosh, that's a long text that could've stayed in my private notes or a journal. I got my eyes on you, pyscript!*

In short, FastAPI is a python framework that makes it easy to implement web APIs. It's documentation (I think it's more of a guided tutorial) is fantastic. I can't possibly cover all of it, you should check it out if you're not familiar with it but are still reading this text. :)

While FastAPI is great for what it does, it is NOT a frontend stack. However, it does support a very cool templating engine, [Jinja2](https://jinja.palletsprojects.com/en/3.1.x/). With jinja I can essentially create a bunch of html templates, load it in python using FaspAPI, and inject data into it. The templates then form a valid html file that can be sent back to the client that made a request to the FastAPI backend.

One issue though, and perhaps something that we take for granted when we start developing a web app in a modern framework, is that while we develop and hit *save*, our browser automatically updates with the new changes. That's not really the case with FastAPI+Jinja. While FastAPI does have a *reload* option (if you're using uvicorn), it does NOT automatically refresh the browser tab that you have opened automatically! This behavior is called hot reload, or hot module reload, or something similar. It might not be a problem at first, but again, we kind of take if for granted with modern frameworks.

## Solution design

I'm not reinventing the wheel here. In fact, the solution already exists, just no one wrote a blog that uses FastAPI. I found the solution at [Create a live-reload server for front-end development](https://www.bscotch.net/post/create-a-live-reload-server), where the backend is using NodeJS.

The idea is pretty straight forward: the dev web server, whiule running, listens to WebSocket on a specific endpoint. The client, when loaded by the browser, will connect to that WS endpoint (using some minimal JavaScript). When the server reloads, it will close all conections. The client, when detecting that a WS connection was lost, will try to reconnect for a few times: if it reconnects, means the server reloaded successfully, if not, time out after a a few tries. If the connection is reestablished, then the JavaScript in the client will trigger a page reload (same as you hitting F5 after every update to the server).

    > Note: this thing is only for development purposes. I would remove it from production!

Enough talking!

## Morph(Cod)ing time

I can't do a better job at teaching you about FastAPI than [its official documentation](fastapi.tiangolo.com/). I'm using `3.10`, but it could work with `3.6` depending which FastAPI version you use. I'm also doing this on a macOS, so adapt the commands accordingly.

1. New project, I like to use [Virtual Environments](https://docs.python.org/3/tutorial/venv.html) for each new python project.

    ```bash
    ~ % mkdir potato
    ~ % cd potato
    potato % python3.10 -m venv env
    potato % source env/bin/activate
    (env) potato %
    ```

    Notice the `(env)` in front of the command line, that's how you know you are in the virtual environment.

1. Install dependencies. This will all exist only in this folder because we are using `venv`. `Jinja2` is included if you install `fastapi[all]`:

    ```bash
    (env) potato % pip install "fastapi[all]"
    ```

1. Create the FastAPI app, mine is in `main.py`:

    ```python
    from fastapi import FastAPI, Request, WebSocket, WebSocketDisconnect
    from fastapi.responses import HTMLResponse
    from fastapi.templating import Jinja2Templates

    from utils import reloader

    templates = Jinja2Templates(directory="templates")

    app = FastAPI(
        title='Potato',
        openapi_url=None,
        docs_url=None,
        redoc_url=None,
    )

    @app.get("/", status_code=200, response_class=HTMLResponse)
    def home(request: Request):
        return views.TemplateResponse('Home.html', {"request": request,
                                                    "reloader": reloader,
                                                    "title": "Tomato"})

    @app.websocket("/ws")
    async def websocket_endpoint(websocket: WebSocket):
        try:
            await websocket.accept()
            data = await websocket.receive_text()
        except WebSocketDisconnect:
            pass
    ```

    The app has two endpoints:
    * `/`: which in this case will return a populated jinja template. This will be accessed through `http://localhost:8000/`
    * `/ws`: the websocket that will restart everytime the server reloads. It's endpoint is `ws://localhost:8000/ws`.

    The home endpoint `/` will read a template from the server (in the `/templates` directory, we will get to the template itself next), pass a couple of data to it (the `title` and `reloader`, more on the reloader later), and return the final html back to the client when requested.

    The WS endpoint has no real use except to exists! It doesn't transfer any sort of data, it reeally is just so the client is able to notice when the server reloads (aka WS connection is lost). We are also cathing (excepting) the `WebSocketDisconnect` because if not, everytime we reload the server, the terminal that is running will be polluted with a trace stack.

1. Create the Jinja template in the `templates` directory:

    ```bash
    mkdir templates
    touch templates/Home.html
    ```

    Now open the `templates/Home.html` and add the following:

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <title>{{ title }}</title>
    </head>
    <body>
        <h1>Tomatoes are fun!</h1>
        {{ reloader|safe }}
    </body>
    </html>
    ```

    This is a Jinja template. The `{{ title }}` will be replaced by the parameters we pass with the FastAPI app from before:

    ```python
    return views.TemplateResponse('Home.html', {"request": request,
                                            "reloader": reloader,
                                            "title": "Potato"})
    ```

    Same with the `{{ reloader|safe }}`, but this will be a JavaScript snippet that we still need to implement. `safe` here is to tell jinja that this variable is safe to render. Which it is, but you woulkdn't if you didn't know what was the contents of `reloader`.



1. The `reloader` script:

    ```html
    <script>
    (() => {
        const socketUrl = "ws://localhost:8000/ws";
        var ws = new WebSocket(socketUrl);
        /*
        * Hot Module Reload
        */
        ws.addEventListener('close',() => {
            const interAttemptTimeoutMilliseconds = 100;
            const maxAttempts = 5;
            let attempts = 0;
            const reloadIfCanConnect = () => {
                attempts++ ;
                if(attempts > maxAttempts){
                    console.error('[WS:error]', 'HMR could not reconnect to dev server.');
                    return;
                }
                socket = new WebSocket(socketUrl);
                socket.addEventListener('error',()=>{
                    setTimeout(reloadIfCanConnect,interAttemptTimeoutMilliseconds);
                });
                socket.addEventListener('open',() => {
                    location.reload();
                });
            };
            reloadIfCanConnect();
        });
    })();
    </script>
    ```

    This snippet is modified from [Create a live-reload server for front-end development](https://www.bscotch.net/post/create-a-live-reload-server). This will be injected in our html template before sending it to the browser.

    Once the browser receives the html with that script, it will execute that self calling function. It will open a websocket connection to our websocket endpoint. When the connection closes, it will wait a few (`100`) millisecconds and try to reconnect. Repeat up to `5` times. If reconnected successfully, refresh the page with `location.reload()`, aka "hot" reload the page.

    To make things easier, wrap that in a python string. Mine is in `utils.py`:

    ```python
    reloader = """
    <script>
    (() => {
        const socketUrl = "ws://localhost:8000/ws";
        /* ... rest of code here */
    </script>
    """
    ```

    So I can import in my FastAPI app (`from utils import reloader`).

1. One last thing. Before you start the uvicorn server again, we need to tell it to watch for changes in `.html` files. By default it will only reload when it detects changes in `.py` files. It's an easy fix though, simply add a flag to the command line like this:

    ```bash
    (env) potato % uvicorn main:app --reload --reload-include '*.html'
    ```

That's it! Now with the development server running, head on to your browser `http://localhost:8000/`, and try saving some change to the source files. For example:

- Change the prop in the `main.py`: from `"title":"Tomato"` to `"title":"Potato"`.
- Change the `templates/Home.html` template: from `<h1>Tomatoes are fun!</h1>` to `<h1>Potatoes are fun!</h1>`.

## Conclusion

This is a very rudimentary hot module reload, and should only be used in dev, not in production. There's no complex state management on the frontend, in which case you should move to a mature frontend framework like React.

All in all, this thing really helps me test out changes in html using only python.