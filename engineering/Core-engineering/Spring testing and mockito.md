
---

Three types of tests

| Type                 | Scope                                                          | Tools                           |
| -------------------- | -------------------------------------------------------------- | ------------------------------- |
| **Unit Test**        | Test single classes in isolation (e.g., services, utilities)   | JUnit, Mockito                  |
| **Integration Test** | Test multiple layers (e.g., controller → service → repository) | Spring Boot Test                |
| **End-to-End (E2E)** | Test the whole application (e.g., via REST APIs)               | TestRestTemplate, WebTestClient |
## 1. Spring context loading:

The `@SpringBootTest` annotation is used when you want to **load the complete Spring Boot application context** in your tests. This is closest to what your app experiences in production — all beans, components, configurations, and properties are loaded. It’s a full integration test setup and is ideal for **end-to-end tests**, **service layer validation**, and **testing component interaction**.

```java
@SpringBootTest
class OrderServiceIntegrationTest {

    @Autowired
    private OrderService orderService;

    @Test
    void testProcessOrder() {
        orderService.process("ORDER123");
        // Verifies full flow — repository, service, etc.
    }
}
```

Behind the scenes, Spring Boot starts an `ApplicationContext`, autowires dependencies, reads `application.properties` or YAML files, and injects everything like it would in a real app. It's powerful but **slower**, so it's best to **limit its use to high-value integration tests**.

Now in the full SpringBootTesting:

`@MockBean` is used in **Spring Boot tests** (like `@SpringBootTest` or `@WebMvcTest`) to **replace a real Spring bean with a Mockito mock**. Unlike plain `@Mock`, which doesn’t involve Spring, `@MockBean` actually **injects the mock into the Spring container**, so any component that `@Autowired`s the real bean will get the mock instead.

```java
@SpringBootTest
public class UserServiceIntegrationTest {

    @Autowired
    private UserService userService;

    @MockBean
    private UserRepository userRepository;

    @Test
    void testGetUserById() {
        // Arrange: stub the repository mock
        User mockUser = new User(1L, "Alice");
        Mockito.when(userRepository.findById(1L)).thenReturn(Optional.of(mockUser));

        // Act
        User user = userService.getUserById(1L);

        // Assert
        Assertions.assertNotNull(user);
        Assertions.assertEquals("Alice", user.getName());

        // Verify repository interaction
        Mockito.verify(userRepository).findById(1L);
    }
}
```

@Mock is provided by mockito and is used to create mocks in unit tests where spring context is not loaded. These mocks are not registered in spring application context.

@MockBean Provided by **Spring Boot’s testing support**. Creates a **mock bean and replaces the real bean** in the **Spring ApplicationContext**. Used in **Spring Boot integration tests** or slice tests (e.g., `@SpringBootTest`, `@WebMvcTest`

Ensures that when Spring injects the bean anywhere, the **mocked version is used**.Useful for **mocking dependencies of Spring beans** during integration testing.

| Aspect         | `@Mock`                        | `@MockBean`                                       |
| -------------- | ------------------------------ | ------------------------------------------------- |
| Provided by    | Mockito                        | Spring Boot Test                                  |
| Use case       | Unit tests (no Spring context) | Integration or slice tests (with Spring context)  |
| Spring context | No (does not register bean)    | Yes (replaces bean in ApplicationContext)         |
| Initialization | Requires manual init or runner | Managed automatically by Spring Boot Test         |
| Effect         | Creates mock object only       | Creates mock and injects it as Spring bean        |
| Purpose        | Mock dependencies manually     | Replace real Spring beans with mocks during tests |

## 2. Testing controllers

The `@WebMvcTest` annotation tells Spring to **only load web-related components**: controllers, filters, controllers' dependencies (if mocked), and MVC configuration. It’s designed specifically for testing **REST APIs or web controllers** without loading service, repository, or database layers.

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ProductService productService;

    @Test
    void shouldReturnProductById() throws Exception {
        when(productService.getProduct("P001"))
            .thenReturn(new Product("P001", "Laptop"));

        mockMvc.perform(get("/products/P001"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("Laptop"));
    }
}
```

## 3. Slice testing:

- Slice testing loads **only the Spring beans relevant to a particular layer**, e.g.:
    - Controller layer (`@WebMvcTest`)
    - Repository layer (`@DataJpaTest`)
    - Service layer (no built-in annotation, but you can use `@ContextConfiguration` with mocks)
- It’s **faster and lighter** than full `@SpringBootTest`.
- Ideal for testing just **one layer’s behavior** with dependencies mocked or excluded.

Example say we want to test this:

```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.getUserById(id);
        if (user == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(user);
    }
}
```

```java
@WebMvcTest(UserController.class)
public class UserControllerTest {
    @Autowired
    private MockMvc mockMvc;
    @MockBean
    private UserService userService;

    @Test
    void testGetUserFound() throws Exception {
        User mockUser = new User(1L, "Alice");
        Mockito.when(userService.getUserById(1L)).thenReturn(mockUser);

        mockMvc.perform(get("/users/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("Alice"));
    }

    @Test
    void testGetUserNotFound() throws Exception {
        Mockito.when(userService.getUserById(1L)).thenReturn(null);

        mockMvc.perform(get("/users/1"))
               .andExpect(status().isNotFound());
    }
}
```

## Controller testing in depth:

At its core, controller testing verifies the HTTP request handling, parameter mapping, response generation, and error management, without involving the underlying business logic or data access layers directly. Instead, those layers are usually mocked to isolate the controller’s behavior.

To effectively test controllers, Spring offers the `@WebMvcTest` annotation, which sets up a minimal Spring application context focused solely on MVC components like controllers, filters, interceptors, and related configuration. This means that unlike a full `@SpringBootTest`, you won’t load the entire application context with services, repositories, or database configurations, making your tests faster and more focused.

Within this setup, the `MockMvc` utility is your main tool. It simulates HTTP requests against your controller layer, allowing you to send GET, POST, PUT, DELETE, or other types of requests, along with parameters, headers, cookies, and payloads as needed. This lets you assert the response status codes, response content (often JSON or XML), response headers, and even the presence or absence of cookies. Because the test runs without a real web server, it’s much faster and easier to execute as part of automated testing.

When writing controller tests, you generally mock the service or repository beans that the controller depends on by using `@MockBean`. This ensures that when the controller calls these dependencies, they return predefined responses or behaviors, isolating your tests to only the controller logic and preventing side effects or slow database interactions.

ontroller testing focuses on verifying the **behavior of Spring MVC controllers**. This means you test how controllers map HTTP requests, process input parameters, invoke services, and construct responses. It usually includes testing:

- URL mapping and HTTP methods (GET, POST, PUT, DELETE, etc.)
- Request parameters, path variables, and headers
- Response status codes (200, 404, 400, 500, etc.)
- Response body content (JSON, XML, etc.)
- Exception handling and error messages

```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void testGetUser() throws Exception {
        when(userService.findUser("123")).thenReturn(new User("123", "Alice"));

        mockMvc.perform(get("/users/123"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("Alice"));
    }
}
```

- We annotate the test with `@WebMvcTest` specifying the controller under test.
- `MockMvc` is autowired to perform mock HTTP calls.
- `UserService` is mocked with `@MockBean` because it’s a dependency injected into the controller.
- We stub the service method to return test data.
- `mockMvc.perform()` simulates an HTTP GET to `/users/123`.
- We use `.andExpect()` to assert the response status and JSON content.

### MockMVC api:

The `MockMvc` API is one of the most powerful and commonly used tools in Spring MVC testing. It provides a way to simulate HTTP requests and responses against your controller layer **without needing to start a real web server**. This makes controller testing faster, more isolated, and more efficient, as it interacts with the MVC framework directly within the test JVM.

At its core, `MockMvc` allows you to **build and execute requests**—GET, POST, PUT, DELETE, PATCH, and more—against your controllers. You create requests with various parameters, headers, content bodies, and then you perform the request, capturing the result for assertions.

##### How It Works Internally

Under the hood, `MockMvc` acts as a thin layer over the Spring MVC DispatcherServlet and related components. Instead of sending the HTTP request over the network, it directly invokes the dispatcher with a mock request object. The dispatcher routes the request to the controller, applies interceptors and filters, executes controller methods, and builds a mock response. This allows the test code to observe the response, status codes, headers, and body content as if it were a real HTTP interaction, but all in-memory.

##### Creating and Configuring Requests

You start building a request using static methods from `MockMvcRequestBuilders`, such as `get()`, `post()`, `put()`, etc. These methods return a `MockHttpServletRequestBuilder` on which you can set:

- **URL and path variables** (e.g., `get("/users/{id}", 1)`)
- **Query parameters** via `.param("key", "value")`
- **Headers** via `.header("Authorization", "Bearer xyz")`
- **Cookies** with `.cookie(new Cookie("session", "abc123"))`
- **Request body content** with `.content(String)` or `.content(byte[])`
- **Content-Type and Accept headers** to specify media types, e.g., JSON or XML, using `.contentType(MediaType.APPLICATION_JSON)` and `.accept(MediaType.APPLICATION_JSON)`


##### Performing the Request

Once configured, you execute the request with `mockMvc.perform(requestBuilder)`. This returns a `ResultActions` object, which lets you chain assertions about the response.

##### Response Assertions

The `ResultActions` API provides the `.andExpect()` method for verifying aspects of the response:

- `status()` checks HTTP status codes (`isOk()`, `isNotFound()`, `isBadRequest()`, etc.)
- `content()` verifies the response body, including exact content, content type, or JSON path expressions (e.g., `.andExpect(jsonPath("$.name").value("Alice"))`)
- `header()` verifies response headers
- `cookie()` verifies cookies set in the response

You can chain multiple `andExpect()` calls to check various parts of the response.

```java
mockMvc.perform(post("/api/users")
        .contentType(MediaType.APPLICATION_JSON)
        .content("{\"name\":\"John\",\"email\":\"john@example.com\"}")
        .accept(MediaType.APPLICATION_JSON))
       .andExpect(status().isCreated())
       .andExpect(header().string("Location", "/api/users/1"))
       .andExpect(jsonPath("$.name").value("John"))
       .andDo(print());
```

## Example:

```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User createdUser = userService.createUser(user);
        URI location = URI.create("/users/" + createdUser.getId());
        return ResponseEntity.created(location).body(createdUser);
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Integer id) {
        return userService.getUserById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
}

```

Tests:

```java

@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void testCreateUser() throws Exception {
        String userJson = """
            {
                "name": "Alice",
                "email": "alice@example.com"
            }
        """;

        User createdUser = new User();
        createdUser.setId(1);
        createdUser.setName("Alice");
        createdUser.setEmail("alice@example.com");

        when(userService.createUser(any(User.class))).thenReturn(createdUser);

        mockMvc.perform(post("/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(userJson)
                .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isCreated())
            .andExpect(header().string("Location", "/users/1"))
            .andExpect(jsonPath("$.id").value(1))  // MEANS JSON PATH OF RETURNED OBJECT
            .andExpect(jsonPath("$.name").value("Alice"))
            .andExpect(jsonPath("$.email").value("alice@example.com"));
    }

    @Test
    void testGetUser() throws Exception {
        User user = new User();
        user.setId(1);
        user.setName("Bob");
        user.setEmail("bob@example.com");

        when(userService.getUserById(1)).thenReturn(Optional.of(user));

        mockMvc.perform(get("/users/1")
                .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.name").value("Bob"))
            .andExpect(jsonPath("$.email").value("bob@example.com"));
    }

    @Test
    void testGetUser_NotFound() throws Exception {
        when(userService.getUserById(999)).thenReturn(Optional.empty());

        mockMvc.perform(get("/users/999")
                .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isNotFound());
    }
}
```

`MockMvc` is part of Spring Test framework. It **mimics the behavior of a Servlet container** and dispatches mock HTTP requests through the Spring MVC stack **without needing an actual web server** (like Tomcat).

MockMvc creates a **mock HTTP request and response** objects (`MockHttpServletRequest` and `MockHttpServletResponse`). These objects simulate real HTTP request and response as if they came from a client over the network.

Internally, MockMvc uses a `DispatcherServlet` instance — the same front controller used in real Spring MVC apps.

- The mock request is passed to this **DispatcherServlet**.
- `DispatcherServlet` routes the request based on URL, HTTP method, and parameters.
- It consults the **HandlerMapping** to find the correct controller method to invoke.
- After locating the right controller and method, it invokes it using **HandlerAdapter**.
- The controller method executes, returning a result (like a `ResponseEntity`, `ModelAndView`, or plain object).
- The `HandlerResult` is processed by **ViewResolvers** or **HttpMessageConverters** to produce the HTTP response content.
- Finally, the generated response is written to the `MockHttpServletResponse`.

- Normally, `DispatcherServlet` runs inside a Servlet container.
- MockMvc creates a **standalone Servlet environment in memory**, bypassing the network layer.
- It uses mock implementations of request and response objects to simulate the HTTP lifecycle.
- Since everything is in-memory, no actual HTTP calls or socket communication occur.
- This makes tests fast and isolated.

```pgsql
mockMvc.perform(request)
    ↓
create MockHttpServletRequest & MockHttpServletResponse
    ↓
DispatcherServlet.dispatchRequest(mockRequest, mockResponse)
    ↓
HandlerMapping → find controller method
    ↓
HandlerAdapter → invoke controller method
    ↓
Controller method executes → returns result
    ↓
ViewResolver / MessageConverter → prepare response content
    ↓
Write content to MockHttpServletResponse
    ↓
Return MvcResult with response accessible for assertions
```

### Workout example::

```java
package com.example.demo.controller;  
  
import com.example.demo.modal.Product;  
import com.example.demo.repository.ProductRepository;  
import org.junit.jupiter.api.*;  
import org.junit.jupiter.params.ParameterizedTest;  
import org.junit.jupiter.params.provider.CsvSource;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;  
import org.springframework.data.domain.Page;  
import org.springframework.data.domain.PageImpl;  
import org.springframework.data.domain.Pageable;  
import org.springframework.http.MediaType;  
import org.springframework.test.context.bean.override.mockito.MockitoBean;  
import org.springframework.test.web.servlet.MockMvc;  
  
import static org.mockito.ArgumentMatchers.any;  
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;  
  
import java.lang.reflect.Array;  
import java.time.LocalDateTime;  
import java.util.Arrays;  
import java.util.Collections;  
import java.util.List;  
import java.util.Optional;  
  
import static org.mockito.Mockito.when;  
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;  
  
@WebMvcTest(ProductController.class)  
class ProductControllerTest {  
  
    @Autowired  
    private MockMvc mockMvc;  
  
    @MockitoBean  
    private ProductRepository productRepository;  
  
    @BeforeEach  
    void init(){  
  
        Product product = new Product();  
  
        product.setId(1L);  
        product.setName("Product 1");  
        product.setDescription("Product 1 description");  
        product.setPrice(100);  
        product.setCreatedAt(LocalDateTime.now());  
        product.setOwnerId(1L);  
  
        when(productRepository.findAll()).thenReturn(Collections.singletonList(product));  
  
        when(productRepository.findById(any())).thenReturn(Optional.of(product));  
  
        when(productRepository.getProductsPage(1,1)).thenReturn(List.of(product));  
  
        Page<Product> page = new PageImpl<>(List.of(product));  
  
        when(productRepository.findAll(any(Pageable.class))).thenReturn(page);  
    }  
  
    @Nested  
    @DisplayName("Get requests test")  
    class GetProductTests{  
  
        @Test  
        void allEndPointTest() throws Exception {  
            mockMvc.perform(get("/all")  
                    .accept(MediaType.APPLICATION_JSON)).  
                    andExpect(status().isOk())  
                    .andExpect(jsonPath("$[0].id").value(1))  
                    .andExpect(jsonPath("$[0].name").value("Product 1"));  
        }  
  
        @ParameterizedTest  
        @CsvSource({  
                "1",  
                "2",  
                "3",  
                "4"  
        })  
        void getByIdTest(String ids) throws Exception {  
            mockMvc.perform(get("/id")  
                .param("id",ids)  
                .accept(MediaType.APPLICATION_JSON))  
                .andExpect(status().isOk())  
                .andExpect(content().string("Product 1"));  
        }  
  
        @Test  
        void testTheCustomPage() throws Exception {  
            mockMvc.perform(get("/custom-page")  
                    .param("pageNo","1")  
                    .accept(MediaType.APPLICATION_JSON))  
                    .andExpect(status().isOk());  
        }  
  
        @Test  
        void checkTheHello() throws Exception {  
            mockMvc.perform(get("/")  
                .accept(MediaType.APPLICATION_JSON))  
                .andExpect(status().isOk())  
                .andExpect(content().string("Hello"));  
        }  
  
        @Test  
        void checkGetPage() throws Exception {  
            mockMvc.perform(get("/page")  
                    .param("pageNo","1")  
                    .accept(MediaType.APPLICATION_JSON))  
                    .andExpect(status().isOk());  
        }  
    }  
  
}
```

