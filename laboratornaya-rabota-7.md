# Персонализация новостной ленты

В этой работе вашей задачей является написать простой персонализированный новостной агрегатор.

Для выполнения этого задания вам потребуется собирать и размечать новости из одного или нескольких источников. В качестве поясняющего примера будем использовать социально-новостной сайт [Hacker News](https://news.ycombinator.com).

### Сбор данных

В предыдущих работах вы уже сталкивались с модулем `requests`, который позволяет получать доступ к веб-страницам. Вспомним, что есть два распространенных способа доступа к веб-страницам: запрос типа `GET` и запрос типа `POST` (в действительности видов http-запросов гораздо [больше](https://ru.wikipedia.org/wiki/HTTP#.D0.9C.D0.B5.D1.82.D0.BE.D0.B4.D1.8B)). Запрос типа `GET` - это когда вы передаете серверу какую-то информацию в адресной строке. Например, если вы перейдете по такому адресу: https://translate.google.com/?hl=ru#en/ru/python, то этим вы просите сервис Google Translate перевести слово "python" с английского на русский (параметры запроса указываются после символа `?`). `POST`-запрос - это когда вам нужно ввести информацию в какую-нибудь форму, например, ввести логин-пароль, который не будет отображать в адресной строке браузера. 

Мы пока будем использовать только `GET`-запросы.

Давайте выполним два разных `GET`-запроса к новостному сайту: 

```python
>>> import requests
>>> r = requests.get("https://news.ycombinator.com/newest")
>>> r.ok
True
>>> r.status_code
200
>>> r = requests.get("https://news.ycombinator.com/abrakadabra")
>>> r.ok
False
>>> r.status_code
404 
```

<div class="alert alert-info">
<b>Замечание:</b> Если у вас не установлен модуль <code>requests</code>, то вы можете установить его командой <code>pip install requests</code> или <code>python -m pip install requests</code>.
</div>

Первый запрос был выполнен успешно, о чем говорит значение `r.ok` и `r.status_code`. Второй запрос был выполнен к несуществующей странице, что привело к ошибке 404 - "Страница не найдена".

Доступ к содержимому страницы можно получить с помощью атрибута `text` (для примера выведено 100 первых символов):



```python
>>> r.text[:100]
'<html op="newest"><head><meta name="referrer" content="origin"><meta name="viewport" content="width='
```

Как вы видите это простая HTML-страница, из которой нам нужно извлечь интересующую нас информацию, а именно:
- заголовок новости;
- автора новости;
- ссылку на новость;
- количество комментариев;
- количество "лайков", которое набрала статья.

Например, в следующей новости:
![](/assets/Screen Shot 2017-01-28 at 21.33.29.png)

- **заголовок** - Show HN: Pydb – a lightweight database with Python syntax queries, using ZeroMQ;
- **автор** - asrp;
- **ссылка** - https://github.com;
- **количество комментариев** - 11;
- **количество "лайков"** - 63.

Для извлечения данных с веб-страниц есть множество разных модулей. Проблема с HTML в том, что большинство браузеров ведет себя "прощающе" (в чем вы могли убедиться выполняя лабораторную работу №5), и поэтому в вебе много плохо-написанных (не по стандарту HTML) HTML-страниц. Впрочем, обработка даже не вполне корректного HTML-кода не так сложна, если под рукой есть подходящие инструменты. Мы будем пользоваться модулем Beautiful Soup 4.

<div class="alert alert-info">
<b>Замечание</b>: Если у вас не установлен модуль <code>bs4</code>, то вы можете установить его командой <code>pip install bs4</code> или <code>python -m pip install bs4</code>.
</div>

Чтобы использовать Beautiful Soup, нужно передать функции `BeautifulSoup` текст веб-страницы (в виде одной строки). Чтобы он не "ругался", также следует указывать название парсера (той программы, которая осуществляет обработку HTML) — с целью совместимости я использую `html.parser` (он входит в поставку Python и не требует установки), но вы можете также попробовать использовать `html5lib`, если он у вас установлен.

```python
>>> from bs4 import BeautifulSoup
>>> page = BeautifulSoup(r.text, 'html.parser')
>>> page
<html op="newest"><head><meta content="origin" name="referrer"><meta content="width=device-width, initial-scal
e=1.0" name="viewport"><link href="news.css?5kjS59ufyw5qyqpjcavc" rel="stylesheet" type="text/css">
<link href="favicon.ico" rel="shortcut icon">
...
```

Перменная `page` представялет собой не просто содержимое HTML-страницы, это объект, который позволяет выполнять запросы. Например, мы можем обратиться к тегу `head`, а внутри него к тегу `title`:

```python
>>> page.head.title
<title>New Links | Hacker News</title>
>>> page.head.title.text
'New Links | Hacker News'
```

Для того, чтобы лучше понять структуру HTML-страницы следует воспользоваться веб-инспектором, который есть в большинстве современных браузеров.

![](/assets/Screen Shot 2017-01-28 at 21.13.57.png)

Если вы посмотрите на структуру HTML-страницы, то сможете заметить, что есть внешняя таблица, которая включает в себя еще три таблицы: заголовок, новостную ленту (которая в свою очередь также состоит из множества строк) и подложку (см. рисунок ниже).

![](/assets/Screen Shot 2017-01-28 at 21.57.13.png)

Возникает вопрос "_Как обратиться к внутренним таблицам?_". Если мы дважды обратимся к атрибуту `table`, то получим заголовок:

```python
>>> page.table.table
<table border="0" cellpadding="0" cellspacing="0" style="padding:2px" width="100%"><tr><td style="width:18px;p
adding-right:4px"><a href="http://www.ycombinator.com"><img height="18" src="y18.gif" style="border:1px white 
solid;" width="18"/></a></td>
<td style="line-height:12pt; height:10px;"><span class="pagetop"><b class="hnname"><a href="news">Hacker News<
/a></b>
<span class="topsel"><a href="newest">new</a></span> | <a href="newcomments">comments</a> | <a href="show">sho
w</a> | <a href="ask">ask</a> | <a href="jobs">jobs</a> | <a href="submit">submit</a> </span></td><td style="t
ext-align:right;padding-right:4px;"><span class="pagetop">
<a href="login?goto=newest">login</a>
</span></td>
</tr></table>
```

У объекта `page` кроме атрибутов есть функции, одной из которых является `findAll` и позволяет найти несколько элементов с одинаковыми тегами:

```python
>>> tbl_list = page.table.findAll('table')
>>> len(tbl_list)
3
```

Соответственно, нулевой элемент списка `tbl_list` это таблица, которая содержит заголовок, первый элемент списка это таблица с новостями и второй элемент списка это подложка.

На текущий момент вашей задачей является написать функцию `get_news`, которая в качестве аргумента принимает HTML-страницу, а возвращает список словарей, где каждый словарь представляет собой запись об одной новости (пример вывода смотри ниже):

```python
>>> news_list = get_news(r.text)
>>> pp(news_list[:3])
[{'author': 'evo_9',
  'comments': 0,
  'points': 1,
  'title': 'Daily Action – Sign Up to Join the Resistance',
  'url': 'https://dailyaction.org/'},
 {'author': 'azuajef',
  'comments': 0,
  'points': 1,
  'title': 'Immigration Ban Blocks Travelers at Airports Around Globe',
  'url': 'https://www.nytimes.com/2017/01/28/us/refugees-detained-at-us-airports-prompting-legal-challenges-to
-trumps-immigration-order.html?_r=0'},
 {'author': 'ColinCochrane',
  'comments': 0,
  'points': 7,
  'title': 'Green card holders included in Trump ban: Homeland Security',
  'url': 'http://mobile.reuters.com/article/idUSKBN15C0KX'}]
```

### Сохранение данных в sqlite

В процессе сбора данных их нужно где-то хранить. Мы будем использовать для хранения [SQLite](https://ru.wikipedia.org/wiki/SQLite) - компактная встраиваемая реляционная база данных. В стандартной библиотеке языка Python есть модуль [sqlite3](https://docs.python.org/3/library/sqlite3.html), который предоставляет интерфейс для работы с SQLite. Этот модуль требует знания языка SQL, поэтому мы воспользуемся другой технологией, которая называется ORM.

ORM (англ. object-relational mapping, рус. объектно-реляционное отображение) — технология программирования, которая связывает базы данных с концепциями объектно-ориентированных языков программирования, создавая "виртуальную объектную базу данных". Существуют как проприетарные, так и свободные реализации этой технологии.

SQLAlchemy — это библиотека на языке Python для работы с реляционными СУБД с применением технологии ORM. Служит для синхронизации объектов Python и записей реляционной базы данных. SQLAlchemy позволяет описывать структуры баз данных и способы взаимодействия с ними на языке Python без использования SQL.

Каждая таблица описывается классом, который должен наследоваться от базового класса, создаваемого при помощи функции `sqlalchemy.ext.declarative.declarative_base()`. В рассматриваемом нами примере будет только один класс - `News`, с атрибутами: заголовок, автор, ссылка, количество комментариев и число "лайков".

```python
from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base()

from sqlalchemy import Column, String, Integer
class News(Base):
    __tablename__ = "news"
    id = Column(Integer, primary_key = True)
    title = Column(String)
    author = Column(String)
    url = Column(String)
    comments = Column(Integer)
    points = Column(Integer)

from sqlalchemy import create_engine
engine = create_engine("sqlite:///news.db")
Base.metadata.create_all(bind=engine)

from sqlalchemy.orm import sessionmaker
session = sessionmaker(bind=engine)
s = session()
```

Функция `sqlalchemy.create_engine()` создает новый экземпляр класса `sqlalchemy.engine.Engine`, который отвечает за подключение к базе данных.

Что касается создаваемой сессии, то вот небольшая выдержка о назначении сессиий из официальной документации:
> In the most general sense, the Session establishes all conversations with the database and represents a “holding zone” for all the objects which you’ve loaded or associated with it during its lifespan. It provides the entrypoint to acquire a Query object, which sends queries to the database using the Session object’s current database connection, populating result rows into objects that are then stored in the Session, inside a structure called the Identity Map - a data structure that maintains unique copies of each object, where “unique” means “only one object with a particular primary key”.

> The Session begins in an essentially stateless form. Once queries are issued or other objects are persisted with it, it requests a connection resource from an Engine that is associated either with the Session itself or with the mapped Table objects being operated upon. This connection represents an ongoing transaction, which remains in effect until the Session is instructed to commit or roll back its pending state.

> All changes to objects maintained by a Session are tracked - before the database is queried again or before the current transaction is committed, it flushes all pending changes to the database. This is known as the Unit of Work pattern.

> When using a Session, it’s important to note that the objects which are associated with it are proxy objects to the transaction being held by the Session - there are a variety of events that will cause objects to re-access the database in order to keep synchronized. It is possible to “detach” objects from a Session, and to continue using them, though this practice has its caveats. It’s intended that usually, you’d re-associate detached objects with another Session when you want to work with them again, so that they can resume their normal task of representing database state.

Более подробно о сессиях можно прочитать [тут](http://docs.sqlalchemy.org/en/latest/orm/session_basics.html).

Далее приведен пример создания объекта и сохранения его в БД:
```python
>>> news = News(title='Lab 7', 
                author='dementiy',
                url='https://dementiy.gitbooks.io/-python/content/lab7.html',
                comments=0,
                points=0)
            
>>> news.id, news.title
(None, Lab 7)
>>> s.add(news)
>>> s.commit()
>>> news.id, news.title
(1, Lab 7)
```

Обратите внимание, что идентификатор объекта (`id`) содержит значение `None` до тех пор, пока мы не сделаем коммит в БД с помощью метода `commit()`.

На текущий момент вашей задачей является сохранить все записи с новостного сайта в БД. Просмотреть содержимое файла `news.db` можно с помощью программы [DB Browser for SQLite](http://sqlitebrowser.org).

![](/assets/Screen Shot 2017-02-01 at 19.34.44.png)

### Разметка данных

Как разметить имеющиеся данные? Давайте создадим простую HTML-страницу, на которой будем выводить список не размеченных новостей, а рядом с каждой новостью будет несколько кнопок со следующими метками:
- "_Интересно_" эта новость вам показалась интересной и вы ее прочитали;
- "_Не интересно_" - эта новость вас не интересует;
- "_Возможно прочитаю_" - вы сомневаетесь интересна вам эта новость или нет.

<div class="alert alert-info">
Меток может быть больше, кроме того мы можем использовать численное значение для оценки нашего отношения к новости, например, от 0 до 5.
</div>

Нажатие на кнопку должно обрабатываться в виде добавления метки в БД к соответствующей новости и удаления новости из списка новостей, так как мы ее уже разметили.

Для создания такой веб-страницы воспользуемся простым и популярным веб-фреймворком - [bottle](https://bottlepy.org/docs/dev/).

Давайте рассмотрим простой пример из документации к этому фреймворку:

```python
from bottle import route, run, template

@route('/')
@route('/hello/<name>')
def index(name="Stranger"):
    return template('hello_template', name=name)

run(host='localhost', port=8080)
```

```html
<!-- hello_template.tpl -->
<b>Hello {{name}}</b>
```

Функция `run` запускает веб-сервер по адресу `localhost` и порту `8080` (см. скриншоты ниже) и обрабатывает запросы до тех пор, пока вы его не остановите (нажатием `Ctrl+c`).

![](/assets/Screen Shot 2017-02-04 at 02.02.18.png)

![](/assets/Screen Shot 2017-02-04 at 01.53.54.png)

`route` это функция-декоратор, которая отвечает за маршрутизацию и связывает адрес ресурса (веб-страницы) с функцией, которая должна быть вызвана при обращении к этому ресурсу. В нашем примере функция `index` отвечает за два разных маршрута: `/` и `/hello/<name>`. Второй маршрут является динамическим, так как он соответствует не одному ресурсу, а целому множеству разных ресурсов: `/hello/dementiy`, `/hello/alice`, `/hello/bob` и т.д. Шаблон (wildcard) заключается в угловые скобки `<` и `>` и имя аргумента функции должно совпадать с именем указанным в шаблоне (в нашем случае это `name`).

В `bottle` реализован механимз шаблонов, предназначенный для генерации веб-страниц. Чтобы начать работу с шаблонизатором достаточно воспользоваться функцией `template`, которая в качестве первого аргумента принимает имя файла, в котором содержится текст шаблона (в нашем случае это `hello_template.tpl`). Затем следует список необязательных именованных аргументов, которые нужно передать шаблонизатору (в нашем примере это `name`).

Вернемся к нашему примеру. Напишем функцию, которая будет отвечать за маршрут `news` и выводить список неразмеченных новостей.


```python
@route('/news')
def news_list():
    s = session()
    rows = s.query(News).filter(News.label == None).all()
    return template('news_template', rows=rows)
```

```hmtl
<!-- news_template.tpl -->
<table border=1>
    <tr>
        <th>Title</th>
        <th>Author</th>
        <th>#likes</th>
        <th>#comments</th>
        <th colspan="3">Label</th>
    </tr>
    %for row in rows:
        <tr>
            <td><a href="{{row.url}}">{{row.title}}</a></td>
            <td>{{row.author}}</td>
            <td>{{row.points}}</td>
            <td>{{row.comments}}</td>
            <td><a href="/add_label/?label=good&id={{row.id}}">Интересно</a></td>
            <td><a href="/add_label/?label=maybe&id={{row.id}}">Возможно</a></td>
            <td><a href="/add_label/?label=never&id={{row.id}}">Не интересно</a></td>
        </tr>
    %end
</table>
```

### Классификация данных

