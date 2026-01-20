
---

### Working and tutorials:

Creating mock object.

```java
MyService mockService = Mockito.mock(MyService.class);
```

In **Mockito**, **stubbing** refers to the process of specifying the **behavior of a mock object** — essentially, telling the mock what to do when a specific method is called with specific parameters.

Without stubbing mocks returns default values.

```java
List<String> mockList = Mockito.mock(List.class);

System.out.println(mockList.get(0)); // Output: null (default for object return types)
System.out.println(mockList.size()); // Output: 0 (default for int)
```

Stubbing and doing something

```java
List<String> mockList = Mockito.mock(List.class);
when(mockList.get(0)).thenReturn("Hello");

System.out.println(mockList.get(0)); // Output: Hello
System.out.println(mockList.get(1)); // Output: null (not stubbed)
```

Verifying if the mock was called or not:

Mocks are stubs + if u verify it if it was verified or not.

```java

List<String> mockList = Mockito.mock(List.class);

mockList.add("test");
mockList.add("another");

verify(mockList).add("test");
verify(mockList).add("another");    
verify(mockList, times(2)).add(anyString());

```

### Full example:

```java
class UserService {
    private final UserRepository repo;

    public UserService(UserRepository repo) {
        this.repo = repo;
    }

    public String getUsername(String id) {
        User user = repo.findById(id);
        return user != null ? user.getName() : "Unknown";
    }
}
```

```java
class UserServiceTest {

    @Test
    void testGetUsername() {
        // 1. Creating  mock
        UserRepository mockRepo = Mockito.mock(UserRepository.class);

        // 2. Stub behavior
        when(mockRepo.findById("123")).thenReturn(new User("123", "Alice"));

        // 3. Inject mock into service
        UserService service = new UserService(mockRepo);

        // 4. Run and assert
        assertEquals("Alice", service.getUsername("123"));

        // 5. Verify interaction
        verify(mockRepo).findById("123");
    }
}
```


Using annotations:

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    UserRepository repo;

    @InjectMocks
    UserService service;

    @Test
    void testGetUsername() {
        when(repo.findById("123")).thenReturn(new User("123", "Alice"));
        assertEquals("Alice", service.getUsername("123"));
        verify(repo).findById("123");
    }
}

```


Discussion::

Note the first example the user repo is passed to UserService through constructor. 

```java
UserRepository mockRepo = Mockito.mock(UserRepository.class);
UserService userService = new UserService(mockRepo);
```

If your class had **multiple dependencies**, you'd have to manually create mocks for each and pass them all into the constructor (or setters). This gets **verbose and repetitive**, especially when testing service layers with many dependencies.

Now with @Mock

```java
@Mock
UserRepository userRepository;
```

Mockito will create a mock version of `UserRepository` and store it in that field.

This annotation tells Mockito to **create an instance of the class** and then **inject mocks into it**. 

```java
@InjectMocks
UserService userService;
```

## What If My Class Uses Spring’s `@Autowired`?

Mockito's `@InjectMocks` works independently of Spring. However, if you're using Spring's `@Autowired` or constructor injection, it still works the same way. Mockito will try to:

- Match mock objects to constructor arguments first (if available)
- Then use field-level injection for the rest
## Under the Hood: How Mock Injection Works

When you run your test with `@ExtendWith(MockitoExtension.class)` (for JUnit 5), the extension calls `MockitoAnnotations.openMocks(this)` behind the scenes. This does several things:

1. **Scans the test class** for fields annotated with `@Mock` and creates mock instances using `Mockito.mock(...)`.
2. **Scans for fields annotated with `@InjectMocks`**.
3. When it finds an `@InjectMocks` field:
    - It creates an instance of the class.
    - It tries to **match the available mocks** to the class’s constructor, fields, or setter methods.
    - It **injects mocks** by:
        - Preferring constructor injection (if constructor parameters match mocks)
        - Then trying field injection
        - Then trying setter injection
4. After this, the `@InjectMocks` field is ready to be used with all its dependencies mocked.

### Spy:

In **Mockito**, a **spy object** is a special kind of test double that allows you to **wrap a real object** and **partially mock** its behavior. This means that you can call real methods on the object unless you explicitly stub them.

| Feature           | `@Mock` or `mock()`                                           | `@Spy` or `spy()`                         |
| ----------------- | ------------------------------------------------------------- | ----------------------------------------- |
| Behavior          | By default, all methods return default values (null, 0, etc.) | Calls the real methods unless stubbed     |
| Use case          | When you want a full mock                                     | When you want to partially mock an object |
| Underlying Object | Fake                                                          | Real instance                             |

```java
import static org.mockito.Mockito.*;

