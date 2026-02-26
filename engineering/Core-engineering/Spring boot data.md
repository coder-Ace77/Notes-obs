
---

### JPA:

**JPA** stands for **Java Persistence API**. It is a specification in Java that provides a standardized way to manage relational data in Java applications. JPA acts as a bridge between object-oriented programming and relational databases by handling the mapping of Java objects to database tables and vice versa. This process is known as **Object-Relational Mapping (ORM)**.

JPA defines how Java classes (entities) are mapped to database tables and how they interact with the database using components like the **EntityManager**, **Persistence Context**, and **transactions**. It simplifies database operations by abstracting the complexities of SQL and providing a rich API for managing data. It is actually **specification** (part of Java EE, now Jakarta EE).

Spring Data JPA is abstraction on top of JPA. The annotation such as `@Entity`, `@Id`, `@OneToMany` are part of JPA itself. Note that JPA is an specification we need an implementation of it which for example is Hibernate.

#### Hibernate:

- The most popular **JPA implementation**.
- Handles object-relational mapping (ORM), SQL generation, and caching.
- Adds many advanced features beyond the JPA spec (criteria queries, lazy loading, second-level cache, etc.).
- With Spring Boot, if you include `spring-boot-starter-data-jpa`, Hibernate is pulled in automatically.

####  Spring Data JDBC

- A simpler alternative to JPA/Hibernate.
- Provides repository support like Spring Data JPA but **without heavy ORM**.
- Works directly with SQL and relational data mapping (not full-blown ORM).
- Good for simpler applications where JPA/Hibernate overhead is unnecessary.

Note :: **JDBC (Java Database Connectivity)** is an API in Java that facilitates communication between Java applications and relational databases such as MySQL, PostgreSQL, and Oracle.

JDBC consists of several essential components that work together to establish and manage database connections:

- **DriverManager**: Manages database drivers and establishes connections to the database.
- **Connection**: Represents an active connection to the database.
- **JDBC Drivers**: Vendor-specific implementations that translate JDBC calls into database-specific protocols.

How JDBC Works

JDBC operates through a layered architecture, typically using one of two models:

- **Two-Tier Architecture**: The Java application communicates directly with the database using a JDBC driver.
- **Three-Tier Architecture**: The application interacts with a middle-tier service, which processes queries and communicates with the database.
- 
JDBC drivers play a critical role in this process. There are four types of drivers:

1. Type-1 (JDBC-ODBC Bridge, deprecated).
2. Type-2 (Native API driver).
3. Type-3 (Network Protocol driver).
4. Type-4 (Thin driver, widely used for its platform independence).


---

## Working with Spring-boot:

To configure your custom datasource we can define it inside application.properties

```java
app.datasource.url=jdbc:h2:mem:mydb
app.datasource.username=sa
app.datasource.pool-size=30
```

### Using spring data repository:

Spring Data can create implementations of Repository interfaces of various flavors. Spring Boot handles all of that for you, as long as those `Repository` implementations are included in one of the auto-configuration packages, typically the package (or a sub-package) of your main application class that is annotated with @SpringBootApplication or @EnableAutoConfiguration.

For many applications, all you need is to put the right Spring Data dependencies on your classpath. There is a `spring-boot-starter-data-jpa` for JPA, `spring-boot-starter-data-mongodb` for Mongodb.

### Entity:

In Hibernate and JPA (Java Persistence API), an entity is a **plain Java object (POJO)** that represents a table in your relational database. Each instance of the entity corresponds to a row in the table, and each field in the class corresponds to a column in the table. By annotating the class with JPA annotations, you tell Hibernate how to map the object to the database table.

```java
import jakarta.persistence.*;   // JPA annotations
import lombok.*;               // Optional, used to reduce boilerplate code

@Entity
@Table(name = "users") // Explicitly specify table name
@Data                 // Lombok annotation to generate getters, setters, equals, hashCode, and toString
@NoArgsConstructor    // Lombok generates a no-args constructor
@AllArgsConstructor   // Lombok generates a constructor with all fields
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // Auto-increment primary key
    private Long id;

    @Column(nullable = false) // Column must not be null
    private String name;

    @Column(unique = true, nullable = false) // Must be unique and not null
    private String email;
}
```

The `@Entity` annotation is mandatory for any class that you want Hibernate to manage as a persistent entity. By placing this annotation, you are telling Hibernate: _“This class should be mapped to a table in the database.”_

Without this annotation, Hibernate will simply treat your class as a normal Java class and will not create or persist any corresponding table or row for it.

The `@Table` annotation allows you to specify the table name that this entity should map to. If you omit it, Hibernate will use the class name by default.

Every entity class must have a **primary key**. The `@Id` annotation marks the field that will act as the primary key in the database table. Without it, Hibernate cannot uniquely identify rows in the table, and it will throw an error.

