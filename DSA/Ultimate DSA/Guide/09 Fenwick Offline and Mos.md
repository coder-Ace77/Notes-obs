---
tags: [dsa, guide, fenwick, bit, offline-queries, mos-algorithm]
chapter: 9
sheet-section: I
---

# Chapter 9 · Fenwick Trees, Offline Queries & Mo's Algorithm

> **Read this before you start the problems.** Each technique comes with a small example, so no prior familiarity is assumed.

Back to [[00 Guide Index]] · Sheet section **I** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

The difficulty in this block is unusual, because the problems contain very little that is conceptually complicated. What they require is noticing a permission you have been given and have not used.

When a problem supplies all its queries in advance, you are not obliged to answer them in the order they were written. You may sort them, group them, or interleave them with a pass over the data in whatever way is convenient. That freedom is often the entire solution, because a query that is difficult to answer at an arbitrary moment becomes easy if you can arrange to be at the right point in a sweep when you answer it.

Problem setters make heavy use of this because the direct solution is obvious and runs just slowly enough to fail. With a hundred thousand elements and a hundred thousand queries, the direct approach performs around ten billion operations while the intended one performs a few million. Nothing in the statement hints at the technique, so the whole problem is recognising that reordering is available.

The second source of difficulty is that the structures involved, particularly Fenwick trees, are easy to write incorrectly in small ways. Index conventions, coordinate compression, and the requirement that adding and removing an element be exact inverses of each other all produce errors that appear only on the second or third query.

---

## What these problems look like

The signals are:

- **All queries are supplied in advance**, as an array rather than through an interactive protocol. This is the permission to reorder.
- **Queries have an obvious sort key**, such as a right endpoint, a threshold, or a time.
- **The question involves counting pairs**, counting inversions, or counting elements below a value within a range.
- **The answer would be easy if the data were processed in sorted order** rather than in index order.
- **The product of the number of elements and the square root of the number of queries fits comfortably**, at around a hundred million operations, with no better structure apparent. That indicates Mo's algorithm.

---

## Part 1 · The Fenwick tree

A Fenwick tree, also called a binary indexed tree, supports updating a single position and querying the total of a prefix, each in logarithmic time, in about six lines.

```cpp
struct BIT {
    int n; vector<long long> t;
    BIT(int n) : n(n), t(n + 1, 0) {}
    void add(int i, long long v) { for (++i; i <= n; i += i & -i) t[i] += v; }
    long long sum(int i) { long long s = 0; for (++i; i > 0; i -= i & -i) s += t[i]; return s; }
    long long range(int l, int r) { return sum(r) - (l ? sum(l-1) : 0); }   // inclusive
};
```

It is worth choosing this over a segment tree whenever the operation can be undone, which covers sums, exclusive or, and counts, and whenever prefix queries are sufficient. That describes most situations. The code is shorter to produce under time pressure and runs around three times faster in practice.

It cannot be used for minimums or maximums, since those cannot be undone, nor for range updates requiring deferred work. Those cases belong to chapter [[08 Segment Trees]].

Two extensions are worth knowing.

**Range updates with point queries.** Storing a difference array inside the tree means adding a value at the left end of a range and subtracting it just past the right end, after which the value at any position is the prefix total up to that position. This is the difference array from chapter [[06 Prefix Sums and Difference Arrays]] made to work with interleaved queries.

**Finding the k-th element by descending the tree.** To locate the smallest index whose prefix total reaches a target:

```cpp
int kth(long long k) {
    int pos = 0;
    for (int pw = 1 << 20; pw; pw >>= 1)
        if (pos + pw <= n && t[pos + pw] < k) { pos += pw; k -= t[pos]; }
    return pos;
}
```

This turns a Fenwick tree into an order-statistics structure, which removes the need for a segment tree in several problems.

---

## Part 2 · Sweeping positions while querying values

This is the most useful pattern in the chapter, and the standard example is counting inversions, meaning pairs of positions where the earlier element is larger than the later one.

Process the array from left to right. Before inserting the current element, ask how many already-inserted elements are larger than it, which is a suffix query on a tree indexed by value. Then insert it.

```cpp
// after compressing the values of a[] into the range 0 to n-1
BIT bit(n); long long inv = 0;
for (int i = 0; i < n; i++) {
    inv += bit.range(a[i] + 1, n - 1);       // already-seen values larger than a[i]
    bit.add(a[i], 1);
}
```

