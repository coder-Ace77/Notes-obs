
---

#### What makes these problems hard

A segment tree is best understood as a framework with two things left unspecified rather than as a single structure. The two things are what each node stores and how the information in two adjacent nodes combines into information about their union.

Every problem in this section supplies a different pair of answers to those two questions. Sometimes the combination is straightforward as with sums. More often the information you actually want cannot be combined at all and the design work consists of finding *additional* information to store whose only purpose is to make the combination possible. 

## The framework

A segment tree over an array is a binary tree in which each leaf corresponds to one element each internal node corresponds to the concatenation of the ranges covered by its two children and each node stores a summary of its own range.

For this to work the operation that combines two summaries must be associative so that grouping does not affect the result and there must be an identity value to use for empty ranges. Sums qualify with an identity of zero maimums qualify with an identity of negative infinity greatest common divisors qualify with an identity of zero and matri products qualify with the identity matri. The identity eists because the recursion must return something for ranges it misses entirely.

What is **not** required is commutativity since nothing in that argument ever swapped two pieces only re-bracketed them. Gluing the pieces in left-to-right order is enough.

**Why a range only ever needs a logarithmic number of blocks.** Every node the recursion visits either misses the query sits entirely inside it or straddles its edge. A node can only straddle if one of the two query endpoints lands strictly inside it. At any given depth the nodes are non-overlapping intervals laid side by side the left endpoint falls in eactly one of them and the right endpoint in eactly one so **at most two nodes per depth straddle** and everything else at that depth stops immediately. Two recursing nodes per level each producing at most two children that stop and get taken over a tree of depth `log n`.

The design work therefore consists of two questions asked in this order:

1. What does a node need to store so that a query over an arbitrary range can be answered by combining the summaries of the logarithmically many nodes that cover it?
2. Is that information sufficient to compute a parent's summary from its two children?

 Frequently the answer to the first question alone turns out to be insufficient because the quantity you want cannot be recovered from the same quantity in the two children. When that happens the response is to store more and the etra fields eist purely to make the combination possible rather than because the problem asked for them.
### The non-lazy template

Ranges are inclusive `[l r]` the root is node `0` and the children of `i` are `2i+1` and `2i+2`.

```cpp
class SegTree {
    void build(vector<int>& arr int l int r int i) {
        if (l == r) { tree[i] = arr[l]; return; }
        int mid = (l + r) / 2;
        build(arr l mid 2*i+1);
        build(arr mid+1 r 2*i+2);
        tree[i] = tree[2*i+1] + tree[2*i+2];
    }
    void upd(int ind int val int l int r int i) {
        if (l == r) { tree[i] = val; return; }
        int mid = (l + r) / 2;
        if (ind <= mid) upd(ind val l mid 2*i+1);
        else            upd(ind val mid+1 r 2*i+2);
        tree[i] = tree[2*i+1] + tree[2*i+2];
    }
    int qry(int ql int qr int l int r int i) {
        if (ql > r || l > qr) return 0;              // no overlap so the identity
        if (ql <= l && r <= qr) return tree[i];      // fully inside
        int mid = (l + r) / 2;
        return qry(ql qr l mid 2*i+1) + qry(ql qr mid+1 r 2*i+2);
    }
public:
    int n; vector<int> tree;
    SegTree(int arr_size) { tree.resize(4*arr_size); n = arr_size; }
    void build(vector<int>& arr) { build(arr 0 arr.size()-1 0); }
    void upd(int ind int val)   { upd(ind val 0 n-1 0); }
    int  qry(int ql int qr)     { return qry(ql qr 0 n-1 0); }
};
```

Here lets first observe the `update` function, philosphy is to start from root and at each node move down the tree in the direction to `indx` index to be updated. The result is that entire path from `root` to the `leaf` node is updated while traversal from `root` node towards leaf. This path is of length `logn`. Also note that since value of leaf is updated only the `nodes` which lie on the path needs to get updated. This updation happens bottom up and thus this style of coding segment tree is bottom up segment tree. More importantanly observe the last line of `upd` function 

