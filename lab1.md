# Лабораторная работа №1

### Шифр Цезаря

Шифр Цезаря является одним из самых простых методов шифрования и заключается в сдвиге всех букв алфавита на три символа вперед:

```
A -> D, B -> E, C -> F, и так далее
```

и, соответственно, три последних буквы алфавита:

```
X->A, Y->B, Z->C
```

Используя шифр Цезаря, слово `PYTHON` будет закодировано следующим образом:

```
PYTHON
||||||
SBWKRQ
```

Ваша задача написать две функции:

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

<div class="alert alert-info">
<strong>Подсказка</strong>: Воспользуйтесь встроенными функциями <tt>ord()</tt> и <tt>chr()</tt>. Функция <tt>ord()</tt> позволяет получить код для указанного символа, например:<br/>
<tt>
>>> ord('A')<br/>
65<br/>
</tt>
Функция <tt>chr()</tt> работает наоборот - возвращает символ по его коду:<br>
<tt>
>>> chr(65)<br/>
'A'</tt>
</div>