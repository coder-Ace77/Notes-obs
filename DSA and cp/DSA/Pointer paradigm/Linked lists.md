
---

A **linked list** is a linear data structure where elements (called **nodes**) are connected using **pointers** instead of being stored in contiguous memory (like arrays).

```cpp
struct Node {
    int data;
    Node* next;
};
```

Types of linked lists:

1. Singly linked list(SLL): Each node has a data and a next pointer which points to the next node.

```r
Head → [data|next] → [data|next] → NULL
```

2. Doubly linked list:

```r
NULL ← [prev|data|next] ↔ [prev|data|next] ↔ NULL
```

3. Circular linked list: 

Here the next of last pointer points back to the first.

Now there are other types of linked list as well for an instance Header Linked list where we have a special dummy node at the start (it stores metadata). Then we have skip list which is a layered structure and allows logn search , sort and insert time.

- A skip list is essentially a **linked list with multiple layers**.
- The bottom layer is a simple sorted linked list of all elements.
- Higher layers contain "express lanes" (shortcuts) that skip over multiple elements, allowing faster traversal.
- Each element may appear in one or more layers, chosen randomly (usually with coin flips).

### Access in linked list:

Usually linked lists can store head pointers. Optionally they may also store tail. With the help of these two pointers one may do many opertions. It should be known that accessing an element whose pointer is given is a constant time operation however accessing an element whose address is unknown is done through sequencial insert. Neverthe less with the help of two pointers we can easily implement queue. 

Normal queue provides three operations:

1. q.front() - get the first element in queue
2. q.push() - add element to the end of queue
3. q.pop() - remove the first element

Without tail push is O(n) operation but with tail pointer it is O(1) operation because without tail we need to move to the last element. 

### Quick refernce for pointer element access in cpp:

Pointer is a variable that stores the memory address of another variable. Pointers variable again have a data type which is made by appending the * to original datatype. for instance int* etc. Note that * in case of pointers can come at two places.

1. with datatype to signify that this datatype is a pointer type.
2. With variable it comes as deferncing operator - which means to get the `value` of address you stored.

```cpp
int x = 10;
int* p = &x; // p stores the address of x
cout<<(*p)<<endl; // dereferncing operator
```

A pointer can also point to nothing called nullptr.

We can also allocate memory at run time in heap using new and delete operator.

```cpp
int* p = new int(42); // allocates memory for int and stores 42
cout << *p; // 42
delete p;   // free memory
```

Or we can go with advanced datatypes

```cpp
struct Node {
    int val;
    Node* next;
};

Node* head = new Node{10, nullptr}; // first node with value 10
head->next = new Node{20, nullptr}; // link second node
```

members of composite types in a pointer variable can be accessed using `->` operator

Traversing the linked list:

```cpp
Node* cur = head;
while(cur != nullptr){
	cout << cur->val << " ";
	cur = cur->next;
}
```

Also whenever pointer is declared it stores the garbage address(whatever bits happen to be in that memory location).

Note: NULL is historically a pointer from C language and defined as macro 
```c
#define NULL 0
```

So cpp NULL is just an integer value 0. nullptr on the other hand is a keyword, and not a micro and has own distinct type `std::nullptr_t`.

#### Linked list basic opertions:

#### Insertion:

we create a new node and move head to this node. `O(1)` operation.

```cpp
void insertAtHead(Node*& head, int v) {
    Node* n = new Node(v);
    n->next = head;
    head = n;
}
```

Insertion at tail we need to move till the last pointer if tail is not present. If tail ptr is there then its faster.

```cpp
Node* temp = head;
while (temp->next) temp = temp->next;
```

```cpp
Node* temp = new Node(10);
tail->next=temp;
tail=temp;
```

Insertion at given position: 

Here we can have many variations for an instance suppose the actual ptr is given after whih we have to insert. Then insertion is `O(1)`. Because the node after which we have to insert is given and we can add the node after it.

```cpp
// suppose curr is ptr after which we have to add
Node* next = curr->next;
Node* temp = new Node(10); // new node to be inserted
curr->next=temp;
temp->next = next;
```