```
tree[i]=tree[2*i+1]+tree[2*i+2]; 
```

This line is true only because of the invariant that tree nodes get updated to their true values bottom up only.  

Three details generalise beyond this particular tree.

The three-case structure inside `qry`  being no overlap full containment and partial overlap is the same in every recursive segment tree in this chapter lazy or not. Everything that varies between problems is the return type the `+` in the two recombination lines and the identity returned on no overlap.
## Lazy propagation

When updates apply to whole ranges rather than single positions the tree needs to defer work and the deferral requires the recursive form because pending operations have to be pushed down along a path from the root.

```cpp
class LazySeg {
    void apply(int i,int l,int r,long long v) {      // "add v" across the whole node
        tree[i] += v * (r - l + 1);                     // inclusive range so +1
        lz[i]   += v;
    }
    void push(int l int r int i) {
        if (lz[i] == 0) return;
        int mid = (l + r) / 2;
        apply(2*i+1,l,mid,lz[i]);
        apply(2*i+2,mid+1,r,lz[i]);
        lz[i]=0;
    }
    void upd(int ql int qr long long v int l int r int i) {
        if (ql > r || l > qr) return;                                // no overlap
        if (ql <= l && r <= qr) { apply(i l r v); return; }        // fully inside
        push(l r i);
        int mid = (l + r) / 2;
        upd(ql qr v l mid 2*i+1);
        upd(ql qr v mid+1 r 2*i+2);
        tree[i] = tree[2*i+1] + tree[2*i+2];                          // never omit
    }
    long long qry(int ql int qr int l int r int i) {
        if (ql > r || l > qr) return 0;
        if (ql <= l && r <= qr) return tree[i];
        push(l r i);
        int mid = (l + r) / 2;
        return qry(ql qr l mid 2*i+1) + qry(ql qr mid+1 r 2*i+2);
    }
public:
    int n; vector<long long> tree lz;
    LazySeg(int sz) : n(sz) tree(4*sz 0) lz(4*sz 0) {}
    void upd(int ql int qr long long v) { upd(ql qr v 0 n-1 0); }
    long long qry(int ql int qr)         { return qry(ql qr 0 n-1 0); }
};
```

### Why deferring the work is correct

Note here invariant is - Once you reach a given node the node will be pure and will not have any pending lazy updates. What means is that the responsibility to make a node pure actually lies on the parent of a given node. And is mostly the premise. 

Here `tree` is the actual array holding the value. With lazy technique it means sometimes value of tree may not be true. `lazy` actually holds the tag for the parent and it does means that values in the subtree of current nodes are correct or not. With simple sum queries `lazy[i]!=0` means the childs are not correct which is sometimes referred to as impure. However that does not means value is incorrect for current node. It means value is correct for the node but the childs are impure.

**A tag is a message addressed to your children. It is not a note to yourself.** When a tag is placed on node `i` `tree[i]` is updated immediately and completely. The tag eists only to record that the same operation still owes an update to everything underneath `i`. So `lz[i] != 0` never means "`tree[i]` is wrong". It means "`tree[2i+1]` and `tree[2i+2]` are wrong and here is the information they need in order to become right".

Now lets talk about two functions 

- apply - This is required to make the current node pure or correct. Once apply is run for a given node the node becomes correct and pure. This function essentially needs to do two things first one being the updation of correct value `tree[i]`. And second point is to update `lazy` tag correctly. Note once again this `lazy` tag refers that bottom nodes will be impure and not this current child. apply runs at two places first is when we reach a full update node where the entire update is to be done `ql<=l && r<=qr` here we need to apply and reason is that we are reaching this node finally and recursion will stop here. Note that we just need to do two things here - make current node pure and remember tag. However what value needs to be updated is actually the value. How and why is this the case we will see in a bit. 

