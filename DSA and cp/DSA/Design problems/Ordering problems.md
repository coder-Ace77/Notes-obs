
---

First we need to able to design the doublly linked list. It has specific ability to delete an arbitrary node at any given point additionally we can also add any node to the front or on back and also delete or retrieve from any point. With the help of this we can in `O(1)` move the general ode to front or back. 

Now one of the most powerfull paradigm is `DLL+MAP` because in production systems we need to maintain some kind of order. Now since linkedlist can be used to make inhernet ordering and map can be used to quickly access the values from `DLL`. 

### LRU cache 

Least recentrly used cache. 

```cpp
class Node{
    public:
    int key,val;
    Node* prev;
    Node* next;

    Node():prev(nullptr),next(nullptr){}
    Node(int k,int v):key(k),val(v),prev(nullptr),next(nullptr){}
};

class LinkedList{
    public:
    Node* head;
    Node* tail;

    LinkedList(){
        head=new Node();
        tail=new Node();
        head->next=tail;
        tail->prev=head;
    }

    void addToHead(Node* node){
        node->next=head->next;
        node->prev=head;
        head->next->prev=node;
        head->next=node;
    }

    void remove(Node* node){
        node->prev->next=node->next;
        node->next->prev=node->prev;
    }

    Node* pop(){
        Node* node=tail->prev;
        node->prev->next=tail;
        tail->prev=node->prev;
        return node;
    }

    void bringFront(Node* node){
        remove(node);
        addToHead(node);
    }
};

class LRUCache {
public:

    LinkedList* list;
    unordered_map<int,Node*> cache;
    int n;

    LRUCache(int capacity) {
        n=capacity;
        list = new LinkedList();
    }
    
    int get(int key){
        if(cache.find(key)==cache.end())return -1;
        Node* node = cache[key];
        list->bringFront(node);
        return node->val;
    }
    
    void put(int key, int value){
        if(cache.find(key)==cache.end()){
            if(cache.size()==n){
                Node* node = list->pop();
                cache.erase(node->key);
                delete node;
            }
            Node* node = new Node(key,value);
            cache[key]=node;
            list->addToHead(node);
        }else{
            Node* node = cache[key];
            node->val=value;
            list->bringFront(node);
        }
    }
};
```

### LFU  least frequently used cache

Note that for each key we maintain a map of what is the current freq of each key. Also maintain the lru map for each of freq. In this way when put comes we can put into some freq list and also maintain `LRU` if having same freq. 

```cpp
class Node{
    public:
    int key,val;
    Node* prev;
    Node* next;
    Node():prev(nullptr),next(nullptr){}
    Node(int k,int v):key(k),val(v),prev(nullptr),next(nullptr){}
};

class LinkedList{

    Node* head;
    Node* tail;
    int n=0;
    
    public:
    LinkedList(){
        head=new Node();
        tail=new Node();
        head->next=tail;
        tail->prev=head;
    }

    void add_front(Node* node){
        head->next->prev=node;
        node->next=head->next;
        node->prev=head;
        head->next=node;
        n++;
    }

    void remove(Node* node){
        node->prev->next=node->next;
        node->next->prev=node->prev;
        n--;
    }

    Node* pop_back(){
        Node* node=tail->prev;
        node->prev->next=tail;
        tail->prev=node->prev;
        n--;
        return node;
    }

    void bring_front(Node* node){
        remove(node);
        add_front(node);
    }

    int size(){
        return n;
    }
};

class LFUCache {
public:
    unordered_map<int, LinkedList*> lists; // frequency -> list of nodes
    unordered_map<int, int> keyFreq;       // key -> frequency
    unordered_map<int, Node*> keyNode;     // key -> Node pointer
    int capacity;
    int minFreq;

    LFUCache(int cap) {
        capacity = cap;
        minFreq = 0;
    }

    void updateFreq(Node* node) {
        int freq = keyFreq[node->key];
        lists[freq]->remove(node);
        
        // If the list for minFreq becomes empty, increment minFreq
        if (freq == minFreq && lists[freq]->size() == 0) {
            minFreq++;
        }

        keyFreq[node->key]++;
        int newFreq = keyFreq[node->key];
        
        if (lists.find(newFreq) == lists.end()) {
            lists[newFreq] = new LinkedList();
        }
        lists[newFreq]->add_front(node);
    }

    int get(int key) {
        if (keyNode.find(key) == keyNode.end() || capacity == 0) return -1;
        Node* node = keyNode[key];
        updateFreq(node);
        return node->val;
    }

    void put(int key, int value) {
        if (capacity <= 0) return;

        if (keyNode.find(key) != keyNode.end()) {
            Node* node = keyNode[key];
            node->val = value;
            updateFreq(node);
        } else {
            if (keyNode.size() >= capacity) {
                // Evict the LRU node from the minFreq list
                Node* nodeToEvict = lists[minFreq]->pop_back();
                keyNode.erase(nodeToEvict->key);
                keyFreq.erase(nodeToEvict->key);
                delete nodeToEvict;
            }

            // New node starts with freq 1
            Node* newNode = new Node(key, value);
            keyNode[key] = newNode;
            keyFreq[key] = 1;
            minFreq = 1; // Reset minFreq to 1 for new elements

            if (lists.find(1) == lists.end()) {
                lists[1] = new LinkedList();
            }
            lists[1]->add_front(newNode);
        }
    }
};
```

