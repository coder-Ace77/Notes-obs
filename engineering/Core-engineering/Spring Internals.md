
---

In **Spring Boot**, the **application context** (or **Spring context**) is the central container that manages the lifecycle and configuration of all your application’s beans and components.

At its core, the **Spring context** is an implementation of the **ApplicationContext** interface from the Spring Framework. It:

- Acts as a **dependency injection container**
- Manages **beans** (objects managed by Spring)
- Handles **configuration**, **scanning**, and **lifecycle events**
- Resolves **@Autowired**, **@Value**, and other annotations
- Enables **component scanning**, **profile management**, etc.

The Spring context is like a **big registry** or **container** that knows about all your services, components, repositories, and configurations — and how they connect to each other.

When Spring Boot starts, it:

1. Creates the **ApplicationContext**
2. Scans and detects all components (`@Component`, `@Service`, `@Repository`, etc.)
3. Registers them as **beans**
4. Injects dependencies wherever needed
5. Starts the embedded server (if it's a web app)

