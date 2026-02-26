
---


A testing library contains 5 important components:-

1. Discovery
2. Execution
3. Assertions
4. Reporting 
5. Mocking 

### Test discovery:

Before tests can be executed, a testing library must identify which parts of the codebase are test cases. This is known as _test discovery_. Most libraries adopt either **convention over configuration** or **explicit annotation/registration** to discover tests.

In **JUnit**, test classes are identified by annotations like `@Test`, `@BeforeEach`, or `@AfterAll`. Java supports reflection via the `java.lang.reflect` package, allowing the framework to inspect class metadata and extract test methods dynamically.

The discovery phase also checks for fixtures, test parameterization, custom markers (like `@slow`, `@database`), and setup/teardown routines. Internally, this is managed through a _test registry_ — a data structure (often a list or tree) that stores discovered test cases and their associated metadata for execution.

### Test collection and organisation:

Once discovered, tests need to be organized into a coherent structure for execution. This involves grouping them into **test suites**. A test suite is essentially a logical collection of test cases, possibly nested, that can be run as a batch. Libraries often organize these suites based on the file structure, test class, or explicit grouping defined by the user.

In **Jest** (JavaScript), the entire test file is considered a suite. Within that, individual tests are grouped with `describe()` blocks, which create nested contexts. During runtime, Jest collects test definitions and stores them in an in-memory structure representing a tree of suites and tests.

### Executions:

The core idea here is to **wrap the execution in a try-catch (or try-except)** block to handle assertion failures and unexpected exceptions. The framework logs the results of each test execution — whether it passed, failed, was skipped, or errored out.

Execution often includes **hooks** like:

- `beforeEach` / `afterEach` in Jest,
- `setUp` / `tearDown` in unit test (Python),
- `@Before` / `@After` in JUnit.

Some test runners (like Jest ) support **parallel execution**. This requires creating separate threads or processes to isolate tests, ensuring that shared state or global variables do not interfere. This is usually achieved using the language’s multiprocessing libraries and inter-process communication mechanisms.

- The core of the execution is calling the method marked with `@Test`. 
- JUnit instantiates the test class (unless configured differently).
- The method is invoked.
- Any **assertions** (`Assertions.assertEquals`, etc.) or **exceptions** during the method define the result:
	- No exception + assertions pass → ✅ success.
    - Assertion fails or exception thrown → ❌ failure or error.

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

Jupiter test engine:

When you run tests on the JUnit Platform (the foundation of JUnit 5), it supports multiple **test engines**, each designed to execute tests written in different styles or frameworks. The Jupiter Test Engine specifically handles all the tests annotated with Jupiter’s annotations like `@Test`, `@BeforeEach`, `@AfterAll`

JUnit 5 is **not a single library**, but a **platform** that supports multiple test engines, including the **Jupiter Test Engine**, the **Vintage Test Engine** (for running older JUnit 3/4 tests), and potentially others.

The **JUnit Platform** provides the foundation for launching tests, discovery, and reporting, but it does not define the test syntax or behavior itself.

The **Jupiter Test Engine** is the **implementation** of the JUnit Platform’s test engine API that runs tests written specifically using the **JUnit Jupiter programming model**.

The Jupiter Test Engine is **one part** of the overall JUnit 5 platform but is responsible only for discovering and executing Jupiter-style tests.
### Listner interface for IDEs:

The **`TestExecutionListener`** interface in JUnit 5 is a key part of the JUnit Platform’s infrastructure that enables monitoring and reporting the progress and results of test execution. Essentially, it is a callback mechanism that lets various tools, such as IDEs, build systems (like Maven or Gradle), or custom reporters, get notified about the lifecycle events of tests running on the platform. This allows these tools to provide real-time feedback about which tests are starting, which have finished, whether they passed or failed, or if any were skipped or aborted.

When a test run begins, the JUnit Platform invokes methods on the registered `TestExecutionListener` implementations to signal specific events. For example, when a test method or test container (like a test class or nested test) starts executing, the listener’s `executionStarted()` method is called with a `TestIdentifier` representing that test or container. Similarly, when execution finishes, the platform calls `executionFinished()` and passes the result, including the success or failure status and any exception thrown. There are also methods like `executionSkipped()` to report tests that didn’t run, and `reportingEntryPublished()` for publishing additional information during test runs.

### Java hooking:

JUnit hooks into the call stack and the internals of a running test method primarily through **Java reflection** and **its test lifecycle management**, allowing it to control, observe, and intervene during test execution without changing the actual test code.

When you run a test method annotated with `@Test`, JUnit doesn't just call the method directly like normal code. Instead, it uses the **Java Reflection API** to dynamically invoke the test method at runtime. Reflection lets JUnit inspect the method metadata (like annotations) and then call the method on an instance of the test class without knowing its details at compile time. This dynamic invocation allows JUnit to insert its own logic before and after the actual test method runs.

Before the test method executes, JUnit runs any lifecycle hooks such as `@BeforeEach` methods, again via reflection. It can set up necessary state or prepare the environment. Then, JUnit invokes the test method itself, catching any exceptions or assertion failures as they occur. This exception handling is crucial because it enables JUnit to determine whether the test passed or failed and to report errors precisely.

After the test method finishes (or fails), JUnit executes `@AfterEach` lifecycle methods, allowing cleanup or additional checks. All of these steps happen within a controlled call stack managed by JUnit’s test engine, which wraps the method invocation in try-catch blocks and notifies registered listeners about test progress and results.

In addition to reflection, JUnit 5 introduces an **extension model** that can intercept calls around test methods. Extensions can wrap the method invocation with additional code, effectively hooking deeper into the call stack and allowing behaviors like parameter injection, conditional test execution, or retry logic.

In summary, JUnit hooks into the call stack of a running test method by using Java reflection to invoke the method dynamically, surrounding that call with setup and teardown hooks, and managing exceptions to monitor test outcomes. This approach gives JUnit full control over test execution while keeping the test code clean and declarative.


> [!NOTE] Java reflection API
> The **Java Reflection API** is a powerful feature of the Java programming language that allows programs to inspect and manipulate the runtime behavior of applications. Through reflection, Java code can analyze itself or other classes, methods, fields, and constructors dynamically at runtime, even if those elements were not known at compile time. This capability is crucial for frameworks, libraries, and tools—like JUnit—that need to operate flexibly without hardcoding specific class or method names.
   At its core, the Reflection API provides a set of classes and interfaces in the `java.lang.reflect` package, such as `Class`, `Method`, `Field`, and `Constructor`. These allow you to obtain metadata about classes (e.g., what methods they have, their parameter types, annotations), create new instances of objects dynamically, invoke methods, and access or modify fields — all during runtime. For example, you can load a class by name, examine its declared methods, and invoke one of those methods on an instance without having direct compile-time references to that class or method.
   Reflection also allows reading and modifying annotations, which is essential for modern Java frameworks that rely heavily on annotations to configure behavior, like dependency injection, transaction management, or test execution. However, using reflection comes with trade-offs: it can bypass normal compile-time safety checks, can impact performance due to dynamic type resolution and method dispatch, and may violate encapsulation principles by accessing private members.
   In testing frameworks like JUnit, reflection enables the discovery of test methods annotated with `@Test` and the invocation of those methods on freshly created instances of test classes. This dynamic invocation allows JUnit to control the execution lifecycle of tests without requiring any special boilerplate in the test code. Essentially, reflection makes Java code adaptable and extensible, providing the foundation for many sophisticated tools and frameworks in the Java ecosystem.

### Assertion Mechanisms: Detecting Failures

Assertions are the crux of testing — they validate expected behavior. A failed assertion indicates that the code did not perform as expected. Internally, assertions work by raising exceptions or returning error objects that the test runner can intercept.

In **JUnit**, assertions are method calls like `assertEquals(expected, actual)`. These methods throw exceptions (e.g., `AssertionError`) when the assertion fails. The testing engine catches these and records them in the test result log.

### Result reporting: 

After execution, testing libraries must collect and report the results. This includes which tests passed, which failed, how long they took, and any associated error messages or stack traces.

Most frameworks maintain a **test result object**, usually a tree structure reflecting the suite hierarchy. Each node contains metadata such as:

- Test name,
- Status (passed/failed/skipped),
- Duration,
- Error message (if any),
- Stack trace.

This result is then formatted and output to the terminal, a file (e.g., JUnit XML), or a CI dashboard. Pytest, for instance, uses `TerminalReporter` to output colorful summaries. Many libraries also support plugins to export results in machine-readable formats like **JSON**, **XML**, or **HTML** for integration with other tools.

Failures are typically displayed with stack traces and contextual information. Tools like Jest and Mocha include diff outputs for failed assertions (e.g., expected vs actual), highlighting differences. 

Junit Testing: 

1. We should test one scenario at a time. 
2. Junit test must be annotated with @Test.
3. We use asserts to check conditions.

```java
class CaluclatorTest{   
    @Test  
    void twoPlusTwoEqualsFour(){  
    
        Caluclator caluclator = new Caluclator();  
        assertEquals(4, caluclator.add(2,2));  
        assertNotEquals(3,caluclator.add(2,3));  
        assertNull(null);  
        assertNotNull(caluclator);  
        assertTrue(calculator.add(2,2)==4);
    }  
}
```

All the above assert methods will throw an error if assert fails however if condition passes the error is not thrown and test passes. 

Now lets go over the assertThrows()

At a high level, `assertThrows()` is used in JUnit to **verify that a specific block of code throws an expected exception**. It allows you to ensure that error handling behaves correctly — for example, if a method should throw `IllegalArgumentException` when given invalid input, `assertThrows()` confirms this.


### **JUnit Invokes the Executable Block**

- Internally, JUnit tries to run the code inside the lambda block (your test logic) by calling `executable.execute()`.
- This happens inside a `try-catch` block.

```java
import static org.junit.jupiter.api.Assertions.*;
class CalculatorTest {
    @Test
    void twoPlusTwoThrowsRuntimeException() {
        Calculator calculator = new Calculator();       
        assertThrows(RuntimeException.class, () -> calculator.add('1', '2'));
    }
}
```


Behind the schenes:

```java
try {
    executable.execute(); 
} catch (Throwable actualException) {
    if (expectedType.isInstance(actualException)) {
        return (T) actualException; 
    } else {
        throw new AssertionFailedError("Unexpected exception type", actualException);
    }
}
throw new AssertionFailedError("Expected exception was not thrown");
```

### Code coverage:

There are multiple code coverage metrics, and the most popular are:
- **Line coverage**: which lines of code have been covered by automated tests?
- **Branch coverage**: which execution paths have been covered by automated tests? For example, a line of code like if a && b can have full line coverage, but only partial branch coverage.

Code coverage is a way to measure how much of your source code is executed while running tests, giving insights into which parts of your code are tested and which are not.

Internally, code coverage tools work by modifying your program’s bytecode—the compiled form of your Java classes—before or during test execution. This process is called bytecode instrumentation. During instrumentation, the tool injects extra instructions (called probes) into the bytecode at strategic points, such as before each line, branch, or method. These probes record whether a specific part of the code has been executed when the tests run.

When you execute your tests, the instrumented code collects runtime data about which probes were hit. This information is stored in memory or temporary files as the program runs. After the tests finish, the coverage tool aggregates this execution data, correlates it back to the original source code lines using debugging metadata, and then generates detailed reports. These reports typically highlight covered lines in green and uncovered lines in red, making it easy to see which parts of your code were tested.

Code coverage tools work by hooking into the **test lifecycle** that JUnit manages. When you run your tests using JUnit—whether from the command line, an IDE, or a build tool like Maven or Gradle—the coverage tool interposes itself into this process. Before any test code or application code runs, the coverage tool **instruments** the compiled classes by modifying their bytecode. This instrumentation adds monitoring probes that record execution data when the instrumented code runs during tests.

As JUnit executes your test methods, it is actually running these instrumented versions of your application classes. The inserted probes collect detailed data about which methods, branches, or lines were executed. Because this happens dynamically during the normal test execution lifecycle, the coverage tool can precisely correlate runtime behavior with your source code.

Once the tests finish, the coverage tool collects and aggregates this execution data. It then generates reports—usually in HTML, XML, or other formats—that visually indicate which parts of the code were covered by the tests and which were not. This allows developers to easily see how effective their tests are, identify dead code or untested branches, and prioritize areas for adding or improving tests.

Notes: In junit it work flow is really simple ->

Just by writing the name of function or creating object of that class that passes or fails need not mean that this test is covered. Essentially these lines must run while running those test cases.

Junit after discovering all the tests will run the methods effectively and test is said to pass if and only if method does not creates error. So without any assertion the test is said to have passed. 

Infact assert methods explicitly throw an error when this kind of methods are called. Since error is thrown the test fails and this becomes the way by which java shows where test failed as we get to know the exact stack trace. 
