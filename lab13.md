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

@aiohttp_jinja2.template('index.html')
async def index(request):
    return {'title': 'Index page'}

app = web.Application()
aiohttp_jinja2.setup(app, loader=jinja2.FileSystemLoader('templates'))
app.router.add_static('/static', 'static', name='static') # аргумент name используется для того, чтобы мы могли обратиться к app.router.static
app.router.add_get('/', index)
web.run_app(app, host='127.0.0.1', port=8080)
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
import jinja2
import aiohttp_jinja2

@aiohttp_jinja2.template('index.html')
async def index(request):
    return {'title': 'Index page'}


async def ws_handler(request):
    ws = web.WebSocketResponse()
    await ws.prepare(request)
    
    async for msg in ws:
        if msg.tp == web.MsgType.text:
            ws.send_str('server response')
        elif msg.tp == web.MsgType.error:
            print('connection closed with exception')
    
    await ws.close()
    print('websocket connection closed')
    return ws


app = web.Application()
aiohttp_jinja2.setup(app, loader=jinja2.FileSystemLoader('templates'))
app.router.add_static('/static', 'static', name='static')
app.router.add_get('/', index)
app.router.add_get('/ws', ws_handler)
web.run_app(app, host='127.0.0.1', port=8080)
```

```js
var socket = new WebSocket('ws://' + window.location.host + '/ws');

socket.onopen = function() {
    console.log("Соединение установлено");
    socket.send("Данные от клиента");
};

socket.onclose = function(event) {
    if (event.wasClean)
        console.log("Соединение закрыто чисто");
    else
        console.log("Обрыв соединения");
    console.log("Код: " + event.code + ", причина: " + event.reason);
    console.log(event)
};

socket.onmessage = function(event) {
    console.log("Получены данные " + event.data);
};

socket.onerror = function(error) {
    console.log("Ошибка " + error);
};
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
        return {'title': 'Authentication'}
    
    async def post(self):
        response = web.HTTPFound('/')
        # http://aiohttp.readthedocs.io/en/stable/web.html#http-forms
        data = await self.request.post()
        response.set_cookie('user', data['name'])
        return response
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