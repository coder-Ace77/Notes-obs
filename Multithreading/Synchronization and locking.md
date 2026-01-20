
---

## Why synchronization

In multithreaded environment multiple threads access and modify shared data without coordination, they can interfere with each other’s operations. This can lead to race conditions where final output depends on the order of execution of thread timings. 

A **race condition** happens when two or more threads access shared data **at the same time**, and the final result depends on **who runs first**. Example two threads taking out the money from the same account can read incorrect balance this happens because both are allowed to run at same time. 

Example

```java
class BankAccount {
    private int balance = 1000;

    public void withdraw(int amount) {
        if (balance >= amount) {
            balance = balance - amount; // Critical section
        }
    }

    public int getBalance() {
        return balance;
    }
}

public class RaceExample {
    public static void main(String[] args) throws Exception {
        BankAccount account = new BankAccount();

        Thread t1 = new Thread(() -> account.withdraw(700));
        Thread t2 = new Thread(() -> account.withdraw(700));

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Final balance: " + account.getBalance());
    }
}
```

Another common problem is **data corruption**. If two threads write to a shared variable simultaneously, the updates can overlap, causing partially written or invalid values.

## What is synchronization?

Synchronization is the mechanism that ensures **controlled access** to shared data in a multithreaded program. It allows only one thread to execute a critical section at a time, preventing interference between threads.

Critical section - A critical section is a block of code that must not be executed by more than one thread at a time. Synchronization protects these sections to ensure data consistency.


For the simple syncronization there are two ways to achive it-

1. Method level
2. Block level

The synchronized keyword is used to control access to critical sections of code so that only one thread can execute the synchronized code at a time.

### Synchronized Method

When you declare an entire method as synchronized, the lock is acquired on the object instance (or on the Class object for static methods) before the method is executed and released after it finishes

Example

```java

package code.data;
public class Account {
    private int balance=0;
    public synchronized void deposit(int money){
        balance+=money;
    }
    public synchronized void withDraw(int money){
        balance-=money;
    }
    public int getBalance(){
        return balance;
    }
}
```


```java

package code.threads;

import code.data.Account;

public class AccountRunners implements Runnable{

    private final Account account1;
    private final Account account2;

    public AccountRunners(Account account1,Account account2){
        this.account1 = account1;
        this.account2 = account2;
    }

    @Override
    public void run(){
        account1.withDraw(100);
        account2.deposit(100);
        System.out.println("BALANCE"+account1.getBalance()+" "+account2.getBalance());
    }
}
```

### Synchronized block 

A synchronized block allows you to specify a particular block of code to be synchronized, along with the object on which to acquire the lock (often called a monitor). This is more fine-grained compared to a synchronized method.

Example

```java
package code.data;

public class Counter {
    private int count = 0;
    private final Object lock;

    public Counter(Object lock){
        this.lock = lock;
    }

    public void increment(){
        synchronized(lock){
            count++;
        }
    }

    public int getCount(){
        return count;
    }
}
```

```java
for(int i=0;i<numberOfThreads;i++){
	Thread th = new Thread(()->{
		counter.increment();
		System.out.println("[CURR]"+counter.getCount());
	});
	threads[i]=th;
	th.start();
}

for (int i = 0; i < numberOfThreads; i++) {
	try {
		threads[i].join();  // waiting for all threads to finish
	} catch (InterruptedException e) {
		e.printStackTrace();
	}
}
```

#### volatile keyword

The volatile keyword in Java is used to indicate that a variable’s value will be modified by multiple threads. Declaring a variable as volatile ensures two key things:

1. When a variable is declared volatile, its value is always read from and written to the main memory instead of a thread’s local cache.This means changes made by one thread are immediately visible to others.
2. volatile establishes a happens-before relationship. Operations on a volatile variable cannot be re-ordered relative to each other.

Uses - 
 If you only need visibility guarantees (not atomicity for compound actions like x++), volatile is lighter and faster than using synchronized. In lazy initialization patterns, volatile ensures that the constructed instance is visible to all threads correctly.

```java
public volatile boolean isRunning = false; // visible to all the threads correctly.
```

#### atomic keyword

Atomic variables in Java—found in the java.util.concurrent.atomic package—are designed to support lock-free , thread-safe  operations on single variables.

You should use atomic variables when you need to perform simple operations (like incrementing, decrementing, or updating) on shared variables in a multithreaded environment. They are especially useful when the overhead of locking is undesirable  and when the logic remains limited to single-step atomic operations. 

Note that atomic variables have lots of internal variables which are used for various pruposes like increment and decrement

```java
package code.data;
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicCounter {
    private AtomicInteger count = new AtomicInteger(0);
    public void increment(){
        count.incrementAndGet();
    }
    public int getVal(){
        return count.get();
    }
}
```

