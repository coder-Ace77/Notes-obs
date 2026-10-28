
---

## Blocking queue

A blocking queue has following characteristics - 

1. enqueue(ele) - Adds element to the front of queue. If queue is full calling thread should be blocked. 
2. dequeue() - returns the element at the end of queue and removes it. If queue is empty, calling thread is blocked untill queue is no longer full. 
3. size() - returns number of elements currently in queue. 

We will be using two semaphores to track number of empty and number of full elements also will be using concurrent datastructure to do the same. 

```java
class BlockingQueue{
	private Semaphore full;
	private Semaphore empty;
	private ConcurrentLinkedDeque<Integer> deque;
	
	public BlockingQueue(int capacity){
		full = new Semaphore(0);
		empty = new Semaphore(capacity);
		
		deque = new ConcurrentLinkedDeque<>();
	}
	
	public void enqueue(int ele) throws InterupttedException{
		empty.acquire();
		deque.addFirst(ele);
		full.release();
	}
	
	public int dequeue() throws InterupttedException{
		int res = -1;
		full.acquire();
		res = deque.pollLast();
		empty.release();
		return res;
	}
	
	public int size(){
		return deque.size();
	}
}
```

full blocks the deque operation and empty blocks the enquue if (full/empty). 