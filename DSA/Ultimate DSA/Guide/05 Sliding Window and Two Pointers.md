---
tags: [dsa, guide, sliding-window, two-pointers]
chapter: 5
sheet-section: E
---

# Chapter 5 · Sliding Window & Two Pointers

---

The **sliding window technique** is an algorithmic optimization pattern used to transform nested-loop computations over contiguous sequences (arrays, strings, or vectors) from $O(N^2)$ or $O(N^3)$ time complexity down to $O(N)$. Instead of recalculating state for overlapping contiguous subarrays from scratch, the technique maintains a running state across a bounded range defined by two pointers (`left` and `right`) and updates the state incrementally as the range "slides" across the structure.

The first version of it is fixed size sliding window.

#### Fixed size window

In this variation, the window length $K$ remains constant throughout the traversal. The goal is to evaluate a specific property (e.g., sum, maximum, average) across every contiguous subarray of size $K$. Template


```cpp
// caclute properties for first window
for(int i=0;i<k;i++){
}

// now calc for next windows
for(int i=1;i+k-1<n;i++){
	
}
```

Bottom line is that first window should be doable in `O(k)` time length of window but for every subsequent window it should `O(1)` operation. 

#### Varaible size sliding window

In this variation, the window expands and contracts dynamically based on a condition or constraint. The objective is usually to find the **longest**, **shortest**, or **total count** of contiguous subarrays satisfying a criteria. Now there is a invariant that needs to be met to apply this technique in the first place and that is and subarray of a valid array must be valid. Using this 

Template

```cpp
while(i<n){
	add(v[i]);
	
	while(!valid(window)){
		remove(window);
		j++;
	}
	
	// now compute at this point the window is valid 
	ans = compute(window);
	i++;
}
```

Now there are three sections and points we maintain a variable size window whose property is that any subarray of valid window is also valid. We keep it as `(i,j)`. Mechnaism is that we first add element to the window. At this point window can become invalid and this is fine since we write another loop which removes elements from stating of window until window does not becoems valid. Once inner while loop is terminated window becomes valid and finally we compute answer. 

Observe that when reach compute state the window actually holds for ending index i longest valid subarray or empty window if not possible. This invariant that compute stage the window is longest valid among all the subarrays ending at `i` means we always traverse all the valid subarrays. 

Most of the difficulty in the harder problems in this block comes from two places. The first is that the condition quietly fails, most often because negative numbers are permitted, and the window has to be replaced with something else. 

The second is that the quantity being maintained inside the window is no longer a simple count or sum, so you have to decide what structure to carry and how to add to and remove from it efficiently. Many times structures are counts or maps. 

Finally it should be established before using this technique that all the valid subarrays of valid window are also valid for example in question shortest subarray with at least given sum. If the numbers are pve this condition of if window is valid subarray keeps on to be valid however as soon as negative numbers are introduced it may be false. 

Also sometimes we track the opposite variation of it window tracked is invalid one and again the point is any subarray of invalid window should keep on invalid. 

Take the condition "at most `k` distinct characters." If the window from `l` to `r` contains at most `k` distinct characters, then the window from `l + 1` to `r` certainly does too, since removing a character cannot increase the number of distinct ones. Shrinking therefore never causes harm, which means that when a window becomes invalid, advancing the left edge is guaranteed to eventually fix it, and no position ever needs revisiting.

The condition fails for "longest run with sum at most `K`" when negative numbers are allowed, because adding an element can decrease the sum. An invalid window can become valid by growing, so the structure the technique relies on is absent. This is why LC 862, which asks for the shortest run with sum at least `K` and permits negatives, is not a window problem at all and instead needs a monotonic deque over prefix sums.

It is worth completing the sentence "if the window from `l` to `r` is valid then the window from `l + 1` to `r` is valid, because ..." before writing anything. 

## Part 2 · The two window shapes

Almost every window problem is one of two shapes, and they differ by a single character.

**The first shape finds the longest valid window.** The right edge always advances, and the left edge advances only while the window is invalid.