Now this provides atomic operations class for simple operations. 

Atomic classes rely on the CPU level mechanism called compare and swap(CAS) CAS checks two things in one step 

- Does the variable still have the value I expect?
- If yes, replace it with a new value.
- If not, do nothing (because another thread changed it).

This entire check-and-update happens **atomically in hardware**, not Java code.

Because the CPU guarantees that **compare + swap happens in a single instruction**.  
No other thread can interrupt it in the middle. 

## Locks

Locks in Java (via the Lock interface) offer more flexible and fine‑grained control over synchronization than the built‑in synchronized keyword.Locks ensure that only one thread can enter a critical section at a time, maintaining correctness.
Synchronized keywork also uses locking but here we are talking about explicit locks. 

### ReentrantLock

Reentrantlock means that a thread can acquire the same lock multiple times without getting blocked.
If a thread already holds a lock, and inside that block it again tries to lock the same lock (directly or indirectly), **Java allows it**.  
Java keeps an internal **hold count** for how many times the lock is acquired, and the lock is released only when the thread calls `unlock()` the same number of times.

Without reentant this will be a deadlock -

```java
class Demo {
    private final ReentrantLock lock = new ReentrantLock();

    public void outer() {
        lock.lock();
        try {
            System.out.println("In outer");
            inner();                    // calls another method
        } finally {
            lock.unlock();
        }
    }

    public void inner() {
        lock.lock();                    // same lock is acquired again
        try {
            System.out.println("In inner");
        } finally {
            lock.unlock();
        }
    }
}
```

AS outer holds lock and inner tries to lock it again. 
The lock keeps a **hold count**:

- After calling `outer()` → hold count = 1
- Inside `inner()` → hold count = 2
- Exiting `inner()` → hold count = 1
- Exiting `outer()` → hold count = 0 (lock fully released)

 Example with counter - 

```java
package code.data;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class CounterRe {
    public final Lock lock = new ReentrantLock();
    private int count = 0;
    public void increment(){
        lock.lock();
        try{
            count++;
        }finally{
            lock.unlock();
        }
    }

    public int getCount(){
        return count;
    }
}
```

Every lock in java implements the Lock interface which has two important methods 
- lock 
- unlock
Unlock must be done in finally as it will always run.

###  ReentrantReadWriteLock

In many applications, data is **read far more often** than it is written.  
A normal lock (like `synchronized` or `ReentrantLock`) blocks all threads even during read operations, even though multiple reads can safely happen together.  This reduces performance in read-heavy systems.

`ReentrantReadWriteLock` splits locking into **two separate locks**:

Read lock-

- Multiple threads can acquire **readLock() simultaneously**.
- Allowed only if **no thread holds writeLock()**.
- Safe when your operation does _not_ change data.

Write lock - 

- Only **one thread** can hold the write lock at a time.
- When write lock is held, **no one can read or write**.
- Ensures complete data consistency during modifications.

A lock is **reentrant** when the same thread can acquire it multiple times without deadlocking itself.  
Example: If a thread already holds the write lock and calls another method that tries to get the same lock again, it won’t block.  This is important for nested method calls.

Because multiple readers can proceed together, this lock drastically improves performance in scenarios like:

- Caches
- Database read-heavy operations
- In-memory datasets accessed by many threads

example

```java
import java.util.concurrent.locks.ReentrantReadWriteLock;

class DataStore {
    private int value;
    private final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    public int read() {
        lock.readLock().lock();
        try {
            return value;
        } finally {
            lock.readLock().unlock();
        }
    }

    public void write(int newValue) {
        lock.writeLock().lock();      
        try {
            value = newValue;
        } finally {
            lock.writeLock().unlock();
        }
    }
}
```

### Differnece between synchronized and Reentrant lock

You cannot try to acquire a synchronized lock with a timeout or check if the lock is available (i.e., no non‑blocking acquisition)

Reentrant gives you extra flexibility—for instance, with methods such as tryLock()(with or without a timeout) you can attempt to acquire the lock in a non‑blocking manner.

The readwrite lock is very good for the cached systems. 

## Semaphores

A semaphore is a synchronization primitive that maintains a count of permits.Threads can acquire these permits (decreasing the count) or release them (increasing the count). When a thread attempts to acquire a permit and none are available, the thread blocks until a permit becomes available or until it's interrupted.

Conceptually, a semaphore has two primary operations:
- aquire() - Obtains a permit, blocking if necessary untill one becomes available
- release(): Returns a permit to the semaphore

### Types - 

### Binary semaphore

 A binary semaphore has only two states (0 or 1 permit) and is mainly used to enforce mutual exclusion, similar to a mutex or lock.

