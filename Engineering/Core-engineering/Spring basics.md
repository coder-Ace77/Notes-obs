
---

### Bean:

_In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container._

Inversion of control means a process in which an object defines its dependencies without actually creating them. 

For example we have address class

```java
public class Address {
    private String street;
    private int number;

    public Address(String street, int number) {
        this.street = street;
        this.number = number;
    }
    // getters and setters
}
```

and company class is using it->

```java
public class Company {
    private Address address;

    public Company(Address address){
        this.address = address;
    }
    // getter, setter and other properties
}
```

And address needs street object which means there are many dependencies to match. 

To create the obejcts normally we have to do this

```java
Address address = new Address("High Street", 1000);
// creating address first and then using it in company object
Company company = new Company(address);
```

Imagine an application with dozens or even hundreds of classes. Sometimes we want to share a single instance of a class across the whole application, other times we need a separate object for each use case, and so on.

Managing such a number of objects is nothing short of a nightmare. **This is where inversion of control comes to the rescue.**

Instead of constructing dependencies by itself, an object can retrieve its dependencies from an IoC container. **All we need to do is to provide the container with appropriate configuration metadata.**

To do that we use annotations. Each annotation means just differnt things and we can define them accordingly.

### Inversion of control and Dependency injection:

Inversion of Control is a broader design principle in which the control of certain processes in the program is transferred from the program itself to an external entity (like a framework or container).

In simpler terms, IoC means that the control over the flow of execution is inverted—it's given to a framework or container, rather than being handled directly by the application. This typically helps in decoupling components in the software, making it more modular, maintainable, and flexible.

Dependency Injection is a specific technique or pattern used to implement IoC. It involves providing the dependencies of a class or component from the outside, rather than the class creating them itself. DI is one of the most common ways to implement Inversion of Control. It makes sure that an object doesn't need to create its own dependencies or manage them; instead, it receives them from an external source, such as a DI container.

**DI Methods**:
1. **Constructor Injection**: Dependencies are passed into the class through its constructor.
2. **Setter Injection**: Dependencies are provided via setter methods after object instantiation.
3. **Interface Injection**: The object provides an injector method to receive its dependencies.

example:

Without inversion:
```java
class Car {
    private Engine engine;
    private Tire tire;

    public Car() {
        engine = new Engine();  // Car is creating its own dependencies.
        tire = new Tire();
    }

    public void start() {
        engine.start();
    }
}
```

With inversion:

Constructor injection:
```java
class Car {
    private Engine engine;
    private Tire tire;

    // Dependencies injected via constructor
    public Car(Engine engine, Tire tire) {
        this.engine = engine;
        this.tire = tire;
    }

    public void start() {
        engine.start();
    }
}
```

Setter injection:

```java
class Car {
    private Engine engine;
    private Tire tire;

    public void setEngine(Engine engine) {
        this.engine = engine;
    }

    public void setTire(Tire tire) {
        this.tire = tire;
    }

    public void start() {
        if (engine != null && tire != null) {
            engine.start();
            tire.inflate();
        } else {
            System.out.println("Engine or Tire not injected!");
        }
    }
}
```

@SpringApplicationContext: Spring _ApplicationContext_ is where Spring holds instances of objects that it has identified to be managed and distributed automatically. These are called beans. **Spring collects bean instances from our application and uses them at the appropriate time.**

	The **ApplicationContext** serves as the container in which Spring beans are defined, initialized, and managed. It is responsible for:
- **Loading and configuring beans** defined in Spring configuration files (XML, annotations, or Java Config).
- **Providing beans** to be injected into other components via DI.
- **Managing the lifecycle** of beans, including instantiation, initialization, and destruction.
- **Supporting event propagation** and handling application-wide events.
### Spring core annotations:

1. @Component: is an annotation that allows Spring to detect our custom beans automatically.

In other words, without having to write any explicit code, Spring will:
- Scan our application for classes annotated with _@Component_
- Instantiate them and inject any specified dependencies into them
- Inject them wherever needed

@Component is used on classes on which we want to put the dependencies. In earlier example it should be done on Car class as we need to inject dependencies on it. 

Observe that Only the class definition was provided the actual handling will be done by spring. But what if we want to have the custom handling. For that we have `@Configuration`.

2. @Configuration: The @Configuration annotation is used to define a configuration class in Spring. A configuration class is responsible for **defining and managing Spring Beans** and is equivalent to an XML-based configuration file.

 Used for defining **custom bean creation logic**, where you explicitly create beans using `@Bean` methods inside a configuration class.

```java
@Configuration
public class AppConfig {

    @Bean
    public Car car() {
        return new Car(engine()); // Creating the Car bean
    }

    @Bean
    public Engine engine() {
        return new Engine(); // Creating the Engine bean
    }
}
```

In Spring, beans are **singleton** by default, which means **only one instance** of a bean will be created per **Spring container** and shared across the application. 

`@Configuration` is essentially a specialization of `@Component`. 
One important feature of `@Configuration` is that Spring **proxies** `@Configuration` classes using **CGLIB (Code Generation Library)**. This proxy mechanism ensures that **bean methods** (like those annotated with `@Bean`) are executed **only once**, even if they are called multiple times.

```java
@Configuration
public class AppConfig {
    @Bean
    public Car car() {
        return new Car(engine()); // car() will only be called once
    }
    @Bean
    public Engine engine() {
        return new Engine("V8");
    }
}
```

Even if you call `car()` multiple times, Spring ensures that it will return **the same instance** of `Car`, ensuring **singleton behavior**.

