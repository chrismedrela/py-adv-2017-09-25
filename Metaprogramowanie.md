---
title: Metaprogramowanie - Zaawansowany Python 2017-09-25
---

# Content

[ToC]

# Ćw.: `@limit(n)`

Napisz dekorator `@limit(n)`, który powoduje, że zwracane jest tylko n pierwszych elementów z kolekcji zwróconej przez dekorowaną funkcję.

**Kod początkowy**

```python
def limit(n):
    ...

@limit(6)
def printme(tekst):
    return tekst.upper()
```

**Oczekiwane zachowanie**

```python
>>> printme("abcdefghijkl")
ABCDEF
```

**Rozwiązanie**

```python
def limit(n):
    def decor(fun):
        def wrapper(*args, **kwargs):
            out = fun(*args, **kwargs)
            return out[:n]
        return wrapper
    return decor
```

# Przerwa do 13:30

# Ćw.: Klasa jako dekorator 

Napisz dekorator `@cached`, który cacheuje wywołania funkcji z tymi samymi argumentami.

**Kod początkowy**

```python
class Cached:
    def __init__(...):
        ...

    def __call__(...):
        ...

@Cached
def fibonacci(n):
    "Return the nth fibonacci number."
    if n in (0, 1):
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

**Oczekiwane zachowanie**

```python
>>> fibonacci(100)
354224848179261915075
```

**Podpowiedź - dekorator `@shout`**

```python
class shout:
    def __init__(self, f):
        print("inside decorator's __init__()")
        self.f = f

    def __call__(self):
        print("before call")
        self.f()
        print("after call")

@shout
def function():
    print('inside')
```

**Rozwiązanie**

```python
class Cached:
    def __init__(self, function):
        self.fun = function
        self.cache = {}

    def __call__(self, *args):
        try:
            return self.cache[args]
        except KeyError:
            print('__call__', self, args)
            value = self.fun(*args)
            self.cache[args] = value
            return value
        except TypeError:
            return self.fun(*args)
```

# Menadżery kontekstu - kontrakt

```python
class Context:
    def __init__(self):
        print('__init__()')

    def __enter__(self):
        print('__enter__()')
        return 42

    def __exit__(self, exc_type, exc_val, exc_tb):
        print('__exit__()')
        
c = Context()  # c = Context.__init__()
print('before with')
with c as cc:  # cc = c.__enter__()
    print('doing work in the context')
    print('c = ', c)
    print('cc = ', cc)
print('after with')
```

```
__init__()
before with
__enter__()
doing work in the context
c =  <__main__.Context object at 0x7f52d8139048>
cc =  42
__exit__()
after with
```

# Ćw.: `SupressOutput`

Napisz kontekst menedżer `SupressOutput`, który powoduje, że w jego środku instrukcje `print` przestają wypisywać na standardowe wyjście.
Hint: zastąp `sys.stdout.write` przez funkcję `blackhole`.

**Kod początkowy**

```python
import sys

def blackhole(*args, **kwargs):
    pass

class SuppressOutput:
    ...

so = SuppressOutput()  # so = SuppressOutput.__init__()
with so as stdout_write:  # stdout_write = so.__enter__()
    print('That won\'t be printed')
    stdout_write('But this one will be printed\n')
```

**Oczekiwane wyjście**

    SuppressOutput.__enter__()
    But this one will be printed
    __exit__()

**Rozwiązanie**

```python
class SuppressOutput:
    def __enter__(self):
        print('Context.__enter__()')
        self.write, sys.stdout.write = sys.stdout.write, blackhole
        return self.write

    def __exit__(self, exc_type, exc_val, exc_tb):
        sys.stdout.write = self.write
        print('__exit__()')
```

# Menadżery kontekstu - dokładny opis zachowania

```python
with EXPR as VAR:
    BLOCK
```
jest równoważne

```python
mgr = (EXPR)
exit = type(mgr).__exit__  # Not calling it yet
value = type(mgr).__enter__(mgr)
exc = True
try:
    try:
        VAR = value  # Only if "as VAR" is present
        BLOCK
    except:
        # The exceptional case is handled here
        exc = False
        if not exit(mgr, *sys.exc_info()):
            raise
        # The exception is swallowed if exit() returns true
finally:
    # The normal and non-local-goto cases are handled here
    if exc:
        exit(mgr, None, None, None)
```

# Ćw.: `@contextmanager` dla `SuppressOutput`

Przepisz klasę `SuppressOutput` używając `@contextmanager`.

**Kod początkowy**

```python
from contextlib import contextmanager
import sys

def blackhole(*args, **kwargs):
    pass

@contextmanager
def suppress_output():
    ...

with suppress_output() as write:
    print("This won't be printed")
    write("This will be printed")
```

**Oczekiwane wyjście**

    This will be printed
    
**Podpowiedź**

```python
from contextlib import contextmanager

@contextmanager
def Shouter():
    print('begin')
    try:
        yield 42
    except Exception:
        print('exception')
    print('end')

with Shouter() as c:
    print('inside')
    print(c)
```

```
begin
inside
42
end
```


**Rozwiązanie**

```python
@contextmanager
def suppress_output():
    write, sys.stdout.write = sys.stdout.write, blackhole
    try:
        yield write
    finally:
        sys.stdout.write = write
```