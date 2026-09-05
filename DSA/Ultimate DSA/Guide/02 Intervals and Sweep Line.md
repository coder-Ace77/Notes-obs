---
tags: [dsa, guide, intervals, sweep-line, scheduling]
chapter: 2
sheet-section: B
---

# Chapter 2 · Intervals, Sweep Line & Scheduling

---

## What makes these problems hard

Interval problems are difficult mainly because the obvious way to store the input is the wrong way to reason about it. When you are given a list of pairs, it feels natural to keep them as pairs and compare them against each other, which leads to nested loops over every pair of intervals and to a growing number of cases about which one starts first.

The technique that removes almost all of that difficulty is to stop treating an interval as a single object. An interval `[l, r)` is better understood as two separate moments in time: one where something switches on at `l`, and one where it switches off at `r`. Once every interval has been broken into two events and all the events have been sorted together, most questions about overlap become a single pass with a running counter.

The remaining difficulty is that this transformation has a small number of conventions attached to it, and getting one of them wrong produces answers that are correct on the examples and wrong on the tests. Most of this chapter is about those conventions and about what to carry through the sweep once a counter is no longer enough.

This is the most frequent idea in product online assessments, and it is almost never described as an interval problem. It normally arrives wrapped in a story:

- **Calendars and bookings.** Whether a meeting can be scheduled, how many rooms are needed, which hour is busiest.
- **Resource allocation.** Servers, licences, parking spaces, taxis, or anything else that is occupied from one time to another.
- **Geometry that is not really geometry.** Rectangles in a plane are intervals in `x` that exist for a range of `y` values, and a skyline is a set of intervals each carrying a height. The sweep in these is a literal vertical line moving from left to right.
- **Ranges over an array.** Applying an increment to positions `l` through `r`, repeated many times, then printing the array.
- **Version histories.** Questions of the form "what was the state at time `t`", answered by processing the queries in time order.


### Turning intervals into events

The core transformation applies to any question about how many intervals are active at once.

Instead of sorting the intervals, break each one into two events: an event at `l` that adds one, and an event at `r` that subtracts one. Sort all `2n` events by coordinate and walk through them while keeping a running total. The largest value that total ever reaches is the maximum number of simultaneously active intervals.

```cpp
vector<pair<int,int>> ev;                  // (coordinate, change)
for(auto& [l, r] : intervals){
    ev.push_back({l, +1});
    ev.push_back({r, -1});
}
sort(ev.begin(), ev.end());
int cur = 0, best = 0;
for (auto& [x, d] : ev) { cur += d; best = max(best, cur); }
```

Those eight lines solve CSES *Restaurant Customers*, LC 253 Meeting Rooms II, and form the skeleton of most of the rest of this chapter.

The one subtlety is what happens when two events share a coordinate. Consider the intervals `[1, 5]` and `[5, 9]`. Whether these count as overlapping depends on the problem: a meeting that ends at 5 frees the room for a meeting starting at 5, so those two do not conflict, whereas two intervals of covered ground that meet at a point may well be considered to overlap.

Because `pair` sorts by its first element and then its second, and `-1` sorts before `+1`, the code above processes removals before additions at a shared coordinate. That means touching intervals are treated as not overlapping, which is usually what is wanted. If the problem needs the opposite convention, sorting by `(coordinate, -change)` reverses it.

The reason to state this explicitly rather than to try both and see which passes is that the difference only shows up on inputs containing touching intervals, which the sample cases frequently do not.

### Half-open intervals

In general there are two kinds of intervals in the problems. First type is having that when if two intervals share a common point will mean they overlapp for example say interval `[1,2]` and `[2,3]` overlapp in this case choose closed intervals as your representation this means if any pair has common point they are said to collide.

Second issue is when two intervals with common point are said to touch at boundry but not collide. In this case we employ a trick each interval will be represented in its open interval form. This is done so that `I=[l,r)` means l is in interval but r is not. This is common trick in these questions and is worth knowing about. 


### Selecting intervals: sort by the right endpoint

A common question is to select as many pairwise non-overlapping intervals as possible. The algorithm is to sort by the *right* endpoint in increasing order, then take each interval whose left endpoint is at or after the right endpoint of the last one taken.

This covers CSES *Movie Festival*, LC 435 (which asks for the fewest removals, the same thing phrased in reverse), and LC 452 (where arrows correspond to groups of overlapping balloons).

It is worth understanding why the right endpoint is the correct sort key, because the alternatives are plausible and wrong.

Let `G` be the greedy solution and `O` an optimal one, both listed in order of right endpoint, and suppose they agree on the first `k` choices and differ afterwards. Greedy picked `g` and the optimal solution picked `o`. Greedy always selects the compatible interval with the smallest right endpoint, so `g.r <= o.r`. Now replace `o` with `g` in the optimal solution. The intervals before this point are identical in both and were compatible with `o`, and `g` was chosen to be compatible with them. Every interval after this point in `O` begins at or after `o.r`, which is at or after `g.r`, so those remain compatible too. The modified solution is the same size and agrees with greedy one step further along, and repeating the argument shows greedy is optimal.

