### REST API для ведения заметок на Django

```py
python3 -m pip install django
```

Создадим новый проект:

```py
django-admin startproject djangorest
```

Установим REST API фреймворк:

```py
python3 -m pip install djangorestframework
```

Подключим фреймворк для использования в нашем проекте. Для этого в файле `djangorest/settings.py` необходимо добавить строку `rest_framework`:

```py
# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
]
```

В одном проекте может быть задействовано несколько приложений, поэтому создадим отдельное \(и единственное\) приложение для заметок:

```py
cd djangorest
python3 manage.py startapp todolist
```

Добавим наше приложение в файле `settings.py`:

```py
# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'todolist',
]
```

Создадим наш первый тест, который проверяет можем ли мы создавать модели и сохранять их в БД. Для этого в файле `todolist/tests.py` добавим следующий класс:

```py
from django.test import TestCase
from .models import Todolist

class ModelTestCase(TestCase):
    def setUp(self):
        self.todolist_name = "Выполнить лабораторную работу №9"
        self.todolist = Todolist(name=self.todolist_name)

    def test_model_can_create_todolist(self):
        old_count = Todolist.objects.count()
        self.todolist.save()
        new_count = Todolist.objects.count()
        self.assertNotEqual(old_count, new_count)
```

Теперь создадим модель `Todolist` в файле `todolist/models.py`:

```py
from django.db import models

class Todolist(models.Model):
    pass
```

И запустим наш тест:

```
python3 manage.py test
```

Вы увидите множество ошибок, которые связаны с тем, что мы еще не создали БД и не добавили туда модель `Todolist`. Давайте сделаем это:

```py
$ python3 manage.py makemigrations
Migrations for 'todolist':
  todolist/migrations/0001_initial.py:
    - Create model Todolist

$ python3 manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions, todolist
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying sessions.0001_initial... OK
  Applying todolist.0001_initial... OK
```

**Замечание**: По умолчанию в Django используется `sqlite3`. В корне проекта будет создан файл `db.sqlite3`, содержимое которого вы можете просмотреть с помощью программы DB Browser for SQLite.

И еще раз запустим наш тест:

```py
python3 manage.py test
Creating test database for alias 'default'...
E
======================================================================
ERROR: test_model_can_create_todolist (todolist.tests.ModelTestCase)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/dementiy/Projects/python-projects/django-tutorials/djangorest/todolist/tests.py", line 9, in setUp
    self.todolist = Todolist(name=self.todolist_name)
  File "/Library/Frameworks/Python.framework/Versions/3.5/lib/python3.5/site-packages/django/db/models/base.py", line 555, in __init__
    raise TypeError("'%s' is an invalid keyword argument for this function" % list(kwargs)[0])
TypeError: 'name' is an invalid keyword argument for this function

----------------------------------------------------------------------
Ran 1 test in 0.006s

FAILED (errors=1)
Destroying test database for alias 'default'...
```

В этот раз сообщений об ошибках не так много и они стали более информативными. В нашем примере не найден атрибут `name` у модели `Todolist`. Давайте изменим нашу модель:

```py
class Todolist(models.Model):
    name = models.CharField(max_length=200, blank=True)
    description = models.TextField(max_length=1000, blank=True)
    completed = models.BooleanField(default=False)
    date_created = models.DateField(auto_now_add=True)
    due_date = models.DateField(null=True, blank=True)
    date_modified = models.DateField(auto_now=True)

    PRIORITY = (
        ('h', 'High'),
        ('m', 'Medium'),
        ('l', 'Low'),
        ('n', 'None')
    )

    priority = models.CharField(max_length=1, choices=PRIORITY, default='n')

    def __str__(self):
        return "{}".format(self.name)
```

И сохраним изменения в БД:

```py
$ python3 manage.py makemigrations
Migrations for 'todolist':
  todolist/migrations/0002_auto_20170225_1224.py:
    - Add field completed to todolist
    - Add field date_created to todolist
    - Add field date_modified to todolist
    - Add field description to todolist
    - Add field due_date to todolist
    - Add field name to todolist
    - Add field priority to todolist
MacBook-Air:djangorest dementiy$ python3 manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions, todolist
Running migrations:
  Applying todolist.0002_auto_20170225_1224... OK
```

Снова запустим тест:

```py
python3 manage.py test
Creating test database for alias 'default'...
.
----------------------------------------------------------------------
Ran 1 test in 0.003s

OK
Destroying test database for alias 'default'...
```

В этот раз уже никаких ошибок не было, а это означает, что мы можем создавать модели и сохранять их в БД.

Теперь нам нужно создать сериализатор \(serializer\). Сериализаторы позволяют представить множество сложных объектов, которые возможно мы получили из БД, в простой и человеко-читаемой форме, например, в формате JSON или XML.

Создадим файл `todolist/serializers.py` со следующим содержимым:

```py
from rest_framework import serializers
from .models import Todolist


class TodolistSerializer(serializers.ModelSerializer):
    class Meta:
        model = Todolist
        fields = ('id', 'name', 'description', 'completed', 'date_created', 'date_modified', 'due_date', 'priority')
        read_only_fields = ('date_created', 'date_modified')
```

Нам нужно создать несколько "вьюеров" \(views\), которые позволят:

* Создать заметку - POST-запрос
* Удалить заметку - DELETE-запрос
* Обновить заметку - PUT-запрос
* Просмотреть одну или несколько заметок - GET-запрос

Теперь в файле `todolist/views.py` создадим два вьювера для создания и просмотра заметок:

```py
from rest_framework import generics
from .serializers import TodolistSerializer
from .models import Todolist


class TodolistCreateView(generics.ListCreateAPIView):
    queryset = Todolist.objects.all()
    serializer_class = TodolistSerializer


class TodolistDetailView(generics.RetrieveAPIView):
    queryset = Todolist.objects.all()
    serializer_class = TodolistSerializer
```

Теперь нам нужно связать вьюеры с соответствующими URL, для этого создайте файл `todolist/urls.py`:

```py
from django.conf.urls import url, include
from rest_framework.urlpatterns import format_suffix_patterns
from .views import TodolistCreateView, TodolistDetail

urlpatterns = {
    url(r'^todolists/$', TodolistCreateView.as_view(), name="create"),
    url(r'^todolists/(?P<pk>[0-9]+)/$', TodolistDetailView.as_view(), name="detail"),
}

urlpatterns = format_suffix_patterns(urlpatterns)
```

Теперь нам этот файл нужно включить в список всех маршрутов `djangorest/urls.py`:

```py
from django.conf.urls import url, include
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^', include('todolist.urls')),
]
```

Итак, давайте запустим сервер:

```
python3 manage.py runserver
Performing system checks...

System check identified no issues (0 silenced).
February 25, 2017 - 13:48:20
Django version 1.10.5, using settings 'djangorest.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

Пройдите по адресу `http://127.0.0.1:8000/todolists`, вы должны увидеть следующее окно:![](/assets/Screen Shot 2017-02-25 at 15.51.30.png)В нашей БД еще нет ни одной записи, давайте добавим новую запись, для этого достаточно заполнить поля и нажать на `POST`:![](/assets/Screen Shot 2017-02-25 at 15.53.36.png)![](/assets/Screen Shot 2017-02-25 at 15.54.04.png)Также мы можем обратиться по адресу `http://127.0.0.1:8000/todolists/1/`, чтобы получить заметку с идентификатором 1:![](/assets/Screen Shot 2017-02-25 at 17.24.22.png)





