
---
An **exception** is an **unexpected event** during the execution of a program that disrupts its normal flow (like dividing by zero, accessing null, etc.).

All exceptions in Java are objects of classes that extend `java.lang.Throwable`.
When some erro occurs how can we stop the process from crashing. Using try catch finally block.

Note: We can chain multiple catch statements to catch multiple types of exceptions.
```java
try{
	// statement that may throw exception
}catch(Exception e){
	// ahndle exception
}finally{
	// finally block is always executed.
}
// rest of program
```

![[Pasted image 20250624091552.png]]

## Types of Exceptions

###  a) **Checked Exceptions** (Compile-time)
- Must be either **caught or declared**
- Examples:
    - `IOException`
    - `SQLException`
    - `FileNotFoundException`

```java
try {
    FileReader f = new FileReader("file.txt");
} catch (FileNotFoundException e) {
    System.out.println("File not found");
}
```

### b) **Unchecked Exceptions** (Runtime)

- Occur during execution
- Don’t need to be explicitly caught
- Examples:
    - `ArithmeticException`
    - `NullPointerException`
    - `ArrayIndexOutOfBoundsException`

### Important methods of throwable:

| Method              | Description                                    |
| ------------------- | ---------------------------------------------- |
| `getMessage()`      | Returns the exception message                  |
| `printStackTrace()` | Prints the stack trace of the exception        |
| `toString()`        | Returns the string representation of exception |
| `getCause()`        | Returns the cause of the exception             |
```java
public class Main {
    public static void main(String[] args) {
        try {
            int a = 10 / 0;
        } catch (ArithmeticException e) {
            System.out.println("Exception caught: " + e.getMessage());
            e.printStackTrace();
        } finally {
            System.out.println("Finally block always runs");
        }
    }
}
```

Throw an exception manually 

```java
throw new IllegalArgumentException("Negative not allowed");
```

### Throws keyword:

```java
public void readFile() throws IOException {
    FileReader fr = new FileReader("file.txt");
}
```

Also called as rethrow and is used when we don't want to handle exception.

![[Pasted image 20250624092712.png]]

### Custom exception:

Checked custom exception:
```java
class MyException extends Exception {
    public MyException(String message) {
        super(message);
    }
}
public class Test {
    public static void main(String[] args) {
        try {
            throw new MyException("Custom error occurred!");
        } catch (MyException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

Unchecked custom exception:

In Java, an **unchecked exception** is any exception that extends the class **`RuntimeException`**. You don’t need to catch or declare it using `throws`.

```java
class InvalidAgeException extends RuntimeException {
    public InvalidAgeException(String message) {
        super(message);  // pass the message to the RuntimeException constructor
    }
}
```

### Java exception propogation:

Method1 calling method2 if method2 gets any error it is passed to mathod1 if not hamdled inside mathod2.
This is called java exception propogation.