The intuition behind the proof is that finishing as early as possible leaves the most room for whatever comes next. Sorting by left endpoint is the right choice when merging intervals, and sorting by right endpoint is the right choice when selecting them.

### Carrying a container through the sweep

A counter is enough when you only need to know how many intervals are active. When you need to know which ones, or the largest value among them, the counter becomes a container.

```cpp
sort(jobs.begin(), jobs.end());                       // by start time
priority_queue<int, vector<int>, greater<>> active;   // min-heap of end times
for (auto& [s, e] : jobs) {
    while (!active.empty() && active.top() <= s) active.pop();   // expire finished jobs
    active.push(e);
    answer = max(answer, (int)active.size());
}
```

The heap version generalises where the counter version does not. LC 2402 Meeting Rooms III needs two heaps, one holding the indices of free rooms and one holding `(end time, room index)` pairs for occupied rooms, because the problem requires knowing which specific room frees up first and, among free rooms, taking the lowest-numbered one.

A `multiset` is worth knowing as an alternative, because unlike a heap it supports removing an arbitrary element:

```cpp
multiset<int> active;
active.insert(x);
active.erase(active.find(x));   // removes ONE copy
int mx = *active.rbegin();
```

#### Map and flooring

Maps can used to do the floor operation `floor(x)` which is to find the largest key atmost `x`. Now note that the pattern is to do upper_bound(x) first which gives you the smallest key greater than `x` and going one key back we get largest key atmost `x`. Now don't confuse with `lower_bound(x)` which will give smallest key atleast `x`. This is why lower bound is not used. 

So the reason is: `upper_bound` strictly excludes `x`, meaning wherever it lands, stepping back one is _guaranteed_ to land on the largest key that is ≤ x  whether that key is `x` itself or something smaller. `lower_bound` doesn't have this clean guarantee because when `x` is present, it points _at_ `x`, not past it, so decrementing overshoots.

The usual code looks like this --

```cpp
auto it = mp.upper_bound(x);
if (it == mp.begin()) {
    // no key <= x exists
} else {
    --it; // now it->first is the largest key <= x
}
```