Traversals of LL:

They can be done using the iterative method and also using recursive method. 

#### Deletion:

Deletion of three types of nodes is prevalent.

1. Begining:

```cpp
Node* temp = head;
head = head->next;
delete temp;
```

2. End 

Again if no tail we have to traverse otherwise we can directly delete.

3. You’re given a **pointer to the node to delete**, but **not the head pointer**.

The trick is to copy the data of next node to current node and delete the the next node.

```cpp
void deleteNode(Node* node) {
    if (!node || !node->next) return;  // cannot delete last node
    Node* temp = node->next;
    node->val = temp->val;
    node->next = temp->next;
    delete temp;
}
```

### Doubly linked list:

Deletion in doubly LL is much faster O(1) due to  presence of a prev pointer in each node. So deletion is O(1) in all cases where pointer is known. 

```cpp
void deleteAtHead(Node*& head) {
    if (!head) return;
    Node* temp = head;
    head = head->next;
    if (head) head->prev = nullptr;
    delete temp;
}
```

```cpp
void deleteAtTail(Node*& head) {
    if (!head) return;
    if (!head->next) { // only one node
        delete head;
        head = nullptr;
        return;
    }
    Node* temp = head;
    while (temp->next) temp = temp->next;
    temp->prev->next = nullptr;
    delete temp;
}
```

#### Reversing a LL:

At any moment of point we will be having three consecutive nodes into consideration.
prev , curr , next -> linked list upto prev is already reversed.

Now we will update the curr->next pointer to point to prev and curr and prev both are moved. 
Observe that at end curr will be null and we will return prev.

```cpp
Node* reverseLL(Node* head) {
    Node* prev = nullptr;
    Node* curr = head;
    Node* next = nullptr;

    while (curr) {
        next = curr->next;   // store next
        curr->next = prev;   // reverse link
        prev = curr;         // move prev forward
        curr = next;         // move curr forward
    }
    return prev;  // new head
}
```

One thing to remember is that at the end curr is null and prev points to the last element.
#### Reversing sub linked list:

Given the `head` of a singly linked list and two integers `left` and `right` where `left <= right`, reverse the nodes of the list from position `left` to position `right`, and return _the reversed list_. 

The inner idea is same of reversing a ll but there are two cases waether left=1 or it starts after some number. The idea is actually to perform 4 steps.

1. First find the just before left and find the node just after right. 
2. Reverse all the nodes in between 
3. Finally we have to reverse the pointers. 
4. And we need to handle the edge case.

```cpp
class Solution {
public:
    ListNode* reverseBetween(ListNode* head, int left, int right) {
        if (!head || left == right) return head;

        ListNode* prev = nullptr;
        ListNode* curr = head;

        // Step 1: move curr to the "left"-th node
        for (int i = 1; i < left; i++) {
            prev = curr;
            curr = curr->next;
        }

        ListNode* conn = prev;   // node before 'left'
        ListNode* tail = curr;   // 'left' node, will become tail after reversal
        ListNode* nxt = nullptr;
        ListNode* prevRev = nullptr;

        // Step 2: reverse from left to right
        for (int i = left; i <= right; i++) {
            nxt = curr->next;
            curr->next = prevRev;
            prevRev = curr;
            curr = nxt;
        }

        // Step 3: reconnect
        if (conn) conn->next = prevRev;  // connect part before left to reversed head
        else head = prevRev;             // if left == 1, new head is prevRev

        tail->next = curr; // connect tail of reversed part to remainder

        return head;
    }
};
```

Now we can also handle the edge case by making a dummy node. Then we don't need to worry about edge node as theres is always a first node dummy.

