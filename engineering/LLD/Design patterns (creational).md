
---

Design patterns are you to solve some repetitive problem.
## Creational design patterns

Creational design patterns are a category of **Object-Oriented Design Patterns** that focus on **how objects are created** in a flexible, controlled, and reusable way.
Sometimes object creation is not simple.  
Maybe an object:
- has 10 parameters
- depends on other objects
- must be created in a certain sequence
- must be reused (singleton)

Creational patterns decouple the **creation** from **usage**, so you follow **Dependency Injection** & **OCP**. 

Why singleton?

1. Efficient utilization of resources. 
2. Inconsistent logging
3. Need of one global object

### Singleton pattern

Ensures only one instance of class exists. Used when required say only one instance of db connection or one logger instance. 

Say we want to make this object Logger singleton

```java
public class Logger{
	public void log(String msg){
		System.out.println("[LOG]:"msg);
	}
}
```

So this is how you make it singleton

```java
public class Logger{
	private static Logger logger = null;
	private Logger(){} // make constructor private so that object can't be directly created
	
	public static Logger getLogger(){ // static method
		if(logger==null){
			logger = new Logger();
		}
		return logger;
	}
	
	public void log(Strin msg){
		System.out.println("[LOG]"+msg);
	}
}

// usage 
Logger logger = Logger.getLogger();
logger.log("Application started");
```

Finally this singleton design pattern might break if two threads try to create the singleton object at the same time. So we need to make this thread safe

Use synchronized blocks or mutex

```java
public class Logger{
	private static volatile Logger logger = null;
	private Logger(){} // make constructor private so that object can't be directly created
	
	public static Logger getLogger(){ // static method
		if(logger==null){
			synchronized(Logger.class){
				if(logger==null){
					logger = new Logger();
				}
			}
		}
		return logger;
	}
	
	public void log(Strin msg){
		System.out.println("[LOG]"+msg);
	}
}

```

## Factory Design pattern

### Simple Factory

The **Factory Design Pattern** is a **creational design pattern** that provides an **interface (a method) to create objects**, but lets subclasses or separate factory classes decide **which exact object to create**.

There are three components to do this - 

- A super interface from which all other concrete implementations are generated on. 

```java
public interface Shape {
    void draw();
}
```

Then we implement the interface to create concrete implementations. 

```java
public class Circle implements Shape {
    public void draw() { System.out.println("Circle"); }
}

public class Square implements Shape {
    public void draw() { System.out.println("Square"); }
}
```

Finally we have a static method where we handle design logic

```java
public class ShapeFactory {
    public static Shape getShape(String type) {
        if (type.equalsIgnoreCase("circle")) return new Circle();
        if (type.equalsIgnoreCase("square")) return new Square();
        return null;
    }
}
```

In the client code 

```java
public class Main {
    public static void main(String[] args) {
        Shape s = ShapeFactory.getShape("circle");
        s.draw();
    }
}
```

What are the good things -

- Loose coupling - Client depends only on the **interface**, not concrete classes.
- Centrlized and easy to create objects. Sometimes objects depend on more objects which is handled here. 
- Object creation can be controlled based on:
- Config files
- User inputs
- Conditions
- Environments

Drawback - 
- Voilates open-close principle if new classes are added.
- Lot of if and else

The above way is called SImpleFactory

### Factory method

Here superclass defines a factory method for creation of objects but subclasses descide which object to create. This allows the class to **defer the object creation to child classes**, enabling **extensibility** without modifying existing code.

Parent class contains - 
- Bussiness logic 
- Empty abstract createProduct() method

Subclass contains - 
- The actual creation of the correct object

It has following parts - 

1. Product interface

```java
interface Transport {
    void deliver();
}
```

2. Concrete products

```java
class Truck implements Transport {
    public void deliver() { System.out.println("Deliver by road"); }
}

class Ship implements Transport {
    public void deliver() { System.out.println("Deliver by sea"); }
}
```

Creator - It is class that has abstract method to create object

```java
abstract class Logistics {
    abstract Transport createTransport();  // Factory Method

    void planDelivery() {
        Transport t = createTransport();
        t.deliver();
    }
}
```

Concrete creators 

```java
class RoadLogistics extends Logistics {
    Transport createTransport() {
        return new Truck();
    }
}

class SeaLogistics extends Logistics {
    Transport createTransport() {
        return new Ship();
    }
}
```

Client 

```java
public class Main {
    public static void main(String[] args) {
        Logistics l = new RoadLogistics();
        l.planDelivery();
    }
}
```

Example of shape

```java
interface Shape {
    void draw();
}
```