```cpp
int l = 0, best = 0;
for (int r = 0; r < n; r++) {
    add(a[r]);
    while (!valid()) { remove(a[l]); l++; }
    best = max(best, r - l + 1);
}
```

**The second shape finds the shortest valid window.** The right edge always advances, and the left edge advances while the window is *still* valid, recording the answer as it goes.

```cpp
int l = 0, best = INF;
for (int r = 0; r < n; r++) {
    add(a[r]);
    while (valid()) { best = min(best, r - l + 1); remove(a[l]); l++; }
}
```

The difference is `while (!valid())` against `while (valid())`, together with where the answer is recorded. In the first shape you record after restoring validity, and in the second you record just before breaking it. Getting these crossed produces a solution that runs and gives wrong answers, and it is the most common structural error in the category.

### Counting windows 

This is the idea that turns counting problems from impossible into routine, and it has two halves.

**The first half concerns which condition to use.** Counting the runs containing exactly `k` distinct integers cannot be done with a window directly, because a shorter piece of a window with exactly `k` distinct values might contain fewer, so shrinking does not preserve the condition.

The way around this is that "at most `k`" does satisfy the shrinking condition, and

$$\text{exactly}(k) = \text{atMost}(k) - \text{atMost}(k-1)$$

so the problem reduces to two runs of an ordinary window.
**The second half concerns how to count inside the window.**

```cpp
long long atMost(vector<int>& a, int k) {
    unordered_map<int,int> cnt;
    long long res = 0;
    int l = 0;
    for (int r = 0; r < (int)a.size(); r++) {
        if (++cnt[a[r]] == 1) k--;
        while (k < 0) { if (--cnt[a[l]] == 0) k++; l++; }
        res += r - l + 1;          // every run ending at r is valid
    }
    return res;
}
```

The line `res += r - l + 1` deserves attention because it does the counting. Once `l` is the smallest left edge for which the window ending at `r` is valid, every window starting at `l`, at `l + 1`, and so on up to `r` is also valid, which follows from the shrinking condition. There are `r - l + 1` of them, and every valid run in the whole array is counted exactly once, at its right end.

That accounting idea transfers even to problems where the "at most" decomposition does not apply. LC 2444 Count Subarrays With Fixed Bounds tracks the most recent position of the minimum bound, of the maximum bound, and of any out-of-range value, and adds a quantity derived from those three at each step. The bookkeeping is different, but the principle of counting each run once at its right end is the same, and recognising that the principle transfers is more useful than remembering either formula.

### Maintaining a maximum or minimum with a deque

For a window of fixed size, or for any window where you need the largest or smallest element, a monotonic deque gives a linear solution where a heap would give `O(n log n)`.

```cpp
deque<int> dq;                              // holds INDICES, values decreasing
for (int r = 0; r < n; r++) {
    while (!dq.empty() && a[dq.back()] <= a[r]) dq.pop_back();   // keep it decreasing
    dq.push_back(r);
    if (dq.front() <= r - k) dq.pop_front();                     // discard out-of-window
    if (r >= k - 1) ans.push_back(a[dq.front()]);                // the front holds the maximum
}
```

Two details matter. The deque stores indices rather than values, because the index is what tells you when an element has fallen out of the window. And the expiry check has to run before the front is read, since otherwise the reported maximum may belong to an element that has already left.

The reason the whole scan is linear despite the inner loop is that every index is pushed once and popped once, so the total work across all iterations is bounded by twice the array length. This is the same accounting that justifies the disjoint-interval map in chapter [[02 Intervals and Sweep Line]] and the monotonic stack in chapter [[07 Monotonic Stacks and Deques]], and recognising it as one argument rather than three makes all of them easier to trust.

When both the maximum and the minimum of a window are needed, as in LC 1438, run two deques side by side, one decreasing and one increasing. The window is valid while the difference between the two fronts is within the limit, and when the left edge advances, the front of either deque is discarded if its index has fallen behind.

A `multiset` is a reasonable alternative here. It is shorter to write and runs in `O(n log n)`, which is fine for LeetCode-scale constraints, so it is worth reaching for under time pressure and keeping the deque for cases where the constraints are tight.

### Windows carrying a real structure

