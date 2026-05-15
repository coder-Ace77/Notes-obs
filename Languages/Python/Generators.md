
---

Generator allow us to generate a sequence of values over time. But you will ask why can we use for loop and list. Key reason being that we don't want entire list to be kept in the memory for memory effieciency.

When generator function is called they become an object that supports iteration protocol. So when they are called they don't return a value and exit rather they suspend and resume the execution around the last point of value generator. 

range function is actually a generator and it genrates the value one by one without memory. It does not craetes full list in memory

Creating a generator -- 

```python
def generator_name(parameters):
    # setup code (optional)
    while condition:
        # produce a value
        yield value
        # code after yield runs on next iteration
```

using it is also simple

```python
gen = simple_generator()
print(next(gen))  # 1
print(next(gen))  # 2
print(next(gen))  # 3
# next(gen) → raises StopIteration
```

so we create a gen object which genrates a value and then using `next(geb=n)` it will create a new generator. 

Actually we can have many yields in a given generator

```python
def normal():
    return 1
    return 2  # never runs

def generator():
    yield 1
    yield 2
```

python for loops are compatable with genrators

```python
def countdown(n):
    while n > 0:
        yield n
        n -= 1

for i in countdown(3):
    print(i)

```

We also can have generator expressions 

```python
squares = (x**2 for x in range(5))
print(next(squares))  # 0
print(next(squares))  # 1

for num in squares:
    print(num)
```

Internally they work as follows

```python
Call generator() → returns generator object
next() → runs until first yield
next() → continues from after last yield
```

Line read example

```python
def read_large_file(filename):
    with open(filename, "r") as f:
        for line in f:
            yield line.strip()

for line in read_large_file("data.txt"):
    print(line)
```

`iter()` is a **built-in function** in Python that **returns an iterator object** from an **iterable**.It converts something you can loop over (like a list, tuple, string, etc.) into an **iterator** — an object that can be used with `next()`.

```python
nums = [10, 20, 30]
it = iter(nums)

print(next(it))  # 10
print(next(it))  # 20
print(next(it))  # 30
# next(it)  # ❌ StopIteration
```

using on a list, string.

An **iterable** is any Python object that can return an **iterator**.When you call `iter()` on it, you get an **iterator**.An **iterator** is an object that produces one element at a time when you call `next()`.

Every iterator supports two methods:

- `__iter__()` → returns itself
- `__next__()` → returns next element (or raises `StopIteration`)


Example of craeting an itertor

```python
class MyList:
    def __init__(self, data):
        self.data = data

    def __iter__(self):
        return iter(self.data)  # delegate to list iterator

ml = MyList([1, 2, 3])
for item in ml:
    print(item)
```
