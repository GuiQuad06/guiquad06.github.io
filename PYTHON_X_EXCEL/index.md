---
layout: default
title: Python x Excel
---

# Python x Excel

Automating spreadsheets from Python.

## Python inside Excel

Excel ships a built-in Python feature, but it is not a very portable skill: it requires
a recent Excel version and must not be blocked by IT.

## Useful modules

### pandas

Play with dataframes.

```python
toto['tata'].value_counts().plot(kind='pie')  # show the dataset as a matplotlib chart
toto['new_col'] = ...                         # add a new column
```

### openpyxl

```python
from openpyxl.workbook import Workbook
from openpyxl import load_workbook
```

[Back to home]({{ site.baseurl }}/)
