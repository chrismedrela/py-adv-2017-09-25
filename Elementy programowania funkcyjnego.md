---
title: Elementy programowania funkcyjnego - Zaawansowany Python 2017-09-25
---

# Content

[ToC]

# Ćw.: Iteracja po własnych typach

Zaimplementuj klasę `countdown` umożliwiającą odliczanie w dół.

**Rozwiązanie**

```python
class countdown:
    def __init__(self,start):
        self.count = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.count <= 0:
            raise StopIteration
        r = self.count
        self.count -= 1
        return r
```

**Oczekiwane zachowanie**

```python
>>> for i in countdown(5):
...    print(i)
5
4
3
2
1
```

# Ćw.: Wielokrotna iteracja po własnych typach

Popraw implementację `countdown` tak, aby można było po takiej klasie iterować wielokrotnie.

**Rozwiązanie**

```python
class CountdownIterator:
    def __init__(self, start):
        self.count = start

    def __next__(self):
        if self.count <= 0:
            raise StopIteration
        r = self.count
        self.count -= 1
        return r

class countdown:
    def __init__(self,start):
        self.start = start

    def __iter__(self):
        return CountdownIterator(self.start)
```

**Oczekiwane zachowanie**

```python
>>> c = countdown(3)
>>> for i in c: print(i)
3
2
1
>>> for i in c: print(i)
3
2
1
```

# Dzień trzeci

# Ćw.: Fibonacci

Napisz generator zwracający kolejne liczby Fibonacciego aż do podanego limitu.
Każdy element ciągu Fibonacciego jest sumą dwóch poprzednich elementów.
Pierwsze liczby w tym ciągu to: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89.

Zadanie rozwiąż na trzy sposoby: 

1. implementując generator jako funkcję `fib`, 
1. implementując go jako iterator `FibSimple`, po którym można iterować tylko raz,
1. oraz iterator `Fib`, po którym można wielokrotnie iterować.

**Oczekiwane zachowanie**

```python
>>> list(fib(100))
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]
>>> list(FibSimple(100))
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]
>>> list(Fib(100))
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]
```

**Rozwiązanie**

```python
def fib(limit):
    a, b = 0, 1
    while a <= limit:
        yield a
        a, b = b, a+b

class FibSimple:
    def __init__(self, limit):
        self.limit = limit
        self.a, self.b = 0, 1

    def __next__(self):
        retval = self.a
        self.a, self.b = self.b, self.a+self.b
        if retval > self.limit:
            raise StopIteration
        return retval

    def __iter__(self):
        return self

class FibIterator:
    def __init__(self, limit):
        self.limit = limit
        self.a, self.b = 0, 1

    def __next__(self):
        retval = self.a
        self.a, self.b = self.b, self.a+self.b
        if retval > self.limit:
            raise StopIteration
        return retval

class Fib:
    def __init__(self, limit):
        self.limit = limit

    def __iter__(self):
        return FibIterator(self.limit)
```

# Ćw.: Suma liczb podzielnych przez 3 lub 5

Znajdź sumę wszystkich liczb całkowitych od 0 do 1000, które są podzielne przez 3 lub 5.

Rozwiąż to zadanie na kilka sposobów:

1. używając pętli for, ale nie używając wyrażeń generatorowych ani funkcji `filter` ani `sum`,
1. używając wyrażenia generatorowego i funkcji `sum`,
1. i używając funkcji `filter` i `sum`.

**Oczekiwane wyjście**

    234168
    234168
    234168

# Przerwa do 11:05

# Ćw.: przetwarzanie logów Apache

Chcemy sprawdzić, ile bajtów danych zostało przesłanych, sumując ostatnią kolumnę danych z pliku logów serwera Apache.
W ostatniej kolumnie zamiast liczby może pojawić się `-`.
Taki wiersz należy zignorować.

Plik logów Apache `"access-log"` wygląda następująco:

    81.107.39.38 - ... "GET /ply/ HTTP/1.1" 200 7587
    81.107.39.38 - ... "GET /favicon.ico HTTP/1.1" 404 133
    81.107.39.38 - ... "GET /ply/bookplug.gif HTTP/1.1" 200 23903
    81.107.39.38 - ... "GET /ply/ply.html HTTP/1.1" 200 97238
    81.107.39.38 - ... "GET /ply/example.html HTTP/1.1" 200 2359
    66.249.72.134 - ... "GET /index.html HTTP/1.1" 200 4447
    81.107.39.38 -  ... "GET /ply/ HTTP/1.1" 304 -

Pliki logów mogą ważyć bardzo dużo.
W związku z tym wczytanie całego pliku do pamięci nie jest najlepszym pomysłem.

Rozwiąż zadanie na dwa sposoby:

1. z użyciem pętli for,
1. i potokowo, tzn. używając kilka wyrażeń generatorowych.

**Rozwiązanie (pierwszy sposób)**

```python
with open("access-log") as wwwlog:
    total = 0
    for line in wwwlog:
        bytes_as_str = line.split()[-1]
        if bytes_as_str != '-':
            total += int(bytes_as_str)
    print("Total", total)
```

**Hint (drugi sposób)**

```python
with open("access-log") as wwwlog:
    bytecolumn = (line.split()[-1] for line in wwwlog)
    bytes = (int(x) for x in bytecolumn if x != '-')
    total = sum(bytes)
    print("Total", total)
```