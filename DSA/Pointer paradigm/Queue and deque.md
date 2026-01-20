
---

A **queue** is a **linear data structure** that follows the **FIFO (First In, First Out)** principle — the first element added is the first one to be removed.
Queue enables two main functions. 

1. push(x) - insert element at the end. 
2. front() - Get the front element from the queue. 

Queue can be implemented using linked list with tailpointer. Queue is used extensively in 

1. Task scheduling 
2. bfs traversals

```cpp
#include <queue>
queue<int> q;
q.push(10);     // enqueue
q.pop();        // dequeue
q.front();      // access front
q.empty();      // check empty
```

Note that queues can be implemented using two ways - 
1. Array implementation
2. Linked list implementation
## Variation in queue

### Circular queue

A queue where last element points to the first. If we implement the queue using array implementation we can use of proximity caching but there is a problem that if the queue becomes full and even if the front is empty still we can not use that space. Cicular queue is bounded queue but can use the empty space in front as well. 

A circular queue has three things - 
1. An array `arr`
2. Two pointers `front` and `rear`. Intially front and rear both point to -1

Code

```cpp
#include <iostream>
using namespace std;

class CircularQueue {
    int *arr;
    int front, rear, size;

public:
    CircularQueue(int n) {
        size = n;
        arr = new int[size];
        front = rear = -1;
    }

    bool isFull() {
        return (front==(rear+1)%size);
    }

    bool isEmpty() {
        return (front==-1);
    }

    void enqueue(int val) {
        if (isFull()){
            cout<<"Queue is Full\n";
            return;
        }
        if (isEmpty()) front = 0;
        rear = (rear + 1) % size;
        arr[rear] = val;
    }

    void dequeue() {
        if (isEmpty()) {
            cout << "Queue is Empty\n";
            return;
        }
        cout << "Removed: " << arr[front] << endl;
        if (front == rear) front = rear = -1;
        else front = (front + 1) % size;
    }
};
```

### Deque

Also called double ended queue. It supports both the insertion and deletion at both ends. 

| Operation       | Description                                            |
| --------------- | ------------------------------------------------------ |
| `push_front(x)` | Insert element at the **front**                        |
| `push_back(x)`  | Insert element at the **rear**                         |
| `pop_front()`   | Remove element from the **front**                      |
| `pop_back()`    | Remove element from the **rear**                       |
| `front()`       | Access front element                                   |
| `back()`        | Access last element                                    |
| `isEmpty()`     | Check if deque is empty                                |
| `isFull()`      | Check if deque is full (for fixed-size implementation) |
Deque can be implemented using array or linked list manner. However cpp deque is not implemented as simple array or linked list. It is actually segmented dynamic array. Deque can act as both queue and stack. 

## Applications of deque

### LRU cache

Least recently used is essentailly a cache eviction policy here once the cache gets full we will remove the items that has not been used since earliest. 

- `LRUCache(int capacity)` Initialize the LRU cache with **positive** size `capacity`.
- `int get(int key)` Return the value of the `key` if the key exists, otherwise return `-1`.
- `void put(int key, int value)` Update the value of the `key` if the `key` exists. Otherwise, add the `key-value` pair to the cache. If the number of keys exceeds the `capacity` from this operation, **evict** the least recently used key.

Solution

```cpp
class LRUCache {
    int capacity;
    list<pair<int, int>> dq; // {key, value}
    unordered_map<int, list<pair<int, int>>::iterator> cache;

public:
    LRUCache(int cap) : capacity(cap) {}

    int get(int key) {
        if (cache.find(key) == cache.end())
            return -1;

        // Move accessed item to front
        auto it = cache[key];
        int value = it->second;
        dq.erase(it);
        dq.push_front({key, value});
        cache[key] = dq.begin();
        return value;
    }

    void put(int key, int value) {
        if (cache.find(key) != cache.end()) {
            dq.erase(cache[key]);
        } else if (dq.size() == capacity) {
            // remove LRU item (from back)
            auto last = dq.back();
            cache.erase(last.first);
            dq.pop_back();
        }
        dq.push_front({key, value});
        cache[key] = dq.begin();
    }
};
```

In c++ list provides the double linked list representation. Here we can insert at front, remove from front etc. Remember that we can delete the element in double ended queue in `O(1)`. 

### LFU cache

LFU (Least Frequently Used) is a cache eviction algorithm that removes the items used the least often. Unlike LRU (Least Recently Used), which focuses on _recency_, LFU focuses on _popularity_.

To achieve an efficient **O(1)** time complexity for both `get` and `put` operations, we need to track two things simultaneously:

1. **The Value:** The actual data associated with a key.
2. **The Frequency:** How many times that key has been accessed.

If two or more keys have the same minimum frequency, the algorithm uses **LRU** as a tie-breaker. It evicts the item that was accessed longest ago within that specific frequency group.