- push - This function exists to make children pure before reaching them and we know that this responsibility lies on parent of a given node. And `push(l,r,i)` Exists for exactly this case. When we reach a given node with id `i`. We try to make both of the childrens correct. So what we do is apply both of them now since at this time `lazy` tag for `i` is correct. We can simply use `lazy[i]` to apply to both of its children. Now as childrens are made correct we update lag to make it `0` to announce that we no longer have its children update pending. Note that push is done before the both childrens recursion is called. Once we return from the recursion of both of the children we update the latest values to the correct value of node with id `i`. This completes the update and query is importantly same for non lazy use case except the fact for the `push`. 

Now some notes about the philosphy of `upd` for the lazy case note that while we try to update may be node very deep down the tree most of the updates need to propogate through the path from root. This matters because if this lazy tag application or implementation was not happening from the very top we could have corrupted the history. However since we always start from the very top and move to the nodes through a path lazy tags get applied one step at a time and finally we get a node on which prev history is already applied. 

### Designing the combination

This is the part that distinguishes being able to use a segment tree from having seen one.

#### The mechanical way to find the fields

The question is always the same: **if I know the quantity for the left half and for the right half do I know it for the whole?** There is a procedure for answering that which is far better than staring at the problem hoping for insight.

**Try to break it.** Look for two different left halves that have the *same* summary but behave *differently* when the same thing is glued onto the right. If you find such a pair no combine function can eist and the pair also tells you what field is missing.

**CSES Prefi Sum Queries** asks for the largest prefi sum within a range where the empty prefi is permitted so the answer is never negative. Try to break "store the best prefi and nothing else":

```
A  = [3 -3]   best prefi = 3
A' = [3]       best prefi = 3        identical summaries

glue B = [10] onto each:
[3 -3 10]    best prefi = 10
[3 10]        best prefi = 13       different answers
```

Identical inputs to `combine` two different required outputs so the function cannot eist. Storing only the best prefi is dead and not because it is hard.

Now the useful half. **What differed between `A` and `A'`?** Their totals `0` against `3`. That is the missing information since the total is what decides how much a prefi is worth once it has swallowed the entire left side. So the fi is to store two values:

```
sum  = the total of the range
best = the largest prefi sum of the range
```

which combine as

```
sum  = L.sum + R.sum
best = ma(L.best L.sum + R.best)
```

The second line says that the best prefi either stops somewhere inside the left child or runs off the end of the left child in which case it is *all* of the left plus a prefi of the right. Those two cases are ehaustive so taking the better of them is correct. The empty prefi of the right child has to be legal otherwise the case "eactly all of the left" is not representable. The `sum` field is stored purely so that this line can be written; the problem never asks for it.

**The general loop** is worth stating on its own because it is what you run under time pressure rather than trying to recall a design. Propose a summary try to break it with a same-summary-different-result pair and whatever distinguished that pair becomes a new field. Repeat. You stop when every output field is computable from the input fields at which point the field set is closed and the tree eists. The procedure is finite and mechanical and its failure mode is informative too: if you keep needing new fields without ever closing the quantity genuinely is not summarisable and the problem is something else.

**CSES Subarray Sum Queries** asks for the largest sum over any contiguous piece within a range and needs four fields:

```
sum  = the total
pref = the largest prefi sum
suf  = the largest suffi sum
best = the largest sum over any piece inside
```

combining as

```
sum  = L.sum + R.sum
pref = ma(L.pref L.sum + R.pref)
suf  = ma(R.suf  R.sum + L.suf)
best = ma(L.best R.best L.suf + R.pref)
```