### All one

We need to support four operations-

- increase the freq of key
- descrease the freq of key
- find a key with min freq
- find a key with max freq

### Solution 

A very easy solution can be to maintain two datastrcutures - 

- map with key to freq count
- a set with entires - `<freq,key>`

Now we can easily handle both the cases. 

```cpp
void inc(string key) {
	if(mp.count(key)){
		st.erase({mp[key],key});
		
	}
	mp[key]++;
	st.insert({mp[key],key});
}

void dec(string key){
	if(mp.count(key)){
		st.erase({mp[key],key});
		
	}
	mp[key]--;
	if(mp[key]==0){
		mp.erase(key);
	}else{
		st.insert({mp[key],key});
	}
}

string getMaxKey(){
	if(st.size()==0)return "";
	return (*(st.rbegin())).second;
}

string getMinKey() {
	if(st.size()==0)return "";
	return (*(st.begin())).second;
}
```

However this gives us a datastructure which works on `O(logn)` in general not `O(1)`. 

To make it `O(1)` we will use a `linkedlist`.  Each node will store set of keys. And each node will be sorted based on freq. Meaning first node of the linkedlist will store the keys with freq 1 and then 2 and so on. 

When we increment a key, we first check if it exists in the hashmap. If the key is new, we look at the node after the dummy head. If that node does not have a frequency of 1, we create a new node for frequency 1. We add the key to this node and update the hashmap. If the key already exists, we find its current frequency node and check the next node, which shows the next higher frequency. If that next node is the tail or does not have the expected frequency, we create a new node with the increased frequency. We then move the key to the right node, remove it from the old node, and delete the old node if it becomes empty.

When we decrement a key, we first check if it is in the hashmap. If it is, we remove it from its current node. If the key’s frequency is greater than one, we check the previous node. If needed, we create a new node for the decreased frequency and add the key to the appropriate previous node, updating the hashmap. If the frequency is one, we remove the key from the hashmap completely.

To find the key with the maximum frequency, we return one of the keys from the last node in the list. For the minimum frequency key, we get a key from the first node after the dummy head. If there are no keys, we return an empty string.

### Insert Delete GetRandom O(1) - Duplicates allowed

The general idea is to use the hashmap + array. Hashmap to handle the `insert` ,`delete` and array to efficiently handle the `getRandom`.

One thing since we are having the removal we have to swap the ind for removal with some element and that needs to be done through the `unordered_set`. 

```cpp
class RandomizedCollection {
public:

    unordered_map<int,unordered_set<int>> mp;
    vector<int> arr;

    RandomizedCollection() {
        
    }
    
    bool insert(int val) {
        bool notPresent = mp[val].empty();
        mp[val].insert(arr.size());
        arr.push_back(val);
        return notPresent;
    }
    
    bool remove(int val){
        if (mp[val].empty()) return false;
        int ind = *mp[val].begin();
        mp[val].erase(ind);
        
        int lastVal = arr.back();
        int lastIdx = arr.size() - 1;
        if (ind != lastIdx){
            arr[ind] = lastVal;
            mp[lastVal].erase(lastIdx);
            mp[lastVal].insert(ind);
        }
        
        arr.pop_back();        
        if (mp[val].empty()) mp.erase(val);
        return true;
    }
    
    int getRandom(){
        int ind=std::rand() % arr.size();
        return arr[ind];
    }
};
```