Note that we don't check past end because iterators support `--` as long as you're not decrementing `begin()` (there's nothing before the first element). Note in empty map end and begin iterator are equal. 

### Maintaining a set of disjoint intervals

A group of problems asks you to add ranges, remove ranges, and query whether a range is covered, all interleaved. To solve these problem we main set of intervals which are disjoint and non overlapping with the open interval `[l,r)` mode. We will be using `map<int,int>` to trakc the intervals. 

Now suppose we have set of such intervals and make a good look at open property here. Now lets discuss some of the queries --

- Finding if every number in the range is covered. We can simply find interval whose left point is at `l`. Now due to open property this interval  has to extend till end if we want to cover ther interval `[l,r)`. Reason is only one interval will be present exactly in the range if there were two we will get a break point and due to open property boundry will not be covered. 

```cpp
bool qry(int l,int r){
	// using upper bound and move back
	auto it = intervals.upper_bound(l); // uppoer bound strictly more than l
	if(it==intervals.begin()) return false; // no interval since first interval is returned  and it is greater than what we want so meaning no interval is present for our use
	
	auto prv = --it;
	
	if(prv->second>=r)return true; // since only this is interval check if this extends upto end or not
	return false;
}
```

- Finding total number of disjoint intervals present is pretty trivial. Its exactly the size of map.
- Finding the total gaps in the set is simply `size of map` minus 1. And its pretty straightforward no two intervals are overlapping due to property. 

Now we need to see how to handle `add` and `remove`  part. Before this its worth mentioning about that `[1,3)+[3,5)` is equal to `[1,5)` and its the beauty of open intervals. So in reality the mp will never hold things like `[1,2)` and `[2,3)` if two intervals are there will always be the gap of atleast 1 between them. Meaning next interval will start from `[r+1` or beyond. 

Add operation's `addRange(l,r)` first step is to find out all the intervals which can merge with this interval and finally store the union of all. Now note that mp had all the intervals stored in non-overlapping form meaning we need to find the last interval whose left end point starts before `left`. Now we start iterating on intervals and merge them untill `it->first<=right`  meaning left end point of interval extends upto right. 

```cpp
void addRange(int left, int right) {
	auto it = mp.upper_bound(left);
	if (it != mp.begin()) {
		auto prev = std::prev(it);
		if (prev->second >= left) {
			it = prev; 
		}
	}

	while (it != mp.end() && it->first <= right) {
		left = min(left, it->first);
		right = max(right, it->second);
		it = mp.erase(it); 
	}
	mp[left] = right;
}
```

Now about delete This is the trickiest one because removing the _middle_ of a tracked interval **splits it into two**.

```cpp
void removeRange(int left, int right) {
    auto it = mp.upper_bound(left);
    if (it != mp.begin()) {
        auto prev = std::prev(it);
        if (prev->second > left) {
            it = prev; // this interval extends into [left, right), must process it
        }
    }
    vector<pair<int,int>> toAdd; // pieces to re-insert after splitting
    while (it != mp.end() && it->first < right) {
        int s = it->first, e = it->second;
        if (s < left) toAdd.push_back({s, left});   // left leftover piece survives
        if (e > right) toAdd.push_back({right, e}); // right leftover piece survives
        it = mp.erase(it);
    }
    for (auto& p : toAdd) mp[p.first] = p.second;
}
```

### Compressing coordinates and sweeping with a segment tree

Collect every coordinate that appears anywhere in the input, sort them, remove duplicates, and use each coordinate's index in that sorted list as the index into your structure.

```cpp
vector<int> xs;                       // every l and every r from every interval
sort(xs.begin(), xs.end());
xs.erase(unique(xs.begin(), xs.end()), xs.end());
auto idx = [&](int v) { return lower_bound(xs.begin(), xs.end(), v) - xs.begin(); };
```

The point that causes errors is that slot `i` of the structure no longer represents a single position but the real interval `[xs[i], xs[i+1])`, whose length is `xs[i+1] - xs[i]` rather than 1. With `k` distinct coordinates there are `k - 1` such slots, not `k`. Drawing four points on a line and counting the gaps between them makes the reason immediate, and doing that before sizing the structure is quicker than debugging it afterwards.

LC 850 Rectangle Area II is the standard application. A horizontal line sweeps upward through the `y` coordinates; at each event an `x`-interval is added to or removed from a segment tree that reports total covered length; and that length multiplied by the gap to the next `y` event contributes to the area. The specific tree needed for this is described in chapter [[08 Segment Trees]], since its node does not use lazy propagation in the usual way.

LC 699 Falling Squares uses the same sweep with a maximum-assignment tree instead.

### When intervals carry values

Everything above assumes that intervals are interchangeable. As soon as each interval has a value and you want the maximum total value, the greedy approach stops working, and the problem becomes dynamic programming.

Sort by right endpoint, then:

```
dp[i] = max( dp[i-1],                     // skip interval i
             value[i] + dp[ p(i) ] )      // take it; p(i) is the last interval ending at or before start[i]
```

`p(i)` is found by binary searching the sorted right endpoints, giving `O(n log n)` overall.

This covers LC 1235 Maximum Profit in Job Scheduling, CSES *Projects*, and LC 2054 Two Best Non-Overlapping Events, which is the special case where exactly two intervals may be chosen.

Recognising that the presence of a value field is what switches the problem from greedy to DP saves several minutes, because it means you do not spend them looking for a sort order that cannot exist.

## The ideas worth carrying forward

1. **An interval is better handled as two events.** Breaking `[l, r)` into an addition at `l` and a subtraction at `r`, then sorting all events together, converts most overlap questions into a single pass.

2. **Half-open intervals remove a class of errors.** Lengths, adjacency and merging all work without adjustment, so converting the input once at read time is worth doing by default.

3. **Ties at a shared coordinate decide whether touching counts as overlapping.** Sorting `(coordinate, change)` puts removals first, which treats touching as non-overlapping. Choose the convention deliberately, since sample cases rarely distinguish them.

4. **Sort by right endpoint when selecting, by left endpoint when merging.** Selecting works because finishing early leaves the most room for later intervals, which the exchange argument in Part 3 makes precise.

5. **Unweighted selection is greedy and weighted selection is dynamic programming.** The presence of a value on each interval is the signal that switches between them.

6. **A map of disjoint intervals supports add, remove and query in amortised logarithmic time**, because each stored interval can only be erased once no matter how many are erased in a single call.

7. **A difference array is a sweep with the sorting already done**, since array indices arrive in order. This is why `diff[l] += k; diff[r] -= k;` followed by a prefix sum works, and it connects this chapter to chapter [[06 Prefix Sums and Difference Arrays]].

8. **Sweeping over one dimension reduces a two-dimensional problem to a sequence of one-dimensional ones.** The structure you carry across the sweep is where the real difficulty of such problems sits.

9. **`multiset::erase(value)` removes every matching element.** Using `erase(find(value))` removes one, and the difference produces wrong answers rather than errors.

10. **Being given all the queries in advance means you may answer them in any order.** Sorting the queries to match the sweep is what makes LC 1851 straightforward, and it is worth checking for in every query problem.

## Check yourself

1. Which two events does an interval `[l, r)` become, and which sort key makes touching intervals count as non-overlapping?
2. Give the exchange argument for sorting by right endpoint, in three sentences.
3. What changes an interval selection problem from greedy to dynamic programming? Write the recurrence and say how `p(i)` is computed.
4. Write the disjoint-interval insertion from memory, then explain why it is amortised logarithmic despite the inner loop.
5. After compressing `k` distinct coordinates, how many slots does your structure need, and what does slot `i` represent?
6. What does `multiset::erase(5)` do, and what was probably intended?
7. In the Skyline problem, why must all events at a given `x` be processed before any point is emitted?
8. Name three problems in this block where sorting the queries is what makes them tractable.
