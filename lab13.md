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

