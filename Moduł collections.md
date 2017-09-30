---
title: Moduł `collections` - Zaawansowany Python 2017-09-25
---

# Content

[ToC]

# Ćw.: Point2D

Napisz klasę `Point2D`, która posiada metodę `distance`.

**Kod początkowy**

```python
from collections import namedtuple
from math import sqrt

class Point2D(...):
    ...
    def distance_to(...):
        ...
```

**Oczekiwane zachowanie**

```python
>>> p1 = Point2D(2, 3)
>>> p2 = Point2D(-1, 7)
>>> p1.distance_to(p2)
5.0
```

**Rozwiązanie**

```python
BasePoint2D = namedtuple('Point2D', 'x y')

class Point2D(BasePoint2D):
    __slots__ = ()
    def distance_to(self, other):
        return sqrt((self.x-other.x)**2 + (self.y-other.y)**2)
```

# Ćw.: Średnia krocząca

Napisz funkcję ``moving_average``, która wylicza średnią kroczącą z przekazanej listy.
[Artykuł na Wikipedii wyjaśniający pojęcie średniej kroczącej.](https://pl.wikipedia.org/wiki/%C5%9Arednia_krocz%C4%85ca)

**Kod początkowy**

```python
from collections import deque

def moving_average(l, n=3):
    ...
```

**Oczekiwane zachowanie**

```python
>>> moving_average(list(range(3,10)))
[4.0, 5.0, 6.0, 7.0, 8.0]
>>> moving_average([40, 30, 50, 46, 39, 44])
[40.0, 42.0, 45.0, 43.0]
```

**Rozwiązanie**

```python
from collections import deque

def moving_average(l, n=3):
    d = deque(l[:n-1])
    d.appendleft(0)
    s = sum(d)
    res = []
    for elem in l[n-1:]:
        s += elem - d.popleft()
        d.append(elem)
        res.append(s / float(n))
    return res
```

**Rozwiązanie 2**
```python
from collections import deque

def moving_average(l, n=3):
    l = iter(l)
    d = deque()
    s = 0
    
    res = []
    
    for i in range(n):
        el = next(l)
        d.append(el)
        s += el
        
    for el in l:
        res.append(s / n)
        s -= d.popleft()
        s += el
        d.append(el)
        
    return res
    
>>> moving_average(range(3, 100))
```

**Rozwiazanie 3**

```python
def moving_average(l, n=3):
    max_i = len(l) - n + 1
    result_list = []

    for i in range(0, max_i):
        sum = 0
        for index in range(i, i+n):
            sum += l[index]

        result_list.append(sum / n)
    
    return result_list
```

**Rozwiazanie 4**

```python
def moving_average(l, n=3):
    return_list = []
    scope = deque()
    prev_sum = None
    for i in l:
        scope.append(i)
        if len(scope) == n and not prev_sum:
            prev_sum = sum(scope)
            return_list.append(prev_sum / n)
            continue
        if len(scope) < n:
            continue
        prev_sum -= scope.popleft()
        prev_sum += scope[-1]
        return_list.append(prev_sum / n)
    return return_list

moving_average(range(3, 10))
```

**Rozwiązanie 5**
```python
def moving_average(a, n=3):
    it = iter(a)
    x = []
    p = 0
    for i in range(n):
        x.append(next(it))
    s = sum(x)
    for i in it:
        yield s / n
        s -= x[p]
        s += i
        x[p] = i
        p += 1
        p %= n
    yield s / n
```

**Rozwiązanie 6**
```python
def moving_average(l, n=3):
    result = []
    initial_sum = 0
    for element in range(n):
        initial_sum += l[element]
    initial_average = initial_sum/n
    result.append(initial_average)
    popped = l.pop(0)
    sum = initial_sum
    for i in range (len(l)-(n-1)):
        sum -= popped
        sum += l[n-1]
        average = sum/n
        result.append(average)
        popped = l.pop(0)
    return result
```

# Ćw.: Zliczanie liter

1. Zlicz jak często występują litery w zadanym ciągu znaków.
   Użyj ``defaultdict`` do zapamiętania, ile razy wystąpiły poszczególne litery.
2. Wykonaj to samo ćwiczenie używając klasy Counter.
3. Korzystając z rezultatu metody ``most_common``, stwórz ``OrderedDict`` pamiętający, ile razy wystąpiły poszczególne litery, w kolejności od najczęściej występującej.

**Rozwiązanie**

```python
from collections import defaultdict, Counter
s = "abcdefgabcaa"

d = defaultdict(int)
for char in s:
    d[char] += 1
print(d)

c = Counter(s)
print(c.most_common())

dd = OrderedDict(c.most_common())
print(dd)
```

**Oczekiwane wyjście**

    defaultdict(<class 'int'>, {'b': 2, 'c': 2, 'f': 1, 'a': 4, 'd': 1, 'g': 1, 'e': 1}) 
    [('a', 4), ('b', 2), ('c', 2), ('f', 1), ('d', 1), ('g', 1), ('e', 1)]
    OrderedDict([('a', 4), ('b', 2), ('c', 2), ('f', 1), ('d', 1), ('g', 1), ('e', 1)])

# Przerwa do 15:35