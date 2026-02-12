
---

`Future` represents the result of an asynchronous computation, but it is **very limited**.

Key features - 

- Once you submit a task using `ExecutorService.submit()`, you get a `Future`.
- You **cannot** manually complete it.
- You **cannot chain** tasks.
- You **cannot attach callbacks**.
- You **cannot combine** multiple Futures.
- To get result → `future.get()` → **blocking call**.
- To check completion → `future.isDone()`.

A **Future** is like taking a token at a repair shop.  
You must **keep checking** or **stand and wait** for work to finish.

example - 

```java
ExecutorService executor = Executor.newSingleThreadExecutor();
Future<Integer> future = executor.submit(()->10+20);
Integer result = future.get(); // blocks untill ready
executor.submit();
```

We can check if done - 

```java
if(future.isDone()){
	// done
}else{
	// do something
}

// we can cancel the task as well

boolean cancelled = future.cancel(true);
```

When you call future.cancel(true), it sends an interrupt signal to the thread running the task.
▪ But Java doesn’t forcefully stop a thread — it just sets the interrupted flag.
▪ So your task must periodically check if it has been interrupted using Thread.interrupted().



### Completable future

### **Advantages over Future**

- Can be **completed manually**.
- Supports **callbacks**.
- Supports **chaining** (`thenApply`, `thenAccept`, `thenCompose`, `thenCombine`).
- Supports **non-blocking** execution.
- Supports **parallel pipelines**.
- Works well with the **ForkJoinPool**.

Example - 

```java
CompletableFuture.supplyAsync(() -> slowTask())
    .thenApply(result -> result + 10)
    .thenAccept(System.out::println);
```

Manual completion 

```java
CompletableFuture<String> cf = new CompletableFuture<>();
cf.complete("Done");
```

```java
CompletableFuture<Integer> a = CompletableFuture.supplyAsync(() -> 10);
CompletableFuture<Integer> b = CompletableFuture.supplyAsync(() -> 20);

CompletableFuture<Integer> sum =
    a.thenCombine(b, (x, y) -> x + y);
```

Chaining 

```java
CompletableFuture.supplyAsync(() -> userId())
    .thenCompose(id -> fetchUserData(id));
```

With completable future we don't work with multithreading directly instead we register some task using `CompletableFuture.supplyAsync(Task)`. Now this returns a completable future on this completable future we can again chain the callback using `.thenAccpt()` this accept function will accpt lambda like - `(resut->)`. And we can even chain them togather.

The supplyAsync call initiates the asynchronous task without blocking the main thread.

• The thenAccept callback is registered to execute once the task is complete, printing both the result and the follow-up message.
• Meanwhile, the main thread can perform other work (as shown by the extra print statement) without waiting for the asynchronous result.
• A simple Thread.sleep at the end ensures the application doesn’t terminate before the asynchronous work completes.

Basic api

```java
// use get as earlier
CompletableFuture<String> future = CompletableFuture.supplyAsync(()->"Hello");

try{
	String res = future.get(); // blocks
}catch(Exception e){
	
}

// we can manually complete the completable future

future.complete('Hello world');
```

Use When: 
• You want to complete a future manually (e.g., timeout fallback, mock).
• If the future is already completed, complete() will return false.

```java
// isDone checks for future to get completed
while(!future.isDone()){
	// waiting
}
```

Transforming results by chaining

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(()->{
	// do something and return
}).thenApply((res)->res+"world")
.thenApplyAsync(s->{
	return s+"!";
})
```

We can combine two futures together.

```java
CompletableFuture<String> combined = future1.thenCombine(future2,(res1,res2)->{

});
```

We can also wait for all the futures to get complete

```java
CompletableFuture<Void> allOf = CompletableFuture.allof(future1,future2);
allOf.thenRun(()->{
	// result
});
```

To handle errors we use `exceptionally` or to `handle` to handle both schenarios. 

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(()->{

}).expectionally(ex->{

});

CompletableFuture<String> handled = CompletableFuture.supplyAsync(()->{

}).handle((result,ex)->{
	if(ex!=null){
		return "Handled error:"+ex.getMessage();
	}
	return "Handled Success"+result;
}); // this again returns the future
```