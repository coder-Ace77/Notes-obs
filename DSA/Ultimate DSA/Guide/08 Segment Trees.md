---
tags: [dsa, guide, segment-tree, lazy-propagation, data-structures]
chapter: 8
sheet-section: H
---

# Chapter 8 · Segment Trees Beyond Range Sum

> **Read this before you start the problems.** Each design is introduced with a worked example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **H** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

Most people learn segment trees once, as a structure that supports range sums with point updates, and then find that this knowledge transfers poorly. The difficulty is that a segment tree is better understood as a framework with two things left unspecified rather than as a single structure. The two things are what each node stores, and how the information in two adjacent nodes combines into information about their union.

Every problem in this section supplies a different pair of answers to those two questions. Sometimes the combination is straightforward, as with sums. More often the information you actually want cannot be combined at all, and the design work consists of finding *additional* information to store whose only purpose is to make the combination possible. Recognising when that is what you need, and working out what the extra fields should be, is the skill this section develops.

Recognising that a segment tree is required is comparatively easy. The number of elements and the number of queries are both in the region of a hundred thousand to two hundred thousand, updates and queries are interleaved, and the direct approach would be quadratic. That pattern of constraints is close to an explicit statement of intent from the problem setter. The difficulty comes afterwards, when you have to invent the node design under time pressure, which is why this chapter is organised around the designs rather than around the problems.

---

## Part 1 · The framework

A segment tree over an array is a binary tree in which each leaf corresponds to one element, each internal node corresponds to the concatenation of the ranges covered by its two children, and each node stores a summary of its own range.

For this to work, the operation that combines two summaries must be associative, so that grouping does not affect the result, and there must be an identity value to use for empty ranges. That is the entire mathematical requirement. Sums qualify with an identity of zero, maximums qualify with an identity of negative infinity, greatest common divisors qualify with an identity of zero, and matrix products qualify with the identity matrix.

The design work therefore consists of two questions, asked in this order:

1. What does a node need to store, so that a query over an arbitrary range can be answered by combining the summaries of the logarithmically many nodes that cover it?
2. Is that information sufficient to compute a parent's summary from its two children?

The second question is where the interesting cases arise. Frequently the answer to the first question alone turns out to be insufficient, because the quantity you want cannot be recovered from the same quantity in the two children. When that happens the response is to store more, and the extra fields exist purely to make the combination possible rather than because the problem asked for them.

---

## Part 2 · The non-lazy template

Segment trees can be written recursively or iteratively. For trees without lazy propagation the iterative bottom-up form is shorter, faster, and carries no risk of exceeding the recursion depth, so it is the one worth learning first.

```cpp
struct SegTree {
    int n; vector<long long> t;
    SegTree(int n) : n(n), t(2*n, 0) {}

    void build(vector<long long>& a) {
        for (int i = 0; i < n; i++) t[n + i] = a[i];
        for (int i = n - 1; i > 0; i--) t[i] = t[2*i] + t[2*i+1];
    }
    void update(int p, long long v) {                 // assign to a single position
        for (t[p += n] = v; p > 1; p >>= 1) t[p>>1] = t[p] + t[p^1];
    }
    long long query(int l, int r) {                   // over [l, r), half-open
        long long resL = 0, resR = 0;
        for (l += n, r += n; l < r; l >>= 1, r >>= 1) {
            if (l & 1) resL = resL + t[l++];
            if (r & 1) resR = t[--r] + resR;
        }
        return resL + resR;
    }
};
```

Three details generalise beyond this particular tree.

The range is half-open, matching the convention used in chapters [[02 Intervals and Sweep Line]] and [[06 Prefix Sums and Difference Arrays]]. Keeping one convention across the whole toolkit prevents more errors than any individual choice of convention.

The query accumulates into two separate variables rather than one. For combinations where order does not matter, such as sums, maximums and greatest common divisors, a single accumulator would work. For combinations where order does matter, such as matrix products or the bracket-matching design in Part 5, the pieces coming from the left must be combined left to right and the pieces from the right must be combined right to left, with the two halves joined at the end. Learning the single-accumulator version means the first order-sensitive problem you meet will fail in a way that is difficult to trace.

The update line uses `t[p] + t[p^1]`, where `p^1` is the sibling of `p`. This is compact but relies on order not mattering. For an order-sensitive combination it should be written explicitly in terms of the two children in their correct order.

---

## Part 3 · Lazy propagation

When updates apply to whole ranges rather than single positions, the tree needs to defer work, and the deferral requires the recursive form because pending operations have to be pushed down along a path from the root.

