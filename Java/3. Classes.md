
---

Classes are fundamental part of java. Mostof the things in java are objects only.
A class is a blueprint for objects — it defines **fields (variables)** and **methods (functions)**.

- A object is an instance of a class. It has 3 characteristics:
	- - State - Represents data related to an object in memory.
	- - Behaviour - Represents the functionality of an object.
	- - Identity - Assigned by JVM to identify each object uniquely.

- How to create an instance of a class ? - new() operator.

```java
class Car {
    // Fields (variables)
    String brand;
    int speed;

    // Constructor
    Car(String brand, int speed) {
        this.brand = brand;
        this.speed = speed;
    }

    // Method
    void drive() {
        System.out.println(brand + " is driving at " + speed + " km/h");
    }
}

// creating object
// <container> <varible name> = new class();
Car myCar = new Car("Tesla", 100);
myCar.drive();
```

| Type                    | Description                                      |
| ----------------------- | ------------------------------------------------ |
| **Regular class**       | Basic user-defined type                          |
| **Abstract class**      | Can't be instantiated; can have abstract methods |
| **Final class**         | Cannot be subclassed                             |
| **Static nested class** | Class inside another class (with `static`)       |
| **Inner class**         | Non-static class inside another class            |
An interface defines a **contract** (methods without implementation). A class **implements** an interface.

```java
interface Animal {
    void makeSound();  // abstract method
}

class Dog implements Animal {
    public void makeSound() {
        System.out.println("Woof!");
    }
}
```

| Feature              | Interface                            | Abstract Class                 |
| -------------------- | ------------------------------------ | ------------------------------ |
| Multiple Inheritance | ✅ Yes                                | ❌ No                           |
| Method Bodies        | ✅ Default/static methods (Java 8+)   | ✅ Can have implemented methods |
| Fields               | Only public static final (constants) | Instance variables allowed     |
| Constructors         | ❌ No constructors                    | ✅ Can have constructors        |
```java
interface Flyable {
    void fly();
}

abstract class Bird {
    void eat() {
        System.out.println("Bird is eating...");
    }
}

class Eagle extends Bird implements Flyable {
    public void fly() {
        System.out.println("Eagle soars high.");
    }
}
```


### Access Modifier:

- Access Modifiers define visibility within the code, not who can physically access or view the code.
-  Access modifiers do NOT control whether other developers, testers, or users can see or use the code.
- They manage how classes interact with each other within the application.

Where are modifiers accessible.

| Modifier    | Class | Package | Subclass | World |
| ----------- | ----- | ------- | -------- | ----- |
| `public`    | ✅     | ✅       | ✅        | ✅     |
| `protected` | ✅     | ✅       | ✅        | ❌     |
| (default)   | ✅     | ✅       | ❌        | ❌     |
| `private`   | ✅     | ❌       | ❌        | ❌     |
1. A package is a namespace that organizes a set of related classes.
2. A public access modifier is a modifier that does not restrict the members at all.A public member (method or field) is accessible within the package as well as outside the package. Basically, everywhere!
3. The private access modifier is the one that has the lowest accessibility level.The scope of private entities (methods or fields) is limited to the class in which they are declared.Can you declare a constructor as private?A  constructor can be declared as private but you cannot create an object of the class from anywhere!

```java
class TestClass {
    //private variable and method
    private int num=100;
    private void printMessage() {
      System.out.println("Hello java");}
}

public class Main {
 public static void main(String args[]) {
   TestClass obj=new TestClass();
   System.out.println(obj.num);
   obj.printMessage();
   }
}
```

### Constructor

