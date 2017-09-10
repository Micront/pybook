# Лабораторная работа №14. Eleven Note

<div class="alert alert-info">
Лабораторная работа основана на <a href="https://github.com/sixfeetup/ElevenNote">замечательном руководстве</a> по Django от компании Six Feet Up.
</div>

```bash
$ pip install Django==1.11.4
$ pip install psycopg2==2.7.3
$ pip install python-decouple==3.1
```

```bash
$ mkdir elevennote && cd $_
$ django-admin startproject config .
$ ls
config manage.py
```

```bash
$ mkdir config/settings
$ mv config/settings.py config/settings/base.py
```

```bash
$ touch config/settings/__init__.py
$ touch config/settings/local.py
$ touch config/settings/production.py
$ touch config/settings/settings.ini
```

`config/settings/base.py`
```python
import os
from decouple import config


def root(*dirs):
    base_dir = os.path.join(os.path.dirname(__file__), '..', '..')
    return os.path.abspath(os.path.join(base_dir, *dirs))


BASE_DIR = root()

SECRET_KEY = config('SECRET_KEY')

INSTALLED_APPS = [
    # ...
]

MIDDLEWARE = [
    # ...
]

ROOT_URLCONF = 'config.urls'

TEMPLATES = [
    # ...
]

WSGI_APPLICATION = 'config.wsgi.application'

AUTH_PASSWORD_VALIDATORS = [
    # ...
]

LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_L10N = True
USE_TZ = True
```

`configs/settings/local.py`
```python
from .base import *

DEBUG = True

INSTALLED_APPS += [
    'django.contrib.postgres',
]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': config('DB_NAME'),
        'USER': config('DB_USER'),
        'PASSWORD': config('DB_PASSWORD'),
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

`config/settings/settings.ini`
```python
[settings]
SECRET_KEY=
DB_NAME=
DB_USER=
DB_PASSWORD=
```

`manage.py`
```python
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings.local")
```

```bash
$ python manage.py migrate
$ python manage.py runserver
```

```bash
$ python manage.py createsuperuser
```


```bash
$ python manage.py startapp notes
```

`notes/models.py`
```python
class Note(models.Model):
    title = models.CharField(max_length=200)
    body = models.TextField()
    pub_date = models.DateTimeField('date published')
```

`notes/admin.py`
```python
from django.contrib import admin
from .models import Note

admin.site.register(Note)
```

```bash
$ python manage.py makemigrations note
$ python manage.py migrate
```

```bash
$ pip install django-admin-honeypot==1.0.0
```

`config/settings/base.py`
```python
INSTALLED_APPS = [
    # ...
    'admin_honeypot',
    # ...
]
```

`config/urls.py`
```python
urlpatterns = [
    url(r'^admin/', include('admin_honeypot.urls', namespace='admin_honeypot')),
    url(r'^secret/', admin.site.urls),
]
```


```bash
$ mkdir requirements
$ pip freeze > requirements/base.txt
```

`notes/admin.py`
```python
class NoteAdmin(admin.ModelAdmin):
    list_display = ('title', 'pub_date', 'was_published_recently')
    list_filter = ['pub_date']

# Replace your other register call with this line:
admin.site.register(Note, NoteAdmin)
```

`notes/models.py`
```python
def was_published_recently(self):
    return self.pub_date >= timezone.now() - timedelta(days=1)
```

```bash
$ mkdir notes/tests
$ mv notes/tests.py notes/tests/test_models.py
```

`notes/tests/test_models.py`
```python
import datetime

from django.utils import timezone
from django.test import TestCase

from .models import Note

class NoteMethodTests(TestCase):

    def test_was_published_recently(self):
    """
    was_published_recently() should return False for notes whose pub_date is in the future.
    """
        time = timezone.now() + datetime.timedelta(days=30)
        future_note = Note(pub_date=time)
        self.assertEqual(future_note.was_published_recently(), False)