```cpp
struct LazySeg {
    int n; vector<long long> t, lz;
    LazySeg(int n) : n(n), t(4*n, 0), lz(4*n, 0) {}

    void apply(int node, int l, int r, long long v) {   // apply "add v" across a whole node
        t[node] += v * (r - l);
        lz[node] += v;
    }
    void push(int node, int l, int r) {
        if (lz[node] == 0) return;
        int m = (l + r) / 2;
        apply(2*node, l, m, lz[node]);
        apply(2*node+1, m, r, lz[node]);
        lz[node] = 0;
    }
    void update(int node, int l, int r, int ql, int qr, long long v) {
        if (qr <= l || r <= ql) return;                              // no overlap
        if (ql <= l && r <= qr) { apply(node, l, r, v); return; }    // fully inside
        push(node, l, r);
        int m = (l + r) / 2;
        update(2*node, l, m, ql, qr, v);
        update(2*node+1, m, r, ql, qr, v);
        t[node] = t[2*node] + t[2*node+1];
    }
    long long query(int node, int l, int r, int ql, int qr) {
        if (qr <= l || r <= ql) return 0;
        if (ql <= l && r <= qr) return t[node];
        push(node, l, r);
        int m = (l + r) / 2;
        return query(2*node, l, m, ql, qr) + query(2*node+1, m, r, ql, qr);
    }
};
```

The three-case structure of no overlap, full containment, and partial overlap is the same in every recursive segment tree, so it is worth memorising the shape and then varying only the application, the pushing, and the recombination line.

The array is allocated with four times the number of elements rather than twice, because the recursive indexing scheme leaves gaps when the size is not a power of two. The extra memory is not worth economising on.

**A pending operation, usually called a lazy tag, is a deferred transformation of an entire range.** Making one work requires three definitions, and it is worth writing all three down before coding:

1. **What the tag represents.** Adding a value to every element, assigning a value to every element, or flipping every bit.
2. **How to apply a tag to a node's summary without descending.** For an addition tag on a sum node this is adding the value multiplied by the range length. For an assignment tag it is replacing the sum with the value multiplied by the length. For a flip tag on a node counting ones it is subtracting the current count from the range length.
3. **How to combine a tag with a tag that is already pending.** For addition the two values add. For assignment the newer one replaces the older. For a tree supporting both addition and assignment, the tag becomes a small structure and the combination rules need care, because an arriving assignment must discard any pending addition while an arriving addition must accumulate on top of a pending assignment.

The third definition is where errors concentrate. Being unable to state the combination rule means the tree does not yet have a consistent design, and writing the rules out as a small table for anything more complex than plain addition is worth the minute it takes.

**CSES Polynomial Queries** is a good exercise in tag design. The update adds an increasing sequence across a range, so the tag is an arithmetic progression described by its first term and common difference. Applying it to a node requires summing that progression across the node's length, which has a closed form, and combining two tags means adding them component by component. Once you have seen that a tag can be any transformation which is closed under combination and whose effect on a summary is computable directly, the whole family of lazy problems becomes approachable.

---

## Part 4 · Designing the combination

This is the part that distinguishes being able to use a segment tree from having seen one.

**CSES Prefix Sum Queries** asks for the largest prefix sum within a range, where the empty prefix is permitted so the answer is never negative. A node storing only that quantity cannot be combined, because the best prefix of a combined range may run through the whole left child and into the right, and nothing in the two stored values reveals the total of the left child.

The fix is to store two values:

```
sum  = the total of the range
best = the largest prefix sum of the range
```

which combine as

```
sum  = L.sum + R.sum
best = max(L.best, L.sum + R.best)
```

The second line says that the best prefix either stays inside the left child, or consumes the left child entirely and then takes a prefix of the right. The `sum` field is stored purely so that this line can be written; the problem never asks for it.

**CSES Subarray Sum Queries** asks for the largest sum over any contiguous piece within a range, and needs four fields:

```
sum  = the total
pref = the largest prefix sum
suf  = the largest suffix sum
best = the largest sum over any piece inside
```

combining as

```
sum  = L.sum + R.sum
pref = max(L.pref, L.sum + R.pref)
suf  = max(R.suf,  R.sum + L.suf)
best = max(L.best, R.best, L.suf + R.pref)
```

The last line carries the idea: the best piece lies entirely in the left child, entirely in the right child, or crosses the boundary, in which case it consists of a suffix of the left joined to a prefix of the right. Deriving these four lines by hand once is worthwhile, because they serve as the model for every subsequent combination you have to design.