**Maintaining a median.** LC 480 and CSES *Sliding Median* need the middle value of the window. There are two workable approaches.

The first uses two heaps, a maximum-heap holding the lower half and a minimum-heap holding the upper half, together with lazy deletion. When an element leaves the window it is recorded as deleted and only actually removed once it surfaces at the top of a heap, with the sizes tracked separately to compensate. This works and is fiddly.

The second keeps a single `multiset` together with an iterator pointing at the median, moved by at most one position after each insertion or removal. This is shorter and is the usual competitive answer.

```cpp
multiset<int> ms;
auto mid = ms.begin();          // maintained to point at the median
// after inserting x:  if (x < *mid) mid--;   then rebalance according to size parity
// before erasing x:   advance the iterator first if necessary, then erase
```

The iterator handling is genuinely awkward, so it is worth writing once carefully and keeping. CSES *Sliding Cost*, which asks for the total distance to the median, is the same structure with two running sums added, one for each half.

## Two pointers

Everything so far has been about a window, which is a pair of pointers that bound a range of the array while we keep track of what is currently inside that range. Two pointers is the wider family that this belongs to, and its other members use a pair of pointers in a completely different way. In these problems the range between the pointers does not matter and nothing is maintained about it. Each pointer is simply a position that we are still considering, and at every step we compare what the two pointers point at and use that comparison to prove that one of the positions can never be part of the answer, which lets us throw it away and move that pointer forward.

 A brute force solution looks at every pair of positions, which is quadratic. A two pointer solution is linear because at every step it can explain why one position is no longer needed, and every position can only be thrown away once. The skill that these problems train is therefore not moving pointers, which is easy, but proving that the position you are about to discard really cannot be in a better answer. 

The sections below go through the common shapes in an order that starts with the easiest proof and ends with the most general one.

### Opposite-end pointers on a sorted array

Suppose we are given an array that is already sorted in increasing order and a target number, and we want to know whether any two different elements add up to the target. The brute force checks every pair and takes `O(n^2)` time. The opposite-end version places one pointer `l` at the first element and the other pointer `r` at the last element and looks at their sum.

```cpp
int l = 0, r = n - 1;
while (l < r) {
    long long s = (long long)a[l] + a[r];
    if (s == target) return true;
    if (s < target) l++;
    else            r--;
}
return false;
```

The reason this is correct comes from the sorted order ,suppose `a[l] + a[r]` is smaller than the target. Then `a[l]` is too small to work with `a[r]`, and `a[r]` is the largest element that is still available, so every other partner `a[l]` could have is no larger than `a[r]` and gives an even smaller sum. That means `a[l]` cannot reach the target with anything that is left, so it can be discarded and `l` moves right. If the sum is larger than the target the same reasoning applies to `a[r]` from the other side, because `a[l]` is the smallest element that is still available and even it is too small to rescue `a[r]`. Either way exactly one element is thrown away per step, so the loop runs at most `n` times.

The loop condition is `l < r` and not `l <= r`, because the two pointers must refer to different elements. Using `<=` would let the same element pair with itself.

**Counting instead of searching.** The same pointers can count pairs whose sum is at most the target. When `a[l] + a[r]` is within the target, then `a[l]` also works with every element between `l + 1` and `r`, since they are all no larger than `a[r]`. That is `r - l` valid pairs counted in one step, after which `a[l]` has been fully accounted for and can be discarded.

```cpp
long long cnt = 0;
int l = 0, r = n - 1;
while (l < r) {
    if (a[l] + a[r] <= target) { cnt += r - l; l++; }
    else r--;
}
```

### Read and write pointers

In this shape both pointers move in the same direction over the same array, but they have different jobs. The **read** pointer scans every element once, and the **write** pointer marks the place where the next element we decide to keep will be stored. The array is rebuilt in place, and because we never allocate a second array the extra memory is constant.

Take the problem of removing duplicates from a sorted array in place and returning how many distinct values remain.

```cpp
int w = 0;
for (int r = 0; r < n; r++) {
    if (w == 0 || a[r] != a[w-1]) a[w++] = a[r];
}
return w;
```

