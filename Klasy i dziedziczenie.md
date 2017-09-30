---
title: Klasy i dziedziczenie - Zaawansowany Python 2017-09-25
---

# Content

[ToC]

# Deskryptory

```python
import random

class Die:
    def __init__(self, sides=6):
        self.sides = sides
        
    def __get__(self, instance, owner):
        print('self =', self)
        print('instance =', instance)
        print('owner =', owner)
        return int(random.random() * self.sides) + 1

    def __set__(self, instance, value):
        print('self =', self)
        print('instance =', instance)
        print('value =', value)

    def __delete__(self, instance):
        print('self =', self)
        print('instance =', instance)

class Game:
    d6 = Die()
    d10 = Die(sides=10)
    d20 = Die(sides=20)
```

# FloatField & Point

```python
class FloatField:
    def __init__(self, field_name):
        self.field_name = field_name
        
    def __get__(self, instance, owner):
        return float(instance._json[self.field_name])

class Point:
    def __init__(self, json):
        self._json = json
        
    x = FloatField('u')
    y = FloatField('v')
    
p = Point({'u': 42, 'v': 18.0})
p.x
```

# Ćw.: Deskryptor `@property`

Napisz deskryptor `Property`, który zachowuje się podobnie do wbudowanego `@property`.

**Kod początkowy**

```python
class Property:
    def __init__(self, fget=None, fset=None):
        ...

    def __get__(...):
        ...

    def __set__(...):
        ...


class MyClass(object):
    def __init__(self, readonly_value):
        self._readonly_value = readonly_value
        self._value = 0

    def _get_readonly_value(self):
        return self._readonly_value

    readonly_value = Property(fget=_get_readonly_value)

    def _get_value(self):
        return self._value

    def _set_value(self, value):
        self._value = int(value)

    value = Property(fget=_get_value, fset=_set_value)
```

**Oczekiwane zachowanie**

```python
>>> d = MyClass("readonly value")
>>> print(d.readonly_value)
readonly value
>>> print(d.value)
0
>>> d.value = '3'
>>> print(d.value)
3
```

**Rozwiązanie**

```python
class Property(object):
    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        self.__doc__ = doc

    def __get__(self, instance, owner):
        if instance is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(instance)

    def __set__(self, instance, value):
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(instance, value)

    def __delete__(self, instance):
        if self.fdel is None:
            raise AttributeError("can't delete attribute")
        self.fdel(instance)
```

# Przerwa do 10:40

# Ćw.: `ComparableMixin`

W przypadku porównywania obiektów, wywoływana jest jedna z sześciu specjalnych metod (``__lt__``, ``__le__``, ``__eq__``, ``__ne__``, ``__gt__`` lub ``__ge__``).
Wystarczy jednak zdefiniować dwie z nich (np. ``__le__`` i ``__eq__``), a pozostałe porównania to odpowiednia kombinacja tych dwóch metod:

    a != b <==> not (a == b)
    a < b  <==> (a <= b) and not (a == b)
    a > b  <==> not (a <= b)
    a >= b <==> (a == b) or not (a <= b)

Zaimplementuj i przetestuj klasę domieszkową ``ComparableMixin``.
Jej użycie powoduje, że wystarczy zdefiniować metody ``__le__`` i ``__eq__``, aby obiekty klasy dziedziczącej po ``ComparableMixin`` mogły być porównywane.

**Kod początkowy**

```python
class ComparableMixin:
    ...

class MyInteger(ComparableMixin): 
    def __init__(self, i):
        self.i = i
    def __le__(self, other):
        return self.i <= other.i
    def __eq__(self, other):
        return self.i == other.i
```

**Oczekiwane zachowanie**

```python
>>> MyInteger(1) > MyInteger(0)
True
```

**Rozwiązanie**

```python
# class ComparableMixin(object):  # Python 2
class ComparableMixin:
    def __ne__(self, other):
        return not (self == other)
    def __lt__(self, other):
        return self <= other and (self != other)
    def __gt__(self, other):
        return not self <= other
    def __ge__(self, other):
        return self == other or self > other
```

# `CountDict`

```python
class CountDict(dict):
    def __getitem__(self, key):
        if key in self:
            return super(CountDict, self).__getitem__(key)
        else:
            return 0
```

# Ćw.: `DefaultDict`

Zaimplementuj klasę `DefaultDict`, która zachowuje się jak zwykły słownik, ale w konstruktorze przyjmuje funkcję generującą domyślne wartości dla nieistniejących kluczy.

**Kod początkowy**

```python
class DefaultDict(dict):
    ...
```

**Oczekiwane zachowanie**

```python
>>> d = DefaultDict(list)
>>> d['a']
[]
>>> d['b'].append(2)
>>> d['b']
[2]
```

**Rozwiązanie**

```python
class DefaultDict(dict):
    def __init__(self, init_value_fabric):
        self._init_value_fabric = init_value_fabric

    def __getitem__(self, key):
        try:
            return super().__getitem__(key)
        except KeyError:
            self[key] = self._init_value_fabric()
            return self[key]
```

# Ćw.: `MetaString`

Zaimplementuj klasę `MetaString`, która zachowuje się dokładnie tak samo, jak zwykły `str`, ale posiada dodatkowy atrybut `meta`, który można przekazać w konstruktorze.

**Kod początkowy**

```python
class MetaString(str):
    ...
```

**Oczekiwane zachowanie**

```python
>>> metastring = MetaString("Hello World", "D. Richie")
>>> metastring
Hello World
>>> metastring.meta
D. Richie
```

**Nieprawidłowe rozwiązanie**

```python
class BadMetaString(str):
    def __init__(self, value, info):
        super().__init__(self, value)
        self.meta = info
```

**Rozwiązanie**

```python
class MetaString(str):
    __slots__ = ['meta']
    
    def __new__(cls, value, info):
        obj = str.__new__(cls, value)
        obj.meta = info
        return obj
```

# [Meta Kwargs (blog post)](https://blog.ionelmc.ro/2015/02/09/understanding-python-metaclasses/)