The implementation has three things - 

- **Key Table:** A hash map storing `key -> (value, frequency, pointer to node)`.
- **Frequency Table:** A hash map storing `frequency -> Doubly Linked List of keys`. This allows us to maintain the LRU order for keys with the same frequency.
- **minFreq:** An integer tracking the current minimum frequency in the cache to enable O(1) eviction.

The `get(key)` Operation

1. If the key doesn't exist, return -1.
2. If it exists:
    - Find the current frequency of the key.
    - Move the key from the current frequency's list to the `frequency + 1` list.
    - Update `minFreq` if the old frequency list is now empty and was the minimum.
    - Return the value.

 The `put(key, value)` Operation

1. If the key exists: Update the value and call the `get` logic to increment frequency.
2. If the key is new:
    - If the cache is full: Evict the last node (LRU) from the `minFreq` list and remove it from the Key Table.
    - Insert the new key with `frequency = 1`.
    - Set `minFreq = 1`.

```cpp
#include <unordered_map>
#include <list>

using namespace std;

struct Node {
    int key, value, freq;
    Node(int k, int v, int f) : key(k), value(v), freq(f) {}
};

class LFUCache {
    int capacity;
    int minFreq;
    unordered_map<int, list<Node>::iterator> keyMap; // key -> iterator in freqMap
    unordered_map<int, list<Node>> freqMap;         // frequency -> list of Nodes

    void updateFrequency(int key) {
        auto it = keyMap[key];
        int f = it->freq;
        int v = it->value;
        
        // Remove from current frequency list
        freqMap[f].erase(it);
        if (freqMap[f].empty()) {
            freqMap.erase(f);
            if (minFreq == f) minFreq++;
        }
        
        // Add to frequency + 1 list
        freqMap[f + 1].push_front(Node(key, v, f + 1));
        keyMap[key] = freqMap[f + 1].begin();
    }

public:
    LFUCache(int cap) : capacity(cap), minFreq(0) {}

    int get(int key) {
        if (keyMap.find(key) == keyMap.end()) return -1;
        int val = keyMap[key]->value;
        updateFrequency(key);
        return val;
    }

    void put(int key, int value) {
        if (capacity <= 0) return;

        if (keyMap.find(key) != keyMap.end()) {
            keyMap[key]->value = value;
            updateFrequency(key);
            return;
        }

        if (keyMap.size() >= capacity) {
            // Evict LRU element from minFreq list
            int keyToEvict = freqMap[minFreq].back().key;
            freqMap[minFreq].pop_back();
            if (freqMap[minFreq].empty()) freqMap.erase(minFreq);
            keyMap.erase(keyToEvict);
        }

        // Insert new element
        minFreq = 1;
        freqMap[1].push_front(Node(key, value, 1));
        keyMap[key] = freqMap[1].begin();
    }
};
```

### Monotonic queue

Monotonic queue is a special queue that maintains the elements in sorted order. Elements are inserted at one end but removed from other but we also maintain a special order (inc/desc)

A **Monotonic Queue** maintains the following invariant:
- For a **Monotonic Increasing Queue**, the elements **inside** are always in **non-decreasing order**.
- For a **Monotonic Decreasing Queue**, the elements **inside** are always in **non-increasing order**.

This means - 
- The **front** of the queue always holds the **minimum (or maximum)** element of the current window.
- As you insert elements, you **remove those from the back** that violate the order property.

#### Sliding window maximum

You are given an array of integers `nums`, there is a sliding window of size `k` which is moving from the very left of the array to the very right. You can only see the `k` numbers in the window. Each time the sliding window moves right by one position.

Solution: 

At first window we enque the elements index which makes our work a little easier. Because when reaching a new window we can check if we have to remove the first element or not. This can be done really efficiently. 

```cpp
vector<int> maxSlidingWindow(vector<int>& arr, int k) {
	deque<int> dq;
	int n = arr.size();
	for(int i=0;i<k;i++){
		while(!dq.empty() && arr[dq.back()]<arr[i]){
			dq.pop_back();
		}
		dq.push_back(i);
	}

	vector<int> ans;
	ans.push_back(arr[dq.front()]);

	for(int i=k;i<n;i++){
		if(dq.front()==i-k)dq.pop_front();
		while(!dq.empty() && arr[dq.back()]<arr[i]){
			dq.pop_back();
		}
		dq.push_back(i);
		ans.push_back(arr[dq.front()]);
	}
	return ans;
}
```

### Maintianing the real time data

Sometimes we want to maintain the real time data. Now our choice of data structure depends heavily on the aggregate we are required to maintained. For instance we may be interested in finding the aggregate 