- Constructor - special method that initializes new objects/instances of class.- It is called when an instance of the [class](https://www.javatpoint.com/object-and-class-in-java) is created. Memory for the object is allocated.- Constructors always have the same name as the class.​​​​- It must have no explicit return type.
	- Types of constructors:
		- Default Constructor (no-arg constructor)
		- Parameterized Constructor

### Oops

- It is a programming model based around objects and data.    
- Many things in life can be represented in code.
- Complex programs are hard to keep organized.
- OOP organizes and structures code
- Groups together data and behaviour into one place.
- Promotes modularization of programs.
- Isolates different parts of a program from each other.

#### Encapsulation

- Bundle an object's data along with the methods that operate on that data within the same class.
- Hiding the data of a class as much as possible, unless it’s necessary to expose.
- Restrict the access to our class as much as possible, so that we can change the properties and the behaviors only from inside the class.
- Packages can be considered as data encapsulation (or data-hiding).
- Encapsulation is done using access modifiers.

#### Inheritance:

![[Pasted image 20250624075453.png]]
- A mechanism for creating new classes from existing classes
-  Allows new classes to inherit fields and methods from their parent class
- To use inheritance, create a new class and specify its parent class using the extends keyword.
-  Use the super keyword to access parent class attributes and methods

```java
// Base class
class Vehicle {
    int maxSpeed = 120;

    Vehicle() {
        System.out.println("Vehicle constructor called");
    }

    void start() {
        System.out.println("Vehicle is starting...");
    }
}

// Derived class
class Car extends Vehicle {
    int maxSpeed = 180; // This hides the parent's maxSpeed

    Car() {
        // Calling the parent class constructor
        super();
        System.out.println("Car constructor called");
    }

    void showSpeeds() {
        System.out.println("Car speed: " + maxSpeed);
        System.out.println("Vehicle speed: " + super.maxSpeed); // Access parent variable
    }

    void start() {
        super.start(); // Call parent class method
        System.out.println("Car is starting with advanced system...");
    }
}
```


#### Polymorphism

- Polymorphism means having many forms. 
- Perform a single action in different ways. 
- Two forms of Polymorphism:
	- Method Overloading
	- Method Overriding

##### Method Overloading:

- When a class has two or more than two methods which are having the same name but different types of order or number of parameters, it is known as Method Overloading.
- Method overloading is resolved during compile time.
- Three ways to overload methods:
	- By changing the number of arguments/parameters.
	- By changing the data type of arguments.
	- By changing the Order of arguments.
- Changing only return type with same parameters of method is not Method Overloading.

##### Method Overriding:

**Method overriding** allows a **subclass** to provide its **own implementation** of a method that is already defined in its **superclass**.

- Only inherited methods can be overridden.
- The overriding method must have same argument list.
- The overriding method must have same return type.
- The overriding method must not have more restrictive access modifier.
- If the overridden method  has default access, then the overriding one must be default, protected or public.
- If the overridden method is protected, then the overriding one must be protected or public.
- If the overridden method is public, then the overriding one must be only public.

- Always use `@Override` annotation (optional but strongly recommended).

```java
class Animal {
    void sound() {
        System.out.println("Animal makes a sound");
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Dog barks");
    }
}
```

| Method Type                  | Can Override? | Reason                 |
| ---------------------------- | ------------- | ---------------------- |
| `private` method             | ❌ No          | Not inherited          |
| `final` method               | ❌ No          | Can't override final   |
| `static` method              | ❌ No          | Hidden, not overridden |
| `abstract` method            | ✅ Yes         | Must be overridden     |
| `default` method (interface) | ✅ Yes         | From Java 8 onward     |

##### Static:

- It’s a field or method of a class that isn’t associated with a specific instance of the class.
- Can be accessed without creating a new class instance.
- A static member is shared among all the instances of the class.

Static method behavior is not dependent on instance variable.
Can access static member variable and modify it.

#### Abstraction

- Hide Implementation Complexity
- Expose only relevant functionality to the user.
- Focus on what the object does instead of how it does it.
- Implemented using abstract classes and interfaces.

In simple words, Abstraction is the practice of breaking a large problem into smaller components, so each smaller problem can be worked on in (relative) isolation.

- abstract keyword is a non-access modifier.
- Can be used with methods and classes.
- Abstract keyword cannot be used with following keywords
	- final
	- static
	- private
- Abstract classes cannot be instantiated.

- Subclass of an abstract class must 
	- Implement all abstract methods in the super-class
	- Or be declared abstract itself.
-  A class that contains at least one abstract method must be declared abstract.- Abstract methods have no body.

#### Interface 

- An interface defines a contract which specifies what a class must do and not how.
- An interface declaration contains 
	- Signatures, but no implementations for methods
	- May also contain constants
- A class that implements an interface must implement all the methods declared in the interface.
- You can use interface names anywhere you can use any other data type name. If you define a reference variable whose type is an interface, any object you assign to it must be an instance of a class that implements the interface.
- Multiple inheritance can be achieved using interfaces.


> [!NOTE] Object class in JAva
> The object class in java is the super class of all objects.Every class in Java is directly or indirectly derived from the Object class. Meaning even if u don't explicitly mention its inheritance it will still be inherited.`java.lang.Object`.

|Method|Description|
|---|---|
|`toString()`|Returns a string representation of the object|
|`equals(Object obj)`|Compares two objects for equality|
|`hashCode()`|Returns an integer hash code|
|`getClass()`|Returns the runtime class of the object|
|`clone()`|Creates and returns a copy (used for cloning) — class must implement `Cloneable`|
|`finalize()`|Called by GC before object is garbage collected (deprecated from Java 9)|
|`wait()`, `notify()`, `notifyAll()`|Thread synchronization methods (used with `synchronized`)

Whenever we print any object reference, it invokes to toString() method which every class inherits from Object class. Its default implementation:

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashcode());
}

