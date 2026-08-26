---
tags: [dsa, guide, dp-optimization, cht, divide-and-conquer, knuth, aliens]
chapter: 17
sheet-section: Q
---

# Chapter 17 · DP Optimization

> **Read this before you start the problems.** Each idea is introduced with a small example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **Q** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

The problems in this chapter start from a position of already having a correct solution. A dynamic program has been designed properly, the state and transition are both right, and the only issue is that the transition costs too much: it is quadratic in the number of items, and the number of items can be a hundred thousand, giving ten billion operations where only a few million are permitted.

This is a distinctive kind of difficulty compared to earlier chapters, because there is no ambiguity about whether the design is correct, only about whether it runs fast enough. That makes the recognition trigger unusually reliable: if you have arrived at a correct quadratic dynamic program and the constraints clearly disallow it, you are in this chapter, and the actual work is choosing which of four techniques applies. The choice is not always obvious, and the four techniques are different enough from one another that using the wrong one wastes a great deal of time before the mismatch becomes apparent.

Before turning to any of the four main techniques, it is worth checking two simpler fixes first, since together they resolve more slow dynamic programs than everything else in this chapter combined: replacing a transition that sums over a contiguous range of earlier states with a running prefix sum, from chapter [[06 Prefix Sums and Difference Arrays]] and chapter [[14 Dynamic Programming Core]], and replacing a transition that takes a maximum, minimum or sum over earlier states satisfying an inequality with a segment tree or Fenwick tree indexed by that inequality's quantity, from chapters [[08 Segment Trees]] and [[09 Fenwick Offline and Mos]]. Only once neither of these applies is it worth reaching for what follows.

---

## Part 1 · Deciding which of the four techniques applies

Suppose the dynamic program has the shape `dp[i] = minimum over j less than i of (dp[j] + cost(j, i))`, or the partitioned form `dp[i][k] = minimum over j of (dp[j][k-1] + cost(j, i))`. Four questions, asked roughly in this order, determine which technique is needed.

**Does `cost(j, i)` factor into a term depending only on `j` multiplied by a term depending only on `i`, plus terms depending on each separately?** If the cost can be written as `a(j) * b(i) + c(j) + d(i)`, the transition is a minimum over a collection of lines evaluated at a point, and the convex hull trick applies.

**Is this a partition into exactly `k` groups, where the optimal split point moves consistently as `i` increases?** If so, divide-and-conquer optimisation applies, reducing the cost by a full factor proportional to the number of items.

**Is this an interval dynamic program of the form `dp[l][r] = minimum over a split point of (dp[l][m] + dp[m][r]) + w(l, r)`, where `w` satisfies a certain regularity condition?** If so, Knuth's optimisation reduces a cubic computation to a quadratic one.

**Is there a hard requirement of exactly `k` groups, where the cost as a function of `k` behaves in a convex way, and the version without that requirement is easy?** If so, a technique usually called the Aliens trick, or Lagrangian relaxation, removes the `k` dimension entirely at the cost of an additional search over a penalty parameter.

---

## Part 2 · The convex hull trick

**Setting it up.** Suppose the transition takes the form

$$dp[i] = \min_{j < i}\big( dp[j] + a_j \cdot b_i \big) + \text{(something depending only on } i\text{)}$$

The inner expression can be read as a line in the variable `b_i`: the slope is `a_j`, the intercept is `dp[j]`, and evaluating that line at the point `b_i` gives its contribution. Read this way, the dynamic program is asking, for each `i`, to evaluate a growing collection of lines at a particular point and take the smallest result, which is precisely the problem of maintaining the lower envelope of a set of lines.

**Deriving the lines by expanding a square.** AC EDPC Z, sometimes called Frog 3, has the transition `dp[i] = min over j less than i of (dp[j] + (h_i - h_j)^2 + C)`. Expanding the square:

$$dp[i] = h_i^2 + C + \min_{j<i}\big( \underbrace{-2h_j}_{\text{slope}} \cdot \underbrace{h_i}_{\text{point}} + \underbrace{dp[j] + h_j^2}_{\text{intercept}} \big)$$

