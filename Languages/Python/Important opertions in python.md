
---
#### Range:

```python
for i in range(0,100): 
```

Now above for loop is essentiaslly a for each loop range is a generator . It takes three numbers `(start,stop,step)` start and step are optionals and stop is exclusive. But we can get lsit from it 

```python
list(range(1,10,2))
```

#### Enumerate:

```python
word = "something good"
for item in enumerate(word):
	print(item) # item here is a 2 tuple with first entry index and second char
	
# for loops also support tuple unpacking
for index,char in enumerate(word): 
	print(index,char)
```

Above we see tuple unpacking which is used almost everywhere in python it can come on the left of = or in places like for loops

### Zip:

It is also a generator. It pairs up corresponding index elements so that we can enumerate on multiple lists at once. Zip can work on multiple lists at once.

```python
for a,b in zip(l1,l2):
	print(a,b)
```

### in:

check for the existance in a various iterables

```python
key in dict # checking if key is in a dict
val in dict.values() # 
x in l # check in list or string
```

### Random library:

```python
from random import shuffle,randint

shuffle(myList) # shuffles the list in place
randint(lower,upper) # geenrates a random number between lower and upper range
```

### Input:

```python
res = input("Text to show while taking input") # it accepts things as strings 
```


Type casting (or type conversion) means **changing the data type** of a value into another type — e.g., converting a string to an integer.

Python supports two types of conversions:

| Type                    | Description                                      |
| ----------------------- | ------------------------------------------------ |
| **Implicit conversion** | Done automatically by Python (safe conversions). |
| **Explicit conversion** | Done manually using built-in functions.          |
example of implicit type conversion

```python
x = 10       # int
y = 2.5      # float

result = x + y
print(result)       # 12.5
print(type(result)) # <class 'float'>
```

explicit

| Function  | Converts To                       |
| --------- | --------------------------------- |
| `int()`   | Integer                           |
| `float()` | Floating point number             |
| `str()`   | String                            |
| `bool()`  | Boolean                           |
| `list()`  | List                              |
| `tuple()` | Tuple                             |
| `set()`   | Set                               |
| `dict()`  | Dictionary (with valid structure) |
examples 

```python
pairs = [("a", 1), ("b", 2)]
d = dict(pairs)
print(d)   # {'a': 1, 'b': 2}
```

### List comprehensions:

List comprtehensions are quick way to create new list in python. If you are using for loop with append to create a list it is a really cool way to do things.

general syntax

```python
new_list = [ expression for item in iterable if condition ]
```

| Part           | Meaning                                                        |
| -------------- | -------------------------------------------------------------- |
| `expression`   | The value or transformation you want to store in the new list. |
| `item`         | The variable representing each element from the iterable.      |
| `iterable`     | A sequence (like a list, tuple, string, range, etc.).          |
| `if condition` | _(optional)_ Filters which items to include.                   |
List comprehension is the syntax sugar for 

```python
result = []
for item in iterable:
    if condition:
        result.append(expression)
```

In the front any expression is formidable any function call or any thing

such as 

```python
l = [('EVEN' if x%2==0 else 'ODD') for x in range(0,11)]
```

We can also do nested list comprehension this however gives us 1d list only not a two d list.

```python
l = [x*y for x in [1,2,3] for y in [1,2,3]]
```

With that any number of such nested loops are possible