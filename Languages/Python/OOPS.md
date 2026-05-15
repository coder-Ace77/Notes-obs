
---

We define obejcts in python using class keyword

```python
class MyClass():
	def __init__(self,param1,..): # constructor
		self.param1 = param1
	def myMethod(self,param):
		pass
	def myFunction(): 
		pass # normal function not a method as not have any self	
		
sample = MyClass(param1="Hello")
```

self is something which differnecites some function from methods(methods are function that are binded to a class). So method must be the first arg of the method. By convention we name classes by camel casing. `self` represents the `object` which is being currently used.

Inheritance and polymorphism

```python
class Animal():
	def __init__(self): # constructor
		print("CONSTRUCTOR")
		
	def eat():
		print("eating")
```

Now to do inheritance 

```python
class Dog(Animal):
	def __init__(self):
		Animal.__init__(self) # calling super class it creates instance of animal class
	
	def eat():
		print("Dog eating")

```

Now dog has all the methods and properties of Animal. If however one defines the function with the same name in inherited class the function will be overwritten.

**Polymorphism** means _“many forms”_ — the same function or method name behaves **differently** based on the type of object it is acting upon.

One function name, multiple behaviors depending on the object.

```python
class Dog:
    def speak(self):
        return "Woof!"

class Cat:
    def speak(self):
        return "Meow!"

animals = [Dog(), Cat()]

for animal in animals:
    print(animal.speak())
```

Creating the abstract class

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self):
        pass

# obj = Animal()   # ❌ TypeError: Can't instantiate abstract class
```

### Special methods or dunder methods:

```python
class Book():
	
	def __init__(self,title,author,pages):
		pass 
```

init method is automatically called at the time of creating of class.

Now how to define the print we do it using `__str__` 

```python
class Book():
	
	def __init__(self,title,author,pages):
		pass 
		
	def __str__():
		return "This is printed"
```

we can define `__len__` method which is run when len() is called

```python
def ___len___():
	return # note it must return something which makes sence
```

### Method types in python

In Python, methods are not all created equal. Depending on how they are defined and what they have access to, they fall into three distinct categories: **Instance Methods**, **Class Methods**, and **Static Methods**.
The primary difference between them lies in their **binding**—what object or data they "see" when they are called.

#### Instance methods

These are the most common type of method. By default, any method you define inside a class is an instance method. They are designed to act upon a specific **instance** of the class.

- **Access:** They can access and modify both instance-level data (via `self`) and class-level data.
- **The `self` parameter:** They must take `self` as their first argument, which points to the specific object in memory.

```python
class Robot:
    def __init__(self, name):
        self.name = name  # Instance attribute

    def say_hello(self):
        return f"Hello, I am {self.name}"
```

#### Class methods

Class methods are bound to the **class itself**, rather than a specific object. They are defined using the `@classmethod` decorator.

**Access:** They cannot access instance-specific data (they don't know which "robot" they are), but they can access and modify class-level attributes.

**The `cls` parameter:** Instead of `self`, they take `cls` as the first argument, which refers to the class definition.

**Use Case:** They are most commonly used as **Factory Methods**—alternative ways to create objects. For example, creating a `Robot` from a JSON string or a database record.

```python
class Robot:
    population = 0  # Class attribute

    def __init__(self, name):
        self.name = name
        Robot.population += 1

    @classmethod
    def get_population(cls):
        return f"Total robots: {cls.population}"
```

#### Static methods

Static methods are the most restricted. They are essentially just regular functions that happen to live inside a class's namespace for organizational purposes.

- **Access:** They have no access to `self` or `cls`. They work purely with the data passed directly to them.
- **The Parameter:** They do not take a special first argument.
- **Use Case:** Use them for utility functions that relate to the class conceptually but don't need to touch any class or instance data.

```python
class Robot:
    @staticmethod
    def is_legal_name(name):
        return name.isalnum()
```

##### Property Methods (`@property`)

While the three above are the main "types," Python also provides the `@property` decorator. This turns a method into a "getter," allowing you to call it like an attribute (without parentheses). This is useful for calculated attributes or adding validation logic without breaking the public API of your class.

```python
class Robot:
    def __init__(self, battery):
        self._battery = battery
    @property
    def status(self):
        return "High" if self._battery > 50 else "Low"
# Called as robot.status, not robot.status()
```

