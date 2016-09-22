# Лабораторная работа №4. Работа с API


Эта работа посвящена взаимодействию с [API ВКонтакте](API ВКонтакте). Чтобы начать работать с API от вас требуется зарегистрировать новое приложеие. Для этого зайдите на форму создания нового Standalone приложения [https://vk.com/editapp?act=create](https://vk.com/editapp?act=create) и следуйте инструкциям. Вашему приложению будет назначен идентификатор, который потребуется для выполнения работы.

Запросы к API ВКонтакте имеют следующий синтаксис ([из документации](https://vk.com/dev/api_requests)):
```
https://api.vk.com/method/METHOD_NAME?PARAMETERS&access_token=ACCESS_TOKEN&v=V
```

где:
* `METHOD_NAME` - это название метода API, к которому Вы хотите обратиться.
* `PARAMETERS` - входные параметры соответствующего метода API, последовательность пар `name=value`, разделенных амперсандом.
* `ACCESSS_TOKEN` - ключ доступа.
* `V` - используемая версия API (в настоящий момент 5.53).


Например, чтобы получить список друзей, с указанием их пола, нужно выполнить следующий запрос:
```
https://api.vk.com/method/friends.get?fields=sex&access_token=0394a2ede332c9a13eb82e9b24631604c31df978b4e2f0fbd2c549944f9d79a5bc866455623bd560732ab&v=5.53
```

<div class="alert alert-danger">
<strong>Внимание:</strong> Токен доступа ненастоящий, поэтому этот запрос работать не будет.
</div>

Чтобы получить токен доступа вы можете воспользоваться написанным для вас скриптом `access_token.py` следующим образом:

```sh
$ python access_token.py YOUR_CLIENT_ID -s friends,messages
```

где вместо `YOUR_CLIENT_ID` необходимо подставить идентификатор вашего приложения.

После выполнения команды откроется новая вкладка браузера, из адресной строки которого вам необходимо скопировать токен доступа.

<div class="alert alert-info">
<strong>Внимание:</strong> На этом этапе вы можете повторить ранее представленный пример запроса, чтобы убедится, что вы делаете все верно.
</div>

Далее приведено содержимое файла `access_token.py`:
```python
import webbrowser
import argparse


def get_access_token(client_id, scope):
    assert isinstance(client_id, int), 'clinet_id must be positive integer'
    assert isinstance(scope, str), 'scope must be string'
    assert client_id > 0, 'clinet_id must be positive integer'
    url = """\
    https://oauth.vk.com/authorize?client_id={client_id}&\
    redirect_uri=https://oauth.vk.com/blank.hmtl&\
    scope={scope}&\
    &response_type=token&\
    display=page\
    """.replace(" ", "").format(client_id=client_id, scope=scope)
    webbrowser.open_new_tab(url)


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("client_id", help="Application Id", type=int)
    parser.add_argument("-s",
                        dest="scope",
                        help="Permissions bit mask",
                        type=str,
                        default="",
                        required=False)
    args = parser.parse_args()
    get_access_token(args.client_id, args.scope)

```


**Задание №1**. Требуется написать функцию прогнозирования возраста пользователя по возрасту его друзей. 

```python
def get_friends(user_id, fields):
    """ Returns a list of user IDs or detailed information about a user's friends """
    assert isinstance(user_id, int), "user_id must be positive integer"
    assert isinstance(fields, str), "fields must be string"
    assert user_id > 0, "user_id must be positive integer"
    # PUT YOUR CODE HERE
    pass
```

Для выполнения этого задания нужно получить список всех друзей для указанного пользователя, отфильтровать тех у кого возраст не указан или указаны только день и месяц рождения.

Для выполнения запросов к API мы будем использовать библиотеку `requests`:

```sh
$ pip3 install requests
```

Список пользователей можно получить с помощью метода [`friends.get`](https://vk.com/dev/friends.get). Ниже приведен пример обращения к этому методу для получения списка всех друзей указанного пользователя:

```python
domain = "https://api.vk.com/method"
access_token = # PUT YOUR ACCESS TOKEN HERE
user_id = # PUT USER ID HERE

query_params = {
    'domain' : domain,
    'access_token': access_token,
    'user_id': user_id
}

query = "{domain}/friends.get?access_token={access_token}&user_id={user_id}&v=5.53".format(**query_params)
response = requests.get(query)
```