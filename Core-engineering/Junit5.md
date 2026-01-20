
---

1. **Junit**: Junit5 is the underlying platform to do testing. 
2. Jupiter: Programming and extension model. 

```scss
Launcher.execute()
  └── JupiterTestEngine.execute()
        └── EngineExecutionContext
              └── executeRecursively()
                    ├── ClassTestDescriptor.execute()
                    │     └── invokeBeforeAll()
                    │     └── for each method:
                    │           └── MethodTestDescriptor.execute()
                    │                 └── invokeBeforeEach()
                    │                 └── invokeTestMethod()
                    │                 └── invokeAfterEach()
                    │
                    └── invokeAfterAll()
```

Jupiter annotations:

1. @Tests: Marks the method as test case. - This method will be discovered and executed as a test by JUnit.
2. @BeforeEach , @AfterEach , @BeforeAll , @AfterAll
3. @DisplayName give custom name to a java test.
```java
@DisplayName("Addition Test")
@Test
void testAddition() {
    assertEquals(4, 2 + 2);
}

```

4. @Disabled
```java
@Disabled("Feature not implemented yet")
@Test
void testFeatureX() {
    // This won't run
}
```

5. @Nested - Groups the class inside a class for better structure.

The `@Nested` annotation in JUnit 5 allows you to structure your test cases in a **hierarchical and readable** manner. It is used to define an **inner class** inside your test class that groups related test methods together under a common context or setup.

```java

class CalculatorTest {

    Calculator calculator;

    @BeforeEach
    void init() {
        calculator = new Calculator();
    }

    @Nested
    class AddTests {

        @Test
        void addingTwoPositiveNumbers() {
            assertEquals(5, calculator.add(2, 3));
        }

        @Test
        void addingZero() {
            assertEquals(3, calculator.add(3, 0));
        }
    }

    @Nested
    class SubtractTests {

        @Test
        void subtractingSmallerFromLarger() {
            assertEquals(2, calculator.subtract(5, 3));
        }

        @Test
        void subtractingSameNumbers() {
            assertEquals(0, calculator.subtract(2, 2));
        }
    }
}

```

- JUnit first runs `CalculatorTest.init()` before each test (even nested ones).
- Each nested class has **access to the outer class’s fields and methods**.

```java
@Nested
class WhenInputIsValid {
    @Test
    void shouldReturnExpectedResult() {
        // test logic
    }
}
```

6. @ParameterisedTests

`@ParameterizedTest` lets you run the **same test method** multiple times with **different inputs**. 
Basically we can run the code with different schenarios in one file.

JUnit calls your test method **repeatedly**, once for each input. The inputs come from one of the many _data sources_ provided by JUnit:

- `@ValueSource` — primitive or string values
- `@CsvSource` — comma-separated values
- `@MethodSource` — values from a method in the test class
- `@EnumSource` — all or selected enum constants
- `@ArgumentsSource` — fully custom input providers

Under the hood, JUnit:

1. Reads the annotated data source,
2. Prepares the inputs (converts strings to target types, for example),
3. Calls your test method with each set of inputs as arguments.

`@ValueSource`

Provides a list of simple values (ints, strings, etc.)

```java

@ParameterizedTest(name = "{0} is positive number")
@ValueSource(ints = {1, 2, 3, 4})
void testIsPositive(int number) {
    assertTrue(number > 0);
}

```

 `@CsvSource`

Provides arguments as comma-separated strings.

```java

@ParameterizedTest
@CsvSource({
    "2, 3, 5",
    "10, 15, 25",
    "0, 0, 0"
})
void testAddition(int a, int b, int expected) {
    assertEquals(expected, new Calculator().add(a, b));
}

```

@MethodSource

Uses a static method in the class (or elsewhere) to supply a **stream of arguments**.

```java

static Stream<Arguments> provideNumbers() {
    return Stream.of(
        Arguments.of(2, 3, 5),
        Arguments.of(4, 1, 5),
        Arguments.of(6, -1, 5)
    );
}

@ParameterizedTest
@MethodSource("provideNumbers")
void testAdditionFromMethod(int a, int b, int expected) {
    assertEquals(expected, new Calculator().add(a, b));
}

```

- JUnit uses a special test runner that recognizes `@ParameterizedTest`.
- The runner reads the input data and schedules multiple executions.
- It passes each input (or combination of inputs) to the test method.
- If any one execution fails (e.g., assertion fails), that particular case is marked as failed in the report.
- You get detailed feedback showing **which input caused the failure**.

```java
@ParameterizedTest
@ValueSource(ints = {1, 2, 3})
void testWithMultipleValues(int value) {
    assertTrue(value > 0);
}
```

### Getting tests from files:

```java

@ParameterizedTest
@CsvFileSource(resources = "/test-data.csv", numLinesToSkip = 1)
void testWithCsvFile(int a, int b, int expected) {
    assertEquals(expected, a + b);
}

```

1. The CSV file must be in the **`resources/`** folder.
2. You can skip the header with `numLinesToSkip = 1`.
3. The method parameters **must match** the number and type of columns in the file.

JUnit automatically **converts** string values from the data source to the types in your test method’s parameters. It uses internal converters for primitives, enums, and even custom types (if you write converters).

If conversion fails, the test fails at runtime with a helpful error message.

**OpenTest4J** is a **common, lightweight test assertion and reporting library** used behind the scenes by **JUnit 5** (and other test frameworks) to handle **failures and assertion errors** in a standard way.

The **test instance lifecycle** defines **how often a new instance of your test class is created**.

| Mode                   | Description                                                     |
| ---------------------- | --------------------------------------------------------------- |
| `PER_METHOD` (default) | A **new test class instance is created for each test method**   |
| `PER_CLASS`            | A **single test class instance is reused for all test methods** |

```java

@TestInstance(TestInstance.Lifecycle.PER_METHOD)
class MyTest {
    @BeforeEach
    void init() {
        System.out.println("BeforeEach");
    }
    @Test
    void test1() { }
    @Test
    void test2() { }
}
```

- JUnit **creates a new instance** of `MyTest` **for each test method**.
- `@BeforeEach` runs on **each new instance**.
- Any **state (fields)** initialized in one test method **does not affect** another.

In default -> beforeall and afterAll are static

