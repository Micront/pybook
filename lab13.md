# Лабораторная работа №13. MicroSlack

Эта лабораторная работа посвящена созданию простого клона мессенджера Slack.

```python
from aiohttp import web

app = web.Application()
web.run_app(app, host='127.0.0.1', port=8080)
```

```python
from aiohttp import web

async def index(request):
    return web.Response(text='Index page')

app = web.Application()
app.router.add_route('GET', '/', index)
web.run_app(app, host='127.0.0.1', port=8080)
```


```python
from aiohttp import web
import jinja2
import aiohttp_jinja2

@aiohttp_jinja2.template('index.html')
async def index(request):
    return {'title': 'Index page'}

app = web.Application()
aiohttp_jinja2.setup(app, loader=jinja2.FileSystemLoader('templates'))
app.router.add_get('/', index)
web.run_app(app, host='127.0.0.1', port=8080)
```

```html
<!DOCTYPE html>                                                             
<html>
    <head>
        <title>{{ title }}</title>
    </head>
    <body>
        <h1>Welcome to {{ title }}</h1>
    </body>
</html>
```

```python
from aiohttp import web
import jinja2
import aiohttp_jinja2

from chat.views import Index

async def create_app():
    app = web.Application()
    aiohttp_jinja2.setup(app, loader=jinja2.FileSystemLoader('templates'))
    app.router.add_static('/static', 'static', name='static') # аргумент name используется для того, чтобы мы могли обратиться к app.router.static
    app.router.add_get('/', Index)
    return app

loop = asyncio.get_event_loop()
app = loop.run_until_complete(create_app())
web.run_app(app, host='127.0.0.1', port=8080)
```

```python
from aiohttp import web
import aiohttp_jinja2


class Index(web.View):
    
    @aiohttp_jinja2.template('index.html')
    async def get(self):
        return {'title': 'Index page'}
```


```html
...
<head>
    <title>{{ title }}</title>
    <script src="{{ app.router.static.url(filename='chat.js') }}"></script>
</head>
...
```


```python
from aiohttp import web
import aiohttp_jinja2


class LoginView(web.View):
    # http://aiohttp.readthedocs.io/en/stable/web.html#class-based-views
    
    @aiohttp_jinja2.template('login.html')
    async def get(self):
        if self.request.cookies.get('user'):
            return web.HTTPFound('/')
        return {'title': 'Login'}
    
    async def post(self):
        response = web.HTTPFound('/')
        # http://aiohttp.readthedocs.io/en/stable/web.html#http-forms
        data = await self.request.post()
        response.set_cookie('user', data['name'])
        return response
```


```html
<!DOCTYPE html>
<html>
    <head>
        <title>{{ title }}</title>
        <link rel="stylesheet" href="//oss.maxcdn.com/semantic-ui/2.1.8/semantic.min.css" type="text/css">
    </head>
    <body>
        <div class="ui three column stackable centered grid">                       
            <div class="column">
                <form action="/login" method="post">
                    <div class="ui fluid action input">
                        <input type="text" name="name" placeholder="Username" required>
                        <button type="submit" class="ui primary button">Log in</button>
                    </div>
                </form>
            </div>
        </div>
    </body>
</html>
```

```python
...
from accounts.views import Login
...

async def create_app():
    ...
    app.router.add_route('*', '/login', Login)
    ...
```


```python
from aiohttp import web

async def auth_cookie_factory(app, handler):
    # http://aiohttp.readthedocs.io/en/stable/web.html#middlewares
    async def middleware(request):
        # http://aiohttp.readthedocs.io/en/stable/web_reference.html#aiohttp.web.BaseRequest.path
        # http://aiohttp.readthedocs.io/en/stable/web_reference.html#aiohttp.web.BaseRequest.cookies
        if request.path != '/login' and request.cookies.get('user') is None:
            # http://aiohttp.readthedocs.io/en/stable/web.html#exceptions
            return web.HTTPFound('/login')
        return await handler(request)
    return middleware
```

```python
...
from middlewares import auth_cookie_factory
...

async def create_app():
    app = web.Application(middlewares=[auth_cookie_factory,])
    ...
```

#### Замена кук на сессии:

```python
from aiohttp import web
import jinja2
import aiohttp_jinja2
import aioredis
from aiohttp_session import session_middleware
from aiohttp_session.redis_storage import RedisStorage

from chat.views import Index
from accounts.views import Login
from middlewares import auth_session_factory, request_session_factory

async def create_app():
    async def close_redis(app):
        app.redis_pool.close()
        await app.redis_pool.wait_closed()
    
    redis_pool = await aioredis.create_pool(('127.0.0.1', 6379), loop=loop)
    app = web.Application(middlewares=[
        session_middleware(RedisStorage(redis_pool)),
        request_session_factory,
        auth_session_factory
    ])
    app.redis_pool = redis_pool
    aiohttp_jinja2.setup(app, loader=jinja2.FileSystemLoader('templates'))
    app.router.add_static('/static', 'static', name='static')
    app.router.add_get('/', Index)
    app.router.add_route('*', '/login', Login)
    app.on_shutdown.append(close_redis)
    return app

loop = asyncio.get_event_loop()
app = loop.run_until_complete(create_app())
web.run_app(app, host='127.0.0.1', port=8080)
```


```python
from aiohttp import web
from aiohttp_session import get_session

async def request_session_factory(app, handler):
    async def middleware(request):
        request.session = await get_session(request)
        return await handler(request)
    return middleware

async def auth_session_factory(app, handler):
    async def middleware(request):
        if request.path != '/login' and request.session.get('user') is None:
            return web.HTTPFound('/login')
        return await handler(request)
    return middleware
```

```python
...
from aiohttp_session import get_session

class Login(web.View):

    @aiohttp_jinja2.template('login.html')
    async def get(self):
        if self.request.session.get('user'):
        ...
    
    async def post(self):
        ...
        self.request.session['user'] = data['name']
        ...
```