// getclass - Returns the object of type Class that represents the runtime class of the object.
// toHexString - Returns a string representation of unsigned integer value in hexadecimal format.
// hashcode - Returns a hash code value for the object (Will be discussed further!)
```

To print something meaning toString() method must be overwritten. 
```java
class Student {
    String name;
    int age;
    Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
    // Overriding toString()
    @Override
    public String toString() {
        return "Student{name='" + name + "', age=" + age + "}";
    }
}
```

Similarly equals() compares for memory location by default.

```java
public class Person {
    String name;

    Person(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Person)) return false;
        Person p = (Person) obj;
        return this.name.equals(p.name);
    }
}

// instanceof operator can be used to test for type of object.
```

Similary we can also change hashcode().
The **default implementation** of `hashCode()` returns a value based on the **memory address** (or a similar unique identifier) of the object.

```java
class Person {
    String name;
    Person(String name) {
        this.name = name;
    }
    @Override
	public int hashCode() {
	    return Objects.hash(name); // handles nulls safely
	}
	// or 
	@Override
	public int hashCode() {
	    return name.hashCode();
	}
}
```

#### Final keyword:

The `final` keyword in Java is a **non-access modifier** that can be applied to **variables**, **methods**, and **classes**, and its meaning depends on where it is used. In general, `final` indicates **immutability** or **non-overridable behavior**.

1. final variable can be assigned only once , making them constant after initialization.
2. If a reference is final, the **reference can't change**, but the object's internal state **can**.
```java
final List<String> list = new ArrayList<>();
list.add("Hello");  // ✅ allowed
list = new ArrayList<>(); // ❌ not allowed
```
3. A `final` method **cannot be overridden** in a subclass.
4. A `final` class **cannot be extended**.

#### Static keyword:

The `static` keyword is used to **declare class-level members** — meaning the member belongs to the class itself rather than to any specific instance of the class. This can be applied to **variables**, **methods**, **blocks**, and **nested classes**.

1. **Static Variable** (class variable): Shared among all instances.
2. **Static Method**: Can be called without creating an object; cannot access instance variables directly.
3. **Static Nested Class**: Doesn’t need an outer class instance to be used.

#### Abstract keyword:

The `abstract` keyword is used to **define abstract classes and methods** — classes or methods that are **incomplete** and meant to be **extended or implemented** by subclasses.

1. **Abstract Class**: Cannot be instantiated; used as a base class.

```java
abstract class Animal {
    abstract void makeSound(); // no body
    void breathe() { System.out.println("breathing"); }
}
```

2. **Abstract Method**: Declared without a body; must be overridden by subclass.

```java
abstract class Shape {
    abstract double area(); // only signature
}

class Circle extends Shape {
    double area() {
        return 3.14 * r * r;
    }
}
```


