
---

Python follows the dynamic and string typing model. Dynamic model means type of variable is determined at the run time not compile time and strong typing means implicit type convertions are not allowed.

```python
"1"+1  # error
```

Every object in Python is represented by a **structure in C** (in CPython, the default interpreter). Now there are variaous implementation of python the default is cpython. In cpython everything is PyObject.

```c
typedef struct _object {
    Py_ssize_t ob_refcnt;  // reference count
    struct _typeobject *ob_type;  // pointer to type info
} PyObject;
```

Every value inherits from `PyObject`.- `ob_type` points to the **type definition** (like `PyLong_Type`, `PyList_Type`, etc.).`ob_refcnt` is used for **reference counting** (part of garbage collection).

### Python memory model:

Python uses **references** to objects rather than storing the actual values in variables. Every object is stored in **heap memory**, and the variable just holds a **pointer** (reference) to that object.

```python
a = [1, 2, 3]
b = a
```

its memory view

```csharp
Heap:
[1, 2, 3]  <- object at some memory address

Stack / Variables:
a ──► [1, 2, 3]
b ──┘
```

so mutating one effects other. 

Shallow copying creates new container, but the elements inside still refernce same objects as original.So although the pointer list is new but the pointers inside pointer list is older one. To do shallow copy 

we have three methods 

```python
import copy

a = [[1, 2], [3, 4]]
b = a.copy()   # or copy.copy(a)
```

```rust
Heap:
[1, 2]  <- object1
[3, 4]  <- object2

New list object:
b = [ref->object1, ref->object2]
a = [ref->object1, ref->object2]
```

**Shallow copy:** new container, same references inside.

A **deep copy** creates a **new container AND recursively copies all nested objects**. The new object is fully independent.

```python
b = copy.deepcopy(a)
```

```python
Original heap:
[1, 2] <- object1
[3, 4] <- object2

Deep copy heap:
[1, 2] <- object1' (new)
[3, 4] <- object2' (new)

b = [ref->object1', ref->object2']
a = [ref->object1, ref->object2]
```

Now understand the diffence with normal `b=a` b is only a new pointer but a and b both point to the same internal c list. With shallow copy `b=a.copy()` the pointer of b interally are diffenrt variables than of a. so now chaning `b[index]=new_val` won't effect a. But if it were a list that would not be recursively copied. So here comes deep copy.

- **Variables store references** to objects, not the objects themselves.
- **Shallow copy** → new container, inner objects shared.
- **Deep copy** → completely independent container and objects.
- **Lists** are mutable → copy makes sense for independence.
- **Tuples** are immutable → shallow copy usually enough, but inner mutable elements can still be shared.
- Memory-efficient code often prefers shallow copy if inner objects won’t be mutated.
### Core data types:

Integers: Python has arbitrary precision integers. Sotred interanlly as the array of digits.
Strings are immutable and stored as unicode. Python uses **hash caching** for fast dictionary lookups.

```python
s1 = "hello"
s2 = "hello"
print(s1 is s2)  # True (interned)
```

#### List and tuples:

| Feature  | List                                 | Tuple                                |
| -------- | ------------------------------------ | ------------------------------------ |
| Type     | `list`                               | `tuple`                              |
| Mutable  | ✅ Yes                                | ❌ No                                 |
| Syntax   | `[1, 2, 3]`                          | `(1, 2, 3)`                          |
| Use case | Dynamic data, frequent modifications | Fixed data, hashable, lightweight    |
| Hashable | ❌ No                                 | ✅ Yes (if all elements are hashable) |

A Python list is essentially a **dynamic array** implemented in C.

```c
typedef struct {
    PyObject_VAR_HEAD
    PyObject **ob_item;  // Pointer to array of PyObject* elements
    Py_ssize_t allocated; // Total allocated slots (capacity)
} PyListObject;
```

When appending, if the list is full, Python **over-allocates** to reduce reallocations.

Whereas tuples internally are fixed sized arrays--

```c
typedef struct {
    PyObject_VAR_HEAD
    PyObject *ob_item[1];  // Inline array of PyObject* elements
} PyTupleObject;
```

Operations:

Python has basic indexing opertor - `[]` 

```python

lst = [1,2,3]
print(lst[i]) # accessing since list is mutable we can also do this
lst[i] = 100

a[start:end] # it is slicing returns a new subarray start is inclusive while end is exclusive

lst.append(100) # to append element at the end

lst.extend([1,2,3,4]) # adds multiple elements 

lst.insert(index,val) # inserting element at certain index and shifts others to right

lst.remove(val) # removes the first occurance

lst.pop(index=last) # removed element at given index with default to last index

lst.index(val) # finds the first index of an element

lst.sort() # in place sort hybrid of merge + insertion sort

lst.reverse() # reversing lsit

new_lst = lst.copy() # shallow copy just creates new refernce

c = a+b # concatination of two lists

b = 3*a # repetion of list

if 10 in a: # membership test
```