The idea that makes this easy to reason about is an **invariant**, which is a statement that is true before and after every iteration. Here it is that `a[0..w)` already contains the answer for everything the reader has seen so far. A new element is kept only if it differs from the last kept element `a[w-1]`, and because the input is sorted, comparing with the last kept value is enough to know whether it has appeared before. When the loop ends, the invariant says `a[0..w)` is the answer for the whole array.

Two facts guarantee that nothing is destroyed. The writer never gets ahead of the reader, so `w <= r` always holds, which means a write can only overwrite a position that has already been read. And elements are copied in the order they were read, so the relative order of the kept elements stays unchanged. Removing every occurrence of a given value, or moving all zeros to the end by first compacting the non-zeros and then filling the rest with zeros, is the same code with a different keep condition.

**More than two regions.** The same invariant idea extends to three groups. Suppose an array contains only 0, 1 and 2 and we want it sorted in one pass without counting. Three pointers cut the array into four regions, and the invariant describes each of them.

```cpp
int low = 0, mid = 0, high = n - 1;
// [0, low)      all zeros
// [low, mid)    all ones
// [mid, high]   not yet looked at
// (high, n)     all twos
while (mid <= high) {
    if (a[mid] == 0)      swap(a[low++], a[mid++]);
    else if (a[mid] == 1) mid++;
    else                  swap(a[mid], a[high--]);
}
```

The detail that people get wrong is that `mid` moves forward after swapping with `low` but not after swapping with `high`. The element that arrives from `low` is known to be a one, because everything between `low` and `mid` has already been classified as a one, so it can safely be passed over. The element that arrives from `high` comes from the unexamined region and could be anything, so `mid` has to stay where it is and look at it. 

### Opposite-end pointers on a greedy comparison

The sorted array gave us a way to know what a comparison meant. Many problems have no sorted order, but they still allow opposite-end pointers because one of the two sides is a **bottleneck**, meaning it limits the result so strongly that its fate can be decided without looking at the other side's details.

Suppose we are given the heights of vertical bars standing at positions `0` to `n-1`, and we choose two of them. The area they enclose is the smaller of the two heights multiplied by the distance between them, and we want the largest possible area.

```cpp
int l = 0, r = n - 1;
long long best = 0;
while (l < r) {
    best = max(best, (long long)min(h[l], h[r]) * (r - l));
    if (h[l] < h[r]) l++;
    else             r--;
}
```

Start with the pointers at the two ends, which gives the largest possible width. The shorter of the two bars decides the height of the area, and the argument for moving it goes as follows. If we keep the shorter bar at `l` and pair it with any bar closer than `r`, the width is smaller and the height is still at most `h[l]`, so the area cannot be better than the one we just recorded. The shorter bar has therefore already given its best answer, and it can be discarded. Moving the taller bar instead would never help, because the height stays limited by the shorter one while the width only shrinks.

The same idea solves the classic water trapping problem. Given the heights of bars of width one, the water that sits above a bar equals the smaller of the tallest bar on its left and the tallest bar on its right, minus the bar's own height. The direct solution builds two arrays of running maximums. With opposite-end pointers we can avoid them, because only the smaller of the two running maximums matters and the smaller side is the bottleneck.

```cpp
int l = 0, r = n - 1, lm = 0, rm = 0;
long long water = 0;
while (l < r) {
    lm = max(lm, h[l]);      // tallest bar seen from the left so far
    rm = max(rm, h[r]);      // tallest bar seen from the right so far
    if (lm < rm) { water += lm - h[l]; l++; }
    else         { water += rm - h[r]; r--; }
}
```

When `lm < rm`, the true tallest bar to the right of position `l` is at least `rm`, which is larger than `lm`, so the water above `l` is limited by the left side alone and equals `lm - h[l]`. That value is final, and `l` can move on. When `lm >= rm` the same reasoning holds mirrored for position `r`. Every step settles exactly one position, and the order in which positions are settled does not matter because each one is settled using only information that is already known to be decisive.

### Monotone boundary pointers

A different use of two pointers appears when, for each position, we are looking for a boundary on the other side, and that boundary only ever moves in one direction as we go along. Take a sorted array and a number `d`, and count the pairs `i < j` whose difference `a[j] - a[i]` is at most `d`.

