
---

### Counter:

Takes list as arg and returns a dict with the value and its freq.

```python
from collections import Counter

c = Counter(lst)
```

### defaultdict:

By default in a normal dict if we try to access a key which is not therd we get keyNotfound exception but with default dict we can assing default values for every key not present

```python
d = defaultdict(lambda:0) # assigining default value 0
```

Everything else works as simple 

