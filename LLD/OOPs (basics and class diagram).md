
---

Object-Oriented Programming (OOP) is a programming paradigm that organizes code into objects, which represent real-world entities. It allows developers to model complex systems by breaking them down into smaller, manageable pieces.

Classes themselves are not the objects but the blueprints to form the object its like guide to create the object. It defines the properties (attributes) and behaviors (methods) that the objects will have.
Constructor is the special method to initialize the attributes of class. 

Objects is an instance of class. It represents a specific realization of the class blueprint, with its own unique set of data.

Advantages of OOPs- 

- Class is created once and many objects are instantiated from it. 
- Modularity - Class helps in organisation of code into logical sections. 
- Abstraction - Focus on the essentail details of entity without worrying about internal workings. 

> [!NOTE] Class diagram
> A **Class Diagram** is a UML (Unified Modeling Language) diagram that shows the **structure** of a system.  It visualizes the **classes**, their **attributes**, **methods**, and the **relationships** between classes.


```pqsql
+---------------------+
|     ClassName       |   ← Class name
+---------------------+
| - attribute1 : Type |   ← Attributes (fields)
| + attribute2 : Type |
+---------------------+
| + method1() : Type  |   ← Methods (functions)
| - method2() : void  |
+---------------------+
```

here - 

`+` Public 
`-` private
`#` protected

## Types of relationships

Obviously real world objects can have relationship between them.
### Association

Association is a general relationship where one class knows about or uses another.
It's like a friendship—two entities are aware of each other, but they exist independently.

Example Customer has a address. Person has a car. 

```lua
Customer --------> Address
```

### Aggregation

Aggreegation is a type of association. 
Whole–part relationship but the part can exist independently.
Example - A team contains player but player can exist independantly. 

```lua
Team ◇----- Player
```

Differences-

Association is a general relationship (friendship) with no ownership implied; objects simply interact or use each other.Aggregation represents a “has-a” relationship where one object (the whole) contains or references other objects (the parts), but the parts can exist independently of the whole.

Aggregation represents a “has-a” relationship where one object (the whole) contains or references other objects (the parts), but the parts can exist independently of the whole. Note that the whole needs to be made up of part and whole is kind of container for example - 

Team -> Players
Library -> Books
Company -> Employees

- The **whole** is something that _contains multiple parts_.
- The **whole’s identity** is defined by its parts (at least partially).
- The part is _physically or logically inside_ the whole.

Now for tricky example Product category realtionship is an assocation not aggregation why because Category is not made up of products nor Product is made up of category. 

### Composition

Strong whole–part relationship. It is stronger form of aggregation where if the whole is destroyed parts can not exist independently. 
Part can not exist without whole. Example Room can not exist independently of house.

```lua
House ◆----- Room
```

Note that in composition the lifecycle of part is dependent on whole so if whole dies part also dies. 
### Inheritance

Inheritance represents an "is-a" relationship where a subclass inherits properties and behaviors from its parent class.

```lua
Animal
  ▲
  |
Dog
```


### Dependency


Dependency represents a temporary relationship where one class uses another class, typically via method parameters or local variables. It’s like borrowing a tool for a short while. For example document does not have a printer rather it simply uses when in need. 

```lua
Order -- -- > PaymentService
```

Note that dependency is not a type of association. rather there is a clear differences between two - 

In association objects are passed (often via constructors or setters) and stored as persistent fields. This creates a long-term relationship where the object is available throughout the lifetime of the class. For example, a Person stores a reference to a Car and uses it whenever needed. 

In dependency objects are passed as method parameters and used only within that method's scope. This creates a short-term, temporary relationship where the object is used just for the duration of the method call, and isn't stored for later use. For example, a Document receives a Printer to print its content and doesn't keep a reference to the Printer afterward.

Also microservice and database is again a dependency. 

Now in interviews it is many times tricky to figure out the realtionship because of many factors. At this point it is necessary to understand that aggregation/composition are defined only on the logical relationships weather the life cycle of part is tied to lifecycle of whole and that is it. Interviewers can add multiple constraints which may be misleading many times. 

### Realization 

Realization represents a relationship where a class implements an interface. It’s like signing a contract to provide specific behaviors.

Example - 

![[Pasted image 20251124133629.png]]



### Example of library management

![[Pasted image 20251124133652.png]]

Readable interface exposes one method read() and is implemented by book Ebook extends Book by adding fileFormat. Book and Author have assocaition relation because Book just refernce author its not like Book is made from Authors. 
Library has aggregation with books as books can logically exist without library at home etc. The diagram is wrong. Reader has a dependency relation with book. Finally reading club is the assocaiton of readers. 