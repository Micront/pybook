# Заметки по ООП в Python

Это краткий конспект по классам, который не является исчерпывающим руководством. Также я не даю определений таким понятиям как ООП, класс, объект и т.п. Все эти определения вы можете легко найти в сети.

### Процедурный подход

Представьте, что вы пишите веб-сервис и перед вами встала следующая задача: "_Как представить пользователя системы в программе?_". Давайте договоримся, что у каждого пользователя должны быть следующие атрибуты:

* имя \(username\)
* адрес электронной почты \(email\)
* пароль \(password\)

Можно представить пользователя в виде нескольких переменных:

```py
username = 'bob'
email = 'bob@example.com'
password = 'foobar'
```

У такого подхода есть несколько недостатков:

1. Мы пониманием, что переменные логически должны быть связаны между собой, но мы эту связь никак не показали. Другими словами, переменная `username`  может содержать имя одного пользователя, `email` относиться ко второму пользователю, а `password` к третьему.
2. Если нам необходимо одновременно взаимодействовать с несколькими пользователями, то для каждого из них нужно создавать по три таких переменных.

Как показать, что существует логическая связь между этими переменными? Мы можем использовать любую подходящую структуру: список, словарь, кортеж, именованный кортеж. Как пример будем использовать словарь \(именованный кортеж, т.е. [`namedtuple`](https://www.blog.pythonlibrary.org/2016/03/15/python-201-namedtuple/), было бы использовать нечестно\):

```py
# Представление отдельно взятого пользователя
user = {
    'username': 'bob', 
    'email': 'bob@example.com',
    'password': 'foobar'
} 

# А так мы теперь можем представить список пользователей
users_list = [
    {'username': 'bob', 'email': 'bob@example.com', 'password': 'foobar'},
    {'username':'joe', 'email': 'joe@example.com', 'password': 'barfoo'},
]
```

Таким образом, объединив несколько значений \(имя, адрес электронной почты и пароль\) в контейнер, мы попытались показать, что существует логическая связь между этими значениями.

Теперь напишем простую функцию, которая возвращает имя пользователя:

```py
def get_username(user):
    return user['username']


>>> get_username(user)
'bob'
```

Так как пользователь представлен словарем, то мы возвращаем значение по ключу `username`, которое соответствует имени.

Одним из недостатков такого подхода является то, что мы не показали логическую связь между данными о пользователе и той функцией, которая должна с ними работать.

В решении этой проблемы нам могут помочь классы, задача которых объединить данные и методы работы с ними.

### Создание простого класса

Начнем с создания простого класса:

```py
class User:
    pass
```

Теперь создадим новый объект класса пользователь:

```py
>>> u = User()
```

Атрибуты хранятся в специальном словаре \(подробнее про модель данных в Python можно почитать [тут](https://docs.python.org/3.6/reference/datamodel.html)\):

```py
>>> u.__dict__
{}
```

Так как мы не создали еще ни одного атрибута, то и словарь будет пустым. Давайте добавим несколько атрибутов  \(Python позволяет динамически привязывать новые атрибуты к объекту, в конце концов это просто словарь\):

```py
>>> u.username = 'bob'
>>> u.password = 'bob@example.com'
>>> u.email = 'foobar'

>>> u.__dict__
{'username': 'bob', 'password': 'bob@example.com', 'email': 'foobar'}

# Следующие два выражения в нашем примере эквиваленты
>>> u.username
'bob'
>>> u.__dict__['username']
'bob'
```

Что будет, если мы обратимся к атрибуту, которого не существует?

```py
>>> u.created_at
Traceback (most recent call last):
  File "<input>", line 1, in <module>
    u.created_at
AttributeError: 'User' object has no attribute 'created_at'
```

Работать с атрибутами можно с помощью следующих функций:

* `hasattr(obj, attr_name)` - проверить наличие атрибута `attr_name` в объекте `obj`. Если атрибут присутствует, то функция возвращает `True`, иначе `False`.
* `getattr(obj, attr_name[, default_value])` - получить значение атрибута. Можно указать значение по умолчанию `default_value`, которое будет возвращено, если атрибута не существует. 
* `setattr(obj, attr_name, value)` - изменить значение атрибута `attr_name` на `value`. Если атрибут не существовал, то он будет создан.

```py
>>> hasattr(u, 'created_at')
False
>>> hasattr(u, 'username')
True

>>> getattr(u, 'created_at')
Traceback (most recent call last):
  File "<input>", line 1, in <module>
    getattr(u, 'created_at')
AttributeError: 'User' object has no attribute 'created_at'

import datetime
>>> getattr(u, 'created_at', datetime.datetime.now())
datetime.datetime(2017, 4, 11, 16, 45, 36, 757869)

>>> setattr(u, 'created_at', datetime.datetime.now())
datetime.datetime(2017, 4, 11, 16, 45, 36, 757869)
```

Функция `setattr` может оказаться полезной, когда нам необходимо добавить в объект множество атрибутов, хранящихся в каком-нибудь контейнере:

```py
u = User()
attrs = {'username': 'bob', 'email': 'bob@example.com', 'password': 'foobar'}
for k, v in attrs.items():
    setattr(u, k, v)
```

Добавим теперь функцию получения имени пользователя в ранее созданный объект:

```py
def get_username(user):
    return user.username

>>> u.get_username = get_username
>>> u.__dict__
{'username': 'bob', 'password': 'bob@example.com', 'email': 'foobar', 'get_username': <function get_username at 0x1038256a8>}
>>> u.get_username(u)
'bob'
```

Обратите внимание, что мы вызываем функцию `get_username` у объекта `u` и в качестве аргумента передаем сам объект `u`. Выглядит странно и некрасиво.

<div class="alert alert-info">
<b>Черная магия питона:</b> Ради справедливости нужно сказать, что мы можем динамически привязать метод к объекту, так, чтобы не пришлось передавать объект в качестве аргумента. Пример приведен ниже.
</div>


```py
>>> from types import MethodType
>>> u.get_username = MethodType(get_username, u)
>>> u.get_username()
'bob'

>>> u.__dict__
{'username': 'bob', 'password': 'bob@example.com', 'email': 'foobar', 'get_username': <bound method get_username of <__console__.U
ser object at 0x103839d30>>}
```

Так как каждый объект класса _Пользователь_ должен содержать одинаковый набор атрибутов, но с разными значениями, то было бы логичным вынести процесс создания атрибутов в отдельный метод. В Python таким методом является `__init__`:

```py
class User:
    def __init__(self, username, email, password):
        self.username = username
        self.email = email
        self.password = password

    def get_username(self):
        return self.username

>>> u = User('bob', 'bob@example.com', 'foobar')
>>> u.email
'bob@example.com'
>>> u.get_username()
'bob'
```

Возникает вопрос "_Откуда взялся `self` в качестве первого аргумента у метода `__init__()`?_". Упрощенно процесс создания и инициализации нового объекта можно описать следующими шагами:

1. `u = User('bob', 'bob@example.com', 'foobar')`
2. Вызывается конструктор объекта [`__new__()`](https://docs.python.org/3.6/reference/datamodel.html#object.__new__), который возвращает "пустой" объект.
3. Созданный объект передается в инициализатор `__init__()` в качестве первого аргумента с именем `self` (такое имя не является обязательным, но используется по соглашению), за ним передаются все остальные аргументы указанные при вызове класса (`'bob', 'bob@example.com', 'foobar'`).
4. У объекта создаются все требуемые атрибуты, например `self.username = username`.
5. Инициализированный объект возвращается на место вызова класса, в примере переменная `u` связывается с созданным объектом.

<div class="alert alert-info">
<b>Замечание:</b> Как было сказано это упрощенная схема создания и инициализации нового объекта. Чтобы понимать этот процесс более полно, то необходимо ввести понятие метаклассов, о которых вы можете прочитать <a href="https://blog.ionelmc.ro/2015/02/09/understanding-python-metaclasses/">тут</a>.
</div>

Вы заметили, что в метод `get_username()` также передается `self`? А также то, что при вызове этого метода мы не передаем никаких аргументов? Давайте рассмотрим два следующих выражения:
```py
>>> u.get_username()
'bob'
>>> User.get_username(u)
'bob'
```

В Python классы также являются объектами (_в Python все является объектом_) и методы, в отличие от атрибутов, принадлежат классу, а не объекту. Поэтому метод `get_username()` вызывается не у объекта, а у класса, а объект передается в качестве аргумента (отсюда следует, что объект должен передаваться в качестве первого аргумента во все методы класса и по соглашению его называют `self`). Таким образом, выражение `u.get_username()` является просто синтаксическим сахаром (упрощенной формой записи) по отношению к `User.get_username(u)`.

<div class="alert alert-info">
<b>Замечание:</b> Конечно все несколько сложнее, рассмотрим следующий пример:<br/>
<code>>>> User.get_username</code><br/>
<code>&lt function User.get_username at 0x103a06268 ></code><br/>
<code>>>> u.get_username</code><br/>
<code>&lt bound method User.get_username of <__console__.User object at 0x1039a8eb8>></code><br/>
</div>

### Пример: создание простой ORM

Что такое ORM? Вот пояснение с сайта [Full Stack Python](https://www.fullstackpython.com/object-relational-mappers-orms.html):

> An object-relational mapper (ORM) is a code library that automates the transfer of data stored in relational databases tables into objects that are more commonly used in application code.

В этом примере (полностью основанном на [этом коде](https://codescience.wordpress.com/2011/02/06/python-mini-orm/)) мы рассмотрим пример создания примитивной ORM для SQLite базы данных, которая имеет [встроенную поддержку](https://docs.python.org/3.6/library/sqlite3.html) в Python.

Создадим БД с таблицей _Пользователи_ и добавим туда несколько записей:

```py
import sqlite3

# Создание нового соединения с БД
conn = sqlite3.connect('users_db.sqlite3')

# Курсор это объект, который позволяет выполнять запросы к БД
cursor = conn.cursor()

# Создание таблицы пользователей
cursor.execute('CREATE TABLE users (id, username, email, password)')

# Добавление новых записей
users = [
   (1, 'john', 'john@thebeatles.com', 'foobar'),
   (2, 'paul', 'paul@thebeatles.com', 'barfoo'),
   (3, 'ringo', 'ringo@thebeatles.com', 'foobaz'),
   (4, 'george', 'george@thebeatles.com', 'bazfoo')
]
cursor.executemany('INSERT INTO users VALUES (?,?,?,?)', users)
conn.commit()

# Вывод всех записей
for row in cursor.execute('SELECT * FROM users'):
   print(row)
```

В результате вы должны увидеть следующие записи:
```py
(1, 'john', 'john@thebeatles.com', 'foobar')
(2, 'paul', 'paul@thebeatles.com', 'barfoo')
(3, 'ringo', 'ringo@thebeatles.com', 'foobaz')
(4, 'george', 'george@thebeatles.com', 'bazfoo')
```

Теперь перейдем к ORM:
```py
import sqlite3


class DataBase:
    def __init__(self, db='db'):
        self.conn = sqlite3.connect(f"{db}.sqlite3")
        self.cursor = self.conn.cursor()

    def get_columns(self, tbl_name):
        self.sql_rows = f"SELECT * FROM {tbl_name}"
        columns = f"PRAGMA table_info({tbl_name})"
        self.cursor.execute(columns)
        return [row[1] for row in self.cursor.fetchall()]

    def Table(self, tbl_name):
        columns = self.get_columns(tbl_name)
        return Query(self.cursor, self.sql_rows, columns, tbl_name)


class Query:
    def __init__(self, cursor, rows, columns, tbl_name):
        self.cursor = cursor
        self.sql_rows = rows
        self.columns = columns
        self.tbl_name = tbl_name

    def filter(self, criteria):
        key_word = "AND" if "WHERE" in self.sql_rows else "WHERE"
        sql = f"{self.sql_rows} {key_word} {criteria}"
        return Query(self.cursor, sql, self.columns, self.tbl_name)

    def order_by(self, criteria):
        return Query(self.cursor, f"{self.sql_rows} ORDER BY {criteria}", self.columns, self.tbl_name)

    def group_by(self, criteria):
        return Query(self.cursor, f"{self.sql_rows} GROUP BY {criteria}", self.columns, self.tbl_name)

    @property
    def rows(self):
        print(self.sql_rows)
        self.cursor.execute(self.sql_rows)
        return [Row(zip(self.columns, fields), self.tbl_name) for fields in self.cursor.fetchall()]


class Row:
    def __init__(self, fields, table_name):
        self.__class__.__name__ = table_name + "_Row"

        for name, value in fields:
            setattr(self, name, value)
    
    def __repr__(self):
        attrs =  ', '.join([f"{attr}={value}" for attr, value in self.__dict__.items()])
        return f"{self.__class__.__name__}({attrs})"
```
Класс `DataBase` отвечает за создание соединения, класс `Query` за формирование запроса к БД, класс `Row` представляет одну запись в таблице.

<div class="alert alert-info">
<b>Замечание:</b> Все строки в формате <a href="https://cito.github.io/blog/f-strings/">f-strings</a>, который был введен в Python 3.6.<br/>
Метод <code>__repr__</code> переопределен, чтобы выводить чуть больше полезной информации об объекте, чем просто его адрес в памяти. Узнать больше про магические методы в питоне можно прочитав статью на <a href="https://habrahabr.ru/post/186608/">Хабре</a>.
</div>

Ниже приведен пример использования:

```py
>>> db = DataBase('users_db')

>>> db.get_columns('users')
['id', 'username', 'email', 'password']

>>> db.Table('users').rows:
SELECT * FROM users
[users_Row(id=1, username=john, email=john@thebeatles.com, password=foobar),
 users_Row(id=2, username=paul, email=paul@thebeatles.com, password=barfoo),
 users_Row(id=3, username=ringo, email=ringo@thebeatles.com, password=foobaz),
 users_Row(id=4, username=george, email=george@thebeatles.com, password=bazfoo)]

>>> db.Table('users').filter('id > 2').rows
SELECT * FROM users WHERE id > 2
[users_Row(id=3, username=ringo, email=ringo@thebeatles.com, password=foobaz),
 users_Row(id=4, username=george, email=george@thebeatles.com, password=bazfoo)]

>>> db.Table('users').order_by('username DESC').rows
SELECT * FROM users ORDER BY username DESC
[users_Row(id=3, username=ringo, email=ringo@thebeatles.com, password=foobaz),
 users_Row(id=2, username=paul, email=paul@thebeatles.com, password=barfoo),
 users_Row(id=1, username=john, email=john@thebeatles.com, password=foobar),
 users_Row(id=4, username=george, email=george@thebeatles.com, password=bazfoo)]

>>> user = db.Table('users').rows[0]
SELECT * FROM users
>>> user.id
1
>>> user.username
'john'
```

**Задания**:
1. Вы должны были заметить, что мы получаем объекты класса `users_Row`, а не класса `User`. Попробуйте внести изменения, чтобы мы получали объекты класса `User`:
```py
>>> user = db.Table('users').rows[0]
>>> type(user)
<class '__main__.User'>
>>> user.get_username()
'john'
```
2. Добавьте метод `limit(N)` в класс `DataBase`, который позволяет получить не больше N записей.
3. Добавьте метод `insert(obj)`, который создает в БД новую запись об объекте `obj`. 

### "Приватные" поля класса

Допустим у нас имеется следующее определение класса, в котором есть методы для изменения и проверки пароля, а также адреса электронной почты:

```py
import hashlib
import random
import string
import re


class User:

    def __init__(self, username, email, password):
        self.username = username
        
        self.set_email(email)
        self.set_password(password)
    
    def set_email(self, email):
        match = re.match('^[_a-z0-9-]+(\.[_a-z0-9-]+)*@[a-z0-9-]+(\.[a-z0-9-]+)*(\.[a-z]{2,4})$', email)
        if not match:
            raise ValueError("Invalid email")
        self.email = email
    
    # http://pythoncentral.io/hashing-strings-with-python/
    # https://docs.python.org/3.5/library/hashlib.html
    def make_salt(self):
        salt = ""
        for i in range(5):
            salt = salt + random.choice(string.ascii_letters)
        return salt

    def set_password(self, pw, salt=None):
        if salt == None:
            salt = self.make_salt()
        self.password = hashlib.sha256(pw.encode() + salt.encode()).hexdigest() + "," + salt

    def check_password(self, user_password):
        password, salt = self.password.split(',')
        return password == hashlib.sha256(user_password.encode() + salt.encode()).hexdigest()
```

Давайте посмотрим на работу с этими методами:

```py
>>> u = User('bob', 'bob@example.com', 'foobar')
>>> u.username
'bob'
>>> u.email
'bob@example.com'
>>> u.password
'fd6c85ddaccb0d13e8b4d45b6e3bcbcc18d3b737a10aef21d297c861d770da6d,PDrGK'

>>> u.set_email('bob-new-email')
Traceback (most recent call last):
  File "<input>", line 1, in <module>
    u.set_email('bob-new-eamil')
  File "<input>", line 12, in set_email
    raise ValueError("Invalid email")
ValueError: Invalid email
>>> u.set_email('bob-new-eamil@example.com')
>>> u.email
'bob-new-eamil@example.com'

>>> u.check_password('barfoo')
False
>>> u.check_password('foobar')
True
```

Допустим, что мы хотим изменить пароль (или адрес электронной почты) и делаем это напрямую обращаясь к атрибуту:
```py
>>> u.password = 'barfoo'
>>> u.check_password('barfoo')
False
```

Почему пароль не прошел проверку? Мы изменили значение атрибута напрямую, не используя функцию `set_password()`, таким образом, мы сохранили пароль в открытом виде. В свою очередь функция `check_password()` хеширует переданный ей пароль в качестве аргумента и затем сравнивает его с паролем, который хранился в атрибуте `password`.

Очевидно, что нужно менять значение пароля или адреса электронной почты с помощью методов `set_password()` и `set_email()`, чтобы избежать подобного рода ошибок. А прямое обращение к полям `password` и `email` нужно ограничить.

### Свойства \(property\)

```py
class UserProfile:

    def __init__(self, user, first_name='', sur_name='', bdate=None):
        assert isinstance(user, User), '`user` field must be a User class instance'
        self._user = user
        self.first_name = first_name
        self.sur_name = sur_name
        self.bdate = bdate
    
        self._age = None
        self._age_last_recalculated = None
        self._recalculate_age()

    def _recalculate_age(self):
        today = datetime.date.today()
        age = today.year - self.bdate.year

        if today < datetime.date(today.year, self.bdate.month, self.bdate.day):
            age -= 1

        self._age = age
        self._age_last_recalculated = today

    def age(self):
        if (datetime.date.today() > self._age_last_recalculated):
            self._recalculate_age()

        return self._age


class User:

    def __init__(self, username, email, password):
        ...
        self.profile = UserProfile(self)
        ...    
```

```py
class UserProfile:
    ...
    @property
    def age(self):
        if (datetime.date.today() > self._age_last_recalculated):
            self._recalculate_age()

        return self._age
```

```py
class UserProfile:
    ...
    @property
    def fullname(self):
        return '{} {}'.format(self.first_name, self.sur_name).title()

    @fullname.setter
    def fullname(self, value):
        name, surname = value.split(" ", maxsplit=1)
        self.first_name = name
        self.sur_name = surname

    @fullname.deleter
    def fullname(self):
        self.first_name = ''
        self.sur_name = ''
```

### Дескрипторы

### Методы класса \(@classmethod и @staticmethod\)

### Наследование

```py
class TimestampedModel:

    def __init__(self, created_at=None, updated_at=None):
        self.created_at = created_at or datetime.datetime.now()
        self.updated_at = updated_at or created_at

    def update(self):
        self.updated_at = datetime.datetime.now()
```

```py
class User(TimestampedModel):

    def __init__(self, username, email, password):
        self.username = username
        self.email = email
        self.password = password
```

```py
class User(TimestampedModel):

    def __init__(self, username, email, password):
        self.username = username
        self.email = email
        self.password = password
        super().__init__()

    @property
    def username(self):
        return self._username

    @username.setter
    def username(self, new_username):
        self._username = new_username
        self.update()
```

```py
import json
import attr
import enum
import datetime


class JSONSerializerMixin:

    @staticmethod
    def to_serializable(value):
        if isinstance(value, datetime.datetime):
            return value.isoformat() + "Z"
        elif isinstance(value, enum.Enum):
            return value.value
        elif attr.has(value.__class__):
            return attr.asdict(value)
        elif isinstance(value, Exception):
            return {
                "error": value.__class__.__name__,
                "args": value.args,
            }
        return str(value)

    def toJSON(self):
        return json.dumps(self.__dict__, default=JSONSerializerMixin.to_serializable)
    
    @classmethod
    def fromJSON(cls, data):
        def datetime_parser(json_dict):
            for k,v in json_dict.items():
                try:
                    json_dict[k] = datetime.datetime.strptime(v, "%Y-%m-%d").date()
                except:
                    pass
            return json_dict
        return cls(**json.loads(data, object_hook=datetime_parser))
```

```py
class User(TimestampedModel, JSONSerializerMixin):
    ...
```

### Множественное наследование и полиморфизм

### Метаклассы

### Абстрактные классы

Продолжим рассматривать пример с классом _Пользователь_. Мы можем разделить пользователей на два типа: **анонимные пользователи**, то есть те пользователи, которые не зарегистрированы или не вошли в систему под своим логином и паролем, и **авторизованные пользователи**.

```py
class AbstractUser:

    def set_password(self, raw_password):
        raise NotImplementedError()

    def check_password(self, raw_password):
        raise NotImplementedError()
    
    def is_anonymous(self):
        raise NotImplementedError()
```

<div class="alert alert-info">
<b>Замечание:</b> Если проводить параллель с другими языками программирования, например, Java, то такой класс более справедливо было бы назвать <a href="http://stackoverflow.com/questions/761194/interface-vs-abstract-class-general-oo">интерфейсом</a>.
</div>

```py
class AnonymousUser(AbstractUser):
    
    def is_anonymous(self):
        return True

class User(AbstractUser):
    def set_password(self, raw_password):
        ...

    def check_password(self, raw_password):
        ...

    def is_anonymous(self):
        return False
```

```py
from abc import ABCMeta, abstractmethod


class AbstractUser(metaclass=ABCMeta):
    def set_password(self, raw_password):
        raise NotImplementedError()

    def check_password(self, raw_password):
        raise NotImplementedError()

    @abstractmethod
    def is_anonymous(self):
        pass
```


