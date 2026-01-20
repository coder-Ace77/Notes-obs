
---
Behaviour design patterns is used to solve the problems where interaction between some objects is required.
## Observational design pattern : 

Observational design pattern is used to solve problem when some event is to be notified to some entities. You have one object (Subject) whose changes should automatically notify and update many other objects (Observers).

It has 4 key types - 

#### Subject(Observable/Publisher)

Subject is where the entire state of events is stored and its task is to notify the observers. It should necessarily implement three functions - 

1. attach() - To attch some objects.
2. detach() - To detach some objects. 
3. notifyObservers() - this method should be called when state changes. 

Additionally it can also implement some method which can act as trigger to send notifications. 

#### Observer(or Subscriber)

Must implement the `update()` method. additonally a common interface `Observer` is made which is retained by the subject. Subject does not need to know the individual observers concrete implementation rather it deals only with common interface. 

#### ConcreteSubject

Stores the state when state changes notifies the observers. 

#### ConcreteObject

Implements the update logic. 

Example of youtube-

Observers 

```java
package youtube;

public interface Subscriber {
    void update(String msg);
}

// its impls
package youtube;

public class EmailSubscriber implements Subscriber{
    private String email;
    public EmailSubscriber(String email){
        this.email = email;
    }

    public void update(String msg){
        System.out.println("An email is sent to "+email+" to watch the video "+msg);
    }
}

package youtube;

public class YouTubeSubscriber implements Subscriber{
    private String name;
    public YouTubeSubscriber(String name){
        this.name=name;
    }

    public void update(String msg){
        System.out.println(name+" is notified to watch video "+msg);
    }
}
```

Subject

```java
package youtube;

public interface YoutubeChannel {
    void addSubscriber(Subscriber subscriber);
    void removeSubscriber(Subscriber subscriber);
    void notifySubs();
}
```

Now the impl

```java
package youtube;
import java.util.ArrayList;
import java.util.List;

public class YoutubeChannelImpl implements YoutubeChannel{
    private String video;
    private List<Subscriber> subscribersList = new ArrayList<>();

    void upload(String video){
        this.video = video;
        notifySubs();
    }
    
    @Override
    public void addSubscriber(Subscriber subscriber){
        subscribersList.add(subscriber);
    }

    @Override
    public void removeSubscriber(Subscriber subscriber){
        subscribersList.remove(subscriber);
    }

    @Override
    public void notifySubs(){
        for(Subscriber subscriber:subscribersList){
            subscriber.update(video);
        }
    }
}
```

Usage 

```java
YoutubeChannelImpl channel = new YoutubeChannelImpl();
Subscriber alice = new YouTubeSubscriber("Alice");
Subscriber bob = new EmailSubscriber("bob.@gmail.com");

channel.addSubscriber(alice);
channel.addSubscriber(bob);

channel.upload("Anime summary");
channel.upload("Political satire");
```

Note that this pattern follows all the key notes of SOLID principles. 
## Strategy design pattern

In simple terms, the Strategy Pattern allows you to define a family of algorithms or behaviors, and choose the one to use during runtime. It is like having a toolbox where you can pick the best tool (or strategy) for the task at hand. This approach avoids hardcoding multiple behaviors into one class and promotes flexibility by separating the behavior logic into different classes.

Some very real example is Payment systems - 

The strategy(algorithm) is implemented through common interface - 

Key features - 

1. Common interface for algos - example Payment inteface
2. Concrete classes for the algos - example UpiPayment
3. Running class - For example payment processor

Example - 

```java
package payment;

public interface Payment {
    void pay(int amount);
}

package payment;
public class CreditCardPayment implements Payment{
    @Override
    public void pay(int amount){
        System.out.println("Paying the amount: "+amount + " by crtedit card.");
    }
}

package payment;

public class PaymentProcessor {
    private Payment payment;
    PaymentProcessor(Payment payment){
        this.payment=payment;
    }
    public void processPayment(int amount){
        payment.pay(amount);
    }
    public void setPaymentStrategy(Payment payment){
        this.payment=payment;
    }
}
```

Usage - 

```java
package payment;

public class Main {
    public static void main(String[] args) {
        Payment credPayment = new CreditCardPayment();
        Payment upiPayment = new UpiPayment();
        PaymentProcessor paymentProcessor = new PaymentProcessor(credPayment);
        paymentProcessor.processPayment(100);
        paymentProcessor.setPaymentStrategy(upiPayment);
        paymentProcessor.processPayment(100);
    }
}
```

## Composite design pattern

Composite is a **structural design pattern** used when you need to treat **individual objects** and **groups of objects** in a **uniform way**.

It lets you build **tree-like structures** (hierarchies) where- 
- **Leaf** = simple object
- **Composite** = object that contains other objects (children)
- Both implement the **same interface** so they can be treated uniformly.

Think of a **File System** - 
- A **File** is a leaf → cannot contain anything.
- A **Folder** is a composite → contains files and other folders.

So here both the files and folders will be implemented by a common interface. 

```go
           Component (interface)
             /         \
            /           \
         Leaf        Composite
                        |
                  children list
```

**Composite implements the same interface as Leaf**, but holds a list of children.

Example say for a Graphic system
We can have-

- Shapes (circle/square)
- Drawings

Common interface which is common to both `Graphic`

```java
interface Graphic {
    void draw();
    void move(int x, int y);
}

class Circle implements Graphic {
    @Override
    public void draw() {
        System.out.println("Drawing Circle");
    }

    @Override
    public void move(int x, int y) {
        System.out.println("Moving Circle to (" + x + ", " + y + ")");
    }
}

import java.util.ArrayList;
import java.util.List;

class Drawing implements Graphic {
    private List<Graphic> children = new ArrayList<>();

    public void add(Graphic g) {
        children.add(g);
    }

    public void remove(Graphic g) {
        children.remove(g);
    }

    @Override
    public void draw() {
        System.out.println("Drawing Composite:");
        for (Graphic g : children) {
            g.draw();
        }
    }

    @Override
    public void move(int x, int y) {
        for (Graphic g : children) {
            g.move(x, y);
        }
    }
}

```

Client 

```java
public class Client {
    public static void main(String[] args) {

        Graphic circle = new Circle();
        Graphic square = new Square();
        
        Drawing drawing = new Drawing();
        drawing.add(circle);
        drawing.add(square);

        drawing.draw();
        drawing.move(10, 20);
    }
}
```

What are the key advantages - 

1. Open close principle is valid here.
2. Recursive structure
3. Uniformity



