
---

JUnit is a unit testing framework of java.
First of all it is an dependency so we need to include as package.

Standard Directory Structure

```bash
project-root/
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/yourpackage/
│   │           └── Calculator.java
│   └── test/
│       └── java/
│           └── com/yourpackage/
│               └── CalculatorTest.java
```



Basic example
```java
package com.yourpackage;

// import for testing
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class CalculatorTest {
    @Test
    void testAddition() {
        Calculator calc = new Calculator();
        assertEquals(5, calc.add(2, 3));
    }
}

```

| Annotation     | Purpose                                         |
| -------------- | ----------------------------------------------- |
| `@Test`        | Marks a test method                             |
| `@BeforeEach`  | Runs **before** each test                       |
| `@AfterEach`   | Runs **after** each test                        |
| `@BeforeAll`   | Runs **once** before all tests (must be static) |
| `@AfterAll`    | Runs **once** after all tests (must be static)  |
| `@DisplayName` | Sets a custom name for a test                   |
| `@Disabled`    | Skips the test                                  |

### Assertion 

| Method                           | Usage Example                  |
| -------------------------------- | ------------------------------ |
| `assertEquals(expected, actual)` | `assertEquals(10, sum);`       |
| `assertTrue(condition)`          | `assertTrue(a > b);`           |
| `assertFalse(condition)`         | `assertFalse(list.isEmpty());` |
| `assertNull(obj)`                | `assertNull(result);`          |
| `assertThrows()`                 | Test for exceptions            |
eg:

```java
@Test
void testException() {
    assertThrows(ArithmeticException.class, () -> {
        int x = 1 / 0;
    });
}
```

You use `assertThrows()` when you expect a method to throw an exception under specific conditions, and you want to **verify that the exception is thrown** (and optionally inspect it).

```java
assertThrows(
    Class<T> expectedType,
    Executable executable
)
```

eg: Normal Class

```java
public class Calculator {
    public int divide(int a, int b) {
        if (b == 0)
            throw new ArithmeticException("Cannot divide by zero");
        return a / b;
    }
}
```

Test class

```java
import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.api.*;

class CalculatorTest {
    Calculator calc;
    @BeforeEach
    void setup() {
        calc = new Calculator();
    }
    @Test
    void testDivideSuccess() {
        assertEquals(5, calc.divide(10, 2));
    }
    @Test
    void testDivideByZeroException() {
        ArithmeticException ex = assertThrows(ArithmeticException.class, () -> {
            calc.divide(10, 0);
        });

        assertEquals("Cannot divide by zero", ex.getMessage());
    }
}

```

