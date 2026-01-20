
---

Thread communication is a fundamental concept in concurrent programming that allows multiple threads to coordinate and share data effectively. Proper thread communication is essential for building robust, efficient, and thread-safe applications. 

Methods - 

1. wait() - When a thread calls the wait() method on an object, it releases the monitor (lock) it holds on that object and goes into a waiting state. 

Use wait() when a thread needs to pause execution until some condition (usually represented by a shared variable) changes. For example, a consumer thread might wait for a producer to produce an item.

2. notify() - The notify() method wakes up a single thread that is waiting on the object’s monitor. If more than one thread is waiting, the scheduler chooses one arbitrarily.

Use notify() when only one waiting thread needs to be awakened (e.g., when one resource becomes available) to continue its execution.

3. notifyAll() - The notifyAll() method wakes up all threads that are waiting on the object’s monitor.

 Use notifyAll() when a change in the condition may be relevant to all waiting threads. For instance, when a producer adds an item to a queue that multiple consumers might be waiting for, you want to wake all waiting threads so they can re-check the condition.

These methods must be called from within a synchronized context (a synchronized block or method) on the same object whose monitor the thread is waiting on. They work together with a shared condition (often a flag or another shared variable) that threads check in a loop to handle spurious wakeups.

These methods work with a thread’s monitor (the intrinsic lock on an object) to coordinate the execution between threads. 

Waiter chef example - 

```java
package code.threads;

public class Waiter implements Runnable{
    private final Object lock;

    public Waiter(Object lock){
        this.lock = lock;
    }

    @Override
    public void run(){
        System.out.println("Order taken");
        synchronized(lock){
            try{
                System.out.println("Order delegated to chef");
                lock.wait();
                System.out.println("Food is ready and ready to be served");
            }catch(Exception e){
                System.out.println("ERROR!!!");
            }
        }
    }
}
```

```java
package code.threads;

public class Chef implements Runnable{
    private final Object lock;

    public Chef(Object lock){
        this.lock = lock;
    }

    @Override
    public void run(){
        try{
            Thread.sleep(3000);
            System.out.println("Chef ready for taking order.");
            synchronized(lock){
                System.out.println("Order taken by chef");
                System.out.println("Processing order");
                Thread.sleep(2000);
                System.out.println("Order processing done");
                lock.notify();
            }
        }catch(Exception e){
            System.out.println("ERROR");
        }
    }
}
```