Concrete shpes
```java
class Circle implements Shape {
    public void draw() { System.out.println("Drawing Circle"); }
}

class Square implements Shape {
    public void draw() { System.out.println("Drawing Square"); }
}

class Rectangle implements Shape {
    public void draw() { System.out.println("Drawing Rectangle"); }
}
```

Creators (abstract)

```java
abstract class ShapeFactory {
    abstract Shape createShape();  // Factory Method

    public Shape getShape() {
        Shape s = createShape();   // calls subclass method
        return s;
    }
}
```

Concrete 

```java
class CircleFactory extends ShapeFactory {
    Shape createShape() { return new Circle(); }
}

class SquareFactory extends ShapeFactory {
    Shape createShape() { return new Square(); }
}

class RectangleFactory extends ShapeFactory {
    Shape createShape() { return new Rectangle(); }
}
```

Now client will be using shape factory to create

```java
public class Main {
    public static void main(String[] args) {
        ShapeFactory factory = new CircleFactory();
        Shape s = factory.getShape();
        s.draw();

        factory = new SquareFactory();
        factory.getShape().draw();
    }
}
```

### abstract factory pattern

Abstract Factory is a creational design pattern that provides an interface for creating _families of related or dependent objects_ without specifying their concrete classes.
Instead of returning a single object, it returns an entire **factory** that can build multiple related objects.

You have **multiple variants** (themes/platforms/brands)  
AND  
each variant has **multiple related objects**.

Example: UI Toolkit

- Windows Theme → WinButton, WinCheckbox
- Mac Theme → MacButton, MacCheckbox

You must ensure:

- If you pick Windows theme → all objects must be Windows type
- No mixing (MacCheckbox with WinButton)

Providing a **factory per family**  
Each factory can create **multiple different products** that belong to the same family.

Example 

```java
interface Button {
    void render();
}
interface TextBox {
    void render();
}
interface ScrollBar {
    void render();
}
```

Components 

```java
class ModernButton implements Button {
    public void render() { System.out.println("Modern Button"); }
}

class ModernTextBox implements TextBox {
    public void render() { System.out.println("Modern TextBox"); }
}

class ModernScrollBar implements ScrollBar {
    public void render() { System.out.println("Modern ScrollBar"); }
}
```

```java
class ClassicButton implements Button {
    public void render() { System.out.println("Classic Button"); }
}

class ClassicTextBox implements TextBox {
    public void render() { System.out.println("Classic TextBox"); }
}

class ClassicScrollBar implements ScrollBar {
    public void render() { System.out.println("Classic ScrollBar"); }
}
```

Abstract factory

```java
interface UIFactory {
    Button createButton();
    TextBox createTextBox();
    ScrollBar createScrollBar();
}
```

Concrete factory

```java
class ModernUIFactory implements UIFactory {
    public Button createButton() { return new ModernButton(); }
    public TextBox createTextBox() { return new ModernTextBox(); }
    public ScrollBar createScrollBar() { return new ModernScrollBar(); }
}
```

```java
class ClassicUIFactory implements UIFactory {
    public Button createButton() { return new ClassicButton(); }
    public TextBox createTextBox() { return new ClassicTextBox(); }
    public ScrollBar createScrollBar() { return new ClassicScrollBar(); }
}
```

```java
public class Main {
    public static void main(String[] args) {

        UIFactory factory = new ModernUIFactory();
        
        Button b = factory.createButton();
        TextBox t = factory.createTextBox();
        ScrollBar s = factory.createScrollBar();

        b.render();
        t.render();
        s.render();
    }
}
```

## Builder pattern

The **Builder Pattern** is a creational design pattern used to construct **complex objects step-by-step**, especially when the object has:

- Many optional parameters,
- Multiple configurations,
- An order-dependent construction process.

There aremany problems while creating objects which have large number of fields. 
1. To much large constructor.
2. Too many constructors if tackle issue of multiple optional fields with constructor overloading
3. Using setters is dangerous as it will not guarantee that correct object state has been created. 

Example of Car

```java
public class Car{
	private String engine;
	...
	
	private Car(CarBuilder carBuilder){
		this.engine = carBuilder.engine;
		....
	}
	// getter setter
	...
	
	private static class CarBuilder{
		private String engine;
		private String country = "India"; // deafault field
		... // same fields
		
		public CarBuilder setEngine(String engine){
			this.engine = engine;
			return this;
		}
		
		... // others 
		
		public Car build(){
			return new Car(this);
		}
	}
}
```

usage - 

```java
Car.CarBuilder builder = new Car.CarBuilder();
Car car = builder.setEngine("V8")
				.setColor("Red")
				.build();
```

Same builder can be used to create multiple instances. Builder is inside the Car class for logical grouping and encapsulation. 