That expansion is the entire technique for this problem. It is worth practising this kind of algebraic expansion until it becomes automatic, since the same manoeuvre handles a squared difference of any two linear expressions and is the recurring step that turns an apparently unrelated cost function into a line.

**The simple case: sorted slopes and sorted queries.** If the slopes of the lines being added are always decreasing and the points being queried are always increasing, a deque combined with a pointer maintains the lower envelope in time proportional to the number of items.

```cpp
struct Line { long long m, c; long long eval(long long x) { return m*x + c; } };
deque<Line> hull;

bool bad(const Line& a, const Line& b, const Line& c) {
    // b becomes unnecessary if a and c intersect to the left of where a and b intersect
    return (long double)(c.c - a.c) * (a.m - b.m) <= (long double)(b.c - a.c) * (a.m - c.m);
}
void addLine(long long m, long long c) {                 // requires slopes added in decreasing order
    Line l{m, c};
    while (hull.size() >= 2 && bad(hull[hull.size()-2], hull.back(), l)) hull.pop_back();
    hull.push_back(l);
}
long long query(long long x) {                            // requires queries in increasing order
    while (hull.size() >= 2 && hull[0].eval(x) >= hull[1].eval(x)) hull.pop_front();
    return hull[0].eval(x);
}
```

**The general case.** When the slopes or the query points are not sorted, a Li Chao tree is the reliable answer: a segment tree over the domain of possible query points, where each node stores whichever line currently gives the best value at that node's midpoint.

```cpp
void insert(int node, int lo, int hi, Line nw) {
    int mid = (lo + hi) / 2;
    bool lef = nw.eval(lo)  < tree[node].eval(lo);
    bool mi  = nw.eval(mid) < tree[node].eval(mid);
    if (mi) swap(tree[node], nw);
    if (hi - lo == 1) return;
    if (lef != mi) insert(2*node,   lo, mid, nw);
    else           insert(2*node+1, mid, hi, nw);
}
```

Insertion and query both cost a logarithmic factor in the size of the domain, and neither requires any assumption about the order in which lines arrive or queries are made.

**It is worth learning the Li Chao tree rather than the deque version as the primary tool**, even though the deque version is shorter, because verifying that slopes and queries are genuinely sorted takes real attention under time pressure, and getting that verification wrong produces a solution that runs and returns a plausible but wrong answer. The Li Chao tree has no such precondition to check.

CSES *Monster Game I* has sorted slopes and queries, so the deque works directly. CSES *Monster Game II* is deliberately constructed so that they are not sorted, requiring the Li Chao tree. Doing both, in that order, is the clearest way to see why the general tool exists.

---

## Part 3 · Divide-and-conquer optimisation

**Setting it up.** The dynamic program has the partitioned form `dp[k][i] = minimum over j less than i of (dp[k-1][j] + cost(j, i))`, and `opt[k][i]`, the value of `j` that achieves this minimum, is non-decreasing in `i` for a fixed `k`.

```cpp
void solve(int k, int l, int r, int optL, int optR) {
    if (l > r) return;
    int mid = (l + r) / 2;
    pair<long long,int> best = {LLONG_MAX, -1};
    for (int j = optL; j <= min(mid - 1, optR); j++)
        best = min(best, {dp[k-1][j] + cost(j, mid), j});
    dp[k][mid] = best.first;
    solve(k, l, mid - 1, optL, best.second);
    solve(k, mid + 1, r, best.second, optR);
}
```

**Why this costs only a logarithmic factor more than linear, per layer of `k`.** At every level of the recursion, the ranges searched for `opt` across the different calls at that level are disjoint except possibly at a shared endpoint, and together they cover the whole range of `i`. This means the total work at any single level is proportional to the number of items, and since there are a logarithmic number of levels, the total work for one value of `k` is the number of items times its logarithm, and across all `k` layers the total is the number of items times the number of layers times that logarithm.

