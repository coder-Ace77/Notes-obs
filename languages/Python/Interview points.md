
---

### List vs tuple

_Answer:_ Tuples are fixed-size and stored in a single memory block, making them slightly faster and memory-efficient. Lists are over-allocated to allow for O(1) appends.

Lists are mutable and tuples are not mutable.

### Memory management in python

Everything in python is object and these objects reside in the heap. Each object in python has fielf called `ob_refcnt`. When object is assigned to variable or may be added to container the refernce count increases. When variable goes out of scope the count descreases. As soon as object reference reaches 0 the object is deallocated. However this garbage collector has one problem remaining cyclic refernces. 

This occurs when two or more objects refer to each other, but are no longer reachable from the main program. For example, if Object A points to Object B, and Object B points back to Object A, their reference counts will never drop to zero, even if the rest of the code can no longer access them. To solve this python has cyclic garbage collector (GC) that runs periodically in background and uses mark and sweep algorithm to detect isolated islands.

### GIL (global interpretor lock)

The python uses reference counting for memory management. Now if two threads are allowed to execute at the same time they can modify the references at the same time leading to race conditions. To solve this python uses mutexes on the object level. Rather it enforces only one thread to be executed on the cpu at a time. 

Python is a interpreter model and to enter inside the interpretor loop thread must aquire the Global interpretor lock(GIL) once aquired no other thread will be able to enter the interpretor. A thread will hold the GIL either I/O is to be performed or switch typically 5ms happens. 

If a thread has held the GIL for longer than the switch interval, it sets a "drop request" flag. The holding thread eventually sees this flag, releases the GIL, and waits for a brief period to allow other threads to grab it. This prevents a single CPU-bound thread from "starving" others.

Now because of GIL python effectively becomes single threaded and is slow for CPU bound tasks. However a multihreaded program can actually perform better than single threaded in case of high I/O operations. 

With python 3.13 the GC algortihm has changed and now python allows experimental `free-threaded` builds here GIL is disabled and makers have used `Biased Reference Counting` which is faster for multithreaded situations but much slower for single threaded because of additional checks. 

### Equality

In Python, determining if two things are "the same" involves a critical distinction between **value equality** and **identity equality**.

Value equality == 

The == operator checks for the value equality and asks the question do two objects contain the same data? When we call the == python under the hood calls the `__eq__()` method. For built-in types like lists or strings, this method is pre-defined to look inside the object and compare the contents piece by piece. Type is ignored and only values are considered. 

```python
10==10.0 # true same value
[1,2]=[1,2] # true 
```

Idenetity equality (is)

`is` operator  checks for the identity equality and it means do the two variables point to the same memory address. Internally, `is` compares the results of the `id()` function for both objects. If `id(a) == id(b)`, then `a is b` is `True`. 

With smaller objects or strings python usually preallocates them and uses them.

```python
x = 10
y = 10
print(x is y) # Returns True (Wait, why?)
```

So smaller string and intergers may internally be using the same object. 

We can also use is with `None`. None is basically a singleton and therefore using is faster than using ==

### Function argument unpacking

In Python, **argument unpacking** allows you to take a collection of data (like a list, tuple, or dictionary) and "unpack" its contents directly into a function’s parameters. The system relies on two operators: `*` (the iterable unpacker) and `**` (the dictionary unpacker).

```python
def calculate_volume(length, width, height):
    return length * width * height

dimensions = [10, 5, 2]

# Instead of calculate_volume(dimensions[0], dimensions[1], dimensions[2])
result = calculate_volume(*dimensions) 
print(result) # 100
```

```python
def create_user(name, age, role):
    print(f"Creating {role}: {name}, {age} years old.")

user_data = {"name": "Alice", "age": 30, "role": "Engineer"}
# Unpacks the dictionary into keyword arguments
create_user(**user_data)
```

### Combining Unpacking with "Slurping"