1. sum - prefix sums if no updates 
2. min/max - monotonic deque
3. median - two prioirty queues. In this pattern we maintain two pq. One which will be storing the top n/2 highest elements. And second which will be storing the bottom(smaller) n/2 elements. So the top of both pqs tells us about the middle most elements always. We try to insert the element in one of queues. Then we also do an operation called balancing such that any pq can not have more than one element from other. Thus using this technique we can calculate the median by boundry elements. 

```cpp
class MedianFinder {
    priority_queue<int> maxHeap; // left half (max heap)
    priority_queue<int, vector<int>, greater<int>> minHeap; // right half (min heap)

public:
    void addNum(int num) {
        if (maxHeap.empty() || num <= maxHeap.top())
            maxHeap.push(num);
        else
            minHeap.push(num);

        // Rebalance
        if (maxHeap.size() > minHeap.size() + 1) {
            minHeap.push(maxHeap.top());
            maxHeap.pop();
        } else if (minHeap.size() > maxHeap.size()) {
            maxHeap.push(minHeap.top());
            minHeap.pop();
        }
    }

    double findMedian() {
        if (maxHeap.size() == minHeap.size())
            return (maxHeap.top() + minHeap.top()) / 2.0;
        else
            return maxHeap.top();
    }
};
```

## Queue using two stacks

Idea is to maintain two stacks one holding the data(instack) other outStack which is used while outputting element. 

1. When we push - We simply push into the inStack 
2. When we pop/peek -  
	1. If outstack is empty- move all elements of instack to outstack which reverses the order. 
	2. Then pop/peek from outstack. 

```cpp
class MyQueue {
    stack<int> inStack;
    stack<int> outStack;

    void transfer() {
        // Move elements only if outStack is empty
        if (outStack.empty()) {
            while (!inStack.empty()) {
                outStack.push(inStack.top());
                inStack.pop();
            }
        }
    }

public:
    void push(int x) {
        inStack.push(x);
    }

    int pop() {
        transfer(); // ensure outStack has elements in correct order
        int val = outStack.top();
        outStack.pop();
        return val;
    }

    int peek() {
        transfer();
        return outStack.top();
    }

    bool empty() {
        return inStack.empty() && outStack.empty();
    }
};
```


### Circular queue

This circular queue uses a fixed-size array and treats it as _circular_ by wrapping indices using modulo arithmetic. When the queue reaches the end of the array, it logically continues from the beginning instead of shifting elements, which keeps every operation in constant time.

The queue stores elements in a vector `a` of size `n`, where `n` is the maximum capacity. The variable `f` represents the index of the **front element**, `r` represents the index of the **rear element**, and `sz` keeps track of how many elements are currently present. Initially, the queue is empty, so `sz` is `0`, `f` is set to `0`, and `r` is set to `-1` to indicate that no element has been inserted yet.

```cpp
class MyCircularQueue {
public:
    int n, f, r, sz;
    vector<int> a;

    MyCircularQueue(int k) {
        n = k;
        a.resize(k);
        f = 0;
        r = -1;
        sz = 0;
    }
    
    bool enQueue(int v) {
        if (sz == n) return false;
        r = (r + 1) % n;
        a[r] = v;
        sz++;
        return true;
    }
    
    bool deQueue() {
        if (sz == 0) return false;
        f = (f + 1) % n;
        sz--;
        return true;
    }
    
    int Front() {
        if (sz == 0) return -1;
        return a[f];
    }
    
    int Rear() {
        if (sz == 0) return -1;
        return a[r];
    }
    
    bool isEmpty() {
        return sz == 0;
    }
    
    bool isFull() {
        return sz == n;
    }
};

```

When `enQueue(value)` is called, the first check ensures the queue is not full by comparing `sz` with `n`. If space is available, the rear index `r` is moved forward by one position using `(r + 1) % n`. The modulo operation is the key part—it wraps the index back to `0` when it reaches the end of the array, giving the structure its circular behavior. The value is then placed at `a[r]`, and `sz` is incremented to reflect the successful insertion.

The `deQueue()` operation removes the front element. It first checks if the queue is empty by testing whether `sz` is `0`. If not empty, the front index `f` is advanced using `(f + 1) % n`, again using modulo to wrap around when necessary. The size `sz` is decremented, effectively discarding the previous front element without physically deleting it, since future insertions will overwrite old values.

The `Front()` operation simply returns the element at index `f`, which always points to the current front of the queue. Similarly, `Rear()` returns the element at index `r`, which always points to the most recently inserted element. In both cases, if the queue is empty, `-1` is returned to indicate that no valid element exists.

The helper functions `isEmpty()` and `isFull()` rely entirely on the `sz` variable. If `sz` is `0`, the queue is empty, and if `sz` equals `n`, the queue is full. This avoids ambiguous conditions that arise when front and rear indices overlap, making the implementation simple and reliable.
