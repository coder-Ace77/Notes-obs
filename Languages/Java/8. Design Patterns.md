
---

### What is LLD?

**LLD (Low-Level Design)** refers to the detailed design phase in software development where the internal structure of individual modules, classes, methods, and interactions between components are defined based on the system’s high-level architecture.

While **High-Level Design (HLD)** outlines the overall system architecture and major components, LLD breaks these down into precise specifications, including class diagrams, relationships, design patterns (like Singleton, Factory, etc.), database schemas, and interface contracts.

![[Pasted image 20250624130846.png]]

- Offers the specifications required for the developer to start coding. 
- Developing scalable, low-maintenance, and robust software is much needed for every engineer. 
- It is a good practice to design modules on paper before jumping into code and it helps to have an idea of where our development process is going.


> [!NOTE] LLD Topics
> - Encapsulation -> Reduces complexity + Data Security
> - Inheritance -> - Eliminate Redundant Code +  Reusability
> - Abstraction  -> Hide Complexity + Isolate Impact of changes
> - Polymorphism -> -  An object can take many forms
> - Association, Aggregation, Composition
> - Good understanding of Object Oriented Design principles such as SOLID principles
> - Good command over Design patterns is a bonus point for entry level developers and kind of expected for senior level developers
> 

#### How to represent LLD? - Unified Modelling Language (UML)

A **UML Class Diagram** (Unified Modeling Language) is a **visual representation** of the **classes**, **interfaces**, and their **relationships** in a system.

```pgsql
+-----------------------+
|      ClassName        |
+-----------------------+
| - attribute1: Type    |
| - attribute2: Type    |
+-----------------------+
| + method1(): Return   |
| + method2(param: T):V |
+-----------------------+
```

Visisbility mode notations:

- `+` → public
- `-` → private
- `#` → protected
- `/` → derived
- `~` → package-private (default)

### Relationships

![[Pasted image 20250624131635.png]]

In a UML class diagram, **relationships** define how different classes or objects are connected to each other and how they interact.

1. **"Is-a"** represents an **inheritance** relationship, where a subclass is a specialized form of a superclass; for example, a `Car` **is a** `Vehicle`
2. **"Has-a"** indicates an **aggregation** or **composition** relationship, where one class contains or is composed of another class; for example, a `Car` **has a** `Engine`, meaning the engine is a part of the car either with loose coupling (aggregation) or tight coupling (composition).
3. **"Uses-a"** describes a **dependency** relationship, where a class temporarily uses another class to perform a function, often as a method parameter or local variable—for instance, a `CarService` class **uses a** `Car` object to perform repairs or maintenance.

#### Association:

We call Association those relationships whose objects have an independent lifecycle and where there is no ownership between the objects.

The most basic relationship is **association**, which represents a logical connection between two classes—for instance, a `Student` and a `Course` may be associated because students enroll in courses. Associations can be **unidirectional** (one class knows about the other) or **bidirectional** (both classes are aware of each other), and often include **multiplicity** indicators (like `1`, `0..1`, `*`, or `1..*`) to express how many instances participate in the relationship.

```java
public class SwipeCard{
	public void Swipe(MAnagaer obj){
		// goes and swipe
	}
}

public class Manager{
	public void logo(SwipeCard obj){
		// do something
	}
}
```

As can be seen there is no ownership between Swipe and managr but there is a logical relationship.

#### Aggregation:

A special type of association is **aggregation**, which denotes a "has-a" relationship with weak ownership—such as a `Team` having multiple `Players`, but the players can exist independently of the team.
There are two kinds of classes here 
1. Whole(Container class) Team 
2. Part (Contained class) Player

Here if parent is deleted childs are not deleted. They exist indepedently.

#### Composition:

We use the term Composition to refer to relationships whose objects don’t have an independent lifecycle, and if the parent object is deleted, all child objects will also be deleted.
A stronger form is **composition**, where the child cannot exist without the parent, indicating a whole-part relationship; for example, a `House` is composed of `Rooms`, and if the house is destroyed, its rooms cease to exist.

