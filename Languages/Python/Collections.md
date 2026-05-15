
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

### Tuples

Tuples are the immutable , ordered collection of items. Defined using parenthesis `()`

```python
# Standard tuple
fruits = ("apple", "banana", "cherry")

# Tuple without parentheses (Packing)
coordinates = 10.0, 20.0

# The Singleton Trap: A tuple with one item MUST have a trailing comma
lonely_tuple = ("only_me",) 
not_a_tuple = ("only_me") # This is just a string!
```

We can access the tuple using `[]` operator. Also we can find length using `len()` operator. Note that although python tuple is immutable if we have mutable object inside tuple we can simply mutate it. 

```python

t = (1,2,3)

for i in range(len(t)):
    print(i,t[i])

for i in t:
    print(i)

for index,val in enumerate(t):
    print(index,val)
```

All above are the standard ways of iterating over the tuples. 

#### Packing and unpacking

Packing is the process of putting values into a tuple. Unpacking is the process of extracting values from the tuples. Unpacking is done by using the comma separated list of variables. Also the order depends. Finally we can use the `*` for the rest for example in the second example `*middle` Contains the rest of values again as the tuple. 

```python
# Unpacking
data = ("Alice", 30, "Engineer")
name, age, profession = data

# Using the Asterisk for "the rest"
numbers = (1, 2, 3, 4, 5)
first, *middle, last = numbers
# first = 1, middle = [2, 3, 4], last = 5
```

Note that tuples are slightly faster , have lower memory overhead and hashable. Finally as we have seen when the functions return multiple values they are treated as the tuples at the calling time and developers heavily use the unpacking syntax for that. 

```python
def func():
	return 1,2,3
	
x,y,z = func()
```

### Enums

An **Enum** (short for Enumeration) is a way to create a set of symbolic names bound to unique, constant values. In Python, these are handled by the `enum` module.

To create an enum, you inherit from the `Enum` class.

```python
from enum import Enum

class Status(Enum):
    PENDING = 1
    RUNNING = 2
    COMPLETED = 3
    FAILED = 4
```

Each member has a **name** and a **value**.
- `Status.PENDING.name` → `"PENDING"`
- `Status.PENDING.value` → `1`
- `Status(1)` → Returns the member `Status.PENDING` (lookup by value)

Also we can change the value type of the Enum as follows

```python 
from enum import Enum

class Status(str,Enum):
	PENDING = "pending"
	RUNNING = "running"
	COMPLETED = "completed"
	FAILED = "failed"
```