The way to hold this in mind is that the tree is indexed by value and its contents represent everything to the left of the current position. Advancing the sweep changes what "to the left" means, and each query asks a question about values within that set. That description covers LC 315, LC 493, LC 327, CF 61E, CF 459D and CSES *Nested Ranges Count*.

**Counting triples** extends the same idea. CF 61E asks for triples of positions in strictly decreasing order of value. For each position taken as the middle of a triple, compute how many larger values lie to its left and how many smaller values lie to its right, then sum the products. That requires two sweeps, one in each direction, and the decomposition of counting candidates on each side of a middle element is reusable. LC 2179 uses the same structure.

---

## Part 3 · Sorting queries by a threshold

When a query carries a parameter and the answer behaves consistently as that parameter grows, sorting the queries by it means the structure only ever grows and never needs anything undone.

**LC 1697** asks, for each query, whether two nodes are connected using only edges below a given weight. Sorting the edges by weight and the queries by their limit, then adding edges into a union-find structure as the sweep passes their weight, means each query can be answered by a connectivity check at that moment. This belongs to chapter [[10 DSU Advanced]].

**LC 2503** asks how many grid cells are reachable from the origin using only cells below a query value. Sorting the queries in increasing order means a flood outwards from the origin only ever expands, so no work is ever repeated.

The property that makes both work is that the structure only grows. When a problem genuinely requires undoing, the response is rollback or a decomposition over time, both of which are covered in chapter [[10 DSU Advanced]].

---

## Part 4 · Sweeping the right endpoint with a last-occurrence marker

This pattern is described in chapter [[08 Segment Trees]] as well, because it belongs to both, and it is the single most reusable offline technique on the sheet.

The problem is to count distinct values within a range, with all queries supplied in advance.

Sort the queries by their right endpoint and sweep that endpoint from left to right. Maintain a tree in which position `j` holds one exactly when `j` is currently the rightmost occurrence of its value among positions up to the current endpoint. When the sweep advances to a new position, if that value appeared earlier, subtract one at its previous position, then add one at the current position. The number of distinct values in a range is then the sum over that range.

The reason this is correct is that each distinct value should contribute exactly once, and counting it at its rightmost occurrence guarantees that the counted position falls inside the query range precisely when the value appears somewhere in it.

**CF 703D** is a good variation. It asks for the exclusive or of all values appearing an even number of times in a range. The useful observation is that the exclusive or of everything in the range, combined with the exclusive or of the distinct values in the range, leaves exactly the values appearing an even number of times. The first quantity is a prefix exclusive or and the second is the sweep above with exclusive or in place of counting, so a difficult-looking problem becomes two straightforward pieces.

---

## Part 5 · Mo's algorithm

Mo's algorithm applies when the queries are ranges, all supplied in advance, there are no updates, and the answer for a range can be adjusted in constant time when either endpoint moves by one. It is the fallback for when no decomposition into prefix queries exists.

The idea is to reorder the queries so that the total movement of the two endpoints is small. Divide the positions into blocks and sort the queries by which block their left endpoint falls in, then by their right endpoint, alternating the direction of that second key from block to block.

```cpp
int B = max(1, (int)(n / sqrt(q + 1)));
sort(qs.begin(), qs.end(), [&](const Q& a, const Q& b){
    if (a.l / B != b.l / B) return a.l / B < b.l / B;
    return ((a.l / B) & 1) ? a.r > b.r : a.r < b.r;      // alternating direction
});

int curL = 0, curR = -1;
for (auto& qu : qs) {
    while (curR < qu.r) add(++curR);
    while (curL > qu.l) add(--curL);
    while (curR > qu.r) remove(curR--);
    while (curL < qu.l) remove(curL++);
    ans[qu.idx] = current;
}
```

The four loops must appear in this order, with both expansions before both contractions. Contracting first can momentarily leave the left pointer beyond the right pointer, which makes the tracked range invalid and corrupts the counters in a way that produces plausible but wrong answers.

The cost is proportional to the number of elements plus the number of queries, multiplied by the square root of the number of elements. The left pointer moves within a block for each query and jumps across the array when the block changes, giving a total governed by the query count times the block size plus the squared element count divided by the block size, which is smallest when the block size is the element count divided by the square root of the query count.

The only design work is writing the functions that add and remove one element while keeping the answer current:

| Problem | What is tracked | What adding does |
|---|---|---|
| CF 86D Powerful array | the sum of value times squared count | `cur += v * (2 * cnt[v] + 1); cnt[v]++` |
| CF 617E XOR and Favorite Number | the number of prefix pairs differing by `k` | `cur += cnt[pre[i] ^ k]; cnt[pre[i]]++` |
| counting distinct values | how many values have a positive count | `if (++cnt[v] == 1) distinct++` |

CF 617E is worth extra attention because of its indexing. The question concerns pieces of the array, which correspond to *pairs of prefix positions*, so the range that Mo's algorithm moves over is a range of prefix indices shifted by one from the range in the statement. Getting that shift right is most of the problem, and it is a good illustration of translating a problem into the index space that a tool operates on.

When a sweep with a Fenwick tree exists, it is preferable, since it costs a logarithmic factor rather than a square-root one and involves fewer things to get wrong. Mo's algorithm is worth reaching for only after you have satisfied yourself that no sweep applies.

---

## Part 6 · Sparse tables

For range minimums and maximums on an array that never changes, a sparse table answers queries in constant time after a preprocessing pass.

```cpp
int LOG = 32 - __builtin_clz(n);
vector<vector<int>> sp(LOG, vector<int>(n));
sp[0] = a;
for (int k = 1; k < LOG; k++)
    for (int i = 0; i + (1<<k) <= n; i++)
        sp[k][i] = min(sp[k-1][i], sp[k-1][i + (1<<(k-1))]);

auto query = [&](int l, int r) {                 // inclusive
    int k = 31 - __builtin_clz(r - l + 1);
    return min(sp[k][l], sp[k][r - (1<<k) + 1]);
};
```

The query combines two overlapping blocks that together cover the range. Overlapping is harmless because taking the minimum of a value with itself changes nothing, which is the property that makes this work for minimums and maximums and not for sums. Remembering that distinction is more useful than remembering the code.

Sparse tables also give constant-time lowest common ancestor queries via an Euler tour, which is covered in chapter [[13 Trees]].

---

## The ideas worth carrying forward

1. **Queries supplied in advance may be answered in any order.** Checking for this in every query problem costs a few seconds and occasionally solves the problem outright.

2. **A Fenwick tree indexed by value, swept along positions, holds everything to the left of the current point.** This converts two-dimensional pair counting into one-dimensional prefix queries.

3. **Counting triples decomposes into candidates on each side of a middle element**, computed by two sweeps in opposite directions.

4. **Counting each distinct value at its rightmost occurrence** is the basis of the whole distinct-values family, and the pattern applies well beyond counting.

5. **Sorted queries mean the structure only ever grows**, so nothing needs undoing. Sorting is what buys that property.

6. **Mo's algorithm is a fallback rather than a first choice.** Look for a sweep before reaching for it.

7. **In Mo's algorithm, both expansions come before both contractions**, since the alternative can briefly invert the range.

8. **A Fenwick tree can be descended in logarithmic time** to answer order-statistics questions, which sometimes removes the need for a segment tree.

9. **Sparse tables work for minimums and maximums because overlapping blocks are harmless.** They do not work for sums.

10. **Prefer a Fenwick tree to a segment tree whenever the operation can be undone and prefixes suffice.** It is shorter, faster, and offers fewer opportunities for error.

---

## Where people lose these problems

**Forgetting to compress the values.** With values up to a billion and a tree indexed by value, compression has to come first.

**Index conventions.** Fenwick trees are naturally one-indexed, since the bit trick fails at zero. The template above hides this behind an increment, but writing one from scratch requires being deliberate about it.

**Mixing inclusive and exclusive ranges between structures.** The Fenwick template here uses inclusive ranges while the segment tree in chapter [[08 Segment Trees]] uses half-open ones, and that inconsistency is worth resolving in your own template file.

**Ordering Mo's loops with contractions first.** The symptom is wrong answers with no obvious cause.

**Choosing the block size badly.** The element count divided by the square root of the query count is the right choice; the square root of the element count is the usual approximation and is acceptable when the two counts are similar. A hardcoded constant is not.

**Adding and removing that are not exact inverses.** In CF 86D, adding performs an update and then increments the count, so removing must decrement first and then update. Getting the order wrong produces a correct first answer followed by incorrect ones, which is a confusing symptom to diagnose.

**Using a sparse table for sums.** Overlapping blocks double-count, so a prefix sum is the correct tool.

**In CSES Josephus Problem II, simulating.** The problem repeatedly removes the k-th remaining element, which is an order-statistics question answered by descending a Fenwick tree.