Unpacking isn't just for calling functions; it’s also used in function **definitions** to allow for a variable number of arguments. This is often called "slurping" or "packing."
- **`*args`**: Collects extra positional arguments into a **tuple**.
- **`**kwargs`**: Collects extra keyword arguments into a **dictionary**.

```python
def flexible_function(*args, **kwargs):
    print(f"Positional: {args}") # A tuple
    print(f"Keywords: {kwargs}") # A dictionary

flexible_function(1, 2, 3, status="active", debug=True)
```

#### Context management

The `with` statement in Python is a control-flow structure designed to simplify **resource management**. By resource we mean the system resources for example database connections, files , network connections. Resource will be freed after the use automatically. 

```python
with open("",'w') as f:
	f.read()
```

We can also support the `with` with our objects. To enable with we need to implement the two functions
`enter` and `exit`.
```python
class ManagedFile:
	def __init__(self,name):
		self.name=name
	def __enter__(self):
		self.file=open(self.name,'w')
		return self.file
	
	def __exit__(self,exc_type,exc_val,exc_tb):
		if self.file:
			self.file.close()
```

### dunder in python

`_` single leading underscore. A single underscore at the start of a name is a **convention** indicating that a variable or method is intended for **internal use** within a class or module.

- **Weak Internal Use Indicator:** It tells other developers, "This is a private detail; use it at your own risk." However, Python does not actually prevent you from accessing `obj._variable`.
- **Module Imports:** If you use `from module import *`, any name starting with `_` will **not** be imported. This helps keep the public API of a module clean.

The double underscore triggers a syntax rule called **Name Mangling**. This is Python’s closest version of a "private" variable.
When the Python interpreter encounters a double underscore member inside a class, it renames it internally to include the class name. For example, `__salary` inside a class `Employee` becomes `_Employee__salary`.

- **Purpose:** The goal is to prevent **name clashes** in subclasses. If a subclass defines a variable with the same name, the mangling ensures they don't overwrite each other in the object's dictionary.
- **Not True Privacy:** You can still access the variable if you use the mangled name (e.g., `obj._ClassName__variable`), but the double underscore serves as a strong "Keep Out" sign.

### Mutabilty 

In Python, every variable is a reference to an object in memory. The distinction between **mutability** and **immutability** defines whether that object can be changed after it has been created.

A mutable object is one that allows you to modify its internal state without changing its identity (its memory address). When you "change" a mutable object, you are editing the data at that specific address.

Common mutable types include:
- **Lists** (`[1, 2]`)
- **Dictionaries** (`{'a': 1}`)
- **Sets**
- **Bytearrays**
- **User-defined classes** (usually)

Because these objects are changeable, if two variables point to the same mutable object, changing one will reflect in the other.


An immutable object cannot be altered once it is created. If you attempt to "modify" an immutable object, Python does not change the existing object; instead, it creates a **brand-new object** in a different memory location and points your variable to it.

Common immutable types include:

- **Integers, Floats, Booleans**
- **Strings**
- **Tuples**
- **Frozen Sets**
- **Bytes**

```python
name = "Python"
# This doesn't change "Python", it creates "python" and reassigns 'name'
name = name.lower()
```


A common point of confusion is that an immutable container (like a tuple) can contain mutable elements (like a list). While the tuple itself cannot be changed (you cannot add or remove elements), the **objects inside it** can still be mutated if they are mutable.

```python
my_tuple = (1, [2, 3])
# my_tuple[0] = 5  <-- This raises an Error (Immutability)
my_tuple[1].append(4) # <-- This works! The list inside is mutable.
```

In python function calls the mechanism used is called `call by assignement`. If you use a mutable object (like a list) as a default parameter in a function, that list is created **only once** when the function is defined, not every time it is called.

```python
def add_item(item, my_list=[]): # The list is created at definition time
    my_list.append(item)
    return my_list

print(add_item(1)) # [1]
print(add_item(2)) # [1, 2]  <-- The list persisted!
```

