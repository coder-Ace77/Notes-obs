
---

Object-Oriented Programming (OOP) is a programming paradigm based on the concept of **"objects,"** which can contain data (attributes) and code (methods). 

Class is the blueprint of objects and objects are instances of classes. Classes are like the real world properties of object. It is not real world object. Object is the real instance which takes memory. 

### Advantages of oops

- **Modularity:** Code is divided into independent objects, making it easier to manage, debug, and maintain.
    
- **Reusability:** Through inheritance, you can use existing code in new programs without rewriting it from scratch.
    
- **Scalability:** OOP is excellent for large, complex projects because it allows teams to work on different modules (classes) simultaneously with minimal interference.
    
- **Security:** Encapsulation protects data from accidental modification by outside code, reducing bugs and vulnerabilities.
    
- **Easy Maintenance:** If a specific part of the program needs an update, you often only need to modify one class rather than the entire codebase.


Limitations

- **Performance Overhead:** OOP programs tend to be larger and slower than procedural ones because they require more CPU overhead and memory to manage objects.
    
- **Over-Engineering:** It's easy to create unnecessarily complex class hierarchies for simple tasks, leading to "spaghetti code" that is hard to follow.
    
- **Memory Management:** Because OOP creates many objects, it can lead to high memory consumption if not managed properly (though modern garbage collectors help with this).


A language is OOP if it lets you define a **Class** (the template) and create **Objects** (the actual instances). Some languages, like **Smalltalk** or **Ruby**, are considered "Pure" OOP because literally everything even a simple number or a string is treated as an object. Other languages like **C++** or **Java** are "Hybrid" because they have some basic data types (like integers) that aren't objects for the sake of speed.

## Pillars of Oops

There are four pillars of OOPS

### Encapsulation

The is the practicing of bundling of data and methods that act on data into a single unit called class.  The whole idea behind encapsulation is to hide the implementation details from users.

The word comes from the Latin _capsula_, meaning "small box" or "chest." In a non-technical sense, it refers to the process of enclosing something in a capsule or a protective coating. 

In reality we have two mechanisms to achieve encapsulation-

1. Data hiding (Access modifiers) You make the data inside a class **private**, meaning it cannot be accessed or changed by code outside that class.
2. Public methods - (Setters and setter) - Since the data is private, you provide "controlled entry points" called **Getters** (to read the data) and **Setters** (to change the data). 

This means all the data and methods are bind and closed away in a single unit of class. Encapsulation is about hiding the data and grouping it with logic (Information Hiding).

### Abstraction

**Abstraction** is the process of hiding the complex, internal implementation details of a system and exposing only the essential features to the user. Note that although both encapsulation and abstraction are about hiding details.**Abstraction** is about **hiding the complexity** of code and showing only the relevant details of objects which makes it (implementation hiding). Meaning for example user does not needs to know what algorithm internally object uses to hash.

In programming, abstraction is primarily achieved through two mechanisms:

- Abstract classes - An abstract class is a "half-finished" blueprint. It defines methods that a subclass _must_ have but doesn't provide the logic for them. Example you have an abstract class `Shape`. It has a method `calculateArea()`. You can't have a generic "Shape" object (what would the area be?), but you can have a `Circle` or a `Square` that provides the specific formula.
- An interface is a strict "contract." it tells a class, "I don't care who you are, but if you want to be a `Payable` entity, you _must_ implement a `processPayment()` method." It provides a way to achieve total abstraction because it contains no data or implementation logic at all.

Observe how both mechanisms provide the way to hide the implement details once you know about the interface that will hide implementation details. 

Usefullness - 

- Reduce complexity
- Enhance maintainabilty
- Avoide code duplication
- Increased security

|**Feature**|**Abstraction**|**Encapsulation**|
|---|---|---|
|**Focus**|**Hiding Complexity.** Focuses on the "outside" view.|**Hiding Data.** Focuses on the "inside" view.|
|**Purpose**|To simplify the interface for the user.|To protect the data from outside interference.|
|**Implementation**|Achieved using Abstract Classes and Interfaces.|Achieved using Access Modifiers (`private`, `public`).|
|**Analogy**|The steering wheel and pedals of a car.|The engine hood that locks the components inside.|