**In CSES Collecting Numbers II, recomputing everything after each swap.** The answer depends on relationships between values that are adjacent in value rather than in position, so a swap only affects a constant number of those relationships. The correct procedure is to subtract the affected relationships, perform the swap, and add them back, taking care when the two swapped values are themselves adjacent in value.

---

## Working through the problem list

### Block 1 · The structures

- **CSES Dynamic Range Sum Queries** — *update single positions and query range sums.* The Fenwick template exactly.
- **CSES Range Xor Queries** — *the same with exclusive or.* Since exclusive or is its own inverse, a range is the combination of two prefixes.
- **CSES Static Range Minimum Queries** — *range minimums on a fixed array.* The sparse table.
- **AC ACL Practice B · Fenwick Tree** — *the same template again in AtCoder form.*

### Block 2 · Sweeping positions, querying values

- **CSES Collecting Numbers II** — *count how many passes are needed to collect numbers in order, under repeated swaps.* Maintaining a global count under local changes.
- **CF 459D Pashmak and Parmida's problem** — *count pairs where a prefix occurrence count exceeds a suffix occurrence count.* Precompute the two counts, then sweep with a Fenwick tree. A clean two-phase problem of medium difficulty.
- **CF 61E Enemy is weak** — *count triples of positions in strictly decreasing order.* The middle-element decomposition.
- **LC 2179 Count Good Triplets in an Array** — *count triples appearing in the same relative order in two permutations.* The same decomposition with a mapping step first.
- **LC 2519 Count the Number of K-Big Indices** — *count positions with at least k smaller elements on each side.* Two sweeps and an intersection.
- **CF 220B Little Elephant and Array** — *count values whose occurrence count within a range equals the value itself.* Offline by right endpoint with an indicator tree, which leads into the next block.

### Block 3 · Sweeping the right endpoint

- **CF 703D Mishka and Interesting sum** — *the exclusive-or problem from Part 4.*
- **CF 1093E Intersection of Permutations** — *count values appearing in a range of one permutation and a range of another, with updates.* Mapping the values of one permutation to their positions in the other turns this into counting points in a rectangle with point updates, which needs a Fenwick tree of ordered sets or a divide-and-conquer approach over time. Genuinely difficult and best treated as a stretch problem.

### Block 4 · Mo's algorithm

- **CF 86D Powerful array** — *sum, over distinct values in a range, of the squared count times the value.* The standard first exercise, with constant-time updates.
- **CF 617E XOR and Favorite Number** — *count pieces of a range whose exclusive or equals a given value.* Mo's algorithm over prefix indices, where the index shift is the difficulty.

### Block 5 · Order statistics and others

- **CSES Josephus Problem II** — *repeatedly remove every k-th remaining person.* Descend the Fenwick tree.
- **LC 1157 Online Majority Element In Subarray** — *find an element occupying a majority of a range.* The word "online" removes the option of sorting the queries. Two approaches work: sampling around twenty random positions in the range, since a majority element is very likely to be hit; or a segment tree whose nodes store a majority candidate and a count, which combine correctly. The segment tree version is the more instructive, and it connects back to chapter [[08 Segment Trees]].
- **AC ACL Practice C · Floor Sum** — *sum the integer parts of a linear function over a range.* Unrelated to Fenwick trees, but a genuinely useful primitive for counting lattice points, and worth having in twenty lines.
- **LC 3245 Alternating Groups III** — *heavy, and reasonable to skip if unavailable.*

---

**A reasonable target here is around 75% of submissions passing first time.**

The failures in this block are mechanical rather than conceptual, arising from compression, indexing, and the requirement that adding and removing be exact inverses. That means they respond to consistent conventions rather than to additional practice.

---

## Check yourself

1. Write the Fenwick add and query from memory. What does the bit trick in the loop accomplish?
2. When do you choose a Fenwick tree over a segment tree, and when can you not?
3. Explain what it means to say the tree is indexed by value and contains everything to the left, in the context of counting inversions.
4. How do two sweeps count triples?
5. Describe the last-occurrence sweep for counting distinct values in a range, and say why counting at the rightmost occurrence is correct.
6. Decompose CF 703D's answer into two simpler quantities.
7. Write Mo's four loops in the correct order and explain why the order matters.
8. What is the best block size for Mo's algorithm, and where does it come from?
9. Why do sparse tables work for minimums but not for sums?
10. Name three problems on this sheet where sorting the queries is the whole solution.
