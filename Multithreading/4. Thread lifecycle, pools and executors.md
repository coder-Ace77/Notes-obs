
---

![[Pasted image 20251127110423.png]]

In java thread api there are following states in enum - 

- NEW - Just object is made and start not called
- RUNNABLE - Either ready for execution or waiting for CPU allocation. Thread is in RUNNABLE when start() is called. 
- BLOCKED - Thread is temporary inactive while waiting to acuire lock usually happens when thread is trying to enter a synchronized block/method already locked by another thread.
- WAITING - Thraed is waiting infinitely for another thread to perform a specific action , enters via method like Object.wait(timeout) , Thread.join() etc. 
- TIMED_WAITING - When thread is suspended via sleep(timeout) so here thread is waiting for some time
- TERMINATED - Finally when thread is dead.

Running is also indicated as RUNNABLE which is state (theoritical). 

## Thread pools

Thread pools are a managed collection of reusable threads designed to execute tasks concurrently. They offer significant advantages in resource management, performance, and application stability.

A thread pool is essentially a collection of worker threads that are created once and reused for executing many tasks.This avoids the overhead of repeatedly creating and destroying threads, which is expensive in terms of memory, system calls, and CPU scheduling. Instead of spawning a new thread for each job, the task is submitted to the pool, placed in a queue, and executed by whichever worker thread becomes free.

In Java, thread pools are implemented through the **Executor Framework**, mainly using the `ExecutorService` interface and the `ThreadPoolExecutor` class. When you submit a task (Runnable or Callable) to an executor, it is queued internally.The thread pool decides whether to assign the task to an existing worker thread or create a new one, depending on the configuration. 
For example, a **fixed thread pool** maintains a constant number of threads—ideal for CPU-bound tasks. A **cached thread pool** can grow or shrink dynamically and is suitable for short-lived or bursty tasks. 
A **single-thread executor** processes tasks sequentially, guaranteeing order. The most flexible option is **ThreadPoolExecutor**, where you control the core pool size, maximum pool size, keep-alive time, queue type, and rejection policy, giving you full control over how the system handles overload or idle periods.

Inside a thread pool, when a task arrives, the executor checks whether the number of running threads is less than the core size; if yes, it starts a new thread. If the pool is full, the task is put into the work queue. If the queue is also full and the pool has not reached the maximum size, new threads are created up to the max.

### Fixed size thread pool

Example 1 - Fixed size thread pool - Creates fixed numbers of threads and has intenal queue for other tasks to wait. 
submit method can take the runnable/ callable object. 

```java
ExecutorService executorService = Executors.newFixedThreadPool(3);;
for(int i=0;i<50;i++){
	executorService.submit(new PrintThread()); // PrintThread is just a runnable
}
executorService.shutdown();
```

Thread pool life cycle

1. **Pool Creation**: When a thread pool is created, it may pre-create some threads (core threads) in the NEW state and immediately start them to RUNNABLE. 
2. Task Execution: When a task is submitted: 
	- An idle thread in the pool executes the task 
	- The thread's state changes according to task operations (RUNNABLE, RUNNING, BLOCKED, WAITING, etc.) 
	- After task completion, the thread returns to the pool (RUNNABLE state waiting for next task) 
3. Pool Shutdown: During shutdown, threads complete their current tasks and are eventually terminated.

A thread waiting **indefinitely** for a signal can cause a leak. Use **timeouts** to prevent this while acquiring locks or waiting on conditions.

```java
lock.wait(timeout);
```

Note that fixed thread pools is good for the cpu bound tasks. and predictable loads. 

### Cached thread pool

A pool that creates **new threads as needed** and **reuses idle threads** (threads die after 60 seconds idle).
This is usefull for the situations where we have IO bound tasks. So once the task is created it will be in blocked state mostly thus most of the threads will not be running at all the time on cpu and clearly will not harm our choices. 

```java
ExecutorService executor = Executors.newCachedThreadPool();

for (int i = 1; i <= 10; i++) {
    int finalI = i;
    executor.submit(() -> {
        System.out.println("Running " + finalI + " in " + Thread.currentThread().getName());
    });
}

executor.shutdown();
```

### Single thread executor

Only one thread running at all the times. 

Used for the event processing , logging and tasks that must not be running in parallel. 

```java
ExecutorService executor = Executors.newSingleThreadExecutor();

executor.submit(() -> System.out.println("Task 1"));
executor.submit(() -> System.out.println("Task 2")); // runs after task 1

executor.shutdown();
```

### ScheduledExecutor service

It adds the task scheduling capabilities. 

```java
ScheduledExecutorService sc = Executors.newScheduledThreadPool(1);
scheduler.scheduleAtFixedRate(runnable,0,2,TimeUnits.SECONDS);
```
### Work-Stealing Pool

A **ForkJoinPool** that dynamically steals work from other threads. used with divide and conquer algorithms, parallel streams and cpu intensive parallel tasks. 