The last line carries the idea and it is worth proving properly since every later design is modelled on it. Take any contiguous piece sitting inside the combined range. Its positions are consecutive so eactly one of three things holds: every position is in the left child every position is in the right child or it has at least one position in each. The third case has a consequence that is easy to skim past. Because the piece is contiguous and spans the boundary it is forced to contain the **last** position of the left child and the **first** position of the right so it is precisely a suffi of the left joined to a prefi of the right and not merely something vaguely crossing the middle. Those three cases are mutually eclusive and cover everything and the maimum over a union of sets is the maimum of the individual maima so taking the best of the three case-winners is correct.

One step there is usually left implicit. In the crossing case you are allowed to maimise the suffi and the prefi **separately** only because the two choices are independent since any suffi of the left may be paired with any prefi of the right and the value is simply their sum. Independent choices with an additive objective can be optimised one at a time. If a constraint linked them such as a cap on the combined length that step would collapse and the design would need more fields. Deriving these four lines by hand once is worthwhile because they serve as the model for every subsequent combination you have to design.

**CF 380C Sereja and Brackets** asks for the length of the longest valid bracket subsequence within a range. A node stores the number of matched pairs the number of unmatched opening brackets and the number of unmatched closing brackets:

```
newPairs = min(L.open R.close)
matched  = L.matched + R.matched + newPairs
open     = L.open  + R.open  - newPairs
close    = L.close + R.close - newPairs
```

The structural fact that makes three numbers sufficient is worth seeing directly. **Take any bracket string and cancel matched pairs repeatedly. What survives always looks like `)))(((`** some closers followed by some openers never an opener before a closer. The reason is immediate: if a leftover opener sat to the left of a leftover closer those two would have matched each other and neither would be leftover. So the residue of a range is completely described by two counts and nothing else about its arrangement eists as far as the outside world is concerned. That is eactly why the triple of matched open and close is a complete summary.

The merge follows. Every leftover opener of the left child sits to the left of every leftover closer of the right child so all such pairs are legal and the only limit is supply giving `min(L.open R.close)` new pairs. Nothing better is available either since the leftover closers of the left child sit to the left of the entire right side and can never find a partner there and the leftover openers of the right child have nothing to their right at all.

**CSES Pizzeria Queries** asks for the minimum of `p[j] + |i - j|` over all `j`. Splitting the absolute value gives `(p[j] - j) + i` when `j` is at or before `i` and `(p[j] + j) - i` when `j` is at or after `i`. Keeping two separate minimum trees one over `p[j] - j` and one over `p[j] + j` turns each query into two range minimums. 

#### Walking down the tree

There is a class of query of the form "find the leftmost position satisfying some property". The direct approach binary searches over positions and performs a range query at each step which costs two logarithmic factors. Walking down the tree costs one.

**CSES Hotel Queries** asks you to find the leftmost hotel with at least a given number of free rooms and then book them. With a tree storing maimums:

```cpp
int descend(long long k int l int r int i) {
    if (tree[i] < k) return -1;                     // no suitable leaf in this subtree
    if (l == r) { tree[i] -= k; return l; }         // a leaf so book here
    int mid = (l + r) / 2;
    int res = (tree[2*i+1] >= k) ? descend(k l mid 2*i+1)
                                 : descend(k mid+1 r 2*i+2);
    tree[i] = ma(tree[2*i+1] tree[2*i+2]);        // recombine on the way back up
    return res;
}
```
## Usage patterns

### Multiple trees over the same array

A multi-tree design is a technique in which a single hard query is split into two or more independent easy queries, each answered by its own plain segment tree, and the final answer is obtained by combining the individual results. It is used when an expression contains a case split, such as an absolute value or a minimum of two terms, that a single tree cannot evaluate directly.

Consider an array `p` of `n` values, where a query gives an index `i` and asks for the minimum value of `p[j] + |i - j|` over every position `j` in the array. The absolute value makes this impossible to answer with a single range-minimum tree directly, so it is split into two cases: when `j <= i`, the expression equals `(p[j] - j) + i`, and when `j >= i`, it equals `(p[j] + j) - i`. Both cases are now plain range-minimum expressions, so two trees are built — one over the array `p[j] - j` and one over the array `p[j] + j` — and each query reads one range minimum from each tree, adds or subtracts `i`, and returns the smaller of the two results.