```cpp
long long countAtMost(const vector<int>& a, int d) {
    long long cnt = 0;
    int i = 0;
    for (int j = 0; j < (int)a.size(); j++) {
        while (a[j] - a[i] > d) i++;       // smallest i that still works with j
        cnt += j - i;                      // every position from i to j-1 works
    }
    return cnt;
}
```

For each `j`, the positions `i` that pair validly with it form a block ending at `j - 1`, and the pointer `i` marks where that block starts. As `j` increases, `a[j]` does not decrease, so any `i` that was too far away for the previous `j` is certainly too far away for this one. That is the **monotonicity** that lets the pointer `i` only ever move forward, which makes the total work linear even though there is a loop inside a loop, because the inner loop's total number of steps over the whole run is at most `n`.

It is useful to be clear about how this differs from a sliding window, since the code looks almost identical. A window carries the contents of the range between the pointers, for example a count or a set, and removes elements as the left edge moves. Here nothing about the range is stored. The pointer `i` is only a remembered answer to the question "where does the valid block for this `j` begin", and we are allowed to reuse the previous answer because that question has a monotone answer. The alternative is to run a binary search for every `j`, which also works and costs an extra logarithmic factor. The pointer removes that factor, but it can only be used after you have checked that the boundary really is monotone, which fails as soon as negative numbers or an unsorted order appear.

### Two pointers inside a binary search

The monotone pointer pass is a counting tool, and counting tools are what a binary search on the answer needs. Suppose we have a sorted array of `n` numbers and we look at all the `n(n-1)/2` differences between pairs, and we want the `k`-th smallest of them. Writing those differences out would need quadratic memory, so we never list them. Instead we ask a counting question: for a value `x`, how many pairs have a difference of at most `x`. That count grows as `x` grows, which is exactly the property a binary search needs, and each count is one linear pass with the code above.

```cpp
sort(a.begin(), a.end());
int lo = 0, hi = a.back() - a.front();
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (countAtMost(a, mid) >= k) hi = mid;     // enough pairs, the answer is mid or smaller
    else                          lo = mid + 1; // too few pairs, the answer is larger
}
return lo;
```

The loop finds the smallest `x` for which at least `k` pairs have a difference of at most `x`. That value is automatically a difference that really occurs, because if no pair had a difference of exactly `x` then the count at `x - 1` would be the same as the count at `x` and the search would have chosen the smaller number. The total cost is `O(n log(range))`, which is far better than anything that tries to list the pairs. The general lesson, which is also used in chapter [[04 Binary Search on the Answer]], is that when the feasibility check of a binary search is a counting question over pairs, two pointers is often the way to make that check linear.

### Greedy two-pointer optimisation via domination

The sections above are all instances of one principle, and this section states it directly. Say that a candidate **A dominates** a candidate **B** if A is at least as good as B no matter what happens in the future, so that B can be removed from consideration permanently without ever changing the final answer. A dominance based algorithm keeps only the candidates that are not dominated, and if each step removes at least one candidate, the algorithm is linear. In the sorted pair problem the dominated candidate was an element too small for any remaining partner, in the bars problem it was the shorter bar, and in the water problem it was the position on the side with the smaller running maximum.

Dominance has two parts that both have to hold, and forgetting the second one is the most common way to go wrong. The first is **value**: A must be at least as good as B for every possible continuation. The second is **lifetime**: A must remain available for at least as long as B does. When candidates can expire, as in a window or under a distance limit, an older candidate expires sooner, so a newer candidate with a value at least as good dominates it, but an older candidate with a better value does not dominate a newer one because it may expire first.

To make this concrete, here is the checklist to run through before trusting any pointer move. Name the candidate being discarded. Name the candidate that dominates it. Convince yourself that the dominance holds for every future the algorithm can still reach, including expiry. And confirm that each step discards at least one candidate, since that is what makes the algorithm linear.

**Expanding from a fixed position.** Given an array and an index `k`, we want a subarray that contains position `k` and has the largest value of (smallest element) multiplied by (length). We start with the single element at `k` and repeatedly extend one step to the left or one step to the right.