import java.util.ArrayList;
import java.util.List;

public class SpyExample {
    public static void main(String[] args) {
        // Real list
        List<String> list = new ArrayList<>();

        // Create a spy of the real list
        List<String> spyList = spy(list);

        // Real behavior
        spyList.add("A");
        spyList.add("B");

        System.out.println(spyList.size()); // Output: 2

        // Stub a method
        when(spyList.size()).thenReturn(100);

        System.out.println(spyList.size()); // Output: 100

        // Verify interaction
        verify(spyList).add("A");
        verify(spyList).add("B");
    }
}

```

### Internals:

**Mockito** is a Java-based **mocking framework** that allows developers to **create mock objects** for unit testing. A mock object is a simulated object that mimics the behavior of real objects in a controlled way. In unit testing, we often want to test a class **in isolation**, without depending on its real collaborators (like databases, external APIs, or complex services). Mockito makes this possible by allowing us to **mock those dependencies**, control their behavior, and verify how they were used — all without invoking their actual implementations.

Mockito works primarily by using **Java's dynamic proxy** mechanism and **bytecode manipulation** (using libraries like Byte Buddy) to create mock instances at runtime. When you create a mock using `Mockito.mock(SomeClass.class)`, you're not getting a real instance — instead, you're getting a **proxy object** that implements the same interface (or extends the class) and can be configured to behave in specific ways. You can instruct the mock to return specific values when methods are called (`when(...).thenReturn(...)`), to throw exceptions (`thenThrow(...)`), or to do nothing (`doNothing()` for void methods). Mockito also allows **verification**: you can check whether a certain method was called, how many times it was called, and with what arguments.

In practice, you first **create a mock** using `@Mock` or `Mockito.mock()`, then **inject it** into the class under test. You use `@InjectMocks` to automatically inject mock dependencies into your class, and `MockitoAnnotations.openMocks(this)` (or use MockitoJUnitRunner) to initialize the mocks. During the test, you can **stub** method responses using `when(...).thenReturn(...)`. After executing the test logic, you can **verify interactions** using `Mockito.verify(...)` to confirm expected behavior.

```java

@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    @Mock
    private UserRepository userRepository;
    @InjectMocks
    private UserService userService;
    @Test
    void testFindUserById() {
        User mockUser = new User("john", "John Doe");
        when(userRepository.findById("john")).thenReturn(Optional.of(mockUser));

        User result = userService.findUser("john");

        assertEquals("John Doe", result.getName());
        verify(userRepository).findById("john"); // Verifies the method was called
    }
}