```cpp
class Solution {
public:
    ListNode* reverseBetween(ListNode* head, int left, int right) {
        if (!head || left == right) return head;

        ListNode dummy(0);
        dummy.next = head;
        ListNode* prev = &dummy;

        for (int i = 1; i < left; i++) { // prev reaches the pos before left
            prev = prev->next;
        }

        ListNode* curr = prev->next; 
        ListNode* nxt = nullptr;
        ListNode* prevRev = nullptr;

        for (int i = left; i <= right; i++) {
            nxt = curr->next;
            curr->next = prevRev;
            prevRev = curr;
            curr = nxt;
        }

        prev->next->next = curr;
        prev->next = prevRev; // prevRev points to last 

        return dummy.next;
    }
};
```

Revering lists in the group of k- 

Again the idea is simple we can easily group the nodes together in groups of k. Then we can reverse the group using earlier technique. And then doing the reversal. 

```cpp
ListNode* rev(ListNode* head){
	ListNode* curr = head;
	ListNode* prev = nullptr;
	ListNode* next = nullptr;
	while(curr){
		next = curr->next;
		curr->next = prev;
		prev = curr;
		curr = next;
	}
	return prev;
}

ListNode* findKth(ListNode* node,int k){
	while(node!=nullptr && k>1){
		node = node->next;
		k--;
	}
	return node;
}

ListNode* reverseKGroup(ListNode* head, int k) {
	ListNode* temp = head;
	ListNode* prevLast = nullptr;
	while(temp!=nullptr){
		ListNode* kth = findKth(temp,k);
		if(kth == nullptr){
			if(prevLast!=nullptr)prevLast->next = temp;
			break;
		}
		ListNode* next = kth->next;
		kth->next = nullptr;
		ListNode* reverseHead = rev(temp);
		if(temp==head){
			head = reverseHead;
		}else{
			prevLast->next = reverseHead;
		}
		prevLast = temp;
		temp = next;
	}
	return head;
}
```
#### finding cycle in a linked list:

Given `head`, the head of a linked list, determine if the linked list has a cycle in it.

The idea is pretty simple if we were allowed to used visited set or map. In that case if we reach any node which we already have traversed then we find the cycle. But there is more tricky way to do that. Using slow and fast pointers.

Proof:

The idea is that we will have two pointers fast and slow fast will move two jumps at a time and slow one jump. I cycle exist they will meet.

If there is no cycle fast will reach nullptr however if cycle exist both will start moving inside the cycle.  The important point is when slow pointer enters the cycle if will have atmost k distance between it and fast pointer. Note that in each step the distance between slow and fast will reduce by 1 meaning in atmost k steps slow and fast will meet together.

```cpp
class Solution {
public:
	bool hasCycle(ListNode *head) {
		if (!head || !head->next) return false;
		ListNode* slow = head;
		ListNode* fast = head;
		while (fast && fast->next) {
			slow = slow->next;
			fast = fast->next->next;
			if (slow == fast) return true;
		}
		return false;
	}
};
```

Merge two sorted list:

We can merge two sorted list without any extra space by using nodes of earlier two lined list. The merger is pretty straight forward if one uses the dummy node.

```cpp
class Solution {
	public:
	ListNode* mergeTwoLists(ListNode* list1, ListNode* list2){
		ListNode* head = new ListNode(0);
		ListNode* curr = head;
		ListNode* p1 = list1;
		ListNode* p2 = list2;
		
		while(p1 && p2){
			if(p1->val<=p2->val){
				curr->next = p1;
				p1=p1->next;
			}else{
				curr->next = p2;
				p2 = p2->next;
			}
			curr = curr->next;
		}
		while(p1){
			curr->next = p1;
			p1=p1->next;
			curr = curr->next;	
		}
		while(p2){
			curr->next = p2;
			p2 = p2->next;
			curr = curr->next;
		}	
		return head->next;
	}
};
```

### Copying node with random pointer:

