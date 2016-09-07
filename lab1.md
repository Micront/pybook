# Лабораторная работа №1

### Шифр Цезаря

[Шифр Цезаря](https://ru.wikipedia.org/wiki/Шифр_Цезаря) является одним из самых простых методов шифрования и заключается в сдвиге всех букв алфавита на три символа вперед:

```
A -> D, B -> E, C -> F, и так далее
```

и, соответственно, три последних буквы алфавита:

```
X -> A, Y -> B, Z -> C
```

Используя шифр Цезаря, слово `PYTHON` будет закодировано следующим образом:

```
PYTHON
||||||
SBWKRQ
```

Ваша задача написать тело для следующих двух функций:

```python
def encrypt_caesar(plaintext):
    """
    >>> encrypt_caesar("PYTHON")
    'SBWKRQ'
    >>> encrypt_caesar("python")
    'sbwkrq'
    >>> encrypt_caesar("")
    ''
    """
    # PUT YOUR CODE HERE
    return ciphertext


def decrypt_caesar(ciphertext):
    """
    >>> decrypt_caesar("SBWKRQ")
    'PYTHON'
    >>> decrypt_caesar("sbwkrq")
    'python'
    >>> decrypt_caesar("")
    ''
    """
    # PUT YOUR CODE HERE
    return plaintext
```

В результате переменные `ciphertext` и `plaintext` должны содержать зашифрованное и расшифрованные сообщения, соответственно.

<div class="alert alert-info">
<strong><a href="https://mail.python.org/pipermail/python-win32/2005-April/003100.html">Подсказка</a></strong>: Воспользуйтесь встроенными функциями <tt>ord()</tt> и <tt>chr()</tt>. Функция <tt>ord()</tt> позволяет получить код для указанного символа, например:<br/>
<tt>
>>> ord('A')<br/>
65<br/>
</tt>
Функция <tt>chr()</tt> работает наоборот - возвращает символ по его коду:<br>
<tt>
>>> chr(65)<br/>
'A'</tt>
</div>

Проверить работу функций можно с помощью примеров, которые приведены в [доктестах](https://docs.python.org/3.5/library/doctest.html) (текст внутри функции, который заключен в тройные кавычки и похож на работу с интерпретатором в интерактивном режиме). Запустить доктесты можно с помощью следующей команды (при условии, что файл с программой называется `caesar.py`):

```sh
$ python3 -m doctest -v caesar.py
```

### Шифр Виженера

[Шифр Виженера](https://ru.wikipedia.org/wiki/Шифр_Виженера) очень похож на шифр Цезаря, за тем исключением, что каждый символ сообщения сдвигается на значение, которое определяется ключом. Ключ это слово, так что каждый символ этого слова указывает на сколько позиций должен быть сдвинут соответствующий символ в шифруемом сообщении, так `A` означает сдвиг соответствующего символа на `0`, `B` на `1` и т.д.

Если длина ключа меньше чем слова, которое подлежит шифрованию, то ключ повторяется необходимое число раз, например:

```
Простой текст:           ATTACKATDAWN
Ключ:                    LEMONLEMONLE
Зашифрованный текст:     LXFOPVEFRNHR
```

Ваша задача написать тело для следующих двух функций так, чтобы переменные `ciphertext` и `plaintext` содержали зашифрованное и расшифрованные сообщения, соответственно.

```python
def encrypt_vigenere(plaintext, keyword):
    """
    >>> encrypt_vigenere("PYTHON", "A")
    'PYTHON'
    >>> encrypt_vigenere("python", "a")
    'python'
    >>> encrypt_vigenere("ATTACKATDAWN", "LEMON")
    'LXFOPVEFRNHR'
    """
    # PUT YOUR CODE HERE
    return ciphertext


def decrypt_vigenere(ciphertext, keyword):
    """
    >>> decrypt_vigenere("PYTHON", "A")
    'PYTHON'
    >>> decrypt_vigenere("python", "a")
    'python'
    >>> decrypt_vigenere("LXFOPVEFRNHR", "LEMON")
    'ATTACKATDAWN'
    """
    # PUT YOUR CODE HERE
    return plaintext
```

<div class="alert alert-info">
Обратите внимание, что символ <tt>A</tt> или <tt>a</tt> в ключе не оказывает никакого влияния на шифруемое сообщение. Если же в качестве ключа мы будем использовать <tt>C</tt> или <tt>c</tt>, то получим шифр Цезаря.
</div>

### RSA шифрование

Одним из современных методов шифрования является алгоритм шифрования RSA, названный так по первым буквам фамилий его  авторов.

Мы не будем здесь вдаваться в [подробности работы](http://kpfu.ru/docs/F366166681/mzi.pdf) этого алгоритма (хотя и рассмотрим техническую часть), но [следюущего объяснения](https://www.quora.com/How-do-you-explain-how-an-RSA-public-key-works-to-a-child) должно быть достаточно, чтобы понимать принципы шифрования с открытым ключом:

> ![](https://qph.ec.quoracdn.net/main-qimg-8ad399b007bf86350675e8cbf5be6e34-c?convert_to_webp=true)
> 
> Show your kid a padlock. This is a kind of lock that locks when you click it (i.e it doesn't require a key) but requires the key to open the lock.
> 
> So, I can send these padlocks to all my friends who want to communicate with me. I will send them only the lock but will keep the key with me.
> 
> My friends can write me messages, put it in a box, lock it with my padlock (by clicking it) and send it to me, even over high risk networks. If the box is intercepted, it's contents will not be compromised since I still have the key with me.
> 
> When the box reaches me, I can open my padlock with my key and read the contents. This way, I can send padlocks (public keys) to people outside which they can use to lock boxes (encrypt messages) without being in danger of the contents being compromised as the padlock key (the private key) is always with me and never exchanged over the network.

Работу алгоритма можно разбить на три шага:
1. Генерация ключей
2. Шифрование
3. Расшифровка

<div class="alert alert-info">От вас в этом задании будет требоваться только выполнить шаг генерации ключей, остальные два шага уже даны (см. исходники к работе).
</div>

На этапе генерации ключей создается два ключа: открытый (public key - ключ, с помощью которого каждый сможет зашифровать сообщение и отправить его нам) и закрытый (private key - ключ, которым мы можем расшифровать полученные сообщения). Для этого выбирается два [простых числа](https://ru.wikipedia.org/wiki/Простое_число) `p` и `q`. Позволим пользователю вводить эти числа, но их нужно будет проверить на простоту. Давайте напишем функцию, которая проверяет является ли число простым:

```python
def is_prime(n):
    """
    >>> is_prime(2)
    True
    >>> is_prime(11)
    True
    >>> is_prime(8)
    False
    """
    # PUT YOUR CODE HERE
    pass
```

<div class="alert alert-info">
Если вы не понимаете как работают функции, то напишите небольшую программу, которая выводит <code>True</code> или <code>False</code>, в зависимости от того является число простым или нет. Затем полученный код скопируйте в приведенную выше функцию (вместо ключевого слова <code>pass</code>) и замените <code>print(True)</code> на <code>return True</code>, а <code>print(False)</code> на <code>return False</code>.
</div>

После того как были выбраны два простых числа находится их произведение `n = p * q` (по ходу объяснения заменяйте комментарий со словами `PUT YOUR CODE HERE` в приведенной ниже функции на соответствующее решение).

```python
def generate_keypair(p, q):
    if not (is_prime(p) and is_prime(q)):
        raise ValueError('Both numbers must be prime.')
    elif p == q:
        raise ValueError('p and q cannot be equal')
    
    # n = pq
    # PUT YOUR CODE HERE

    # phi = (p-1)(q-1)
    # PUT YOUR CODE HERE

    # Choose an integer e such that e and phi(n) are coprime
    e = random.randrange(1, phi)

    # Use Euclid's Algorithm to verify that e and phi(n) are comprime
    g = gcd(e, phi)
    while g != 1:
        e = random.randrange(1, phi)
        g = gcd(e, phi)

    # Use Extended Euclid's Algorithm to generate the private key
    d = multiplicative_inverse(e, phi)
    
    # Return public and private keypair
    # Public key is (e, n) and private key is (d, n)
    return ((e, n), (d, n))
```

Затем вычисляется функция Эйлера по формуе `phi=(p-1)(q-1)`.

Далее выбирается число `e`, отвечающее следующим критериям:
* оно должно быть простое;
* оно должно быть меньше `phi`;
* оно должно быть [взаимно простое](https://ru.wikipedia.org/wiki/Взаимно_простые_числа) с `phi`.

Самым простым способом узнать являются ли два числа взаимно простыми это вычислить наибольший общий делитель, с помощью алгоритма Евклида, и проверить, что он **равен единице**. На этом этапе вашей задачей является реализации алгоритма  Евклида:

```python
def gcd(a, b):
    """
    >>> gcd(12, 15)
    3
    >>> gcd(3, 7)
    1
    """
    # PUT YOUR CODE HERE
    pass
```

Последним этапом на шаге генерации ключей является вычисление `d` такого что `d * e mod phi = 1`. Для его вычисления используется расширенный (обощенный) алгоритм Евклида (см. стр. 23 [этого учебного пособия](http://kpfu.ru/docs/F366166681/mzi.pdf) с подробными объяснениями).

```python
def multiplicative_inverse(e, phi):
    """
    >>> multiplicative_inverse(7, 40)
    23
    """
    # PUT YOUR CODE HERE
    pass
```

Таким образом, полученные пары `(e,n)` и `(d,n)` являются открытым и закрытым ключами соответственно.