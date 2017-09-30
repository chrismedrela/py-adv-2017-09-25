---
title: Testowanie - Zaawansowany Python 2017-09-25
---

# Content

[ToC]

# Przykładowy test w `unittest`

```python
# converters.py
import re

def url_converter(url):
    if not url:
        raise ValueError("Empty url")

    pattern = r'(http://[\w-]+(\.[\w-]+)*((/[\w-]*)?))'
    regexp = re.compile(pattern)
    return regexp.sub('<a href="\\1">\\1</a>', url)

# test_converters.py
import unittest

from converters import url_converter

class UrlConverterTests(unittest.TestCase):
    def test_converts_url_to_ahref(self):
        url = "http://www.python.org"
        expected_ahref = '<a href="http://www.python.org">http://www.python.org</a>'
        result = url_converter(url)
        self.assertEqual(result, expected_ahref)

if __name__ == "__main__":
    unittest.main()
```

Uruchomienie:

```
python test_converters.py
```

W Jupyter Notebooku:

```python
unittest.main(argv=['python'], exit=False)
```

Tworzenie pliku w Jupyter Notebook:

```
%%writefile my_file.py
zawartosc
pliku
```

# Uruchamianie testów

Fail fast & locals:

```bash
python test_converters.py -f --locals
```

W Jupyter Notebooku:

```bash
unittest.main(argv=['python', '-f', '--locals'], exit=False)
```