linked list of length `n` is given such that each node contains an additional random pointer, which could point to any node in the list, or `null`.
Construct a [**deep copy**](https://en.wikipedia.org/wiki/Object_copying#Deep_copy) of the list.

Creating the simple deep copy is really easy reason being you just need to add the nodes and move forward. However once there is an additional pointer which can point to any node except itself. we can not blindly create a new node reason being that node may have been present somewhere else. So we will use hashmap to keep info that if the copied node to that node is created if created we can use that node only otherwise we can create new node and map that.

```cpp
class Solution {

	public:
	
	Node* copyRandomList(Node* h1) {
		Node* head = new Node(0);
		Node* curr = head;
		unordered_map<Node*,Node*> mp;

		while(h1){
			if(mp.count(h1)){
				curr->next = mp[h1];
				if(h1->random){
					if(mp.count(h1->random)){
						curr->next->random = mp[h1->random];
					}else{
						curr->next->random = new Node(h1->random->val);
						mp[h1->random]=curr->next->random;
					}
				}
				h1 = h1->next;
				curr = curr->next;
			}else{
				curr->next = new Node(h1->val);
				mp[h1] = curr->next;
				if(h1->random){
					if(mp.count(h1->random)){
						curr->next->random = mp[h1->random];
					}else{
						curr->next->random = new Node(h1->random->val);
						mp[h1->random]=curr->next->random;
					}
				}
				h1 = h1->next;
				curr = curr->next;
			}
		}
		return head->next;
	}

};
```

## Finding nth from end

To solve this we have two pointers slow and fast. Intially we move fast n pts ahead of slow and then start moving both together. We will stop when fast pointer reaches end and slow is will now be at current position.
Why it works observe that fast and slow have distance of n between them at all points. So when fast reaches end slow will be at n pos from end. 

We can solve a related problem deleting the n th node from end - 
We use dummy node to handle the edge case of empty list at the end. 

```cpp
ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* dummy = new ListNode(0);
        dummy->next = head;
        ListNode* fast = dummy;
        ListNode* slow = dummy;
        for (int i = 0; i <= n; i++) {
            fast = fast->next;
        }

        while (fast != nullptr) {
            slow = slow->next;
            fast = fast->next;
        }
        ListNode* toDelete = slow->next;
        slow->next = slow->next->next;
        delete toDelete;

        ListNode* newHead = dummy->next;
        delete dummy;
        return newHead;
    }
```


## Partition List

Given the `head` of a linked list and a value `x`, partition it such that all nodes **less than** `x` come before nodes **greater than or equal** to `x`.

You should **preserve** the original relative order of the nodes in each of the two partitions.

Solution is that we maintain two dummy pointers one maintaining the list which has all the nodes with value less than x. Similarly we have another separate list which maintains the list of all the numbers larger than x. Finally we move the pointers to have all the things in place. 

```cpp
class Solution {
public:
    ListNode* partition(ListNode* head, int x) {
        ListNode* slist = new ListNode(0, nullptr);
        ListNode* blist = new ListNode(0, nullptr);
        ListNode* small = slist;
        ListNode* big = blist;

        while (head != nullptr) {
            if (head->val < x) {
                small->next = head;
                small = small->next;
            } else {
                big->next = head;
                big = big->next;
            }
            head = head->next;
        }

        small->next = blist->next;
        big->next = nullptr;

        ListNode* result = slist->next;
        delete slist;
        delete blist;
        return result;        
    }

};
```


## Patition list

Given the `head` of a linked list and a value `x`, partition it such that all nodes **less than** `x` come before nodes **greater than or equal** to `x`.

You should **preserve** the original relative order of the nodes in each of the two partitions.

Solution 

Here the idea is again to maintain two different linked list which are maintained using the dummy variable paradigm. Finally in the end we rejoin both the lists. 

```cpp
class Solution {
public:
    ListNode* partition(ListNode* head, int x) {
        ListNode* slist = new ListNode(0, nullptr);
        ListNode* blist = new ListNode(0, nullptr);
        ListNode* small = slist;
        ListNode* big = blist;

        while (head != nullptr) {
            if (head->val < x) {
                small->next = head;
                small = small->next;
            } else {
                big->next = head;
                big = big->next;
            }
            head = head->next;
        }
        small->next = blist->next;
        big->next = nullptr;
        ListNode* result = slist->next;
        delete slist;
        delete blist;
        return result;        
    }
};
```

