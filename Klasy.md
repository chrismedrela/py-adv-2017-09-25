---
title: Klasy - Zaawansowany Python 2017-09-25
---

# Content

[ToC]

# Ćw.: Własny słownik

Napisz klasę zachowującą się jak słownik, ale wykorzystującą składnię `dict.key` zamiast `dict[key]`.

**Kod początkowy**

```python
class Record:
    def __init__(self):
        ...

    def __getattr__(...):
        ...

    ...
```

**Oczekiwane zachowanie**

```python
>>> person = Record()
>>> person.first_name = 'John'
set first_name to John
>>> print(person.first_name)
get first_name
John
>>> del person.first_name
del first_name
```

**Rozwiązanie**

```python
class Record:  # (object)
    def __init__(self):
        # Nie możemy użyć poniższego kodu:
        #     self._dict = {}
        # Ponieważ zakończyłby się on rekurencyjnym wywoływaniem metody __setattr__
        super().__setattr__('_dict', {})

    def __getattr__(self, name):
        print('get', name)
        return self._dict[name]

    def __setattr__(self, name, value):
        print('set', name, 'to', value)
        self._dict[name] = value

    def __delattr__(self, name):
        print('del', name)
        del self._dict[name]
```



```
class Record:
    def __init__(self):
        self.__dict__['d'] = {}

    def __getattr__(self, attr):
        print('get ' + attr)
        return self.d[attr]

    def __setattr__(self, attr, val):
        print('set ' + str(attr) + ' to ' + str(val))
        self.d[attr] = val
        
    def __delattr__(self, attr):
        print('del ' + attr)
```

```
class Record(dict):

    def __setattr__(self, item, value):
        print('set ' + str(item) + ' to ' + str(value))
        self.__dict__[item] = value

    def __getattr__(self, item):
        print('get ' + str(item))
        return self.__dict__[item]

    def __delattr__(self, item):
        print('del ' + item)
        del self.__dict__[item]
```

# Ćw.: Klasa `Date`

Napisz klasę `Date`, którą można utworzyć na dwa sposoby: 

1. wywołując "konstruktor" `__init__` i przekazując trzy liczby (dzień, miesiąc i rok) 
1. wywołując metodę klasy `from_string` i przekazując napis reprezentujący datę w formacie `DD-MM-YYYY`.

Hint: do sparsowania daty użyj `str.split('-')`.

**Kod początkowy**

```python
class Date:
    def __init__(self, day, month, year):
        self.day = day
        self.month = month
        self.year = year

    @classmethod
    def from_string(cls, date_as_string):
        day, month, year = date_as_string.split('-')
        return cls(int(day), int(month), int(year))
```

**Oczekiwane zachowanie**

```python
>>> d1 = Date(20, 1, 2016)
>>> d2 = Date.from_string('20-01-2016')
>>> d1.day, d1.month, d1.year
(20, 1, 2016)
>>> d2.day, d2.month, d2.year
(20, 1, 2016)
```