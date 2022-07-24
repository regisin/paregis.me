---
title: "FastAPI for frontend development with hot reload"
date: 2022-07-31T00:00:01-05:00
draft: true
description: "How to have hot reload when using Jinja templates with FastAPI."
categories:
- Tutorial
---

## Introduction

This is a relatively simple and opinionated way of doing this. It's also the only way so far that I know how to do this.

### Why am I writing about this?

### What is hot module reload? How does it work?
Check svelte/read/vue/etc.

In practice, client opens websocket to server, refresh if websocket disconnects.

### What is Jinja?

### What is FastAPI?



## Storytime

I can't cover the beginning of time, so I assume you have `python` installed. I used `3.10`, but could work with `3.6` depending which FastAPI version you use.

1. Create a new project environment, mine is called `potato`. I like to keep my projects isolated and independent, so I use python's [Virtual Environments](https://docs.python.org/3/tutorial/venv.html) (I'm using Mac so the commands might be slightly different).

    ```bash
    ~ % mkdir potato
    ~ % cd potato
    potato % python3.10 -m venv env
    potato % source env/bin/activate
    (env) potato %
    ```

    Notice the `(env)` in front of the command line, that's how you know you are in the virtual environment.

1. Install dependencies. This will all exist only in this folder because we are using `venv`:

    ```bash
    (env) potato % pip install "fastapi[all]"
    (env) potato % pip install jinja2
    ```

1. Create the simplest FastAPI app in a `main.py`:

    ```python
    from fastapi import FastAPI
    from fastapi.responses import HTMLResponse
    from fastapi.templating import Jinja2Templates

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
                                                    "title":"Potato"})
    ```

    It's a very simple app with only one route: `localhost:8000/`. There are no openapi docs because we are not creating an API, we are being naughty and using FastAPI for a frontend workflow!

    We will modify this script a little bit later. For now, notice the `Jinja2Templates`. It points to a directory that doesn't exist, and is also empty (??).

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
                                            "title":"Potato"})
    ```

    We will use the other placeholder (`{{ reloader|safe }}`) for our hot module reloader.

1. Now you can run your FastAPI app with uvicorn with `(env) potato % uvicorn main:app --reload`.

    > **Great! We're done, right?**
    
    Not so fast! That `--reload` flag in the command will trigger the Uvicorn *server* to reload, it has nothing to do with the client. If you don't believe me, try changing `"title":"Potato"` to `"title":"Tomato"`.

1. Tell uvicorn to watch template files too. By default, uvicorn will trigger a reload whenever a `.py` file is changed in the project directories. To make it also reload when `.html` files are edited, run the server like this: `(env) potato % uvicorn main:app --reload --reload-include '*.html'`. Now try modifying `templates/Home.html` and save itand watch the console where the server is running reload.

    Awesome, now whenever we modify a `.py` OR `.html` files, the server will take notice and reload. However, if you had your browser opened on the dev server `http://localhost:8000`, you will not see the changes unless you refresh it.
    
    That's kind of annoying... sure I can hit `F5` all the time, but it gets old fast! We need to trigger that same reload to happen in the browser.

1. How hot module reload works? I got the inspiration from (this website)[https://www.bscotch.net/post/create-a-live-reload-server].
    
    1. The idea is to create an endpoint on the server that listens to the WebSocket protocol `ws://`.
    1. Then the client will have a tiny bit of JavaScript that opens a connection to the web socket.
    1. If the client detects that the server disconnected, it tries to reloads the page if reconnected successfully.
    1. If it cannot connect to the web socket server, give up after a few tries.

1. Create the Web Socket endpoint.

    Let's modify the FastAPI and add a new endpoint, but this time a WebSocket endpoint:

    ```python
    from fastapi import FastAPI, WebSocket, WebSocketDisconnect
    from fastapi.responses import HTMLResponse
    from fastapi.templating import Jinja2Templates

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
                                                    "title":"Potato"})

    @app.websocket("/ws")
    async def websocket_endpoint(websocket: WebSocket):
        try:
            await websocket.accept()
            while True:
                pass
        except WebSocketDisconnect:
            pass
    ```

    It's really just that, the endpoint only exists to open and close the connection, no messages are passed in this web socket. We also `except` the `WebSocketDisconnect` to keep the console clean. Otherwise, everytime the server reloads that exception will be raised and the stack will pollute the screen. Every. Single. Time.

1. Inject the template with some JavaScript to connect to the websocket:

    Remember that piece of the template that we were not using? The `{{ reloader|safe }}`? We will use it now!

<script>
(() => {
    const socketUrl = "ws://localhost:8000/ws";
    var ws = new WebSocket(socketUrl);
    /*
     * Hot Module Reload
     */
    ws.addEventListener('close',() => {
        console.log('[WS:close]', 'HMR websocket closed.');

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

## Limitations
Doesn't keep state.

## Summary