# Настрока рабочего окружения

В этой главе будут описаны все инструменты, которые понадобятся для успешной сдачи этого курса. Раздел будет обновляться, так что следите за изменениями и задавайте вопросы (а также отвечайте на вопросы друг друга), если что-то непонятно.

### Установка интерпреатора Python

Вам будет нужен [интерпретатор](http://stackoverflow.com/questions/2377273/how-does-an-interpreter-compiler-work) языка Python (что такое интерпретатор мы разберем на занятии). Где его взять? На [юникс-подобных](https://ru.wikipedia.org/wiki/UNIX-подобная_операционная_система) операционных системах (Linux, MacOS и др) интерпретатор Python, скорее всего, уже установлен, проверить это вы можете набрав в терминале (командной строке) команду python, например:
```sh
$ python
Python 2.7.10 (default, Oct 23 2015, 19:19:21) 
[GCC 4.2.1 Compatible Apple LLVM 7.0.0 (clang-700.0.59.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

Обратите внимание на версию, в моем случае это 2.7.10. Нам она не подходит ("Почему?" - можно почитать [тут](https://wiki.python.org/moin/Python2orPython3) и [тут](http://sebastianraschka.com/Articles/2014_python_2_3_key_diff.html)). Нам нужен интерпретатор третьей версии. Если вы видите что версия вторая, то попробуйте набрать команду python3:
```sh
$ python3
Python 3.5.2 (v3.5.2:4def2a2901a5, Jun 26 2016, 10:47:25) 
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```
Если же появляется сообщение об ошибке, то интерпретор нужно установить. Скачать интерпретор под нужную операционную систему вы можете с официального сайта https://www.python.org/downloads/release/python-352/ (если вы пользователь linux системы, то установить интерпретатор можете из репозитория).
После установке обязательно проверьте, что интерпретор запускается:
```sh
$ python3
>>> print("Hello, World!")
Hello, World!
>>> quit()
```

Это интерактивный режим работы с интерпретатором языка Python ([REPL](https://ru.wikipedia.org/wiki/REPL)). Три символа больше `>>>` являются приглашением к вводу команд, чтобы завершить работу с интерпретатором нужно набрать команду `quit()`.

<div class="alert alert-info">
<strong>Важно</strong>: Если вы устанавливаете Python на Windows, то есть вероятность, что, набрав команду python или python3 после установки, вы получите сообщение об ошибке (скорее всего путь к интерпретатору не указан в переменной окружения PATH). Пожалуйста, прежде чем задавать вопросы "Почему ничего не работает?", прочитайте как правильно все настроить <a href='https://docs.python.org/3/using/windows.html#configuring-python'>https://docs.python.org/3/using/windows.html#configuring-python</a>.
</div>

Я рекомендую установить модуль `bpython`. Это тоже интерпретатор Python, но с дополнительными функциями (подсветка синтаксиса, подсказки и др):
```sh
$ pip3 install bpython
$ python3 -m bpython
```
<div class="alert alert-info">
В этом примере мы использовали pip для установки нового модуля. Иногда устанавливаемый модуль требует других модулей для своей работы, тогда дополнительные модули устанавливаются автоматически (говорят "по зависимостям"). Но может возникнуть ситуация, когда вам придется в ручную установить нужную библиотеку. Если вы не знаете как это сделать, то поищите ответ на <a href="http://stackoverflow.com">stackoverflow.com</a>, скорее всего кто-то уже столкнулся с той же проблемой, что и вы.
</div>

### Выбор редактора кода

Вам будет нужен редактор кода. Это может быть простой текстовый редактор с подсветкой синтаксиса (http://sublimetext.com/3) или же полноценная среда разработки (https://www.jetbrains.com/pycharm/download/). Я вам рекомендую использовать оба варианата (средой разработки пользоваться только после того, как научитесь работать с интерпретатором и системой контроля версий, что будет обязательно для нашего курса).

Для ваших работ заведите себе папку, назовите ее cs102 (можете назвать иначе). В этой папке создайте новый файл, откройте его с помощью редактора SublimeText (или любого другого, который вам больше понравился) с именем hello.py:


Откройте терминал (командную строку), перейдите в папку с созданным файлом (рекомендую вам создать alias, т.е. короткое имя для пути, чтобы в будущем вы всегда могли быстро перейти в эту папку):



Таким образом, мы написали и запустили простейшую программу (скрипт). Запомните эти шаги.

### Система контроля версий

Мы будем пользоваться системой контроля версий (на занятии разберем зачем это нужно, но можете начать читать вот это руководство). Вам нужно зарегистрироваться либо на https://github.com, либо на https://bitbucket.org/. Все изменения, которые будут происходить с вашими работами, могут храниться локально (у вас на компьютере), а могут и удаленно, таким образом вы всегда сможете продолжить работу над своим проектом. На bitbucket есть возможность создать бесплатный приватный (закрытый) репозиторий (хранилище) для вашего проекта. Далее приведен пример с использованием bitbucket (рекомендую проделать эти шаги и на github).

Зарегистрируйтесь на сайте BitBucket.org. По окончанию регистрации вам будет предложено создать новый репозиторий (либо вкладка Repository -> Crete new repository). Укажите следующие параметры для вашего проекта (в дальнейшем их можно будет изменить):



Далее раскройте список со словами I'm starting from scratch:

Достаточно повторить указанные шаги, но вместо файла с разработчиками, мы сделаем коммит (загрузим изменения на сервер) нашей программы hello.py:
```sh
$ git init
Initialized empty Git repository in /Users/user/cs102/.git/
$ git remote add origin https://Dementiy@bitbucket.org/Dementiy/cs102.git
$ git add hello.py 
$ git commit -m 'Initial commit with my first program on Python'

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account`s default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'user@Air-user.(none)')
$ git config --global user.email "Dementiy@yandex.ru"
$ git config --global user.name "Dmitriy"
$ git commit -m 'Initial commit with my first program on Python'
[master (root-commit) b9c8e00] Initial commit with my first program on Python
 1 file changed, 1 insertion(+)
 create mode 100644 hello.py
$ git push -u origin master
Password for 'https://Dementiy@bitbucket.org': 
Counting objects: 3, done.
Writing objects: 100% (3/3), 258 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://Dementiy@bitbucket.org/Dementiy/cs102.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```

Важно: Если после выполнения команды git init вы получили сообщение об ошибке о том, что команда git не найдена, то скорее всего вы работаете под Windows и git нужно установить.  Также обратите внимание, что пришлось выполнить простейшую настройку git с помощью команды git config.

Теперь проверьте на bitbucket, что новый файл появился. Для этого зайдите в раздел "Source":

Этого на текущий момент достаточно.

<div class="alert alert-info">
Если вы не будете задавать вопросы, то к сожалению вам никто не сможет помочь. Piazza является основным средством общения (после лекций и практик), где вы можете задать вопрос, попросить помощи, высказать свои пожелания или недовольства касательно курса. Постарайтесь получить удовольствие от наших занятий. Успехов!
</div>