### Inheritance

**Inheritance** is the mechanism in OOP that allows one class to acquire the properties (fields) and behaviors (methods) of another. It creates a **parent-child relationship** between classes.

The main goal of inheritance is **code reusability**: instead of writing the same code over and over for similar objects, you write it once in a "Parent" class and let "Child" classes inherit it.

**Superclass (Parent/Base Class):** The class whose features are inherited. It represents a more general concept. 
**Subclass (Child/Derived Class):** The class that inherits the features. It represents a more specific version of the parent and can add its own unique features.

**The "IS-A" Relationship:** Inheritance is often described as an "is-a" relationship. For example:

- A `Car` **is-a** `Vehicle`.
- A `Manager` **is-a** `Employee`.
- A `SavingsAccount` **is-a** `BankAccount`.

Inheritance allows the reusability of code. 

Types of inheritance - 

- Single inheritance - A subclass inherits from only one superclass. (Supported by Java, Python, C++).
- **Multilevel Inheritance:** A child class acts as a parent for another class (Grandparent -> Parent -> Child).
- **Hierarchical Inheritance:** Multiple subclasses inherit from a single superclass (e.g., `Dog` and `Cat` both inherit from `Animal`).
- **Multiple Inheritance:** A class inherits from more than one parent class.Note:
    **C++** supports this directly, but **Java** does not (to avoid complexity like the "Diamond Problem"), instead using Interfaces to achieve similar results.

Limitations of inheritance - 

- **Tight Coupling:** The child class is highly dependent on the parent. Changes in the parent can break the child (the "Fragile Base Class" problem).
- **Reduced Flexibility:** You cannot change the parent of a class at runtime.
- **Memory Overhead:** An object of a subclass carries all the data members of the superclass, even if it doesn't use them.