**CF 380C Sereja and Brackets** asks for the length of the longest valid bracket subsequence within a range. A node stores the number of matched pairs, the number of unmatched opening brackets, and the number of unmatched closing brackets:

```
newPairs = min(L.open, R.close)
matched  = L.matched + R.matched + newPairs
open     = L.open  + R.open  - newPairs
close    = L.close + R.close - newPairs
```

The derivation is a single sentence: the opening brackets left over in the left child can pair with the closing brackets left over in the right child, and everything else follows.

**CSES Pizzeria Queries** asks for the minimum of `p[j] + |i - j|` over all `j`. Splitting the absolute value gives `(p[j] - j) + i` when `j` is at or before `i`, and `(p[j] + j) - i` when `j` is at or after `i`. Keeping two separate minimum trees, one over `p[j] - j` and one over `p[j] + j`, turns each query into two range minimums. Splitting an absolute difference into two expressions of this kind is a named move that reappears in LC 2926 below and in chapter [[17 DP Optimization]].

---

## Part 5 · Walking down the tree

There is a class of query of the form "find the leftmost position satisfying some property". The direct approach binary searches over positions and performs a range query at each step, which costs two logarithmic factors. Walking down the tree costs one.

**CSES Hotel Queries** asks you to find the leftmost hotel with at least a given number of free rooms and then book them. With a tree storing maximums:

```cpp
int descend(int node, int l, int r, long long k) {
    if (t[node] < k) return -1;                    // no suitable leaf in this subtree
    if (r - l == 1) { t[node] -= k; return l; }    // a leaf, so book here
    int m = (l + r) / 2;
    int res = (t[2*node] >= k) ? descend(2*node, l, m, k)
                               : descend(2*node+1, m, r, k);
    t[node] = max(t[2*node], t[2*node+1]);
    return res;
}
```

The technique rests on the line choosing between the two children. The maximum stored in the left child tells you whether a suitable leaf exists there, so the search never enters a subtree that cannot contain the answer and never needs to backtrack.

CSES *List Removals* and CSES *Salary Queries* are the order-statistics forms of the same idea, where the tree stores counts and the descent locates the k-th remaining element.

**The counting-inversions family** consists of LC 315, LC 327 and LC 493, which all ask you to count pairs of positions where the earlier one relates to the later one in some way. Three interchangeable approaches solve all three: a Fenwick tree over compressed values swept from left to right, which is chapter [[09 Fenwick Offline and Mos]]; a merge sort counting crossing pairs during the merge; and a segment tree indexed by value. Solving LC 315 with all three takes about forty minutes and permanently connects three techniques that are otherwise learned separately.

---

## Part 6 · Indexing by value instead of position

A segment tree does not have to be indexed by position in the array. Indexing it by value, so that a node covers a range of possible values rather than a range of positions, makes two things easy: counting how many elements are below a threshold becomes a prefix query, and finding the k-th smallest becomes a descent.

**LC 2926 Maximum Balanced Subsequence Sum** is the clearest illustration of why this matters. The recurrence is that the best total ending at position `i` equals `a[i]` plus the best total over all earlier positions `j` where `a[j] - j` is at most `a[i] - i`. Compressing the values of `a[i] - i` and building a maximum tree indexed by them turns each step into one range query and one point update, giving a solution in `n log n` time.

The general form is worth stating, because it applies well beyond this problem. **When a dynamic programming transition takes a maximum, minimum or sum over all earlier states satisfying an inequality, index a segment tree by the quantity in that inequality, and the transition becomes a range query.** CF 474E uses the same shape, and this pattern is much of the practical justification for section H.

**CSES Distinct Values Queries** asks how many distinct values appear in a range, with all queries supplied in advance. Sorting the queries by their right endpoint and sweeping that endpoint from left to right, you maintain a structure in which position `j` holds one if `j` is currently the rightmost occurrence of its value, and zero otherwise. Advancing to a new right endpoint means zeroing the previous occurrence of the new value and setting the current position. The number of distinct values in a range is then the sum over that range. CF 1000F and CF 522D are variations on the same sweep, and the technique belongs as much to chapter [[09 Fenwick Offline and Mos]] as to this one.

**CSES Range Queries and Copies** introduces persistence. Each update creates only the logarithmically many nodes along one root-to-leaf path and shares the rest with the previous version, producing a new root that represents the updated array while all earlier roots remain valid. Storing the roots in a list gives access to every historical version. The concept is simple and the implementation is fiddly, so it is worth allocating real time to it.