The same technique applies whenever a hard operation decomposes into several independent easy operations. Building one tree per bit of a number turns a range XOR update into a simple flip on each tree.

### Zero/one presence trees

A presence tree is a segment tree in which every leaf holds either 0 or 1, representing whether a position is currently active or has been removed. The value stored at each node is the sum of its leaves, which gives the count of active positions in that range. This count is used in two ways: to answer how many positions remain in a given range, and to locate the k-th active position through a descent that carries a running count downward. 

Two operations are common with presence trees:

- **Finding and removing the k-th active element.** Descend to the left child if its count is at least `k`; otherwise subtract the left child's count from `k` and descend right. Removing the element found is then a single point update.
- **Finding the nearest active position at or after a given index.** The same descent, applied to the question "what is the leftmost active position at or after `x`."

Consider `n` seats, numbered left to right, all initially free. Queries are of two kinds: mark a specific seat as taken, and given a count `k`, find the `k`-th seat, counting from the left, that is still free, then mark that seat as taken too. A presence tree answers both in `O(log n)`: the first is a plain point update, and the second is the descent described above, using the running count to decide whether to go left or right at every step.

### Merge-sort tree

A merge-sort tree is a static segment tree in which each node stores a sorted copy of the values in its range, instead of a single aggregated value. It is used to answer range queries such as "how many values in `[l, r]` are less than or equal to `x`" when the queries are not known in advance and therefore cannot be processed with the offline sweep described earlier in this guide.

The tree is built in `O(n log n)` time, since each value appears in `O(log n)` nodes, one for every level it belongs to. A query visits the same `O(log n)` nodes an ordinary range query would visit, and at each node performs a binary search on its sorted list, giving a total query time of `O(log^2 n)`.

Consider an array of `n` numbers, where each query gives a range `[l, r]` and a value `x`, and asks how many numbers in that range are less than or equal to `x`. If the queries arrive one at a time and must be answered immediately, rather than all being known in advance, they cannot be sorted and processed with the offline sweep described earlier. A merge-sort tree answers each one independently, in `O(log^2 n)`, without needing to know the others ahead of time. The same structure also answers "what is the `k`-th smallest value in `[l, r]`," either by merging the sorted lists from the visited nodes or by binary searching on the answer and counting, in each node, how many values fall below it.

### A combo query answered by two trees together

A combo query is a technique in which a node stores two different summaries, and a single descent through the tree consults both summaries together at every step, rather than answering two separate queries and combining their results afterward.

Consider `n` rows of a theatre, where row `i` currently has `a[i]` free seats. Queries are of two kinds: given a group size `k`, either seat the whole group together in a single row, or seat the group using as few rows as possible, filling each row completely from the left before moving to the next. Each node stores two fields: the maximum number of free seats in any single row within its range, and the total number of free seats within its range. The first query type is answered with a plain descent using only the maximum field, exactly as in the walking-down technique above. The second query type also descends, but the choice at each step uses the sum field: if the left child's total is enough to hold everything still needed, the group is placed starting there and the descent continues into the left child first, and whatever the left side could not hold is only then passed into the right child.

What distinguishes this technique from using two independent trees is that the meaning of the second field during the descent depends on a decision already made using the first field higher up in the tree. The two fields are not queried separately and merged afterward; they are used together, step by step, during a single walk down the tree.

### Compression technique

Note that we are writing the tree on array which leads to the max index upto `1e6`. The issue occurs when we are req to track following queries - 

- upd(x,c) update count of element x by c
- qry(a,b) - Return total count of all elements in between a and b

This problem is easy to solve when x is atmost `1e6`. However what to do when its `1e9` or beyond. We introduce coordinate compression. Even though the maximum salary or value might be $10^9$, the total number of distinct values processed across $N$ initial elements and $Q$ queries is bounded by at most $N + 2Q$.

