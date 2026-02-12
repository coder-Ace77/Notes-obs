
---

Threads are the fundamental unit of execution in any programming language. 
Process vs thread in breif

![[Pasted image 20251126181919.png]]

Some applications can sprawn multiple process they are used if - 

- One needs strong isolation between different parts of application.  example browsers may sprawn different process for different tabs.
- Running completely independent task. 
- Levarage multiple CPU cores for separate computational task. 

While a multithreaded application where we have multiple threads in one environment threads enables applications to behave in real times for the actions and events. 

Multiple threads can run simultaneously, allowing programs to perform multiple tasks at once.

In java there are two ways to create threads. 

## Threads 

Extending thread class (inherit) and override the run method.  

```java
class MyThread extends Thread{
	
	@override 
	public void run(){
		// do something
	}
}

public class Main{
	public static void main(String args[]){
		Mythread t1 = new Mythread(); // running example
		t1.start();
	}
}
```

It is not a good of way of creating  threads - 

1. Java allows extending **only one class**. If you extend `Thread`, your class **cannot extend anything else**, which is very restrictive.

Example 

```java
class Dog extends Animal, Thread {   // ❌ ERROR: Java doesn't allow multiple inheritance
    public void run() {
        System.out.println("Dog running in thread");
    }
}
```

1. It mixes task logic with thread management this voilates separation of concern.  Runnable keeps them separate 
	- Runnable = **what** to run
	- Thread = **where** to run it
2. A thread instance can not be restarted once it finishes. With runnable a task can be exec multiple times. 
3. Thread instances does not work with thread pools. 
4. Every `Thread` object creates an **OS-level thread**, which is expensive.
## Runnable

Runnable interface represents a unit of work that is meant to be run on separate thread. Unlike thread runnable just represents what should be run and where should it be run is delegated to `Thread` object or Thread pool which manages the execution environement (also called thread management). `Runnable` contains a single method: `run()`. This method holds the logic that the thread will execute. 

A class that extends `Thread` cannot extend any other class, but by implementing `Runnable`, your class remains free to extend another class if needed. 

```java
class Animal {
    void eat() { System.out.println("Animal eating"); }
}

class Dog extends Animal implements Runnable {
    public void run() {
        System.out.println("Dog running in thread");
    }
}
```

Full example - 

```java
class MyRunnable implements Runnable{
	
	@override
	public void run(){
		// do something
	}
}

public class Main(){
	public static void main(){
		MyRunnable r = new MyRunnable();
		
		Thread task1 = new Thread(r);
		Thread task2 = new Thread(r); // separation of concerns so thead class will manage the environment
		
		task1.start();
		task2.start();
		
		// or 
		
		
		
	}
}
```

An important characteristic of Runnable is its limitation: it does not return a result and cannot throw checked exceptions.This design makes Runnable suitable for tasks where you want execution without expecting a response—for example, background logging, updating UI elements, or performing periodic activities.

![[Pasted image 20251126185751.png]]

## Callable interface

It was introduced in Java 5 and unlike Runable it can return result and can throw checked exceptions. Callable is part of java.util.concurrent package. Callable defines a single method, `call()`, which acts similarly to Runnable’s `run()` method. But call() can return the value of any type through generics. 

It can perform a computation, signal problems properly, and send the outcome back to the thread that submitted it. This return-capability makes Callable ideal for tasks such as database queries, mathematical calculations, file operations, and any other work where a result must be collected asynchronously.
Callable works with Future objects to retrieve results after task completion.

example

Note that callable will return Future and not the string as value is expected to be available sometime in future.

```java
// Callable<Type> represents the return type
class MyCallable implements Callable<String>{
	@override
	public String call() throws Exception{ // can throw checked exception
		return "Hello world";
	}
}

public class Main{

	public static void main(){
		MyCallable c = new MyCallable();

		FutureTask<String> ft = new FutureTask<>(c);
		Thread t = new Thread(ft);
		t.start();
		
		String s = ft.get();    // waits for result
		System.out.println(s);	
	}
}
```

Some important points -

- Difference between run and start - The start() method begins thread execution and calls the run() method, while the run() method simply contains the code to be executed. Directly calling run() won't create a new thread; it will execute in the current thread.
- We can not call start twice on same thread object. Calling start() twice on the same Thread object will throw an IllegalThreadStateException. A thread that has completed execution cannot be restarted.
- Thread safety refers to code that functions correctly during simultaneous execution by multiple threads. It can be achieved through synchronization, immutable objects, concurrent collections, atomic variables, and thread-local variables.
- If an uncaught exception occurs in a thread's run() method, the thread terminates. The exception doesn't propagate to the parent thread and doesn't affect other threads.

## Thread object -

A **Thread object** is Java’s representation of an actual OS thread.A Thread is simply a **Java object**, but when you call `start()`, the JVM asks the operating system to create a **real thread**, which then runs your code independently of the main thread.

A thread object has:

- **a name**
- **a thread ID (tid)**
- **a priority**
- **a state**
- **a task to run (`run()` method)**

### Thread lifecycle - 

A thread can go under following states and these can be get from Thread.State enum

- New - Thread object created but the start is not called. 
- Runnable - Eligible to run; may be running or waiting for CPU
- Blocked - Waiting to acquire a synchronized lock.
- Waiting - Waiting indefinitely for another thread to notify it.
- Timed-wait - Waiting for a specific period (sleep, wait(timeout), join(timeout)).
- Terminated - Thread's `run()` finished.

### Thread api

- Start - Starts a new thread of execution.  This internally calls `run()` in a new call stack.
- run() - Contains the code executed by the thread.  **Never call `run()` manually**—it will run in the same thread.

```java
Thread t = new Thread(runnalbe);
t.run(); // runs in same thread
t.start(); // new thread
```

- sleep() - pauses the execution of current thread

```java
Thread t = new Thread(() -> {
    try { Thread.sleep(2000); } catch (Exception e) {}
    System.out.println("Worker done");
});

t.start();
t.join();        // main waits
System.out.println("Main continues");
```

- join() - makes one thread wait for another thread to finish. 
- interupts - Politely ask the thread to stop.
- isInterupted() - Checks if thread has been interupted.
- setName() and getName() 

```java
Thread t = new Thread(() -> {});
t.setName("Worker-1");
System.out.println(t.getName());
```

- setPriority(int p) - 1 min , 5 norm , 10 max
- currentThread() - gives the refernece to thread currently execting

```java
System.out.println(Thread.currentThread().getName());
```

- isAlive()

Note that there are two ways to get this apis - First is using thread object outside of execution then we can direclty call these methods. Inside one can use Thread.currentThread(). 

Note that you should use `this` iff you extend thread. With runnable this is not a thread. 
Finally there are daemon threads which are very low priority and exist in background. 

|Type|Description|
|---|---|
|**User thread**|Main tasks of your program|
|**Daemon thread**|Background tasks (garbage collector, scheduler)|

## Using lambda

Lambda expressions make thread creation **short, clean, and modern** because Java allows you to pass a _functional interface_ wherever a thread expects a task.

Since runaable is functional interface(one method) only we can simply pass the lambda in thread object. 

```java
Thread t = new Thread(() -> System.out.println("running"));
t.start();
```