---

## Part 7 · A tree that needs no lazy propagation

The sweep in chapter [[02 Intervals and Sweep Line]] for LC 850 needs a tree supporting range additions of plus one and minus one, reporting the total length covered by at least one interval. This design is unusual and worth understanding rather than copying.

Each node stores two values: a count of how many intervals fully cover this node's range, arising from range updates that stopped at this node, and the total covered length within the node's range. The covered length is recomputed on the way back up:

```
len = (cnt > 0) ? (realRight - realLeft) : (leftChild.len + rightChild.len)
```

Because a positive count means the entire node is covered regardless of anything below it, there is never a need to push information downwards, which removes lazy propagation entirely. Removals always exactly cancel earlier additions, so the count never becomes negative.

---

## The ideas worth carrying forward

1. **Two questions define any segment tree**: what a node stores, and how two nodes combine. Asking both explicitly, in writing, before coding is what makes the design work tractable.

2. **Store extra fields until the combination becomes possible.** The `sum` field exists to compute the best prefix; the open and close counts exist to compute the matched count. These fields are structural rather than requested by the problem.

3. **Every interesting combination has a crossing term.** The best piece is in the left, in the right, or made from a suffix of the left and a prefix of the right, and that third case carries the content of the design.

4. **Keep the left and right accumulators separate in the iterative query**, so that order-sensitive combinations work correctly.

5. **A lazy tag needs three definitions**: what it means, how it changes a summary directly, and how it combines with a tag already pending. An inability to state the third means the design is not yet complete.

6. **Walking down the tree replaces a binary search over queries**, using the child summaries to decide which way to go, which removes one logarithmic factor.

7. **Indexing by value rather than position** makes order statistics and threshold counting into range queries.

8. **A transition constrained by an inequality becomes a range query on a tree indexed by that inequality's quantity.** This pattern alone justifies much of section H.

9. **An absolute difference splits into two expressions**, one for each side, handled by two separate trees.

10. **Sorting queries by right endpoint and maintaining a rightmost-occurrence marker** solves the whole distinct-values family, and the pattern recurs widely.

11. **Allocate four times the size for recursive trees and twice for iterative ones.** This is not worth optimising and not worth getting wrong.

---

## Where people lose these problems

**Building for one combination and discovering another is needed.** Writing the two design questions down first costs under a minute, against twenty minutes of rewriting.

**Using the wrong identity value.** Sums use zero, maximums use a very small number, minimums use a very large one, and greatest common divisors use zero. Using zero as the identity for a maximum tree over possibly-negative values produces silently wrong answers.

**Failing to push pending operations before descending.** The push belongs in both the update and the query, immediately after the containment check. Without it, queries that partially overlap a node read stale values.

**Omitting the recombination line at the end of an update.** Easy to leave out and immediately fatal.

**Getting the combination of assignment and addition tags wrong.** An arriving assignment must discard a pending addition, and an arriving addition must accumulate on top of a pending assignment.

**Overflow when applying a tag.** A value of `10^9` applied across two hundred thousand positions reaches `2 * 10^14`, so 64-bit arithmetic is needed.

**Using a single accumulator with an order-sensitive combination.** The result is wrong in a way that depends on the query range, which makes it hard to diagnose.

**In LC 699, mishandling adjacency.** The maximum-assignment tag combines by taking the larger value, which is valid here because heights only ever increase. The square boundaries need compressing, and each square should be treated as half-open so that squares merely touching at an edge do not interact.

---

## Working through the problem list

### Block H1 · Lazy propagation

These are best done in order, since each adds exactly one complication.

- **CSES Range Update Queries** — *add a value across a range, then read single positions.* The simplest possible pending operation, and also solvable with a difference array and a Fenwick tree, which is worth noticing.
- **CSES Range Updates and Sums** — *support range addition, range assignment, and range sums.* The combined-tag problem, and the most instructive one in this block. Write out the combination table before coding.
- **AC ACL Practice K · Range Affine Range Sum** — *apply a linear transformation across a range and query sums.* The tag is a linear function, and two of them combine by function composition. Once linear functions are visible as tags, tags in general become much clearer.
- **AC ACL Practice L · Lazy Segment Tree** — *count inversions in a binary array under range flips.* A node stores the counts of zeros and ones and the inversion count, and a flip exchanges the first two while replacing the third with the product minus itself. An elegant piece of design.
- **CSES Polynomial Queries** — *add an increasing sequence across a range.* The arithmetic-progression tag.
- **CSES Increasing Array Queries** — *harder, and a reasonable stretch.*
- **LC 2569 Handling Sum Queries After Update** — *flip bits across a range in one array and use it to update another.* The LeetCode version of the ACL flip problem, and considerably gentler.
- **LC 699 Falling Squares** — *squares drop onto a line; report the height after each.* Compression with a maximum-assignment tag.
- **CF 52C Circular RMQ** — *range addition and range minimum on a circular array.* A wrapping range becomes two ordinary ranges.

