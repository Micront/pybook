### REST API для ведения заметок на Django

В этой работе мы напишем простой REST API сервис для ведения заметок. Если вы не знаете, что такое REST API, то советую обратиться к этой ссылке: [http://www.restapitutorial.ru/lessons/whatisrest.html](http://www.restapitutorial.ru/lessons/whatisrest.html).

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

В этой работе для проверки нашего кода мы будем писать тесты согласно принципу \(философии\) [TDD](https://ru.wikipedia.org/wiki/Разработка_через_тестирование%29, простыми словами: сначала пишем тест - потом код. В Django для создания тестов используется [unittest-фреймворк]%28https://docs.python.org/3/library/unittest.html#module-unittest).

Создадим наш первый тест, который проверяет можем ли мы создавать модели и сохранять их в БД. Для этого в файле `todolist/tests.py` добавим следующий класс:

```py
from django.test import TestCase
from .models import Task

class ModelTestCase(TestCase):
    def setUp(self):
        self.task_name = "Выполнить лабораторную работу №9"
        self.task = Task(name=self.task_name)

    def test_model_can_create_task(self):
        old_count = Task.objects.count()
        self.task.save()
        new_count = Task.objects.count()
        self.assertNotEqual(old_count, new_count)
```

Теперь создадим модель `Task` в файле `todolist/models.py`:

```py
from django.db import models

class Task(models.Model):
    pass
```

И запустим наш тест:

```
python3 manage.py test
```

Вы увидите множество ошибок, которые связаны с тем, что мы еще не создали БД и не добавили туда модель `Task`. Давайте сделаем это:

```py
$ python3 manage.py makemigrations
Migrations for 'todolist':
  todolist/migrations/0001_initial.py:
    - Create model Task

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
ERROR: test_model_can_create_task (todolist.tests.ModelTestCase)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/dementiy/Projects/python-projects/django-tutorials/djangorest/todolist/tests.py", line 9, in setUp
    self.task = Task(name=self.task_name)
  File "/Library/Frameworks/Python.framework/Versions/3.5/lib/python3.5/site-packages/django/db/models/base.py", line 555, in __init__
    raise TypeError("'%s' is an invalid keyword argument for this function" % list(kwargs)[0])
TypeError: 'name' is an invalid keyword argument for this function

----------------------------------------------------------------------
Ran 1 test in 0.006s

FAILED (errors=1)
Destroying test database for alias 'default'...
```

В этот раз сообщений об ошибках не так много и они стали более информативными. В нашем примере не найден атрибут `name` у модели `Task`. Давайте изменим нашу модель:

```py
class Task(models.Model):
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
    - Add field completed to task
    - Add field date_created to task
    - Add field date_modified to task
    - Add field description to task
    - Add field due_date to task
    - Add field name to task
    - Add field priority to task
MacBook-Air:djangorest dementiy$ python3 manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions, todolist
Running migrations:
  Applying todolist.0002_auto_20170225_1224... OK
```

**Замечание**: ....

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

Теперь нам нужно создать [сериализатор](http://www.django-rest-framework.org/api-guide/serializers/%29 %28serializer\). Сериализаторы позволяют представить множество сложных объектов, которые возможно мы получили из БД, в простой и человеко-читаемой форме, например, в формате JSON или XML.

Создадим файл `todolist/serializers.py` со следующим содержимым:

```py
from rest_framework import serializers
from .models import Task


class TaskSerializer(serializers.ModelSerializer):
    class Meta:
        model = Task
        fields = ('id', 'name', 'description', 'completed', 'date_created', 'date_modified', 'due_date', 'priority')
        read_only_fields = ('date_created', 'date_modified')
```

Нам нужно создать несколько "вьюеров" \(views\), которые позволят:

* Создать заметку - POST-запрос
* Удалить заметку - DELETE-запрос
* Обновить заметку - PUT-запрос
* Просмотреть одну или несколько заметок - GET-запрос

Снова создадим тест в файле `todolist/tests.py`:

```py
from rest_framework.test import APIClient
from rest_framework import status
from django.core.urlresolvers import reverse

class ViewTestCase(TestCase):
    def setUp(self):
        self.client = APIClient()
        self.task_data = {
            'name': 'Выполнить 9 лабораторную работу',
            'description': 'Прочитать руководство по django rest framework',
            'priority': 'h'
        }
        self.response = self.client.post(reverse('create'), self.task_data, format='json')

    def test_api_can_create_a_task(self):
        self.assertEqual(self.response.status_code, status.HTTP_201_CREATED)
```

Запустите тест также как мы делали это ранее.

Теперь в файле `todolist/views.py` создадим два вьювера для создания и просмотра заметок:

```py
from rest_framework import generics
from .serializers import TaskSerializer
from .models import Task


class TaskCreateView(generics.ListCreateAPIView):
    queryset = Task.objects.all()
    serializer_class = TaskSerializer


class TaskDetailsView(generics.RetrieveAPIView):
    queryset = Task.objects.all()
    serializer_class = TaskSerializer
```

Теперь нам нужно связать вьюеры с соответствующими URL, для этого создайте файл `todolist/urls.py`:

```py
from django.conf.urls import url, include
from rest_framework.urlpatterns import format_suffix_patterns
from .views import TaskCreateView, TaskDetailsView

urlpatterns = {
    url(r'^todolists/$', TaskCreateView.as_view(), name="create"),
    url(r'^todolists/(?P<pk>[0-9]+)/$', TaskDetailView.as_view(), name="detail"),
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

Пройдите по адресу `http://127.0.0.1:8000/todolists`, вы должны увидеть следующее окно:![](/assets/Screen Shot 2017-02-25 at 15.51.30.png)В нашей БД еще нет ни одной записи, давайте добавим новую запись, для этого достаточно заполнить поля и нажать на

`POST`:![](/assets/Screen Shot 2017-02-25 at 15.53.36.png%29![]%28/assets/Screen Shot 2017-02-25 at 15.54.04.png%29Также мы можем обратиться по адресу `http://127.0.0.1:8000/todolists/1/`, чтобы получить заметку с идентификатором 1:![]%28/assets/Screen Shot 2017-02-25 at 17.24.22.png)

Теперь давайте добавим возможность объединения заметок в группы, назовем их "Списком задач". Для этого нам нужно добавить модель для списка задач `todolist/models.py`:

```py
class Tasklist(models.Model):
    name = models.CharField(max_length=200)

    def __str__(self):
        return "{}".format(self.name)
```

Теперь необходимо отразить, что задача входит в конкретный список задач \(то есть отношение один ко многим - один список задач включает множество задач\), сделаем это с помощью внешнего ключа \(про работу с внешними ключами можно почитать [тут](https://docs.djangoproject.com/en/1.10/topics/db/examples/many_to_one/%29\):

```py
class Task(models.Model):
    # ...
    tasklist = models.ForeignKey(Tasklist, related_name='tasks', on_delete=models.CASCADE)
    # ...
```

Также нам надо добавить сериализатор для списка задач:

```py
from .models import Tasklist


class TasklistSerializer(serializers.ModelSerializer):
    tasks = serializers.StringRelatedField(many=True)

    class Meta:
        model = Tasklist
        fields = ('name', 'tasks')
```

Все изменения надо зафиксировать в БД:

```
python3 manage.py makemigrations
python3 manage.py migrate
```

Прежде чем мы продолжим давайте определимся со списком методов, которые будем предоставлять пользователю:

* `/todolists` - получение всех списков задач или создание нового списка \(GET и POST методы\)
* `/todolists/list_id`_ - _редактирование или удаление списка задач с идентификатором `list_id` \(GET, PUT, DELETE\)
* `/todolists/list_id/tasks`_ _-  просмотр задач или создание новой задачи в списке задач с идентификатором `list_id` \(GET и POST\)
* `/todolists/list_id/tasks/task_id` - редактирование или удаление задачи `task_id` в списке задач с идентификатором `list_id` \(GET, PUT, DELETE\)

Давайте отразим этот список методов в наших маршрутах `todolists/urls.py`:

```py
from django.conf.urls import url, include
from rest_framework.urlpatterns import format_suffix_patterns
from .views import TasklistCreateView, TasklistDetailsView, TaskCreateView, TaskDetailsView
from rest_framework.authtoken.views import obtain_auth_token

urlpatterns = {
    url(r'^todolists/$', TasklistCreateView.as_view(), name="lists"),
    url(r'^todolists/(?P<pk>[0-9]+)/$', TasklistDetailsView.as_view(), name="list-detail"),
    url(r'^todolists/(?P<list_id>[0-9]+)/tasks', TaskCreateView.as_view(), name="tasks"),
    url(r'^todolists/(?P<list_id>[0-9]+)/tasks/(?P<pk>[0-9]+)', TaskDetailsView.as_view(), name="task-detail"),
}

urlpatterns = format_suffix_patterns(urlpatterns)
```

Нам нужно добавить вьюверы для списков задач и изменить вьюверы для самих задач:

```py
class TasklistCreateView(generics.ListCreateAPIView):
    queryset = Tasklist.objects.all()
    serializer_class = TasklistSerializer


class TasklistDetailsView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Tasklist.objects.all()
    serializer_class = TasklistSerializer


class TaskCreateView(generics.ListCreateAPIView):
    serializer_class = TaskSerializer

    def get_queryset(self):
        queryset = Task.objects.all()
        list_id = self.kwargs.get('list_id', None)
        if list_id is not None:
            queryset = queryset.filter(tasklist_id = list_id)
        return queryset

    def perform_create(self, serializer):
        list_id = self.kwargs.get('list_id', None)
        try:
            tasklist = Tasklist.objects.get(pk=list_id)
        except Tasklist.DoesNotExist:
            raise NotFound()
        serializer.save(tasklist=tasklist)


class TaskDetailsView(generics.RetrieveUpdateDestroyAPIView):
    serializer_class = TaskSerializer

    def get_queryset(self):
        queryset = Task.objects.all()
        list_id = self.kwargs.get('list_id', None)
        if list_id is not None:
            queryset = queryset.filter(tasklist_id = list_id)
        return queryset
```

Обратите внимание, что мы теперь получаем задачи предварительно фильтруя их по идентификатору списка задач.

**Задание**:

1. Добавьте возможность указания тегов для задач. Для этого вам необходимо создать новую модель `Tag`, у тега будет поле с именем \(`name`\). Следует иметь ввиду, что одна задача может иметь множество тегов, а одинаковые теги, могут быть применимы к разным задачам. Таким образом, это отношение многие ко многим. Посмотрите [этот вопрос](http://stackoverflow.com/questions/37828358/manytomany-with-django-rest-framework) и ответ для решения этого задания.
2. Добавьте поле владелец \(`owner`\) к списку задач, таким образом, пользователь должен видеть только те списки задач \(или задачи\), которые создавал он. Для выполнения этого задания вам необходимо иметь возможность создавать пользователей.