For typical constraints where $N, Q \le 2 \times 10^5$, there are at most $5 \times 10^5$ unique numbers involved. Range count queries care only about the **relative ordering** of values, not their absolute scale.

By mapping each unique value in the $10^9$ space to a dense rank in the range $[0, M-1]$, we compress the universe of values down to a manageable size $M \approx 5 \times 10^5$.

To perform coordinate compression offline (when queries are known in advance):

Collect every value that will ever be updated or queried into a single vector. For query bounds $[a, b]$, both endpoints $a$ and $b$ must be included.

```cpp
vector<int> pts;

// 1. Initial values
for (int x : initial_array) pts.push_back(x);

// 2. Query values
for (auto& q : queries) {
    if (q.type == UPDATE) {
        pts.push_back(q.new_val);
    } else if (q.type == QUERY) {
        pts.push_back(q.a);
        pts.push_back(q.b);
    }
}
```

Sort the collected points and remove duplicates to build a strictly increasing sequence of unique values.

```cpp
sort(pts.begin(), pts.end());
pts.erase(unique(pts.begin(), pts.end()), pts.end());
int M = pts.size(); // Size of our compressed coordinate space
```

Now once `pts` are available people have multiple ways -

- use simple maps `unordered_map` or `map` to map the sparse set of values to `dense` space. 
```cpp
int curr_ind=0;
for(auto pt:pts){
	if(mp.find(pt)==mp.end()){
		mp[pt]=curr_ind++;
	}
}
```

Now it is usable but does not works well since we can create test cases where unordered_map are quanteed to follow `O(n)` per query. However there is better way to keep all the pts in a vector and use `lower_bound`. Since value is guarantted to exist in the vector and will be unq we will always get correct index. 

```cpp
int get_compressed_idx(int x) {
    return lower_bound(pts.begin(),pts.end(),x)-pts.begin();
}
```


### Handling the frequencies in segment trees

### What can be stored

Vitually tree in segment tree can store anything you can come up with. However there are some things which usually come pretty frequently based on usage. 

- First thing is the interger or count itself. Where its count of summary of something.
- Second thing is small array stroing multiple things like count as well as sum. 
- Third thing is the small set which one needs to maintain for an example set of characters. One can think of maintaining small hashmap per node. But actually a integer with its bits can actually fullfill the job. 
- Sets can also be stored per node or an hashmap. 

Handling frequency queries and quickly querying on the range is more difficult. We are trying to handle following queries here -

- How many elements in the range `l,r` are of value atleast `x`.
- How many elements in the range `l,r` have frequeny atleast `x`.  and these kinds of queries. 

First data structure we see is merge sort tree. 

### Merge sort tree

In a standard segment tree, each node represents a range $[L, R]$ and stores a single merged value (like a sum or maximum). In a **Merge Sort Tree**, each node instead stores a **sorted list** of all elements belonging to that range.

At the leaf level, each node stores a single-element list `[arr[i]]`. As we move up, each parent node combines the sorted lists of its two children using the standard `merge()` step from Merge Sort.

Now observe that each node has the not only the summary but a detailed analysis stored on a node. However while querying you can not traverse this data structure linearly reason is then our query will take around linear time and around `O(logn)` nodes are touched meaning `O(nlogn)` time is taken. 

Now lets see basic question which can be solved using this - 

```
qry(l,r,x) = freq of x in the range l,r
```

Now note that we already have stored the sorted list in each node equal to size of range of node. Now space complexity for merge segment tree - 

We are storing `r-l+1` elements for a node marking `l,r` observe that only `n` elements are used across the entire horizon at any given depth `d`. Since we have `O(logn)` depth segment trees full space complexity is `O(nlogn)`. 

Now how do we handle query - 