```

```bash
$ python manage.py test
Creating test database for alias 'default'...
F
======================================================================
FAIL: test_was_published_recently (note.tests.NoteMethodTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/home/vagrant/Projects/elevennote/elevennote/note/tests.py", line 18, in test_was_published_recently
     self.assertEqual(future_note.was_published_recently(), False)
AssertionError: True != False

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
Destroying test database for alias 'default'...
```

`notes/models.py`
```python
def was_published_recently(self):
    now = timezone.now()
    return now - timedelta(days=1) <= self.pub_date <= now
```

`config/urls.py`
```python
url(r'^notes/', include('note.urls', namespace="note")),
```

```bash
$ touch notes/urls.py
```

`notes/urls.py`
```python
from django.conf.urls import url
 
from . import views

urlpatterns = [
    url(r'^$', views.index, name='index'),
    url(r'^(?P<note_id>[0-9]+)/$', views.detail, name='detail'),
]
```

`notes/views.py`
```python
from django.shortcuts import get_object_or_404, render
from django.http import HttpResponse

from .models import Note


def index(request):
    latest_note_list = Note.objects.order_by('-pub_date')[:5]
    context = {
        'latest_note_list': latest_note_list,
    }
    return render(request, 'note/index.html', context)

def detail(request, note_id):
     note = get_object_or_404(Note, pk=note_id)
     return render(request, 'note/detail.html', {'note': note})
```

```bash
$ mkdir -p templates/notes
```

`config/settings/base.py`
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [root('templates')],
        # ...
    },
]
```

`templates/note/index.html`
```html
{% if latest_note_list %}
   <ul>
   {% for note in latest_note_list %}
     <li><a href="{% url 'note:detail' note.id %}">{{ note.title }}</a></li>
   {% endfor %}
   </ul>
{% else %}
  <p>No notes are available.</p>
{% endif %}
```

`templates/note/detail.html`
```html
<h1>{{ note.title }}</h1>
<p>{{ note.body }}</p>
```

`notes/tests/tests_views.py`
```python

```

```bash
$ python manage.py startapp accounts
$ touch accounts/urls.py
```

`config/urls.py`
```python
urlpatterns = [
    # Handle the root url.
    url(r'^$', lambda r: HttpResponseRedirect('notes/')),

    # Admin
    url(r'^admin/', include('admin_honeypot.urls', namespace='admin_honeypot')),
    url(r'^secret/', admin.site.urls),

    # Accounts app
    url(r'^accounts/', include('accounts.urls', namespace="accounts")),
 
    # Notes app
    url(r'^notes/', include('note.urls', namespace="note")),
]
```

```bash
$ touch accounts/urls.py
```

`accounts/urls.py`
```python
from django.conf.urls import url
from django.contrib.auth import views

urlpatterns = [
    url(r'^login/$', views.login, name='login'),
    url(r'^logout/$', views.logout, name='logout'),
]
```

`templates/registration/login.html`
```html
<form action="{% url 'accounts:login' %}" method="post" accept-charset="utf-8">
  {% csrf_token %}
  {% for field in form %}
    <label>{{ field.label }}</label>
    {% if field.errors %}
      {{ field.errors }}
    {% endif %}
    {{ field }}
  {% endfor %}
  <input type="hidden" name="next" value="{{ next }}" />
  <input class="button small" type="submit" value="Submit"/>
</form>
```

`notes/views.py`
```python
# ...
from django.contrib.auth.decorators import login_required

@login_required
def index(request):
    # ...


@login_required
def detail(request):
    # ...
```


```bash
$ mkdir static
$ cd static
$ wget https://github.com/twbs/bootstrap/releases/download/v4.0.0-alpha.6/bootstrap-4.0.0-alpha.6-dist.zip
$ unzip bootstrap-4.0.0-alpha.6-dist.zip
$ mv bootstrap-4.0.0-alpha.6-dist bootstrap
$ cd ..
```

`config/settings/base.py`
```python
# ...
STATIC_URL = '/static/'
STATICFILES_DIRS = [
    root('static'),
]
```

```bash
# Fix me
wget https://github.com/sixfeetup/ElevenNote/raw/master/templates-ch06.zip
unzip templates-ch6.zip   # If prompted to replace, say (Y)es
cd ..
```


`accounts/views.py`
```python
from django.contrib.auth import authenticate, login
from django.contrib.auth.forms import UserCreationForm
from django.views.generic import FormView


class RegisterView(FormView):
    template_name = 'registration/register.html'
    form_class = UserCreationForm
    success_url='/'
    
    def form_valid(self, form):
        #save the new user first
        form.save()
        
        #get the username and password
        username = self.request.POST['username']
        password = self.request.POST['password1']
        
        #authenticate user then login
        user = authenticate(username=username, password=password)
        login(self.request, user)
        return super(RegisterView, self).form_valid(form)
```

`accounts/urls.py`
```python
from django.conf.urls import url
from django.contrib.auth import views as auth_views
from django.core.urlresolvers import reverse_lazy

from .views import RegisterView

urlpatterns = [
    url(r'^login/$', auth_views.login, name='login'),
    url(r'^logout/$', auth_views.logout, {"next_page" : reverse_lazy('login')}, name='logout'),
    url('^register/', RegisterView.as_view(), name='register'),
]
```


`config/settings/base.py`
```python
# ...
LOGIN_REDIRECT_URL = '/'
```

### Добавляем API с помощью DRF

```bash
$ pip install djangorestframework
$ python manage.py api
```

`config/settings/base.py`
```python
INSTALLED_APPS = [
    # ...
    'rest_framework',
    # ...
    'api',
]
```