A **Sealed** class (seen in Java 17+ and C#) is used to restrict inheritance. It allows a developer to specify exactly which other classes are allowed to inherit from it. This is great for security and creating "closed" hierarchies where you know every possible subclass.

You can call a base class method without an instance if the method is **Static**. In Java or C++, you simply use the class name: `BaseClassName.methodName()`. Within a child class, you can also use keywords like `super.methodName()` (Java) or `base.methodName()` (C#) inside a static context if applicable.

**Override:** Extends the base class implementation. The runtime ensures the most specific version of the method is called, even if the object is referenced by its parent type.

Diamond problem - 

This occurs when Class D inherits from Class B and Class C, both of which inherit from Class A. If Class A has a method that B and C both override, and D calls that method, the compiler won't know whether to use B’s version or C’s version.

Java avoids multiple inheritance of _classes_ primarily to prevent the **Diamond Problem**. It keeps the language simpler and avoids the ambiguity of which parent's method to use if two parents share the same method signature. Java allows multiple inheritance of _type_ through **Interfaces** instead.

 If Class A inherits from Class B, what all is inherited?
The child class inherits:
- All Public and Protected members (fields and methods).
- Default (package-private) members, but only if the child is in the same package.
- Note: **Constructors** and **Private members** are _not_ inherited, though private members still exist in the object's memory.

Object slicing - This happens in C++ when you assign a derived class object to a base class object. Because the base class object is smaller, the "extra" parts of the derived class (its unique variables and methods) are literally "sliced off" and lost.

Friend Function / Friend Class / Inline Function

- **Friend Function/Class:** (C++ only) A function or class that is granted access to the `private` and `protected` members of another class.
- **Inline Function:** A hint to the compiler to replace the function call with the actual code of the function to save the overhead of a function call.

**Local Class:** A class defined inside a method.
**Nested Class:** A class defined inside another class (Static vs. Inner).
**Simulating Final Class:** In older C++, you could simulate a "Final" class (one that can't be inherited) by making the constructor private and using a friend class.

**Overloading** happens within the _same_ class (same name, different parameters). When you change a parent's method in a child class, it is **Overriding**. However, a child class can have a method with the same name but different parameters than the parent; in many languages, this is simply seen as the child class "hiding" the parent's versions.

Generalisation is the another name of inhertitance. eg  - A Car is a generalisation of a Sedan.

### Polymorphism

Polymorphism means a same fature canhae different forms. In practice that means a single piece of code based on the situation can act differently. There are two main forms of polymorphism

- Compile-Time Polymorphism (Static Binding) - This occurs when the compiler determines which method to call at the time of compilation. The primary way to achieve this is through Method Overloading. Multiple methods in the same class have the same name but different parameters (different type, number, or order)

- Run time polymorphism - This occurs when the specific method to be executed is determined at "run-time" based on the actual object being referenced. This is achieved through Method Overriding. A subclass provides a specific implementation of a method that is already defined in its parent class.
- **Virtual Methods:** In languages like C++, you must explicitly mark a method as `virtual` to allow it to be overridden for polymorphism. In Java, methods are virtual by default

#### Advantages of Polymorphism

- **Extensibility:** You can add new classes (like a new programming language to **CodeArena**) without changing the existing code that handles submissions.
- **Code Cleanliness:** It reduces the need for long `if-else` or `switch` blocks to check an object's type.
- **Interchangeability:** It allows you to write code that works with a general "type," making your system much more flexible and modular.

## Binding

Binding refers to the process by which the compiler or the runtime environment connects a function call to the actual code that should be executed.

In Early Binding, the connection between the method call and the method body happens at **compile-time**.

- **Mechanism:** The compiler knows exactly which method to run based on the reference type.
    
- **Performance:** It is faster because there is no overhead of looking up the method during execution.
    
- **Used In:** Private, static, and final methods in Java, as well as overloaded methods.
    
- **Example:** If you call a static helper method in your **Fastdev** project, the compiler binds that call immediately.

In Late Binding, the decision of which method to call is delayed until **run-time**.

- **Mechanism:** The program looks at the actual object in memory (not just the reference type) to decide which overridden method to execute.
    
- **Performance:** Slightly slower due to the "virtual table" lookup.
    
- **Used In:** Method overriding and virtual functions.
    
- **Key Benefit:** This is the core mechanism that makes **polymorphism** possible. For your **CodeArena** engine, late binding allows you to call `execute()` on a general `Submission` object and have it automatically run the specific logic for a `PythonSubmission` or `JavaSubmission`.

### Struct vs classes

While both are used to group variables and functions together, their usage and rules vary significantly between languages.

In C++, the only technical difference between a `struct` and a `class` is the **default access level**:

- **Struct:** Members are **public** by default. They are typically used for "Plain Old Data" (POD) structures where you just want to group data together without much logic.
- **Class:** Members are **private** by default. They are used when you want to implement full Object-Oriented principles like **Encapsulation** and **Abstraction**.


In C# and other languages

- **Value vs. Reference:** In C#, a `struct` is a **value type** (stored on the stack), while a `class` is a **reference type** (stored on the heap).
    
- **Performance:** Structs can be faster for very small, short-lived data objects because stack allocation is cheaper than heap allocation and garbage collection.

### Cohesion and coupling

Cohesion refers to how closely related and focused the responsibilities of a single module or class are. It measures the "internal strength" of a module. 

High Cohesion: A class does one thing and does it well. All its methods and properties are directed toward a single, well-defined purpose.
 
**Low Cohesion:** A class has many unrelated responsibilities. It is often called a "God Object" because it tries to do everything. Low cohesion makes code difficult to maintain, test, and understand because a change in one feature might accidentally break an unrelated feature in the same class.

Coupling refers to the degree of direct knowledge or dependency one module has on another. It measures the "interdependence" between modules.

- **Low (Loose) Coupling:** Modules are independent. They interact through simple, well-defined interfaces and don't need to know how other modules work internally.
- **High (Tight) Coupling:** Modules are deeply intertwined. Changing one module forces you to change all the modules it is connected to.