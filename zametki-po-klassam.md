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

Так как каждый объект класса пользователь должен содержать одинаковый набор атрибутов, но с разными значениями, то было бы логичным вынести процесс создания атрибутов в отдельный метод. В Python таким методом является `__init__`:

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

Вы заметили, что в метод `get_username()` также передается `self`? А также то, что при вызове этого метода мы не передаем никаких аргументов? Давайте рассмотрим два следующих выражения:
```py
>>> u.get_username()
'bob'
>>> User.get_username(u)
'bob'
```

В Python классы также являются объектами (_в Python все является объектом_) и методы, в отличие от атрибутов, принадлежат классу, а не объекту. Поэтому метод `get_username()` вызывается не у объекта, а у класса, а объект передается в качестве аргумента (отсюда следует, что объект должен передаваться в качестве первого аргумента во все методы класса и по соглашению его называют `self`). Таким образом, выражение `u.get_username()` является просто синтаксическим сахаром (упрощенной формой записи) по отношению к `User.get_username(u)`.

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

Очевидно, что нужно менять значение пароля или адреса электронной почты с помощью методов `set_password()` и `set_email()`, чтобы избежать подобного рода ошибок. А прямое обращение к полям `password` и `email` - ограничить.

### Свойства \(property\)

### Методы класса \(@classmethod и @staticmethod\)

### Наследование

```py
class TimestampedModel:

    def __init__(self, created_at = None):
        self.created_at = created_at or datetime.datetime.now()
        self.updated_at = None

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
```

```py
class User(TimestampedModel, JSONSerializerMixin):
    ...
```

### Множественное наследование и полиморфизм

### Метаклассы

### Абстрактные классы