Finally we have custom Thread pool executor which has full control over - 

- Core threads
- Max threads
- Queue
- Keep-alive time
- Rejection policy

With fixed pools it becomes crucial to have balanced usage and throughput. If queue is too large we will get delay in execution and with small queue we will get frequent task rejection. 

Notes - 

-  After task completion, the thread doesn't terminate but returns to the pool, ready to execute another task. This reuse eliminates the overhead of constantly creating and destroying threads.
-  The queue stores tasks when all core threads are busy. A larger queue can handle more pending tasks but consumes more memory. If the queue reaches capacity, the pool creates additional threads up to maxPoolSize. If maxPoolSize is reached and the queue is full, the rejection policy is applied.
-  shutdown() initiates a graceful shutdown, allowing queued tasks to complete but not accepting new tasks. shutdownNow() attempts to stop all executing tasks immediately and returns a list of tasks that were awaiting execution.
-  Yes, if the thread is interrupted during TIMED_WAITING, it can throw an InterruptedException and complete its run method, transitioning to TERMINATED state.
-  Thread starvation occurs when threads are unable to gain regular access to shared resources and make progress. Thread pools help prevent this by controlling the number of active threads and implementing fair scheduling policies.

Advanced points-

- Executor service allows us to have queing policies, scheduling and eviction policies. 

```java
ScheduledExecutorService sc = Ececutors.newScheduledThreadPool(1);
scheduler.schedule(runnable,3,TimeUnit.SECONDS);
```

Monitoring

```java
System.out.println("Active Threads: " + executor.getActiveCount());
System.out.println("Queued Tasks: " + executor.getQueue().size());
```

### Java executor interface and classes

The Java concurrency framework provides several key interfaces and classes for working with Thread Executors.
Most basic unit is `Executor` and is most basic form.

```java
public interface Executor {
    void execute(Runnable command);
}
```

ExecutorService is a subinterface of `Executor` and supports task submission

```java
ExecutorService service = Executors.newFixedThreadPool(3);

service.submit(runnable);
service.shutdown();
```

Then ThreadPoolExecutor which is an implementation of ExecutorService. It provides the full control over the core/max threads, queue type, keep-alive time, rejection policy. Great for performance-tuned, production-grade thread pool management.

```java
ThreadPoolExecutor customPool = new ThreadPoolExecutor(
	int corePoolSize,
	int maxPoolSize,
	long keepAliveTime,
	TimeUnit unit,
	BlockingQueue<Runnable> workQueue
);
```

### Methods of Executor service

#### Task submission methods

```java
executor.execute(runnable); // does not returns anything
```

Task runs asynchronously. No result is expected or tracked.

Submit

```java
ExecutorService es = new Executors.newSingleThreadExecutor();
Future<String> future = executor.submit(callable);
future.get(); // waits until reuslt is not available
```

▪ Returns a Future. You can block and get the result using future.get().

InvokeAll is used to run a Collection of callable tasks.

```java
List<Future<String>> res = executor.invokeAll(tasks); // tasks are the collections of callable
```

▪ Runs all tasks in parallel. Waits until all finish. You get a list of Futures.

`Note: We also have an overloaded method invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) that allows you to specify a timeout for the completion of all tasks.`
▪ If a task throws an exception, it stops future executions.

Schedule

```java
schedule(Runnable command, long delay, TimeUnit unit):
```

There are two kinds of shutdown

```java
void shutdown();
 
List<Runnable> shutdownNow(); // returns list of reamining tasks
```

Finally we have some executor factory methods to create executors. 

```java
ExecutorService fixedPool = Executors.newFixedThreadPool(nThreads);

ExecutorService singlePool = Executors.newSingleThreadExecutor();

ExecutorService cachedPool = Executors.newCachedThreadPool(nThreads);

ExecutorService scheduledPool = Executors.newScheduledThreadPool(corePoolSize);
```

Finally some notes about configuration properties about ThreadPoolExecutor on how it decides weather ot create new thread or put it in queue. 

- If fewer than corePoolSize threads are running, create a new thread.
- If corePoolSize or more threads are running, add the task to the queue.
- If the queue is full and fewer than maximumPoolSize threads are running, create a new thread. 
- If the queue is full and maximumPoolSize threads are running, reject the task.

If we don't explicitly shutdown the service the executor's threads will continue running, preventing the JVM from shutting down normally (unless they're daemon threads). This can cause memory leaks and resource exhaustion. Always call shutdown() or shutdownNow() when done with an executor.

Difference between scheduleAtFixedRate and scheduleWithFixedDelay

scheduleAtFixedRate attempts to execute tasks at a consistent rate regardless of how long each task takes (tasks might overlap if execution takes longer than the period). scheduleWithFixedDelay waits for the specified delay time after each task completes before starting the next execution.

Exception handling - Answer: For submit() methods, exceptions are stored in the returned Future and thrown when calling get().

