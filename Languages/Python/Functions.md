
---

Anatomy 

Parameters are the variables listed in the function definition.
Arguments are values passed to the function. 
```python
def func_name(args1,arg2):
	# do something
	return val
	
func_name(1,3) # calling
```

Python can accept positional arguments which are assigned based on the order or on the basis of keywords where we explicitly name the parameter so order does not matter.

```python

func(arg2=v2,arg1=v1) # order does not matter
```

We can define the fallback or default argument if user does not provides the value. 

```py
def power(base,power=2):
	return base**power
```

Sometimes you don't know how many items are coming your way.

- `*args` - Collects extra positional arguments into tuple
- `**kargs` - Collects extra keyword arguments into dictrionary. 

A function will return `None` by default if no return statement is provided. Also we can return multiple values at once which python  will treat as tuple. 

```python

def get_stats():
	return val1,val2 # will be treated as tuple.
	
v1,v2 = get_stats() # tuple unpacking
```

In python the functions are first class citizens meaning that assign them to variables, pass as arguments to other functions, return as the functions. Functions can also be defined inside some function. 

Also we can use the tuple unpacking - This is the process in which we pass a tuple to a function however unpack it meaning we convert the tuple to its individual values using `*` operator. Similarly dictionary can be passed as the keyword arguments using `**` operator.

```python

def add(a,b):
	reutrn a+b
	
t = (1,2)
add(*t) # unpacking

def power(base,exp=2):
	return base**exp
	
arg = {base:2,exp:5}
power(**arg)
```

Note the differnece between that we can pass it at two places. 

### Lambda functions

Lambda functions are used for the one time only. Structure of lambda

```python
lambda arguments:expression
```

Note that arguments can be multiple like normal functions. Colon separates the input from logic. Note that expression is evaulated and auto returned. Note that lambda functions are called in the same way is normal functions are. 

```python
square = lambda x:x*x
print(square(x))
```

Exmaple in sort. - 

List's sort function expression expects the function which could tell which expression value will be used for the comparison.

Example

```python
users = [("Alice",30),("Anil",20)]
users.sort(lambda x:x[1]) # expression returns the first arg
```

