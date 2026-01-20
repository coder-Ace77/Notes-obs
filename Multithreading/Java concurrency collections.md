
---

## Concurrent hash maps

### Basic working

A normal `HashMap` is not thread-safe — multiple threads updating or resizing it simultaneously can corrupt the map or even cause infinite loops.  
`ConcurrentHashMap` solves this by allowing **safe concurrent reads and writes** without blocking all threads. It provides high performance in multi-threaded systems where many threads read and write frequently.

Like `HashMap`, it uses an array of buckets, where each bucket starts as a linked list of nodes.  
Each node stores: `hash`, `key`, `value`, `next`.

Instead of locking the entire map, CHM uses **CAS operations** to insert nodes.  
If a bucket is empty, it uses CAS to place a new node without locking.  
CAS ensures only one thread wins even if many try to write simultaneously

If two threads collide on the same bucket and need to modify it, Java uses **synchronized locking on that specific bucket node**, not the entire map.  
This makes locking very small and local, improving performance.

Reads are **lock-free**.  
Multiple threads can read the map without any locking, because updates are done atomically and visible safely (volatile fields + CAS).

### API

```java
map.put("name", "Adil");// puts a field
String name = map.get("name"); //lock free due to cas + volatile
map.remove("name"); 

map.putIfAbsent("count", 1); //atomic 

map.replace("status", "active");

map.replace("status", "active", "inactive"); // replace(key,oldval,newval) works atomically no lock

map.remove("token", "abc123"); // remove iff and only if

// atomic compute operations

map.compute("count", (k, v) -> v == null ? 1 : v + 1); //Updates a key atomically using a lambda.

map.computeIfAbsent("user1", k -> loadFromDB(k)); // updates if key is absent thread safe

map.forEach((k, v) -> System.out.println(k + ": " + v)); // safe even if updates are happening

String result = map.search(1, (k, v) -> v.equals("admin") ? k : null); // thrad safe

if (map.containsKey("user1")) { ... } // thread safe existance checks
```

size and isEmpty both are approximate in hashmap. Compute performs two actions atomically - 

1. It computes new value for a given key
2. Updates the returned value, or removes entry if function returns `null`.

Internally, ConcurrentHashMap finds the bucket for the key and then applies **per-bin locking**, meaning only that bucket is temporarily locked.  
Inside this locked section, it reads the old value, applies your function, and updates or deletes the node.  
No other thread can modify the same key during this time, making the whole update atomic and safe.

## CopyOnWriteArrayList

`CopyOnWriteArrayList` (from `java.util.concurrent`) is a **thread-safe variant of `ArrayList`**.  
It allows multiple threads to **read the list without locking**, while write operations (add, remove, set) create a **new copy of the underlying array**.

Key idea: **reads are fast and non-blocking, writes are expensive** because they involve copying the array.

Internally it creates volatile `Object[]` array.
When a thread calls `add()`, `remove()`, or `set()`, a **new array is created** with the updated elements.
The volatile reference is then **atomically updated** to point to the new array.

Since reads can be done directly and write is done on copied array no other thread can interfere. Volatile ensures that all threads see the latest array after a write.

| Method                      | Behavior                                                                     |
| --------------------------- | ---------------------------------------------------------------------------- |
| `add(E e)`                  | Appends element; copies array internally                                     |
| `add(int index, E e)`       | Inserts at index; copies array                                               |
| `set(int index, E element)` | Replaces element; copies array                                               |
| `remove(Object o)`          | Removes element; copies array                                                |
| `get(int index)`            | Fast, lock-free read                                                         |
| `iterator()`                | Returns **snapshot iterator**: does not reflect modifications after creation |
| `size()`                    | Reads from current array; lock-free                                          |
Iterator does **not throw `ConcurrentModificationException`**

```java
import java.util.concurrent.CopyOnWriteArrayList;

public class Demo {
    public static void main(String[] args) {
        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
        // Adding elements
        list.add("Apple");
        list.add("Banana");
        list.add("Cherry");
        // Reading elements (no locks)
        for (String fruit : list) {
            System.out.println(fruit);
        }
        // Adding element while iterating
        for (String fruit : list) {
            System.out.println(fruit);
            list.add("Date"); // Safe, iterator sees old snapshot
        }
        System.out.println(list); // All elements including "Date"
    }
}
```

## Concurrent linked list

A **Concurrent Linked List** is a **thread-safe linked list** that allows multiple threads to **insert, remove, and traverse** nodes concurrently without corrupting the structure.  
In Java, the main implementation is **`ConcurrentLinkedDeque`** and **`ConcurrentLinkedQueue`** from `java.util.concurrent`.  
These lists are **non-blocking**, meaning they do not use global locks for most operations, making them highly scalable in multi-threaded systems.

Built as a **singly-linked (queue) or doubly-linked (deque) list** of nodes.

```java
class Node<E> {
    volatile E item;
    volatile Node<E> next;
    volatile Node<E> prev; // for deque
}
```
Uses **`volatile` fields** and **CAS (Compare-And-Swap)** operations to safely update references in multi-threaded scenarios.
CAS ensures that only one thread can update a pointer at a time without using locks.

Non blocking insert -

- For a queue (`ConcurrentLinkedQueue`), adding an element appends a node at the tail.
- CAS is used to atomically update the tail pointer.
- Multiple threads can safely add nodes simultaneously.

Non blocking remove - 

- Removing the head uses CAS to move the head pointer to the next node.
- If another thread modifies the list at the same time, CAS retries until successful.

Iterators are **weakly consistent** -

- They reflect some, but not necessarily all, modifications since creation.
- Do not throw `ConcurrentModificationException`.

```java
import java.util.concurrent.ConcurrentLinkedDeque;

ConcurrentLinkedDeque<String> deque = new ConcurrentLinkedDeque<>();
deque.addFirst("First");
deque.addLast("Last");
deque.pollFirst(); // removes "First"
deque.pollLast();  // removes "Last"
```

## Blocking queue

Blocking queues provide a thread-safe mechanism for exchanging data between threads, typically in producer-consumer scenarios.

Unlike non-blocking concurrent collections like ConcurrentLinkedQueue, blocking queues explicitly coordinate between threads using one of two fundamental approaches:

### Intrinsic locking 

Most blocking queue implementations use locks (such as ReentrantLock) internally to synchronize access to the underlying data structure.

### Condition variables 

 Conditions are used in conjunction with locks to allow threads to wait efficiently when they cannot proceed (e.g., when the queue is full or empty) and to be notified when they can continue.When a producer thread tries to add an item to a full queue using put(), the thread is suspended until space becomes available.Similarly, when a consumer thread tries to take an item from an empty queue using take(), the thread waits until an item is added.

```java
BLockingQueue<Integer> q = new BlockingQueue<>();
// putting something in queue
new Thread.start(()->{
	q.put(100);
}).start();

// take blocks until the element is available
Integer ele = queue.take();
```

