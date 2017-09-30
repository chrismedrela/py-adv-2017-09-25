---
title: Funkcje - Zaawansowany Python 2017-09-25
---

# Content

[ToC]

# Ćw.: Fibbonacci

```python
class CallableClass:  # (object) w Pythonie 2
    def __init__(self):
        self.counter = 0
        
    def __call__(self):
        self.counter += 1
        print(self.counter)


def fibbonacci(n):
    if n == 0:
        return 0
    elif n == 1:
        return 1
    else:
        return fibbonacci(n-1) + fibbonacci(n-2)
```

**Rozwiązanie**

```python
class Fibbonacci:  # (object)
    def __init__(self):
        self.cache = {0: 1, 1: 1}
        
    def __call__(self, n):
        try:
            return self.cache[n]
        except KeyError:
            retval = self(n-1) + self(n-2)
            self.cache[n] = retval
            return retval
```

# Ćw.: Klasy jako funkcje

Przekształć poniższą klasę na funkcję:

```python
class CallableClass:
    def __init__(self):
        self._counter = 0
    
    def __call__(self):
        self._counter += 1
        print("You have called me {0} times".format(self._counter))
```

**Kod początkowy**

```python
def callable_func():
    ...
```

**Oczekiwane zachowanie**

```python
>>> f = callable_func()
>>> f()
You have called me 1 times
>>> f()
You have called me 2 times
>>> f()
You have called me 3 times
>>> f2 = callable_func()
>>> f2()
You have called me 1 times
>>> f()
You have called me 4 times
```

**Rozwiązanie**

```python
def callable_func():
    counter = 0
    def inner():
        nonlocal counter
        counter += 1
        print("You have called me {0} times".format(counter))
    return inner
```

# Ćw.: Lambda

```python
lista = [5, 3, 8, 0]
list(map(lambda x: x+1, lista))
list(filter(lambda x: x%2==0, lista))

# Hint
[_ for _ in _]
[_ for _ in _ if _]
```

**Rozwiązanie**

```python
[x+1 for x in lista]
[x for x in lista if x%2==0]
```

# Przerwa do 13:30

# Ćw.: `globals()` i `locals()`

Zmodyfikuj wartość zmiennej globalnej używając `globals()`.
Uniknij użycia bezpośredniego przypisania `x = ...`.

```python
>>> x = 2
>>> g = globals()
>>> ...
>>> x
42
```

Następnie umieść powyższy kod w funkcji, tak że `x` jest zmienną lokalną.
Użyj `locals()` zamiast `globals()`.

# Ćw.: Parametry nazwane

Napisz funkcję `dict_without_Nones`, która zachowuje się tak samo jak `dict`, ale usuwa klucze, dla których wartość jest `None`.
Zadanie rozwiąż na dwa sposoby:

1. używając pętli for,
2. i używając dictionary comprehension: `{_: _ for _ in _ if _}`,
3. używając funkcji `filter` i `map` (opcjonalnie).

**Oczekiwane zachowanie**

```python
>>> dict_without_Nones(a="1999", b="2000", c=None)
{'a': '1999', 'b': '2000'}
```

**Rozwiązanie 1**

```python
def dict_without_Nones(**kwargs):
    result = {}
    for k, v in kwargs.items():
        if v is not None:
            result[k] = v
    return result
```

**Rozwiązanie 2**

```python
def dict_without_Nones(**kwargs):
    result = {k, v for k, v in kwargs.items() if v is not None}
```

**Rozwiązanie 3**

```python
def dict_without_Nones(**kwargs):
    ignore_None = lambda key_value: key_value[1] is not None 
    return dict(filter(ignore_None, kwargs.items()))
```

**Unpack 2to3**

```diff
--- x.py        (original)
+++ x.py        (refactored)
@@ -1 +1 @@
-print (lambda (a,b):a)((1,2))
+print((lambda a_b:a_b[0])((1,2)))
```

[PEP3113 wyjaśniający dlaczego usunięto unpacking w sygnaturze funkcji](https://www.python.org/dev/peps/pep-3113/)

# Ćw.: Pułapka domyślnego atrybutu

Napisz funkcję `append_func`, która dodaje element do listy.
Element oraz lista powinny być jedynymi argumentami tej funkcji.
Lista powinna być argumentem opcjonalnym.
Jeżeli nie jest podana, należy utworzyć pustą listę.
Hint: użyj `list.append(element)`.

**Oczekiwane zachowanie**

```python
>>> append_func(3, [1, 2])
[1, 2, 3]
>>> append_func('c', ['a', 'b'])
['a', 'b', 'c']
>>> append_func('e')
['e']
>>> append_func('f')
['f']
```

**Nieprawidłowe rozwiązanie**

```python
def append_func(item, seq=[]):
    seq.append(item)
    return seq
```

**Rozwiązanie**

```python
def append_func(item, seq=None):
    if seq is None:
        seq = []
    seq.append(item)
    return seq
```

**Prawie prawidłowe rozwiązanie 1**
```python
def append_func(x, y=None):
    return (y or []) + [x]
```

**Prawie prawidłowe rozwiązanie 2**
```python
def append_func(x, y=[]):
    return y + [x]
```


```
> sorted?
Signature: sorted(iterable, /, *, key=None, reverse=False)
```

# Zapychacz pamięci
```python
import time


class C:
    def __init__(self):
        self.x='0'*1000*1000*100

    def __del__(self):
        raise Exception
f=[]
while True:
    f.append(C())
    time.sleep(1)
```
