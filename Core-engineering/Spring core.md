
---

Inversion of Control is a **design principle** where the control of creating and managing objects is shifted from the **application code** to an external framework or container.

Normally, in traditional programming:

- Your code **controls** when and how objects are created.
- You write `new` everywhere, wiring up dependencies yourself.

With IoC:
- The **framework/container** takes over that control.
- Your code just declares what it needs, and the framework provides it.

```java
class Service {
    private Repository repo;

    public Service() {
        this.repo = new Repository(); // You are creating the dependency
    }

    public void doWork() {
        repo.save();
    }
}
```

Now Spring’s IoC **container** decides _which_ implementation of `Repository` to provide, without you writing `new Repository()`

```java
@Service
public class Service {
    private final Repository repo;

    @Autowired
    public Service(Repository repo) {
        this.repo = repo;
    }
}
```

Dependency injection (DI) is a specialized form of IoC, whereby objects define their dependencies (that is, the other objects they work with) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method.

The `org.springframework.beans` and `org.springframework.context` packages are the basis for Spring Framework’s IoC container. The [`BeanFactory`](https://docs.spring.io/spring-framework/docs/6.2.11/javadoc-api/org/springframework/beans/factory/BeanFactory.html) interface provides an advanced configuration mechanism capable of managing any type of object.

ApplicationContext is the sub-interface of BeanFactory.In short, the `BeanFactory` provides the configuration framework and basic functionality, and the `ApplicationContext` adds more enterprise-specific functionality.

A bean is an object that is instantiated, assembled, and managed by a Spring IoC container.

### Container overview:

The `org.springframework.context.ApplicationContext` interface represents the Spring IoC container and is responsible for instantiating, configuring, and assembling the beans.
The container gets its instructions on the components to instantiate, configure, and assemble by reading configuration metadata. 
The configuration metadata can be represented as annotated component classes, configuration classes with factory methods, or external XML files or Groovy scripts.

In a Spring Boot scenario, the application context is implicitly bootstrapped for you based on common setup conventions.

A **POJO (Plain Old Java Object)** by itself is just a class.  
For Spring to manage it (create objects, inject dependencies, configure lifecycle), you need to provide **metadata**.

### Configuration metadata:

As the preceding diagram shows, the Spring IoC container consumes a form of configuration metadata. This configuration metadata represents how you, as an application developer, tell the Spring container to instantiate, configure, and assemble the components in your application.

The Spring IoC container itself is totally decoupled from the format in which this configuration metadata is actually written.

XML-based configuration metadata configures these beans as `<bean/>` elements inside a top-level `<beans/>` element. The following example shows the basic structure of XML-based configuration metadata:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="..." class="...">  
		<!-- collaborators and configuration for this bean go here -->
	</bean>
	<bean id="..." class="...">
		<!-- collaborators and configuration for this bean go here -->
	</bean>
	<!-- more bean definitions go here -->
</beans>
```

The `id` attribute is a string that identifies the individual bean definition.

The `class` attribute defines the type of the bean and uses the fully qualified class name.

### Annotated class component:

Instead of XML, you annotate Java classes directly with stereotypes `@Component`.These classes are detected automatically by `@ComponentScan`. Spring Boot (through `@SpringBootApplication`, which includes `@ComponentScan`) automatically scans the package and sub-packages.

```java
@Service
public class UserService {
    public String getUser() {
        return "Spring User";
    }
}
```

Configuration: 

Instead of relying on scanning, you explicitly tell Spring:  
“Create this bean by calling this factory method.” These beans don’t require `@ComponentScan` they are **always registered** because they’re explicitly declared.

Essentially we are using factory methods to explicitly define the beans.

```java
@Configuration
class AppConfig{
    @Bean
    public UserService userService() {
        return new UserService(); // explicitly created
    }
    
    @Bean
    public UserController userController() {
        return new UserController(userService()); // manual wiring
    }
}
```

### Bean overview:

A Spring IoC container manages one or more beans. These beans are created with the configuration metadata that you supply to the container (for example, in the form of XML `<bean/>` definitions).

Within the container itself, these bean definitions are represented as `BeanDefinition` objects, which contain (among other information) the following metadata:

- A package-qualified class name: typically, the actual implementation class of the bean being defined.
    
- Bean behavioral configuration elements, which state how the bean should behave in the container (scope, lifecycle callbacks, and so forth).
    
- References to other beans that are needed for the bean to do its work. These references are also called collaborators or dependencies.
    
- Other configuration settings to set in the newly created object — for example, the size limit of the pool or the number of connections to use in a bean that manages a connection pool.

### Dependency injection:

Dependency injection (DI) is a process whereby objects define their dependencies (that is, the other objects with which they work) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The container then injects those dependencies when it creates the bean. This process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies on its own by using direct construction of classes or the Service Locator pattern.

### Constructor based dependency injection:


