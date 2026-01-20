
---

General regular functions

```java
returnType methodName(parameters) {
    // method body
}
```

Mathod types:

| Type                     | Example                     |
| ------------------------ | --------------------------- |
| **Instance method**      | `obj.method()`              |
| **Static method**        | `ClassName.method()`        |
| **Parameterised method** | `void setName(String name)` |
| **Return-type method**   | `int sum(int x, int y)`     |
| **Void method**          | `void display()`            |

#### Lambda functions:

A lambda expression is a short block of code which takes in parameters and returns a value. Lambda expressions are similar to methods, but they do not need a name and they can be implemented right in the body of a method.

```java
parameter -> expression
(parameter1, parameter2) -> { code block }
(parameters) -> { expression or block }
```

Example:

```java
import java.util.ArrayList;

public class Main {
  public static void main(String[] args) {
    ArrayList<Integer> numbers = new ArrayList<Integer>();
    numbers.add(5);
    numbers.add(9);
    numbers.add(8);
    numbers.add(1);
    numbers.forEach( (n) -> { System.out.println(n); } );
  }
}
```