```
1. Find the relevant nodes. 
2. Now in a given node fully inside range (ql,qr) we need to find the count of x 
3. Which is easy since range is sorted. use binary search (`std::upper_bound - std::lower_bound`) on its stored sorted list to count occurrences of $X$. Sum the counts across all matching nodes
```

Note that time complexity is `O((logn)^2)`.

Now if the sorted data structure used is `vector`, we will face difficulty in update since its inefficient to update a sorted vector. Rather we can use `set` however it has higher cost associated. 

When each node holds a `std::multiset` instead of a `std::vector`:

- ✏️ **Point Update ($O(\log^2 N)$):** When `arr[i]` changes from $X$ to $Y$, we visit the $O(\log N)$ ancestor nodes that contain index $i$. In each ancestor node's multiset, we erase $X$ and insert $Y$ in $O(\log N)$ time.
- 🔍 **Range Frequency Query ($O(\log^2 N)$):** We visit $O(\log N)$ canonical nodes covering $[QL, QR]$ and use `equal_range()` or iterator subtraction on each multiset in $O(\log N)$ time.

Now lets move to another problem

```
qry(l,r,x) = how many elements in the range have values strictly less than x
```

Now if you know we can use the `pbds` in `cpp` to solve the problem. Since pbds allows us to find how many elements atmost `x` in `O(logn)` time. We can use that instead of our simple set. 

### Value segment tree 

In a standard Segment Tree, node indices correspond to **array positions** ($0$ to $N-1$).

In a **Value-Indexed Segment Tree** (sometimes called a Frequency Segment Tree or Coordinate Tree), the leaves correspond to **element values** (from $1$ to $\text{MAX\_VAL}$). Each leaf at index $V$ stores the **frequency** of value $V$ in our collection.

Usage of such trees is first we can very easily find freq of some ranges. May be max or total occurance etc. Adding or removing an element just means incrementing or decrementing its value leaf and updating the ancestors.

#### Concept Note: Lazy-Only Segment Trees

When a segment tree only handles **range updates** and **point queries**, internal nodes do not need to store or compute aggregate values (like range sum, min, or max). Instead, range updates assign lazy tags to target subtrees, and point queries traverse down to a specific leaf, pushing pending tags along that single path. Because range queries across internal nodes are never performed, parent nodes never need to combine child values (`tree[i] = tree[left] + tree[right]`). Internal node `tree` entries remain unused, while leaves hold the fully resolved values.

```cpp
void push(int l, int r, int i) {
    if (lazy[i] == NO_TAG) return;
    int mid = (l + r) / 2;
    apply(l, mid, 2 * i + 1, lazy[i]);
    apply(mid + 1, r, 2 * i + 2, lazy[i]);
    lazy[i] = NO_TAG;
}

void upd(int ql, int qr, int val, int l, int r, int i) {
    if (ql > r || l > qr) return;
    if (ql <= l && r <= qr) {
        apply(l, r, i, val);
        return; // Fully covered: tag and return
    }
    push(l, r, i);
    int mid = (l + r) / 2;
    upd(ql, qr, val, l, mid, 2 * i + 1);
    upd(ql, qr, val, mid + 1, r, 2 * i + 2);
    // Notice: No parent re-calculation step needed here!
}

int qry(int ind, int l, int r, int i) {
    if (l == r) return tree[i]; // Leaf reached: returns exact point value
    push(l, r, i);
    int mid = (l + r) / 2;
    if (ind <= mid) return qry(ind, l, mid, 2 * i + 1);
    else return qry(ind, mid + 1, r, 2 * i + 2);
}
```

- **`push()`**: Pushes pending lazy tags down to direct child nodes whenever traversing through node `i`.
    
- 🏷️ **`upd()`**: Sets the tag directly on fully contained ranges and recurses on partial overlaps. It skips any post-recursion parent updates.
    
- 🎯 **`qry()`**: Clears lazy tags along the path from the root down to index `ind` and returns `tree[i]` once it hits the leaf `l == r`.