```cpp
int l = k, r = k, mn = a[k];
long long best = a[k];
while (l > 0 || r < n - 1) {
    if (l == 0)                 r++;
    else if (r == n - 1)        l--;
    else if (a[l-1] > a[r+1])   l--;      // extend toward the larger neighbour
    else                        r++;
    mn = min({mn, a[l], a[r]});
    best = max(best, (long long)mn * (r - l + 1));
}
```

Extending toward the larger neighbour is correct because, for every possible length, this procedure produces the window containing `k` with the largest possible minimum. Taking the smaller neighbour would lower the minimum to at most that smaller value, whereas taking the larger neighbour keeps it at least as high, and the window of that length that took the smaller neighbour is dominated. Since the score depends only on the minimum and the length, the best window of each length is the only one worth considering.

**Eliminating starting points.** Suppose there are `n` stations arranged in a circle. At station `i` we collect `gain[i]` fuel, and travelling to the next station costs `cost[i]`. We want a station to start at, with an empty tank, such that we can go all the way around, or to learn that none exists.

```cpp
int total = 0, tank = 0, start = 0;
for (int i = 0; i < n; i++) {
    int d = gain[i] - cost[i];
    total += d;
    tank += d;
    if (tank < 0) { start = i + 1; tank = 0; }     // every start from the old one up to i fails
}
return total >= 0 ? start : -1;
```

This works because of a domination argument that discards many candidates at once. If we start at `s` and the tank first becomes negative at station `i`, then any station `s'` strictly between `s` and `i` is also a bad start. When we travelled from `s` we arrived at `s'` with a tank that was not negative, so starting fresh at `s'` with an empty tank is never better than what we had, and since we ran out by `i` starting from `s`, we would also run out by `i` starting from `s'`. All of those starts are removed with a single jump of the pointer to `i + 1`. The final check on `total` handles the other half of the argument, which is that if the whole circle has enough fuel overall then the last surviving start must work.

**Pairing with the best partner.** Suppose people have the given weights and each boat carries at most two people with a combined weight of at most `limit`. We want the smallest number of boats.

```cpp
sort(w.begin(), w.end());
int l = 0, r = n - 1, boats = 0;
while (l <= r) {
    if (w[l] + w[r] <= limit) l++;      // the lightest person shares the boat
    r--;                                // the heaviest person always leaves
    boats++;
}
return boats;
```

Look at the heaviest remaining person. If this person cannot share a boat with the lightest person, then they cannot share with anyone, so they must travel alone. If they can share with the lightest person, then pairing them with the lightest is at least as good as pairing them with anyone else, because the lightest person is the easiest to fit in every possible later situation. In both cases the heaviest person is settled, which is another form of domination.

**Collapsing a deque into a variable.** The monotonic deque from the earlier part of this chapter is itself a dominance structure, because when a new element is at least as large as an older one it dominates it, being both better and available for longer, so the older one is popped. The deque therefore keeps a chain of non-dominated candidates. Consider choosing two positions `i < j` to maximise `a[i] + a[j] + i - j`, which can be rewritten as `(a[i] + i) + (a[j] - j)`. If there is no restriction on how far apart `i` and `j` may be, nothing ever expires, so the single best earlier value dominates all the others and one variable is enough.

```cpp
int bestLeft = a[0] + 0, ans = INT_MIN;
for (int j = 1; j < n; j++) {
    ans = max(ans, bestLeft + a[j] - j);
    bestLeft = max(bestLeft, a[j] + j);
}
```

If we add the restriction `j - i <= k`, then older candidates do expire, so an older and better candidate is no longer safe to discard, and we need the full chain that the deque maintains.

```cpp
deque<int> dq;                                   // indices, with a[i] + i decreasing
int ans = INT_MIN;
for (int j = 0; j < n; j++) {
    while (!dq.empty() && dq.front() < j - k) dq.pop_front();     // expired
    if (!dq.empty()) ans = max(ans, a[dq.front()] + dq.front() + a[j] - j);
    while (!dq.empty() && a[dq.back()] + dq.back() <= a[j] + j) dq.pop_back();
    dq.push_back(j);
}
```