1. Whole(container) class -> Hotel
2. Part(Contained) class -> Contact

#### Diagram notations:

|Type|Symbol|Description|
|---|---|---|
|Association|`→` or line|One class references another|
|Aggregation|◇ (hollow diamond)|"Has-a" relationship, weak ownership|
|Composition|◆ (solid diamond)|Strong ownership (child cannot exist without parent)|
|Inheritance|⬆ (solid arrow, triangle)|`extends` or `implements`|
|Dependency|---→ (dashed arrow)|One class depends temporarily on another|
|Multiplicity|1, _, 0..1, 1.._|Indicates how many objects are involved

#### Cardinality in class relationships:

![[Pasted image 20250624133246.png]]

Example:

![[Pasted image 20250624133415.png]]


> [!NOTE] Explaination
> The given UML class diagram models a **Sales Order System**. At the core is the `Order` class. Each `Order` is associated with exactly one `Customer` but a customer can place multiple orders (1 to 0..* association). An `Order` is also composed of multiple `OrderDetail` entries (1 to 1..* composition), each representing a line item in the order and tied to a single `Item` (0..* to 1). Orders are paid for through one or more `Payment` instances (1 to 1..* association), with `Payment` being an abstract class extended by concrete types: `Cash`, `Check`, and `Credit`, each adding their specific attributes and an `authorized()` method where applicable. We use *inheritance (is-a)** for payment types, **composition (has-a)** between `Order` and `OrderDetail`, and **association (uses-a)** between other classes.


## Solid Principles:

---