The `@GeneratedValue` annotation tells Hibernate how to generate the primary key. Common strategies are:

- `GenerationType.IDENTITY`: The database auto-increments the ID.
- `GenerationType.AUTO`: Hibernate chooses a strategy depending on the database.
- `GenerationType.SEQUENCE`: Uses a database sequence (commonly used in Oracle, PostgreSQL).
- `GenerationType.TABLE`: Uses a separate table to generate IDs.

The `@Column` annotation gives you fine-grained control over how a field maps to a column in the database. You can specify attributes such as:

- `nullable`: Whether the column allows `NULL` values.
- `unique`: Whether the column must have unique values.
- `length`: Maximum length of a string column.
- `name`: Custom column name if you do not want to use the field name.

## JPA repository:

As cleared now JPA is just the specification hibernate is the implementation. We usually extend JPA repository and hibernate uses it to get actual implementation.

In Spring Boot, a **JPA Repository** is an interface that allows you to perform database operations on an entity without writing boilerplate code such as `INSERT`, `UPDATE`, `DELETE`, or even `SELECT` queries.

The JPA Repository provides a **higher-level abstraction** for the **Data Access Layer**. Instead of writing manual SQL or even Hibernate-specific code, you simply define a repository interface that extends `JpaRepository`, and Spring Boot automatically generates an implementation for you at runtime.

To understand JPA Repository, it is important to know its inheritance chain:

1. **`Repository`** – The most basic interface, which is a marker interface (it does not contain methods but marks the class as a repository).
2. **`CrudRepository<T, ID>`** – Extends `Repository` and provides basic CRUD operations such as `save()`, `findById()`, `findAll()`, and `deleteById()`.
3. **`PagingAndSortingRepository<T, ID>`** – Extends `CrudRepository` and adds methods for pagination and sorting of records.
4. **`JpaRepository<T, ID>`** – Extends `PagingAndSortingRepository` and adds JPA-specific methods such as batch operations, flushing, and custom queries.

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // Custom query methods can be defined here
    User findByEmail(String email);
}
```

## **Built-in Methods of JpaRepository**

By extending `JpaRepository`, you get access to a wide variety of methods without writing any implementation yourself. Some of the most important ones are:

- **Save and Update**:
    - `save(S entity)` → Saves an entity or updates it if it already exists.
    - `saveAll(Iterable<S> entities)` → Saves multiple entities at once.
- **Read (Find)**:
    
    - `findById(ID id)` → Finds an entity by its primary key.
    - `findAll()` → Returns all entities.
    - `findAllById(Iterable<ID> ids)` → Finds multiple entities by their IDs.
    - `count()` → Returns the total number of entities.
    
- **Delete**:
    
    - `deleteById(ID id)` → Deletes an entity by its ID.
    - `delete(T entity)` → Deletes a given entity.
    - `deleteAll()` → Deletes all records in the table.
        
- **Pagination and Sorting**:
    
    - `findAll(Pageable pageable)` → Returns a paginated list of entities.
    - `findAll(Sort sort)` → Returns all entities sorted according to given criteria.
        
- **JPA-Specific Features**:
    
    - `flush()` → Synchronizes persistence context with the database.
    - `saveAndFlush(T entity)` → Saves an entity and immediately flushes changes.
    - `deleteInBatch(Iterable<T> entities)` → Deletes a batch of entities efficiently.

One of the most powerful features of JPA Repository is **query derivation from method names**. You can define methods in the repository interface following a specific naming convention, and Spring Data JPA will generate the query automatically.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    User findByEmail(String email);  // SELECT * FROM users WHERE email = ?
    List<User> findByName(String name); // SELECT * FROM users WHERE name = ?
    List<User> findByNameAndEmail(String name, String email); // AND condition
    List<User> findByNameOrEmail(String name, String email);  // OR condition
    List<User> findByNameContaining(String keyword); // LIKE query
    List<User> findByNameStartingWith(String prefix); // LIKE 'prefix%'
    List<User> findByNameEndingWith(String suffix);   // LIKE '%suffix'
}
```

Writing the custom queries:

```java
@Query("SELECT u FROM User u WHERE u.email = ?1")
User getUserByEmail(String email);
```

Now there are two options either use JPQL or use native SQL 

```java
@Query("SELECT u FROM User u WHERE u.email = ?1")
User getUserByEmail(String email);
```

Here, `User` refers to the entity class, not the table name. JPQL works with entities, not raw tables.

```java
@Query(value = "SELECT * FROM users WHERE email = ?1", nativeQuery = true)
User getUserByEmailNative(String email);
```

Example with pagination:

```java
Page<User> findAll(Pageable pageable);

// in service layer

Page<User> usersPage = userRepository.findAll(PageRequest.of(0, 5, Sort.by("name").ascending()));
```