The `@Configuration` annotation indicates that the class is a **configuration class** for Spring’s **IoC container**. It essentially tells Spring that the class contains **bean definitions**.

The `@Bean` annotation, on the other hand, is used to **define individual beans** in a configuration class. It is placed on methods within a class to tell Spring that the method's return value should be registered as a bean in the Spring container.

So finally what is the differnce between @Component and @Configuration the differnce is that we can explicitly define the creation logic with configuration but not with @Component.

Beans annotated with `@Component` are **automatically discovered** and managed by Spring during classpath scanning, typically using `@ComponentScan`. Spring will automatically instantiate these classes and manage their lifecycle.

Example:

Suppose you need to create the object of some type based on run time environment. So in that case only @Configuration is an viable option.

```java
package com.example;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.beans.factory.annotation.Value;

@Configuration
public class AppConfig {

    @Value("${environment.type}")  // Read the environment type from a property file or system variable
    private String environmentType;

    @Bean
    public PaymentGateway paymentGateway() {
        if ("production".equals(environmentType)) {
            return new ProductionPaymentGateway();  // Return the real payment gateway in production
        } else {
            return new DevelopmentPaymentGateway();  // Return the mock payment gateway in other environments
        }
    }
}
```

Tip: We can get the application config here as 

```java
// Initialize Spring context with configuration class
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

// Get the PaymentGateway bean
PaymentGateway gateway = context.getBean(PaymentGateway.class);

// Use the PaymentGateway
gateway.processPayment(100.0);
```

 **@Bean** - **Purpose**: Declares a method that returns a bean to be managed by the Spring container.
 **@Scope**  **Purpose**: Specifies the scope of a Spring bean (e.g., singleton, prototype).


```java
@Scope("prototype")
@Component
public class MyComponent {
    // this bean will be created anew each time it's requested
}
```

#### Spring strereotype annotations:

Spring has provided a few specialized stereotype annotations: _@Controller_, _@Service_ and _@Repository_. They all provide the same function as _@Component_.

**They all act the same because they are all composed annotations with _@Component_ as a meta-annotation for each of them.** They are like _@Component_ aliases with specialized uses and meaning outside Spring auto-detection or dependency injection.

### Component scan:

Before we rely on @Component, we must understand that its only a plain annotation. However spring also uses @ComponentScan to gather it into Application context.

_@SpringBootApplication_ is a composed annotation that includes _@ComponentScan_.

### Important spring boot annotations:

1. @SpringBootApplications: It is a composition of three different annotations:
	1. Configuration: Marks the class as a source of **Spring bean definitions**.
	2. **`@EnableAutoConfiguration`** Tells Spring Boot to automatically **configure beans** for you based on the **classpath** and existing **bean definitions**. If `spring-boot-starter-web` is in the classpath, it configures a Tomcat server + Spring MVC automatically. If `spring-boot-starter-data-jpa` is there, it sets up an EntityManagerFactory, DataSource, and TransactionManager
	3. Tells Spring to scan the current package (and subpackages) for classes annotated with:
	    - `@Component`
	    - `@Service`
	    - `@Repository`
	    - `@Controller` / `@RestController`
		- This way, Spring automatically detects and registers your beans without extra configuration.

2. @RestController: This annotation is used to indicate that a class is a RESTful controller. It combines `@Controller` and `@ResponseBody`.
	1. `@Controller`
		- Marks a class as a **Spring MVC controller**.
		- Spring will detect it during **component scanning** (`@ComponentScan`) and register it as a bean.
		- A controller usually handles web requests via methods annotated with `@RequestMapping` (or shorthand annotations like `@GetMapping`, `@PostMapping`, etc.).
		 Note that it will return a view not a json kind of object. 
	2.  `@ResponseBody`
		- Tells Spring that the return value of the method should be written **directly into the HTTP response body**.
		- No view resolution is done — the raw return value is sent back (usually as JSON or plain text).
		- Response body can be applied on method as well as class itself.

3. **@RequestMapping:** This annotation is used to map web requests to specific handler methods. It can be applied at the class or method level. We also have some more mappings like 
		@GetMapping , @PostMapping which can also be used to do the same.
4. ** @Autowired:** This annotation is used to automatically wire dependencies in Spring beans. It can be applied to fields, constructors, or methods.
		If applied at a field it will mean the setter injection However we can also employ this at the constructor level as well.
	constructor injection.
```java
@Service
public class OrderService {
    private final PaymentService paymentService;
    @Autowired
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

With lombok we also write required args constructor that simply created the constructor for us.

6.  **@Service:** This annotation is used to indicate that a class is a specialized type of Spring bean, typically used for business logic.
7. **@Repository:** This annotation is used to indicate that a class is a specialized type of Spring bean, typically used for database access.

Path variable and requesrt param

```java
@RestController
@RequestMapping("/api")
public class MyController {
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        // Retrieve user with the given ID
    }
}
```

```java
@RestController
@RequestMapping("/api")
public class MyController {
    @GetMapping("/users")
    public List<User> getUsers(@RequestParam("status") String status) {
        // Retrieve users with the given status
    }
}
```

@RequestBody

```java
@RestController
@RequestMapping("/api")
public class MyController {
    @PostMapping("/users")
    public void createUser(@RequestBody User user) {
        // Create a new user
    }
}
```


Refernce : [List of All Spring Boot Annotations, Uses with examples - Tutorial World](https://tutorialworld.in/spring-boot/spring-boot-annotations/)

