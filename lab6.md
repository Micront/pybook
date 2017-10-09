# Лабораторная работа №?. Дерево отрезков

Имеетя строка, состоящая из круглых скобок, например, каждый символ такой строки это либо открывающая круглая скобка `(`, либо закрывающая `)`. Задан интервал `[l , r]`, для которого необходимо найти самую длинную сбалансированную подпоследовательность скобок. Подпоследовательность считается сбалансированной, если для каждой открывающей скобки по индексу `i` в подпоследовательности существует одна закрывающая скобка по индексу `j`, так что `j > i`.

Примеры:

```
# Пример 1
string:  ())))()
index:   1234567

# Наиболее длинная сбалансированная подпоследовательность
string:  ()()
index:   1267

# Пример 2
string:  )(()())(
index:   12345678

# Наиболее длинная сбалансированная подпоследовательность
string:  (()())
index:   234567

# Пример 3 (для этого примера не существуюет подпоследовательности)
string:  )(
index:   12
```

Пример решения задачи:

```python
def query(string, left, right):
    opened = 0
    answer = 0
    for i in range(left, right + 1):
        if string[i] == '(':
            opened += 1
        elif string[i] == ')' and opened > 0:
            opened -= 1
            answer += 1
    return answer


if __name__ == "__main__":
    brackets = input('String? ')
    queries = int(input('Queries? '))
    for i in range(queries):
        l, r = map(int, input('Interval? ').split())
        print(query(brackets, l - 1, r - 1))
```

```sh
$ python3 brackets.py
String? )(()())(
Queries? 3
Interval? 0 7
1
Interval? 3 6
1
Interval? 0 2
0
```

Очевидно, что это решение не является оптимальным, ваша задача предоставить лучшее решение, используя струкутру данных [Дерево отрезков](https://ru.wikipedia.org/wiki/Дерево_отрезков).

```python
class SegmentTree:
    def __init__(self, string):
        """ Class initialization """
        # initialization code goes here
        self.__build(0, 0, len(string) - 1)

    def __build(self, current, left, right):
        """ Private function to build the tree """
        # build function code goes here
        pass

    def __query(self, current, left, right, l, r):
        """ Private version of query function """
        # query function code goes here
        pass

    def query(self, l, r):
        """ Public version of query function """
        q =  self.__query(0, 0, len(self.string) - 1, l - 1, r - 1)
        # return the answer for the given query
        pass
```

```sh
$ python3 -i brackets_seg_tree.py
>>> brackets = input()
)(()())(
>>> segment_tree = SegmentTree(brackets)
>>> segment_tree.query(1, 5)
1
>>> segment_tree.query(1, 8)
3
```