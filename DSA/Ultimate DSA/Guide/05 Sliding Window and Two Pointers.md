---
tags: [dsa, guide, sliding-window, two-pointers]
chapter: 5
sheet-section: E
---

# Chapter 5 · Sliding Window & Two Pointers, Hard Variants

> **Read this before you start the problems.** Each technique is introduced with a small example, so no prior familiarity is assumed.

Back to [[00 Guide Index]] · Sheet section **E** in [[1. Ultime DSA 2026 calibration]] · Builds on your note [[2. Sliding window]]

---

## What makes these problems hard

A sliding window works because of a promise: the left edge of the window never moves backwards. That promise is what makes the whole scan linear, since each element enters the window once and leaves it once.

The promise is only safe under a particular condition, which is that shrinking a valid window always leaves it valid. When that holds, an invalid window can always be repaired by advancing the left edge, and there is never a reason to reconsider a position you have already passed. When it does not hold, the technique produces answers that look plausible and are wrong.

Most of the difficulty in the harder problems in this block comes from two places. The first is that the condition quietly fails, most often because negative numbers are permitted, and the window has to be replaced with something else. The second is that the quantity being maintained inside the window is no longer a simple count or sum, so you have to decide what structure to carry and how to add to and remove from it efficiently.

There is also a shift in how these problems are phrased. The older form asks for the longest or shortest window satisfying a condition. The current form more often asks you to count how many windows satisfy it, which requires a different counting argument even when the window mechanics are identical.

---

## What these problems look like

The signal is that the problem concerns contiguous pieces of an array or string. Once you have noticed that, the useful question is which of four things it is:

- **Longest or shortest run satisfying a condition** points to a sliding window, provided the shrinking condition holds.
- **Counting runs satisfying a condition** points to a window combined with the decomposition in Part 3, or to prefix sums with a hash map from chapter [[06 Prefix Sums and Difference Arrays]].
- **An aggregate over every window of a fixed size** points to a monotonic deque, or a heap, or a multiset.
- **Anything involving sums where negative values are permitted** usually points away from windows entirely and towards prefix sums.

That last one is the most useful disqualifier to have in mind, because it is the case where a window looks applicable and is not.

---

## Part 1 · The condition that makes a window valid

The technique applies when shrinking a valid window leaves it valid, or equivalently when growing an invalid window leaves it invalid.

Take the condition "at most `k` distinct characters." If the window from `l` to `r` contains at most `k` distinct characters, then the window from `l + 1` to `r` certainly does too, since removing a character cannot increase the number of distinct ones. Shrinking therefore never causes harm, which means that when a window becomes invalid, advancing the left edge is guaranteed to eventually fix it, and no position ever needs revisiting.

The condition fails for "longest run with sum at most `K`" when negative numbers are allowed, because adding an element can decrease the sum. An invalid window can become valid by growing, so the structure the technique relies on is absent. This is why LC 862, which asks for the shortest run with sum at least `K` and permits negatives, is not a window problem at all and instead needs a monotonic deque over prefix sums.

It is worth completing the sentence "if the window from `l` to `r` is valid then the window from `l + 1` to `r` is valid, because ..." before writing anything, in the same way as the consistency check in chapter [[04 Binary Search on the Answer]]. It takes a few seconds and distinguishes a correct approach from a plausible one.

---

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

LC 76 Minimum Window Substring is the second shape, where the validity test asks whether all required characters are present. Rather than re-scanning a frequency map on every step, the standard approach keeps a counter of how many distinct required characters have reached their required count, incremented only at the moment a character's count first reaches its requirement. That makes the validity test constant time.

---

## Part 3 · Counting windows rather than measuring them

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

---

## Part 4 · Maintaining a maximum or minimum with a deque

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

---

## Part 5 · Windows carrying a real structure

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

**Bitwise operations over a window.** LC 1521, LC 898 and LC 2419 look like window problems but rely on a different property. As a run is extended, its running AND can only lose bits and its running OR can only gain them, and each change removes or adds at least one bit permanently. Since there are only about thirty bits, the running AND takes at most about thirty distinct values across all left endpoints for a fixed right endpoint.

That means you can carry the small set of distinct values forward:

```cpp
vector<int> cur;                       // distinct AND-values of runs ending at r; about 30 entries
for (int r = 0; r < n; r++) {
    for (int& v : cur) v &= a[r];
    cur.push_back(a[r]);
    cur.erase(unique(cur.begin(), cur.end()), cur.end());
    for (int v : cur) best = min(best, abs(v - target));
}
```

This is not a sliding window at all, and it is grouped here because the problems appear in the same section. The underlying observation recurs in chapter [[24 Bitwise and XOR Basis]] and is worth treating as its own technique.

---

## The ideas worth carrying forward

1. **A window requires that shrinking preserves validity.** Completing the sentence "if `[l, r]` is valid then `[l+1, r]` is valid, because ..." takes a few seconds and identifies the cases where the technique does not apply.

2. **Negative values in a sum condition usually break the window.** That combination points to prefix sums instead.

3. **The two shapes differ by one character, and by where the answer is recorded.** The longest-window shape records after restoring validity, and the shortest-window shape records before breaking it.

4. **`res += r - l + 1` counts every valid run exactly once, at its right end.** This single line does the counting work in most counting problems in this block.

5. **Counting runs with exactly `k` of something becomes two runs of an "at most" window**, because "at most" satisfies the shrinking condition while "exactly" does not.

6. **The counting-at-the-right-end principle transfers even when the decomposition does not.** LC 2444 uses different bookkeeping and the same accounting.

7. **A monotonic deque stores indices**, since the index is what determines when an element leaves the window.

8. **Two deques side by side give both the maximum and the minimum in linear time**, and a `multiset` gives the same thing in three lines with an extra logarithmic factor.

9. **Each element entering and leaving once is what makes these linear.** The same argument covers deques, monotonic stacks and disjoint-interval maps.

10. **A running AND or OR over an extending run changes only about thirty times**, so the distinct values form a small set that can be carried forward.

11. **A window can carry any aggregate**, including a frequency map, a multiset, two heaps or a monotonic deque. Writing your solutions with explicit `add`, `remove` and `valid` helpers makes the mechanics identical across problems and makes the code easier to debug.

---

## Where people lose these problems

**Applying a window to a sum problem with negative values.** This is the characteristic failure of the category. When negatives are possible and the condition concerns a sum, chapter [[06 Prefix Sums and Difference Arrays]] is the right place to look.

**Shrinking with an `if` where a `while` is needed.** One new element can require several removals before the window becomes valid again.

**Recording the answer at the wrong point in the shortest-window shape.** The record has to happen inside the shrinking loop, before the removal that breaks validity.

**Leaving zero-count entries in a frequency map.** If the number of distinct values is being read from the map's size, entries must be erased when their count reaches zero rather than merely decremented. Keeping a separate integer count is safer and faster.

**Reading the deque front before expiring stale indices.** The first several answers then include elements that have already left the window.

**Calling `multiset::erase` with a value.** That removes every matching element rather than one, and `erase(find(x))` is what was intended. This is the same trap as in chapter [[02 Intervals and Sweep Line]].

**In LC 1888, missing the doubling.** The second operation moves the first character to the end, which means the reachable strings are the windows of length `n` in the string concatenated with itself. Once that is noticed the problem becomes a fixed-size window with two counters, and without it the problem looks intractable.

**In LC 2009, working in terms of operations rather than retained elements.** After sorting and removing duplicates, the answer is the array length minus the largest number of distinct values that fit inside a window spanning `n - 1` in value. Reframing minimum changes as maximum retentions is what makes the window appear.

---

## Working through the problem list

### Block 1 · The basic mechanics

- **LC 1004 Max Consecutive Ones III** — *find the longest run of ones allowing up to k zeros to be flipped.* The longest-window shape in its simplest form.
- **LC 76 Minimum Window Substring** — *find the shortest substring of s containing all characters of t.* The shortest-window shape with the counter described in Part 2. Worth repeating until it flows, since it underpins several later problems.
- **LC 239 Sliding Window Maximum** — *report the maximum in every window of size k.* The deque template, worth being able to write in about ninety seconds.
- **CSES Subarray Distinct Values** — *count runs containing at most k distinct values.* Direct practice for the counting line.