**Verifying the monotonicity condition in practice.** The rigorous justification is a condition on `cost` called the quadrangle inequality, but the practical way to check it during a contest is to write the straightforward, quadratic-in-both-dimensions solution, print `opt[k][i]` for increasing `i`, and look directly at whether it is non-decreasing. This takes a matter of minutes, gives complete certainty, and additionally produces a reference implementation to test the faster version against, which is not a shortcut so much as the standard way this is actually verified in practice.

CSES *Subarray Squares* and CSES *Houses and Schools* are the standard first examples. LC 1478 and LC 813 are small enough that the naive cubic solution also passes, but are worth solving with divide-and-conquer optimisation anyway purely for the practice.

---

## Part 4 · Knuth's optimisation

**Setting it up.** An interval dynamic program of the form `dp[l][r] = w(l, r) + minimum over split points m of (dp[l][m] + dp[m+1][r])`, where `w` satisfies the same regularity condition as in the previous section and is monotone with respect to interval containment.

**What the condition buys you.** It guarantees that the optimal split point for a slightly larger interval lies between the optimal split points of two related, slightly smaller intervals:

$$opt[l][r-1] \le opt[l][r] \le opt[l+1][r]$$

Restricting the search for each interval's split point to that narrow range, rather than searching the whole interval, causes the total work across all intervals to telescope down to a cost proportional to the square of the number of items, rather than the cube.

```cpp
for (int len = 2; len <= n; len++)
    for (int l = 0; l + len - 1 < n; l++) {
        int r = l + len - 1;
        dp[l][r] = LLONG_MAX;
        for (int m = opt[l][r-1]; m <= min(r - 1, opt[l+1][r]); m++) {
            long long v = dp[l][m] + dp[m+1][r] + w(l, r);
            if (v < dp[l][r]) { dp[l][r] = v; opt[l][r] = m; }
        }
    }
```

This technique appears infrequently in assessment-style problems relative to its difficulty, and it is worth learning the shape without investing heavily beyond that. CSES *Knuth Division*, which merges piles at a cost equal to the sum of the two merged piles, is the standard example, and it is worth noting that this particular problem also has a much simpler greedy solution using a heap in the style of Huffman coding, with Knuth's optimisation serving mainly as a way to prove that the dynamic programming formulation is correct at all.

---

## Part 5 · The Aliens trick

This is the most conceptually distinctive of the four techniques, and it is worth understanding even though it appears rarely, because when it does appear it tends to separate candidates sharply.

**The setting.** A problem requires partitioning something into exactly `k` groups while minimising a cost, and the requirement of exactly `k` groups forces an extra dimension into the dynamic program, making it slower than it would otherwise be.

**The move.** The exact-`k` requirement is dropped, and in its place, a fixed penalty `λ` is charged for every group used:

$$g(\lambda) = \min_{\text{any number of groups } m} \big( \text{cost} + \lambda m \big)$$

This unconstrained version has no dimension for the group count, so it typically runs in linear or close-to-linear time. Solving it for a given `λ` also reveals, as a side effect, how many groups the optimal unconstrained solution happened to use, call that count `m(λ)`.

**Why this works.** As `λ` increases, each additional group becomes more expensive, so `m(λ)` decreases, or at least never increases, as `λ` grows. Because of this monotonicity, `λ` can be found by binary search until `m(λ)` equals `k` exactly, from chapter [[04 Binary Search on the Answer]]. Once such a `λ` is found, the answer to the original, constrained problem is `g(λ) − λk`.

**The condition this requires.** The cost of the best solution using exactly `k` groups, viewed as a function of `k`, must be convex. Geometrically, `g(λ)` is finding the point on the convex hull of that function where the supporting line has slope `−λ`. If the function is not convex, some values of `k` correspond to points that never lie on that hull no matter what `λ` is chosen, and the technique simply cannot reach them.

**The practical complication.** Multiple values of `k` can be simultaneously optimal for the same `λ`, which means `m(λ)` can jump by more than one as `λ` crosses certain values rather than passing smoothly through every integer. The standard response is to consistently prefer either the smallest or the largest optimal group count when ties occur, to binary search over integer values of `λ` when costs are integers, and to accept that the search may only find `m(λ)` less than or equal to `k` rather than exactly equal, relying on the fact that the formula `g(λ) − λk` remains correct regardless because of convexity.

