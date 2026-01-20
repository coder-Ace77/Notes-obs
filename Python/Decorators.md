
---

Decorators allows us to add some additional functionality to existing function. They use @operator and are palxced on top of function.

A **decorator** is simply a **function that takes another function as input**, **adds some extra functionality**, and then **returns it (or replaces it)** — _without modifying the original function’s code._

```python
def greet():
    print("Hello!")

def decorator(func):
    def wrapper():
        print("Before function runs")
        func()
        print("After function runs")
    return wrapper

decorated_greet = decorator(greet)
decorated_greet()
```

Now we can simply use the @decorator to define that kind of function. This is used very much in web frame works like flask and fastapi

```python
@decorator
def greet():
    print("Hello!")

# above line is equal to 
greet = decorator(greet)

greet()
```

If the function which we decoreate takes args the wrapper must take them too

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        print("Before call")
        result = func(*args, **kwargs)
        print("After call")
        return result
    return wrapper

@decorator
def add(a, b):
    return a + b

print(add(3, 5))

```

One practical example is timing a function

```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.5f} seconds")
        return result
    return wrapper

@timer
def compute():
    time.sleep(1)
    print("Done")

compute()
```

Another example can be of authentication

```python
def require_login(func):
    def wrapper(user):
        if not user.get("logged_in"):
            print("Access denied! Please log in.")
            return
        return func(user)
    return wrapper

@require_login
def view_profile(user):
    print(f"Welcome, {user['name']}!")

user1 = {"name": "Adil", "logged_in": True}
user2 = {"name": "Guest", "logged_in": False}

view_profile(user1)
view_profile(user2)
```

Example in fastapi

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello, world!"}
```

Observe that @app.get is a decorator