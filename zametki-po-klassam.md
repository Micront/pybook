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

Атрибуты хранятся в специальном словаре:

```py
>>> u.__dict__
{}
```

Так как мы не создали еще ни одного атрибута, то и словарь будет пустым. Давайте добавим несколько атрибутов  \(python позволяет динамически привязывать новые атрибуты к объекту, в конце концов это просто словарь\):

```py
>>> u.username = 'bob'
>>> u.password = 'bob@example.com'
>>> u.email = 'foobar'

>>> u.__dict__
{'username': 'bob', 'password': 'bob@example.com', 'email': 'foobar'}
```

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

```py
u = User()
attrs = {'username': 'bob', 'email': 'bob@example.com', 'password': 'foobar'}
for k, v in attrs.items():
    setattr(u, k, v)
```

```py
def get_username(user):
    return user.username

>>> u.get_username = get_username
>>> u.get_username(u)
'bob'
```

```py
>>> from types import MethodType
>>> u.get_username = MethodType(get_username, u)
>>> u.get_username()
'bob'
```

```py
class User:
    def __init__(self, username, email, password):
        self.username = username
        self.email = email
        self.password = password

    def get_username(self):
        return self.username
```

```py
>>> u = User('bob', 'bob@example.com', 'foobar')
>>> u.email
'bob@example.com'
>>> u.get_username()
'bob'
```

### "Приватные" поля класса

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