The sheet's instruction to revisit LC 410 using the Aliens trick is worth following precisely because that problem has already been solved with binary search on the answer, from chapter [[04 Binary Search on the Answer]], and with a direct quadratic dynamic program, from chapter [[14 Dynamic Programming Core]]. Seeing the same problem solved a third way, through an entirely different mechanism, is more valuable than solving three unrelated problems once each.

---

## The ideas worth carrying forward

1. **Check the two cheaper fixes first.** A running prefix sum or a segment tree indexed by the transition's constraint resolves more slow dynamic programs than the four techniques in this chapter combined.

2. **The decision between the four techniques comes down to three questions**: does the cost factor into a product of a `j`-term and an `i`-term; is this an exact-`k` partition with a monotone optimal split point; is this an interval dynamic program satisfying the regularity condition needed for Knuth's optimisation.

3. **The convex hull trick is a minimum over a collection of lines.** Expanding a squared difference is usually what reveals the slope and intercept.

4. **The Li Chao tree has no ordering precondition and is the safer default**, even though the deque-based approach is shorter when its preconditions genuinely hold.

5. **Verify the monotonicity needed for divide-and-conquer optimisation by writing the naive version and printing `opt`.** This is standard practice, not a shortcut, and it produces a reference solution for testing at the same time.

6. **Divide-and-conquer optimisation costs a logarithmic factor per layer, because the search ranges at each level of the recursion are disjoint and together cover the full range.**

7. **The Aliens trick exchanges a hard constraint for a penalty per unit**, converting a minimum over exactly `k` groups into a minimum over any number of groups with a price attached to each one.

8. **The Aliens trick requires the cost, as a function of the group count, to be convex**, and the binary search over the penalty works because the resulting group count changes consistently as the penalty changes.

9. **Solving the same problem through binary search, direct dynamic programming, and the Aliens trick, as with LC 410, teaches more than three unrelated problems would.**

---

## Where people lose these problems

**Reaching for the convex hull trick when a prefix sum would have done.** The cheaper fixes are worth checking first, every time.

**Overflow inside the line-comparison test.** The products involved can reach far beyond the range of a standard 64-bit integer when slopes and intercepts are both large, so using extended precision for that comparison, rather than the underlying values, is the safer choice.

**Assuming slopes or queries are sorted when they are not.** The deque-based approach fails silently in this case, returning a plausible but wrong answer, which is precisely why the Li Chao tree is worth using by default.

**Sizing the Li Chao tree's domain too narrowly.** The domain must cover every possible query point, including negative ones if the problem allows them.

**Applying divide-and-conquer optimisation to a cost function where `opt` is not actually monotone.** This produces wrong answers on some tests only, and it is exactly what the empirical verification step is meant to catch beforehand.

**Applying Knuth's optimisation without checking the underlying regularity condition.** The same failure mode as above.

**Applying the Aliens trick to a cost function that is not convex as a function of the group count.** The symptom is that the binary search converges to a penalty value where the resulting group count jumps over `k` entirely rather than landing on it, which is itself the signal that convexity does not hold here.

**Being inconsistent about whether the penalty is charged per group or per split between groups.** These differ by exactly one, and choosing a convention and applying it consistently avoids an off-by-one that is otherwise easy to introduce.

**Starting the next layer of divide-and-conquer optimisation before the previous layer has been fully computed.** Each layer depends entirely on the one before it, and this dependency must be respected strictly.

---

## Working through the problem list

**Before beginning, it is worth confirming that chapter [[08 Segment Trees]] and the accelerated-transition material in chapter [[14 Dynamic Programming Core]] are solid**, since a large share of what feels like it needs this chapter's techniques is actually solved more simply by those instead.

### Block 1 · Feeling the limits of the direct approach

