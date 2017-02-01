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

### Разметка данных


### Классификация данных

