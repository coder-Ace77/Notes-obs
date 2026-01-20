
---

File handling can be done through open function 

```python
open(file, mode)
```

| Mode  | Meaning | Description                                           |
| ----- | ------- | ----------------------------------------------------- |
| `'r'` | Read    | Default mode. File must exist.                        |
| `'w'` | Write   | Creates a new file or overwrites existing.            |
| `'a'` | Append  | Adds data to end of file. Creates file if not exists. |
| `'x'` | Create  | Creates new file; fails if file exists.               |
| `'b'` | Binary  | Used for binary files (e.g., images, PDFs).           |
| `'t'` | Text    | Default mode. Reads/writes text.                      |
Reading from file: So read() function reads entire file while we can also read line by line using iterator

```python
f = open("example.txt", "r")
data = f.read()
print(data)
f.close()

f = open("example.txt", "r")
for line in f:
    print(line.strip())
f.close()
```

read takes optional arg which is how many bytes to read. 

```python
f = open("example.txt", "r")
part = f.read(10)  # read first 10 characters
print(part)
f.close()
```

Writing to files so there are two modes `'r'` and `'a'` read and append mode read mode jsut starts overriding current file while append will append to a certain file.

Now instead of manually closing the file we can use with that automatically closes the files

```python
with open("example.txt", "r") as f:
    data = f.read()
    print(data)
```

If we try to open non existant files it returns fileNotFound exception. We can also check if file exist by using file check.

```python
import os

if os.path.exists("example.txt"):
    print("File exists!")
else:
    print("No such file.")
```

we can also easily write the file data 

```python
import json

data = {"name": "Adil", "age": 22}

# Write JSON
with open("data.json", "w") as f:
    json.dump(data, f)

# Read JSON
with open("data.json", "r") as f:
    obj = json.load(f)

print(obj)
```

file opening and handling internally have a cursor that moves forward when one reads a file. Note however we can move back using seek method

```python
f.seek(<pos to go to>)

f.readlines() # returns the lines however we have \n at the end 
```