- **LC 813 Largest Sum of Averages** — *partition an array into groups maximising the sum of each group's average.* Small enough for a direct cubic solution, worth writing first, then rewriting with divide-and-conquer optimisation, printing `opt[k][i]` along the way to confirm it behaves as expected.
- **LC 1478 Allocate Mailboxes** — *place mailboxes to minimise total distance to houses.* The same shape, with the per-interval cost precomputed as the sum of distances to a chosen point within it.
- **LC 2547 Minimum Cost to Split an Array** — *partition an array minimising a cost based on runs of repeated values.* The intended solution is quadratic and relies on maintaining the cost incrementally as a boundary moves, which is a prerequisite skill for everything that follows in this chapter.

### Block 2 · Divide-and-conquer optimisation

- **CSES Subarray Squares** — *the canonical problem for this technique.* Do this first.
- **CSES Houses and Schools** — *the same technique with a distance-based cost, where prefix sums make each interval's cost constant time to compute.*
- **CF 321E Ciel and Gondolas** — *the same problem at larger scale, where the naive approach genuinely does not fit.*
- **CF 868F Yet Another Minimization Problem** — *a partition problem where the cost of an interval cannot be computed directly but can be updated incrementally as its boundaries move, in the style of Mo's algorithm from chapter [[09 Fenwick Offline and Mos]].* The most instructive problem in this block, since it shows that the requirement is really that cost be cheap to update between adjacent intervals, not that it be computable from scratch in constant time.

### Block 3 · The convex hull trick

- **AC EDPC Z · Frog 3** — *the introductory problem, worked through above.* Do the algebraic expansion by hand before looking at any implementation.
- **CSES Monster Game I** — *sorted slopes and sorted queries.* The deque-based approach.
- **CSES Monster Game II** — *the same problem with neither sorted.* The Li Chao tree, or an ordered-container-based approach. Doing this immediately after the first version makes the contrast clear.
- **CF 319C Kalila and Dimna in the Logging Industry** — *a classic application requiring an argument about processing order before the convex hull trick itself can be applied.* Greedy reasoning combined with the technique.
- **AC EDPC Z, revisited** — *implement both the deque-based version and the Li Chao tree from scratch, and keep both in a personal template file.*

### Block 4 · Knuth's optimisation

- **CSES Knuth Division** — *merge piles at minimum total cost.* The one problem in this chapter for this technique. Note the simpler heap-based alternative and understand why both approaches are valid.

### Block 5 · The hardest problems

- **LC 410, revisited with the Aliens trick** — *the three-lens exercise described above.*
- **CF 660F Bear and Bowling 4** — *divide-and-conquer optimisation applied to a cost function that is not immediately obvious.* Genuinely difficult, and best attempted once everything earlier in this chapter feels comfortable.

---

**A reasonable target here is around 50 to 60% of submissions passing first time, and completion matters more than that number.**

This is the lowest-frequency chapter on the sheet relative to how much effort it demands, so the return on it is less about the specific problems being likely to appear and more about what working through it does to your general instinct for noticing when a dynamic program is too slow and why. That instinct improves every dynamic program written afterwards, which is a better way to think about this chapter's value than treating it as material likely to be tested directly.

---

## Check yourself

1. Which two cheaper optimisations should be ruled out before reaching for anything in this chapter?
2. Expand `dp[j] + (h_i - h_j)^2` and identify the slope, the intercept, and the query point.
3. Why is a Li Chao tree generally preferable to a deque-based approach under time pressure?
4. State the monotonicity condition needed for divide-and-conquer optimisation, and describe the practical way to check it.
5. Why does divide-and-conquer optimisation cost only a logarithmic factor per layer of `k`?
6. State the bound Knuth's optimisation places on `opt[l][r]`.
7. Explain the Aliens trick in three sentences: what is dropped, what is added, and what is searched over.
8. Why does the Aliens trick require the cost, as a function of the group count, to be convex? What happens if it is not?
9. In CF 868F, what does divide-and-conquer optimisation require of the cost function, and how is that weaker than the usual requirement?
10. Name the three distinct ways LC 410 can be solved, as covered across this guide.