### Block H2 · Designing the combination

- **AC ACL Practice J · Segment Tree** — *maximum queries with a descent.* A warm-up.
- **CSES Prefix Sum Queries** — *the two-field design from Part 4.* Do this first in this block.
- **CSES Subarray Sum Queries** — *the four-field design.* The model for every later combination, and worth deriving by hand.
- **CF 380C Sereja and Brackets** — *the bracket design.*
- **CSES Pizzeria Queries** — *two trees and the split absolute value.*
- **CF 474E Pillars** — *a longest-chain dynamic program with a value constraint.* A segment tree over compressed values accelerating the transition, which leads directly into the idea in Part 6.

### Block H3 · Descending the tree

- **CSES Hotel Queries** — *assign each group to the leftmost hotel with enough rooms.* The descent template.
- **CSES List Removals** — *repeatedly remove the k-th remaining element.* Order-statistics descent.
- **CSES Salary Queries** — *count salaries in a range, with updates.* A tree over values, and the first real taste of Part 6.
- **LC 315 Count of Smaller Numbers After Self** — *for each element, count smaller elements to its right.* Worth doing all three ways, which is the highest-value single exercise in section H.
- **LC 493 Reverse Pairs** — *count pairs where the earlier element exceeds twice the later one.* The same skeleton, with overflow to watch in the comparison.
- **LC 327 Count of Range Sum** — *count ranges whose sum falls within given bounds.* The same skeleton applied to prefix sums, combining this chapter with chapter [[06 Prefix Sums and Difference Arrays]].

### Block H4 · Value space and merging

- **CSES Distinct Values Queries** — *the offline sweep from Part 6.*
- **CF 1000F One Occurrence** — *report a value occurring exactly once in a range.* The same sweep, storing the value rather than a count.
- **CF 522D Closest Equals** — *find the closest pair of equal values in a range.* The same family.
- **LC 2276 Count Integers in Intervals** — *the interval-counting problem from chapter [[02 Intervals and Sweep Line]].* The sheet asks you to revisit it here, and the reason is to compare the disjoint-interval map against a tree over value space, and to see that the map is itself a merging structure with an amortised guarantee.
- **LC 2926 Maximum Balanced Subsequence Sum** — *the transition-acceleration problem from Part 6.* If only one problem from this block stays with you, this is the one to choose.
- **CSES Range Queries and Copies** — *query historical versions of an array.* Persistence, and worth allocating an evening.

### Block H5 · Optional extensions

These use balanced binary search trees with implicit keys, supporting splitting, merging and reversing sequences.

- **CSES Cut and Paste**, **CSES Substring Reversals**, **CSES Reversals and Sums** — *sequence manipulation problems.*
- **CF 1187D Subarray Sorting** — *decide whether sorting arbitrary ranges can transform one array into another.* A segment tree used as a feasibility oracle, which is an unusual and pleasing use.

These are genuinely optional for assessment purposes and are worth doing for enjoyment or for competitive programming rather than because the sheet lists them.

---

**For blocks H1 and H2 the measure worth tracking is fluency rather than accuracy**, since this is a calibration block.

The question to answer is whether you can write a working lazy segment tree from an empty file in under fifteen minutes. If not, that is the finding, and the response is repetition of the same few problems rather than a larger number of new ones.

---

## Check yourself

1. State the two design questions for any segment tree.
2. Give the four fields and four combination lines for the largest piece sum in a range, and explain the crossing term.
3. Why does the largest-prefix design need to store the total as well?
4. Name the three things a pending operation must define, and give the combination rule for a linear-function tag.
5. Why must the two accumulators be kept separate in the iterative query?
6. Write the descent step for finding the leftmost position with a value of at least `k` in a maximum tree.
7. What does indexing a tree by value mean, and which two query types does it make easy?
8. Given a transition that maximises over earlier states satisfying an inequality, what do you index the tree by, and what are the two operations per step?
9. Describe the sweep that answers how many distinct values appear in a range.
10. Why does the covered-length design need no lazy propagation?