Seeing these two versions next to each other is the clearest way to understand when a data structure is necessary. The deque is the general solution and the single variable is what it shrinks to when the lifetime part of dominance disappears.

### Choosing between them

| What the problem looks like | Shape to try |
|---|---|
| A sorted array and a question about pairs or a fixed number of elements | Opposite-end pointers on a sorted array |
| Rearranging or filtering an array in place with constant extra memory | Read and write pointers |
| Two ends and a bottleneck, with no sorted order | Opposite-end pointers on a greedy comparison |
| For each position, a boundary on the other side that only moves one way | Monotone boundary pointers |
| Finding the best value where counting pairs below a threshold is easy | Two pointers inside a binary search |
| Candidates that can be proven useless once something better appears | Domination |

### Where people lose these problems

**Using `l <= r` where `l < r` is needed.** In pair problems the two pointers must refer to different elements, and equality lets an element pair with itself. The boats problem is the opposite case, where a single remaining person is a legitimate case and `l <= r` is correct, so the condition has to be derived from the problem each time.

**Forgetting to sort.** Opposite-end pointers on a sorted array have no proof without the order. If indices must be returned, sort pairs of value and position.

**Skipping duplicates incorrectly.** In the three-number problem the skip must happen after recording a match and for the fixed element as well, otherwise the same triple appears several times, and a skip that is placed before the match is recorded can lose valid answers.

**Moving the wrong pointer on a tie.** When the two compared values are equal, check that the argument still holds for the pointer you move. In the bars problem either pointer is safe on a tie, and in the water problem the tie goes to the right side, but this has to be checked, not assumed.

**Advancing `mid` after swapping with `high`.** The element that arrives is unexamined, so it has to be looked at before `mid` moves.

**Assuming the boundary is monotone.** A monotone boundary pointer needs the underlying order to be monotone. With negative numbers in a sum, or with an unsorted array, the pointer would move past positions that later turn out to be needed.

**Overflow.** Sums of two or three values and products of a height and a width can leave the 32-bit range, so store them in `long long`.

**Using domination without the lifetime check.** If candidates expire, a better but older candidate is not safe to keep as the only one, and the solution fails exactly on the inputs where the best candidate is far away.

### Check yourself

1. State the shrinking condition. Give a specific condition where it fails, and say what you would use instead.
2. Write the two window shapes side by side. Where does each record the answer, and why?
3. Explain in one sentence why `res += r - l + 1` counts every valid run exactly once.
4. Why can "exactly k distinct" not be handled by a window directly, and what is the way around it?
5. Why does the monotonic deque store indices?
6. Give the argument for why the deque scan is linear, and name two other structures on this sheet that rely on the same argument.
7. How many distinct values can the AND of a run take as the left end varies for a fixed right end, and why?
8. Rewrite `y_i + y_j + |x_i - x_j|`, for `i < j`, as a term depending only on `i` plus a term depending only on `j`.
9. What reframing turns LC 2009 into a window problem?
10. In a sorted pair search, why is it safe to discard `a[l]` when `a[l] + a[r]` is below the target? State the argument in one sentence.
11. How does counting pairs with sum at most a target discard `a[l]` while adding `r - l` to the count?
12. State the invariant of a read and write pointer pair, and explain why the writer can never overwrite an element that has not been read.
13. In the three-group partition, why does `mid` advance after a swap with `low` but not after a swap with `high`?
14. Why is the shorter bar the one to discard in the most-water problem? Why does moving the taller bar never help?
15. In the water trapping version with two pointers, why is the water above position `l` final when `lm < rm`?
16. What is the difference between a monotone boundary pointer and a sliding window, and what must be verified before using the pointer?
17. Why is the smallest `x` with at least `k` pairs of difference at most `x` guaranteed to be a real pair difference?
18. Define domination. What are its two parts, and which one is forgotten when a window or distance limit is present?
19. Why can the circular route search skip every start between the failing start and the failing station?
20. When does the monotonic deque collapse into a single variable, and what makes it necessary again?