```

Mockito provides a rich set of features beyond basic mocking. It supports **spying**, which allows you to partially mock real objects (real methods and mocked methods mixed).

### Internal working of mockito:

Mockito isn't just magic — it uses powerful Java features like **reflection**, **dynamic proxies**, and **bytecode manipulation** to simulate the behavior of real objects. Let’s break down how this works step-by-step in detailed paragraphs.

 Whether you're telling it to return a specific value, throw an exception, or verify that something was invoked — Mockito controls the flow **without touching your real business logic**. This interception is made possible using **Java Reflection** and **bytecode generation**.

When you call:

```java
MyService service = Mockito.mock(MyService.class);
```

Mockito internally uses **one of two strategies** to create the mock object:

- If `MyService` is an **interface**, Mockito uses Java's built-in **dynamic proxy** mechanism (`java.lang.reflect.Proxy`) to generate a proxy class at runtime.
- If `MyService` is a **concrete class**, Mockito uses **bytecode manipulation** via the **Byte Buddy** library to create a **subclass** of `MyService` at runtime. This subclass **overrides all methods** and injects mock behavior.

These proxy or subclass instances are what you interact with in your tests — not the real class.

Every method call on the mock is **intercepted** and routed to a central handler inside Mockito. This is where it decides:

- Is this method part of a **stubbing**? (e.g., `when(...).thenReturn(...)`)
- Is it part of a **verification**? (e.g., `verify(...).someMethod()`)
- Should it just return the **default value**? (e.g., `null`, `0`, `false`, empty list)
- Should it **throw an exception**?

The interception happens by overriding the method or using an invocation handler. All the intercepted calls are tracked and stored in an internal registry for verification and reporting.

```java
when(mock.getName()).thenReturn("John");
```

Mockito registers a **stubbed method** in its internal map. It uses a combination of:

- The method signature,
- The method arguments,
- The instance itself

to create a **key**. When the method is called during the test, Mockito checks the key against this internal map to determine what value or behavior to return.

```java
verify(mock).getName();
```

Mockito checks its internal call history (a **call log**) for the mock object. Every intercepted method call is recorded with details such as:

- Method name
- Arguments passed
- Invocation time

Mockito compares the recorded calls against what you're verifying. If there's a mismatch, it throws an exception with a **detailed failure message**, showing what was expected and what was actually called.

Note: If you don't create the mock objects then Mockito returns the dafault values so that tests don't terminate unexpectedly.

### Components of Mock:

#### MockHandler:

At the heart of every mock is the **`MockHandler`** interface (usually implemented by `MockHandlerImpl`). This class is responsible for **handling all intercepted method calls** on a mock object. Whenever a mock is invoked, that call is delegated to the `MockHandler`, which acts as a **central dispatcher**. It checks whether the call corresponds to a **stubbing** (like `thenReturn`) or if it's part of a **verification** (`verify(...)`), and then it responds accordingly — either by returning a predefined value, recording the call, or throwing an error.

#### Invocation:

The `Invocation` class represents a **single method call** on a mock. It encapsulates all the details of that call, such as:

- The **method signature**
- The **mock object** itself
- The **arguments passed**
- The **return type**
- The **call time** and **stack trace**

This is crucial because Mockito records **every method invocation** in an `Invocation` object and stores it for later use

### InvocationContainerImpl:

`InvocationContainerImpl` is an internal class that acts as the **storage** for all method invocations and stubbing instructions related to a mock. It keeps two main things:

1. A **history of all invocations** (as `Invocation` objects)
    
2. A list of **stubbing rules**, so that when a specific method is called with certain arguments, Mockito knows what to return

`VerificationMode` is used when you run `verify(...)` statements. It defines **how Mockito checks whether a method was called**, how many times, and with what arguments. For example, when you write:

```java
verify(mock, times(2)).doSomething();
```

The `times(2)` part returns a `VerificationMode` instance that internally holds:

- The **expected number of invocations**
- Possibly a **timeout or delay** (in asynchronous cases)
- Optional modes like `never()`, `atLeast()`, `atMost()`, etc.


When you create and use a mock, these components work together like this:

1. `MockCreationSettings` stores how the mock is configured.
2. `MockHandler` intercepts every method call on the mock.
3. For each method call, an `Invocation` object is created.
4. `InvocationContainerImpl` stores the `Invocation` and checks for matching stubbings.
5. During `verify()`, the `VerificationMode` compares the expected vs. actual invocations.