### Block 2 · Counting runs

- **LC 992 Subarrays with K Different Integers** — *count runs containing exactly k distinct integers.* The decomposition from Part 3, best attempted straight after the CSES problem above so the difference is visible.
- **LC 2537 Count the Number of Good Subarrays** — *count runs containing at least k pairs of equal elements.* This condition behaves consistently on its own, since adding elements only creates pairs, so a single window suffices and the counting is done from the left end instead. A useful contrast.
- **LC 2444 Count Subarrays With Fixed Bounds** — *count runs whose minimum and maximum are exactly the given values.* Three tracked positions rather than a decomposition.
- **LC 3234 Count the Number of Substrings With Dominant Ones** — *count substrings where the number of ones is at least the square of the number of zeros.* Relies on the number of zeros in a valid substring being small. Genuinely difficult and best saved for last.

### Block 3 · Windows carrying structures

- **LC 1438 Longest Continuous Subarray With Absolute Diff ≤ Limit** — *find the longest run whose largest and smallest elements differ by at most a limit.* Two deques, or a multiset. Worth doing both ways.
- **LC 2762 Continuous Subarrays** — *count runs where any two elements differ by at most two.* The same structure as 1438, combined with the counting line.
- **CSES Sliding Median** — *report the median of every window of size k.* The multiset-with-iterator approach.
- **LC 480 Sliding Window Median** — *the same problem in LeetCode form*, with overflow to watch for when averaging two middle values.
- **CSES Sliding Cost** — *report the minimum total cost of making every element in each window equal.* The median structure with two running sums, and the payoff for building it carefully.
- **CSES Maximum Subarray Sum II** — *find the largest sum over runs whose length lies between a and b.* Prefix sums with a deque holding the smallest prefix values in the permitted range. This bridges into chapter [[06 Prefix Sums and Difference Arrays]] and teaches the technique that LC 862 also needs.

### Block 4 · Reframing

- **LC 2009 Minimum Number of Operations to Make Array Continuous** — *change the fewest elements so that the array holds n consecutive distinct values.* The retention reframing described above.
- **LC 1888 Minimum Flips to Make Alternating** — *make a binary string alternate, using flips and rotations.* The doubling reframing.
- **LC 1793 Maximum Score of a Good Subarray** — *find the run containing index k maximising its minimum times its length.* Expand outwards from `k`, always extending towards the larger neighbour. A two-pointer scan starting from the middle, which is an under-used shape.
- **LC 1499 Max Value of Equation** — *maximise `y_i + y_j + |x_i - x_j|` subject to a limit on `|x_i - x_j|`.* Rewriting the expression for `i < j` as `(y_i - x_i) + (y_j + x_j)` turns it into a maximum-deque over `y - x` inside a window. The rewrite is the whole problem, and separating an expression into a part depending on `i` and a part depending on `j` is a move that reappears in chapter [[17 DP Optimization]].
- **LC 1521 Find a Value of a Mysterious Function Closest to Target** — *find the run whose AND is closest to a target.* The bounded-distinct-values technique from Part 5.
- **CF 6E Exposition** — *find the longest run whose height range is within a limit.* The Codeforces form of LC 1438.

---

**A reasonable target here is around 80% of submissions passing first time.**

The mechanics are simple and the ways to go wrong form a short list. When accuracy falls below that, the cause is usually either confusion between the two shapes or a shrinking condition that was never checked.

---

## Check yourself

1. State the shrinking condition. Give a specific condition where it fails, and say what you would use instead.
2. Write the two window shapes side by side. Where does each record the answer, and why?
3. Explain in one sentence why `res += r - l + 1` counts every valid run exactly once.
4. Why can "exactly k distinct" not be handled by a window directly, and what is the way around it?
5. Why does the monotonic deque store indices?
6. Give the argument for why the deque scan is linear, and name two other structures on this sheet that rely on the same argument.
7. How many distinct values can the AND of a run take as the left end varies for a fixed right end, and why?
8. Rewrite `y_i + y_j + |x_i - x_j|`, for `i < j`, as a term depending only on `i` plus a term depending only on `j`.
9. What reframing turns LC 2009 into a window problem?