```python
t = (1,2,3)  # creating of new tuple
print(t[index]) # accessing
b = t[1:3] # slicing creates new tuple

c = a+b # concatination

for i in t: # iteration
```

### Dictionary:

Dictionary is a mutable hash table

```c
typedef struct {
    Py_ssize_t used;       // Number of items
    Py_ssize_t fill;       // Slots filled (including deleted)
    PyObject **keys;       // Array of keys
    PyObject **values;     // Array of values
    size_t mask;           // Table size - 1 (power of 2)
} PyDictObject;
```

Important operations:

| Operation                          | Description                | Average Time | Worst Case |
| ---------------------------------- | -------------------------- | ------------ | ---------- |
| `d[key]`                           | Get value by key           | O(1)         | O(n)       |
| `d[key] = value`                   | Insert/update key          | O(1)         | O(n)       |
| `del d[key]`                       | Delete key                 | O(1)         | O(n)       |
| `key in d`                         | Membership test            | O(1)         | O(n)       |
| `len(d)`                           | Number of items            | O(1)         | O(1)       |
| `d.clear()`                        | Remove all items           | O(n)         | O(n)       |
| `d.copy()`                         | Shallow copy               | O(n)         | O(n)       |
| `copy.deepcopy(d)`                 | Deep copy                  | O(n)         | O(n)       |
| Iteration (`for k,v in d.items()`) | Iterates keys/values/items | O(n)         | O(n)       |
```python

d = {"a":1,"b":2} # creation

value = d["a"] # computes hash key

d["c"] = 3 # inset or update slot

del d["b"] # marks slot as deleted

"a" in d # membership test

for k in d: ... # iteration
for k, v in d.items(): ... # iteration

```

### Type objects in python:

**type objects** in Python — the foundation of _how every object knows what it is_.  
This is part of Python’s **data model** and **object system internals**.

In Python, **everything is an object**, including functions, classes, numbers, and even types themselves.  
Each object has an associated **type object** that defines:
- What operations it supports
- How it behaves internally (its C struct)
- How it interacts with other objects

```python
a = 10
print(type(a))  # <class 'int'>
```

In CPython, every type object is a `PyTypeObject` struct, which defines how instances of that type behave.
```python
typedef struct _typeobject {
    PyObject_VAR_HEAD
    const char *tp_name;       // Type name, e.g. "list"
    Py_ssize_t tp_basicsize;   // Size in memory
    destructor tp_dealloc;     // Called when refcount = 0
    getattrfunc tp_getattr;    // Attribute access
    setattrfunc tp_setattr;    // Attribute setting
    as_number *tp_as_number;   // Numeric operations
    as_mapping *tp_as_mapping; // Dict-like operations
    ...
} PyTypeObject;
```

each PyObject has a field `ob_type` `type(x)` just returns this pointer

### Strings:

Strings in python are immutable meaning altough we can access element by 
```python
s = "abs"
s[0] # accessing element but we can not change it or mutate it
```

common operations:

|Operation|Description|Complexity|
|---|---|---|
|`len(s)`|Number of characters|O(1)|
|`s[i]`|Access character|O(1)|
|`s[a:b]`|Slice substring|O(k)|
|`s1 + s2`|Concatenation|O(n + m)|
|`s * k`|Repeat string|O(n * k)|
|`s1 in s2`|Membership test|O(n·m) worst|
|`s.find(x)`|Find substring|O(n·m)|
|`s.count(x)`|Count occurrences|O(n)|
|`s.replace(a,b)`|Replace substring|O(n)|
|`s.strip()`|Remove whitespace|O(n)|
|`s.upper()/lower()`|Case change|O(n)|
|`s.split()`|Split into list|O(n)|
|`'sep'.join(list)`|Join list to string|O(n)|
|`s.startswith()/endswith()`|Prefix/suffix check|O(k)|
|`s.format()` / `f"..."`|String formatting|O(n)|

String slicing 

```python
s = "Python"
print(s[0])     # 'P'
print(s[-1])    # 'n' negative index means from end
print(s[1:4])   # 'yth'
```

concatenation

```python
s = ""
for i in range(5):
    s += str(i)
    
s = ''.join(str(i) for i in range(5)) # efficient
```

String formatting styles

| Method              | Example                      | Description          |
| ------------------- | ---------------------------- | -------------------- |
| **Old style**       | `"%s %d" % ("Age:", 20)`     | C-style formatting   |
| **`str.format()`**  | `"{} {}".format("Age:", 20)` | Flexible formatting  |
| **f-string (3.6+)** | `f"Age: {20}"`               | Fastest and cleanest |

### Sets:

They are unique collection of objects. 

```python
s = set()
s = set(lst)

s.add(1) # adding element to a set
```