```java
package code.data;

import java.util.concurrent.Semaphore;

public class SemaphoreCnt {

    private final Semaphore semaphore = new Semaphore(1);
    private int count = 0;
    
    public void increment(){
        try{
            semaphore.acquire();
            count++;
        }catch(InterruptedException e){
            System.out.println("ERROR");
        }finally{
            semaphore.release();
        }
    }

    public int getCount(){
        try{
            semaphore.acquire();
            return count;
        }catch(InterruptedException e){
            System.out.println("ERROR");
        }finally{
            semaphore.release();
        }
        return count;
    }
}
```

There is another semaphore counting semaphore where we have more then one permit count

```java
private final Semaphore semaphore = new Semaphore(5);
```

Usages of semaphore- 

- Managing pool of resources such as database connections or pools of files. 
- Implementing producer consumer solution
- Enforcing mutual exclusion - Binary semaphore

Basic difference between lock and semaphore

A Lock allows only one thread to access a resource at a time (mutual exclusion), while a Semaphore can allow a specified number of threads to access resources concurrently. A Lock is owned by a specific thread that must release it, whereas Semaphore permits can be acquired and released by different threads. Locks support multiple condition variables, while Semaphores work on a simpler permit-based model.

In Java's Semaphore implementation, calling release() without a prior acquire() is perfectly legal. It simply increases the permit count beyond its initial value. This behavior can be useful in certain scenarios, such as dynamically increasing the number of available resources. However, this can lead to unexpected behavior if not managed carefully, as it might allow more concurrent access than originally intended.

Barrier synchronization - 

A barrier ensures that no thread can proceed past a certain point until all threads have reached that point. Here's how to implement it with semaphores:

```java
    private final Semaphore mutex = new Semaphore(1);
    private final Semaphore barrier = new Semaphore(0);
```

```java
public void await() throws InterruptedException {

    mutex.acquire();       // 1
    count--;               // 2

    if (count == 0) {      // 3
        barrier.release(parties - 1);   // 4
        count = parties;                // 5
        mutex.release();                // 6
    } else {
        mutex.release();                // 7
        barrier.acquire();              // 8
    }
}
```

This function can be called at barrier to stop the execution of all the threads at that time. mutex is first aquired so that only one thread can update the counter which counts how many threads are reamining to reach barrier. Initially it starts from number of threads and reduces one by one as more number of people reach. Observe that for all the threads other than last else will be run and thread will wait at that point due to aquire. aquire starts with 0. So no body will be able to move forward until last one arrives and it will be the one to allow others. parties-1 is released as last thread is not waiting while others are waiting. 

Read write lock using semaphores-

```java
private int readerCount = 0;
private final Semaphore mutex = new Semaphore(1);
private final Semaphore writeLock = new Semaphore(1);

private void lockRead(){
	mutex.aquire();
	readerCount++;
	
	if(readerCount==1){ // lock to read mode if first reader
		writeLock.aquire();
	}
	
	mutex.release();
}

public void unlockRead() throws InterrruptException{
	
	mutex.aquire();
	readerCount--;
	
	if(readerCount==0){ // unlock read mode if no reader
		writeLock.relase();
	}
	mutex.release();
}

public void unlockWrite() {
	writeLock.release();
}

public void lockWrite(){
	writeLock.aquire();
}
```

### Reentrant lock internals 

`ReentrantLock` (from `java.util.concurrent.locks`) is an **explicit mutual exclusion lock**.

- Exclusive access to a critical section
- Reentrant behavior → the same thread can acquire it multiple times without deadlocking.

Reentrant lock is not a spin lock.

**Spin lock**: Thread repeatedly checks a variable in a loop (spins) until it acquires the lock. CPU cycles are consumed during spinning.

**ReentrantLock default implementation** (non-fair lock):
- Internally uses **AQS (AbstractQueuedSynchronizer)**
- Threads that fail to acquire the lock are **blocked** and put into a **wait queue** (`LockSupport.park()`)
- They do **not spin continuously** in a tight loop
- This makes it **blocking**, not a pure spin lock

Some minimal spinning may happen in **non-fair locks** in Java **for very short durations** (to reduce context switching), but this is **not the main mechanism**.  
So ReentrantLock is **mostly blocking**, not a classical spin lock.

|Feature|ReentrantLock|Spin Lock|
|---|---|---|
|Lock acquisition|Blocking (park/unpark)|Busy-wait loop|
|CPU usage|Low while waiting|High while waiting|
|Fairness|Optional|Usually not fair|
|Reentrancy|✅|Usually ❌|
|Condition support|✅|❌|
|Suitable for|Contention-heavy cases|Very short critical sections, low threads|

