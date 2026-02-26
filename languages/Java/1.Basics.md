
---

```java
// File: MyClass.java
package mypackage;              // optional
import java.util.Scanner;       // optional imports
public class MyClass{          // class
    // fields
    int number;
    // constructor
    public MyClass(int n) {
        this.number = n;
    }
    // method
    public void display() {
        System.out.println("Number: " + number);
    }
    // main method
    public static void main(String[] args) {
        MyClass obj = new MyClass(5);
        obj.display();
    }
}
```


> [!NOTE] Package
>A **package** in Java is like a **folder or namespace** that groups related classes.
> 


### Why use packages?

- Avoid **name conflicts** (e.g., two classes named `Utils` in different packages).
- Improve **organization** of large codebases.
- Control **access levels** using `public`, `private`, etc.
- Improve imports

```java
package com.myapp.models;
public class User {
    private String name;
    // other fields and methods
}
```

And in another file:
```java
import com.myapp.models.User;
public class Main {
    public static void main(String[] args) {
        User u = new User();
    }
}
```

### Structure of a Java Project

```kotlin
MyProject/
│
├── src/
│   └── com/
│       └── myapp/
│           ├── Main.java
│           └── models/
│               └── User.java
│
└── out/ or bin/   (compiled .class files)
```

### Compile and run:

Without package

```bash
javac MyClass.java
java MyClass
```

With a package

```bash
javac -d . MyClass.java
java mypackage.MyClass
```

### File naming rules:

1. If a Java file contains a **public class**, the filename **must exactly match** the public class name, including **case sensitivity**.
2. A `.java` file can contain **only one public class**.It **can** contain multiple **non-public** classes (default access).The file name must match the **public class**, if present.
3.  Package Names Use Lowercase Dot-Separated Paths.

Eg:

```java
package com.example.myapp;
```

Then file must be in 

```swift
src/com/example/myapp/User.java
```

### Anatomy of main class

Main class has the method
```java
public static void main(String[] args)
```

This main function is the entry point of program and is called by JVM.


> [!NOTE]  🧠 Notes
> You can **only have one `main()` method per class**, but you can have multiple classes each with their own `main()` in the same project.
> You can run any class that contains `public static void main(String[] args)`.

Thus we may have multiple classes containing the main and any of them can be executed. Infact 
command
```bash
java myClass
- You can run any class that contains `public static void main(String[] args)`.```

myClass must have a main function inside it.
