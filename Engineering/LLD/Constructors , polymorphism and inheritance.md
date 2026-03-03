
---

Constructors are called automatically when object of class is instantiated. They have no return type and there can be multiple constructors based on overloading of constructors. 

### Types 

- Default constructors is an implicit constructor. If you do not explicitly create any constructor for a class, Java automatically provides a default constructor.This constructor initializes instance variables to their default values based on the data type.
- Explicit default constructor- Also called no arg constructor is where we can define our own default fields. You can define your own default constructor to set custom default values for the class fields instead of relying on Java's implicit default values.
- Parameterised constructor - > A parameterized constructor takes arguments to initialize the object with specific values.
- Copy constructor - > A copy constructor initializes an object using another object of the same class.

```java
public Movie(String title, int duration) {
	this.title = title;
	this.duration = duration;
}

public Movie(Movie other){
	this.title = other.title;
	this.duration = other.duration;
}
```

A private constructor is used to restrict object creation from outside the class. It is commonly used in Singleton Design Pattern.

```java
private Movie(){} // making default constructor private
```

Key points - 

1. A class can have multiple constructors but must differ in parameter lists. 
2. Constrcutors can call other constructors through this().
3. Constructors can call parent class constructors using super() in java.
4. Always use constructors to ensure objects are in a consistent and valid state.
5. Utilize copy constructors carefully to avoid shallow copying when deep copying is required.


> [!NOTE] Important
> Constructor can not be final as constructor can not be inherited. (final methods can not be inherited)
> Constructor belong to objects and not class so they can not be static 
> Since constructor is used to intialize concrete object this means it can not be abstract. 
> If we define any type of constructor implicit default constructor will not be provided. 
> Parent class constructor is called first and child class constructor is called later on. 


## This keyword

The "this" keyword in Java is a built-in reference to the current object within a class's methods or constructors. It allows access to the object's properties, methods, and other members. 
this allows the use of constructors in other constructors. 

The "this" keyword can be used to return the current instance of a class. This approach is commonly used in implementing fluent interfaces, where multiple methods are called on the same object in a single statement. This pattern enhances code readability and provides a seamless way to chain method calls

```java
class Person {
  private String name;
  Person setName(String name) {
	  this.name = name;
	  return this; // Enables method chaining
  }
  void display() {
    System.out.println("Name: " + this.name);
  }
}

public class Main {
  public static void main(String[] args) {
    Person p = new Person();
    p.setName("Bob").display();
  }
}
```

"this" can be used to pass the current object as a parameter to another method.

## Polymorphism

Polymorphism means a thing can have different behaviour in different contexts. Polymorphism is the ability of a **single interface** (like a method name or an operator) to take on **different forms** or behaviors depending on the context.

### Compile time or Static polymorphism

Compile-time or Static polymorphism occurs when the method to be executed is determined at compile time. It is achieved using method overloading or operator overloading.

When there are multiple functions with the same name but different parameters, then the functions are said to be overloaded, hence this is known as Function or Method Overloading. 
Functions can be overloaded by changing the number of arguments or/and changing the type of arguments.

### Run time polymorphism

Runtime polymorphism occurs when the method to be executed is determined during runtime. It is achieved through method overriding and is closely tied to inheritance.

Method overriding is the essential technique which enables run time polymorphism. 

```java
class Animal {
    public void sound() {
        System.out.println("The animal makes a generic sound.");
    }
}

class Dog extends Animal {
    @Override
    public void sound() {
        System.out.println("The dog barks."); // Specific implementation
    }
}

class Cat extends Animal {
    @Override
    public void sound() {
        System.out.println("The cat meows."); // Another specific implementation
    }
}

// Main method context
Animal myPet = new Dog(); // Parent class reference points to a child object
myPet.sound();           // Output: The dog barks. (The actual Dog object's method is called
```

Method resolution happens at runtime based on the actual object type.

Polymorhpism allows code reusability,flexibility and extensibility. 
Dynamic method dispatch introduces slight overhead as the JVM resolves the method during runtime.