- Principles of Good Object Oriented Design proposed by Uncle Bob Martin

	- Single Responsibility Principle - The Single Responsibility Principle states that a class should have only one reason to change, meaning it should only have **one job or responsibility**. This keeps classes focused and easy to maintain. For example, if a class handles both user data storage and report generation, a change in reporting logic would affect user data logic too, increasing the risk of bugs. By separating responsibilities into different classes, each class becomes simpler, easier to test, and less error-prone when changes are required.
	
	- Open Closed Principle - The Open/Closed Principle says that software entities (classes, modules, functions) should be **open for extension but closed for modification**. This means you should be able to add new behavior without changing existing code. This is typically achieved through inheritance or interfaces. For example, if you have a payment system, instead of modifying existing code to add a new payment type like UPI, you should create a new class that extends or implements the existing interface. This protects stable code and reduces the risk of breaking existing functionality.

	- Liskov Substitution Principle - The Liskov Substitution Principle ensures that **subclasses can be used in place of their parent class without breaking the system**. In other words, if `S` is a subtype of `T`, then objects of type `T` should be replaceable with objects of type `S` without affecting program correctness. For example, if `Bird` is a superclass and `Penguin` is a subclass, you shouldn’t override flying behavior in a way that breaks expectations (since penguins can't fly). Violating LSP leads to runtime issues and undermines polymorphism.
	
	- Interface Segregation Principle - The Interface Segregation Principle states that **clients should not be forced to depend on interfaces they do not use**. Instead of having one fat interface with many unrelated methods, split them into smaller, more specific interfaces. For example, if you have a `Worker` interface with methods like `work()`, `eat()`, and `sleep()`, a robot that only needs `work()` shouldn’t be forced to implement `eat()` and `sleep()`. This keeps systems flexible and prevents bloated class implementations.

	- Dependency Inversion Principle - 
	
		High-level modules should not depend upon low-level modules; they should depend on abstractions.
		
		The Dependency Inversion Principle encourages **relying on abstractions rather than concrete implementations**. High-level modules (like business logic) should not depend on low-level modules (like file access); both should depend on interfaces. For example, a `ReportGenerator` should depend on an `IStorage` interface, not a `FileStorage` class directly. This makes the system more modular and allows you to swap out implementations (e.g., file storage, database, cloud) without changing the core logic.


> [!NOTE] Debrief DIP
> Your **business logic (high-level)** should not directly depend on **specific implementations (low-level)**.Abstractions should not depend on details. Details should depend on abstractions.


example:

Without DIP

```java
public class FileStorage {
    public void save(String data) {
        System.out.println("Saving data to file: " + data);
    }
}

public class ReportGenerator {
    private FileStorage fileStorage = new FileStorage(); // Direct dependency

    public void generate(String report) {
        // Business logic
        fileStorage.save(report);
    }
}

```

With DIP(loosely coupled):

```java
public interface Storage {
    void save(String data);
}
public class FileStorage implements Storage {
    public void save(String data) {
        System.out.println("Saving data to file: " + data);
    }
}

public class DatabaseStorage implements Storage {
    public void save(String data) {
        System.out.println("Saving data to database: " + data);
    }
}

public class ReportGenerator {
    private final Storage storage;

    // Constructor Injection (or use setter injection)
    public ReportGenerator(Storage storage) {
        this.storage = storage;
    }

    public void generate(String report) {
        System.out.println("Generating report: " + report);
        storage.save(report);
    }
}
```

Use now:

```java
public class Main {
    public static void main(String[] args) {
        Storage storage = new FileStorage(); // Or new DatabaseStorage()
        ReportGenerator generator = new ReportGenerator(storage);

        generator.generate("Sales Report");
    }
}
```

Earlier report generator class used dependency of fileStorage direclty so in future if it changes any way entire implementation had to be rewritten. 

But now with DIP it depends upon Storage which is an abstraction not an implementation. Any change like change in storage structure will still be good.

![[Pasted image 20250624141817.png]]

### Factory Design Pattern

The **Factory Design Pattern** in Java is a **creational design pattern** that provides an elegant way to create objects without specifying the exact class of the object that will be created. 

It achieves this by defining a **common interface or superclass** for all possible types of objects that need to be created, and then using a **factory class or method** to return instances of the appropriate class based on input parameters, conditions, or logic.

Step1 -- Product interface

```java
public interface Shape {
    void draw();
}
```

Step2 -- Concrete Products

```java
public class Circle implements Shape {
    public void draw() {
        System.out.println("Drawing a Circle");
    }
}

public class Square implements Shape {
    public void draw() {
        System.out.println("Drawing a Square");
    }
}
```

Step3 -- Factory Class

```java
public class ShapeFactory {
    public Shape getShape(String shapeType) {
        if (shapeType == null) return null;
        if (shapeType.equalsIgnoreCase("CIRCLE")) return new Circle();
        else if (shapeType.equalsIgnoreCase("SQUARE")) return new Square();
        return null;
    }
}
```

Client code:

```java
public class FactoryPatternDemo {
    public static void main(String[] args) {
        ShapeFactory factory = new ShapeFactory();

        Shape shape1 = factory.getShape("CIRCLE");
        shape1.draw();  // Output: Drawing a Circle

        Shape shape2 = factory.getShape("SQUARE");
        shape2.draw();  // Output: Drawing a Square
    }
}
```

The core idea is to **encapsulate object creation** so that if the creation process becomes complex, or if the exact class to instantiate is determined at runtime (e.g., based on user input, configuration, or context), the client remains clean, simple, and loosely coupled.


### Builder Pattern

The **Builder Pattern** in Java addresses the problem of constructing complex objects with many optional parameters or configurations in a readable, maintainable, and safe way.
When a class has multiple fields—some required, some optional—using telescoping constructors (multiple overloaded constructors) becomes messy and hard to manage, and using setters exposes the object to invalid intermediate states. The use of setters is called as Java Bean Pattern. However this makes the field mutable which is not a good idea.

The Builder Pattern solves this by separating the construction of an object from its representation, allowing step-by-step construction through a nested `Builder` class with fluent methods.

Telescoping constructor problem:

```java
public class User {
    private String name;
    private int age;
    private String email;
    private String address;

    // Telescoping constructors - hard to read and maintain
    public User(String name) {
        this(name, 0, null, null);
    }

    public User(String name, int age) {
        this(name, age, null, null);
    }

    public User(String name, int age, String email) {
        this(name, age, email, null);
    }

    public User(String name, int age, String email, String address) {
        this.name = name;
        this.age = age;
        this.email = email;
        this.address = address;
    }
}
```

Using builder pattern

```java
public class User {
    // Required
    private final String name;

    // Optional
    private final int age;
    private final String email;
    private final String address;

    // Private constructor — can only be built via Builder
    private User(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.email = builder.email;
        this.address = builder.address;
    }

    // Static nested Builder class
    public static class Builder {
        // Required
        private final String name;

        // Optional with defaults
        private int age = 0;
        private String email = "";
        private String address = "";

        // Constructor with required fields
        public Builder(String name) {
            this.name = name;
        }

        // Setter-style methods returning Builder (fluent API)
        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder email(String email) {
            this.email = email;
            return this;
        }

        public Builder address(String address) {
            this.address = address;
            return this;
        }

        // Final build() method
        public User build() {
            return new User(this);
        }
    }

    // To display user data
    @Override
    public String toString() {
        return String.format("User{name='%s', age=%d, email='%s', address='%s'}",
                name, age, email, address);
    }
}
```

The idea is that there is a static class inside outer class whose constructor will initially create all the required/ mandatory fields. Then optional fields will be set up by calling the setter methods of builder. Since these setter methods return object of builder class we can chain them. Finally build() method returns the object of original class by calling the constructor. Observe the constructor of outer class it takes object of innrer builder class. Secodly it is private.

Usage example

```java
public class Main {
    public static void main(String[] args) {
        User user1 = new User.Builder("Adil")
                .age(23)
                .email("adil@example.com")
                .address("IIT Bhilai")
                .build();

        User user2 = new User.Builder("Rahul").build();

        System.out.println(user1);
        System.out.println(user2);
    }
}
```

![[Pasted image 20250624144216.png]]

#### Lombok

- Lombok is a library for Java that simplifies the creation of getter and setter methods, constructors, and other common boilerplate code. 
- It also provides annotations like @Builder to easily generate builder patterns

Example:
```java
import lombok.Builder;
import lombok.Getter;
@Getter
@Builder
public class EnrollmentForm {
    private final String firstName;
    private final String lastName;
    private final int age;
    private final String gender;
    private final boolean isGraduate;
    private final boolean hasExperience;
    private final String city;
    private final String state;
    private final boolean isEarning;
}

// usage
public static void main(String[] args) {
        EnrollmentForm form = EnrollmentForm.builder()
                .firstName("John")
                .lastName("Doe")
                .age(25)
                .gender("Male")
                .isGraduate(true)
                .hasExperience(true)
                .city("New York")
                .state("NY")
                .isEarning(true)
                .build();
        System.out.println("form");
}
```

### Singleton Pattern

The **Singleton Pattern** is a **creational design pattern** in Java that ensures a **class has only one instance** throughout the application and provides a **global point of access** to that instance. It is commonly used when exactly one object is needed to coordinate actions across the system—such as in logging, configuration management, caching, or thread pools.

#### How to achieve

The main idea is to **restrict object creation** by making the constructor private, so that no other class can instantiate it directly. Instead, a static method (commonly `getInstance()`) is provided to return the **single shared instance**.

```java
public class Logger{
    private static Logger instance;
    private Logger() {}
    public static Logger getInstance() {
        if (instance == null) {
            instance = new Logger();
        }
        return instance;
    }
}
```

```java
// usage
public class Main{
	public static void main(){
		Logger logger = Logger.getInstance();
	}
}
```

![[Pasted image 20250624145624.png]]
