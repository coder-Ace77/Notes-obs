
---

Pypi is the repository fro open source third party packages. Package are open source library written and distributed by others.

Module is just a .py script. Package is just the collection of modules inside a folder with the condition that there should be `__init__.py` script in that folder. This file does not needs to have anything but it can be empty as well. 

Now one can either entire module or some part of module say module name is hello

```python
def hello():
	return "Hello world" # in a module
```

In another file main

```python
from hello import hello

hello()
```

Package can also have a subpackage meaning a dir with `__init__` and we can call then using dot notation

```python
from MainPackage.Subpackage import hello
```

Package names are usually Camel case

### name and main:

It is observed that python does not have any main function. Python has a `__name__` variable when a script is run directly this name variable is set to `__main__`. Also whenever a module is imported in a python script entire module is runned. So usually it is not ideal to put any code on the top level of file. Its better to define all such statements in the functions and use the if.

```python

if __name__=='__main__':
	 # runs when run through script
else: 
	# runs when not runs through script
```

Every Python file (module) automatically gets a built-in variable called **`__name__`**.When you **run the file directly**, `__name__` is set to `"__main__"`.When the file is **imported as a module**, `__name__` is set to the module’s name (the filename).