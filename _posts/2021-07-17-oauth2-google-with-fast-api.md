---
layout: post
title: Oauth with Python FastAPI
date: '2021-07-29'
categories: posts
published: true
tags: [programing, python, fastapi]
---

Recently I was exploring FastApi micro-framework of Python. There are many interesting features that are found in FastApi. While I was exploring FastApi in on of my project, I was trying to integrate Google Oauth 
in my project. 

> Fast API is a modern, open-source, fast, and highly performant Python web framework used for building Web APIs and is based on Python 3.6+ standard type hints.

I don't find much documentation on Google Oauth2 login docs compared with other framework. So decided to share my experience here.

### FastApi setup

```python
from typing import Optional
from fastapi import FastAPI, Request
```

The Optional and FastAPI imports are standard among FastAPI applications. `Optional` is for type supporting in python. It is used for validating user's request before entering Api.


```python
@app.get('/')
async def home(request: Request):
    user = request.session.get('user')
    if user is not None:
        return {id: user.id}
    return {"error": "Un-authorized"}

```

### Implementing Google Auth in FastAPi

FastApi is created on top of Starlette. A FastAPI app is basically a Starlette app, that is why you can just use Authlib Starlette integration to create OAuth clients for FastAPI.

```python
from authlib.integrations.starlette_client import OAuth
from starlette.config import Config

config = Config('.env')  # read config from .env file
oauth = OAuth(config)
oauth.register(
    name='google',
    server_metadata_url='https://accounts.google.com/.well-known/openid-configuration',
    client_kwargs={
        'scope': 'openid email profile'
    }
)
```

Above code reads client_secret and client_id from configuration file. If you want to do manually via code, please refer code.

```python
oauth.register(
    'google',
    client_id='...',
    client_secret='...',
    ...
)
```

Above are few ways to pass client_id and client_secret. Instead I would prefer to read secrets from environment file instead.

```python
from starlette.config import environ
oauth.register(
    'google',
    client_id=environ['CLIENT_ID'],
    client_secret=environ['CLIENT_SECRET'],
    ...
)
```

#### Final code for initializing O-auth

```python
from authlib.integrations.starlette_client import OAuth
from starlette.config import environ
oauth = OAuth()
oauth.register(
    name='google',
    server_metadata_url='https://accounts.google.com/.well-known/openid-configuration',
    client_id=environ['CLIENT_ID'],
    client_secret=environ['CLIENT_SECRET'],
    client_kwargs={
        'scope': 'user:email'
    }
)
```

However, unlike Flask/Django, Starlette OAuth registry is using [HTTPX](https://github.com/encode/httpx), So as part of this setup, please make sure `httpx` is installed

> httpx is a fast and multi-purpose HTTP toolkit allow to run multiple probers using retryablehttp library, it is designed to maintain the result reliability with increased threads.

```bash
pip install httpx
```

### Creating routes for Google login and Google callback url

```python
@app.route('/login/google')
async def login_via_google(request):
    google = oauth.create_client('google')
    redirect_uri = request.url_for('authorize_google')
    return await google.authorize_redirect(request, redirect_uri)

@app.route('/auth/google')
async def authorize_google(request):
    google = oauth.create_client('google')
    token = await google.authorize_access_token(request)
    user = await google.parse_id_token(request, token)
    print(user)
    #set session or do something with the token and profile
    return {}

```

###  Using OAuth 2.0 to Access Google APIs

Go to google API console, Fetch credentials and save it in your project.
<div style="width:500px; margin: 10px auto"><img src="/img/blogs/fast-api/google-console.png" /></div>

References:
1. [Authlib](https://docs.authlib.org/en/latest/client/oauth2.html)
2. [Authlib blog](https://blog.authlib.org/2020/fastapi-google-login)
