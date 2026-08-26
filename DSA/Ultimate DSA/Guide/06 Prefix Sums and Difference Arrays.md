---
tags: [dsa, guide, prefix-sums, difference-array, offline]
chapter: 6
sheet-section: F
---

# Chapter 6 · Prefix Sums, Difference Arrays & Offline Sweeps

> **Read this before you start the problems.** Each technique comes with a small example you can follow without having seen the problem it comes from.

Back to [[00 Guide Index]] · Sheet section **F** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

The techniques in this chapter are the simplest on the sheet to describe. A prefix sum is one loop, and a difference array is two lines. The difficulty is not in learning them but in the fact that they are usually one component of a larger solution, and the surrounding work is where things go wrong.

There is also a reframing here that is worth stating early, because it does most of the work. Any question about a contiguous piece of an array is really a question about a *pair* of prefix values, since the sum of a range is the difference of two prefix sums. That turns a search over every possible piece, of which there are quadratically many, into a single pass in which you ask a question about the prefix values you have already seen. What structure answers that question depends on how the range condition is phrased, and this is where the block's difficulty ladder comes from: some conditions are answered by a hash map, some by an ordered container, and some by a Fenwick tree from a later chapter.

The remaining difficulties are mechanical. Two-dimensional versions have signs that are easy to get wrong, negative values interact badly with the modulo operator, and prefix sums overflow 32-bit integers at very ordinary input sizes.

---

## What these problems look like

There are two families here, and they are inverses of one another.

**Prefix sums** answer questions about ranges of a fixed array in constant time after a single pass of preprocessing. The harder versions are two-dimensional, or built on a hashed key rather than a position, or combined with a hash map to count ranges rather than to evaluate one.

**Difference arrays** apply changes to ranges in constant time each, with the actual array reconstructed once at the end. The harder versions are two-dimensional, or the change applied depends on the position within the range.

The most useful decision rule in this chapter is about when each applies:

| Updates | Queries | What to use |
|---|---|---|
| none | many, over ranges | prefix sums |
| many over ranges, all before any query | one, at the end | difference array |
| single positions, interleaved with queries | over ranges, interleaved | Fenwick tree, chapter [[09 Fenwick Offline and Mos]] |
| over ranges, interleaved with queries | over ranges, interleaved | lazy segment tree, chapter [[08 Segment Trees]] |

The row that matters is the second one. A difference array only works when every update happens before any query, because the array is only correct once the final pass has been run. As soon as updates and queries interleave, a different structure is needed.

---

## Part 1 · The core identity

Define `P[0] = 0` and let `P[i]` be the sum of the first `i` elements. Then the sum of the range from `l` to `r` inclusive is

$$\text{sum}(l, r) = P[r+1] - P[l]$$

It is worth always using this convention, with an array of size `n + 1` and a leading zero, because it removes the special case where `l` is zero. That special case is where most prefix-sum errors originate.

The reframing follows from the identity. A question about the piece from `l` to `r` is a question about the pair `P[l]` and `P[r+1]`. Rather than iterating over pieces, you iterate over right endpoints and, for each one, ask how many earlier prefix values stand in the required relationship to the current one.

The structure that answers that question depends on the relationship:

| The relationship | What answers it | Example |
|---|---|---|
| the earlier prefix equals the current minus `k` | a hash map of counts | count ranges summing to `k` |
| the earlier prefix is congruent to the current, modulo `k` | a hash map keyed by remainder | LC 974, CSES Subarray Divisibility |
| the earlier prefix equals the current, after a transformation | a hash map | LC 525, LC 523 |
| the earlier prefix is at least `k` below, and you want the closest | a monotonic deque | LC 862 |
| the earlier prefix lies within a range of values | a Fenwick tree or merge sort | LC 327 |
| the earlier prefix is less than half the current | a Fenwick tree or merge sort | LC 493 |

The last three escalate out of this chapter into chapters 07 and 09, and that escalation is essentially the difficulty ordering of this section.

---

## Part 2 · Counting ranges with a hash map

```cpp
unordered_map<long long, int> cnt;
cnt[0] = 1;                          // the empty prefix
long long pre = 0, ans = 0;
for (int x : a) {
    pre += x;
    ans += cnt[pre - k];             // ranges ending here that sum to k
    cnt[pre]++;
}
```

The initialisation `cnt[0] = 1` accounts for ranges that begin at index zero, whose preceding prefix is the empty one. Leaving it out produces an answer that is short by exactly the number of prefixes equal to `k`, which is a distinctive enough symptom to recognise.

**LC 525 Contiguous Array** asks for the longest range containing equally many zeros and ones. Mapping each zero to `-1` converts "equal counts" into "sum of zero", which converts into "two prefixes with the same value". Storing the first index at which each prefix value appeared and maximising the distance gives the answer.

That transformation is worth naming, because it recurs. Turning a balance condition into a sum condition by encoding the two options as `+1` and `-1` also handles bracket matching and problems about one letter outnumbering another.

**LC 523 Continuous Subarray Sum** asks for a range whose sum is divisible by `k`, which becomes two prefixes with the same remainder. **LC 974** counts them rather than finding one, so it stores counts instead of first indices. Both need care with the modulo operator on negative values.

---

## Part 3 · Difference arrays

To add a value to every element in a range:

```cpp
diff[l]   += v;
diff[r+1] -= v;
// after all updates have been applied:
for (int i = 1; i < n; i++) diff[i] += diff[i-1];
```

The reason this works is that the running total at position `i` counts every update whose range began at or before `i` and has not yet ended. It is the counting sweep from chapter [[02 Intervals and Sweep Line]] with the sorting step removed, because array indices already arrive in order.

LC 2381 is this technique with a modulo applied at the end, and LC 2536 is the two-dimensional version.

---

## Part 4 · The two-dimensional versions

**For querying the sum of a rectangle**, build

$$P[i][j] = a[i-1][j-1] + P[i-1][j] + P[i][j-1] - P[i-1][j-1]$$

and read off a rectangle as

$$\text{sum}(r_1, c_1, r_2, c_2) = P[r_2+1][c_2+1] - P[r_1][c_2+1] - P[r_2+1][c_1] + P[r_1][c_1]$$

Both formulas are inclusion and exclusion over rectangles. Drawing the four rectangles once on paper makes the signs follow from the picture rather than needing to be memorised.

**For adding a value to a rectangle**, mark the four corners:

```cpp
d[r1][c1]     += v;
d[r1][c2+1]   -= v;
d[r2+1][c1]   -= v;
d[r2+1][c2+1] += v;
// then take running sums along rows, then along columns
```

The same inclusion and exclusion, running in the other direction. Allocating the array with dimensions `(n+1)` by `(m+1)` means the `+1` indices are always in range, which removes every boundary check from the code. That single decision is most of the difference between this being straightforward and being unpleasant.

**LC 2132 Stamping the Grid** is a good demonstration of the two techniques working together, and of why they are separate tools. It asks whether every empty cell of a grid can be covered by stamps of a fixed size, where a stamp may not overlap an occupied cell. The solution uses:

1. A two-dimensional prefix sum over the occupied cells, which makes it constant time to check whether a stamp placed at a given position covers only empty cells.
2. A two-dimensional difference array, which records a stamp at each position where one fits.
3. A running-sum pass over that difference array, giving the number of stamps covering each cell.
4. A final check that every empty cell is covered at least once.

The insight is that "can a stamp go here" and "what does a placed stamp cover" are separate questions needing separate structures, and recognising that is what makes the problem tractable.

---

## Part 5 · Prefix techniques for operations other than sums

The identity works for any operation that can be undone, since the range value is recovered by combining the prefix at the right end with the inverse of the prefix at the left end.

**Exclusive or** works, and is particularly convenient because it is its own inverse, so a range is `X[r+1] ^ X[l]`. This covers LC 1310 and CSES *Range Xor Queries*.

**Products modulo a prime** work using modular inverses, but break if any element is zero, so zeros need counting separately.

**Minimum and maximum do not work**, because there is no way to remove an element's contribution from a prefix. This is a genuine boundary rather than a technicality, and it is where sparse tables from chapter [[09 Fenwick Offline and Mos]] and segment trees from chapter [[08 Segment Trees]] become necessary. Attempting to build a prefix-minimum array and use it for ranges is a common early mistake.

**Counts of each character** work, where `cnt[c][i]` holds the number of occurrences of character `c` among the first `i` characters. This gives constant-time range counts per character, and it is extremely useful in string problems from chapter [[18 Strings]] while being frequently forgotten.

---

## Part 6 · Wrapping ranges

**LC 918 Maximum Sum Circular Subarray** asks for the largest sum over a contiguous piece of an array that is allowed to wrap around the end.

There are two cases. If the best piece does not wrap, ordinary Kadane's algorithm finds it. If it does wrap, then the part *not* taken is a non-wrapping piece, and to maximise what is taken you minimise what is left, so the answer is the total minus the smallest non-wrapping sum.

The answer is therefore the larger of the ordinary Kadane result and the total minus the minimum. There is one exception: when every element is negative, the total minus the minimum equals zero, which corresponds to taking nothing at all, and that is not permitted. Guarding with a check on whether the ordinary result is negative handles it.

The general principle is worth extracting. When a wrapping problem is solved by considering what is left out, check whether leaving out everything is a legal option, because it usually is not.

---

## Part 7 · Answering queries out of order

A problem is described as offline when all the queries are supplied in advance, which means you are free to answer them in any order you choose. That freedom is frequently the whole solution.

The pattern is to sort the queries by some key, sort the data by the same key, and move through both together while maintaining a structure that is correct as of the current position.

LC 1851 Minimum Interval to Include Each Query is a good example. Sorting the queries in increasing order lets you add intervals to a heap as their left endpoints are passed and discard those whose right endpoints have been passed, so the shortest interval covering the current query is always at the top.

The general machinery for this is the subject of chapter [[09 Fenwick Offline and Mos]]. What matters at this stage is recognising the option, since checking whether all the queries are available in advance takes a few seconds and occasionally converts an impossible-looking problem into a straightforward one.

---

## The ideas worth carrying forward

1. **Use `P[0] = 0` with an array of length `n + 1`.** This removes the special case at the left edge, which is where most prefix-sum errors come from.

2. **A contiguous piece is a pair of prefix values.** Iterating over right endpoints and asking a structure about earlier prefixes converts a quadratic search into a single pass.

3. **Initialise the count of the empty prefix to one.** Omitting it produces an answer short by a recognisable amount.

4. **Encoding two options as `+1` and `-1` turns a balance condition into a sum condition.** This is what makes LC 525 work, and it applies to bracket problems and majority problems too.

5. **A difference array is a sweep that needed no sorting**, because array indices are already in order.

6. **The two-dimensional difference update marks four corners with alternating signs**, and allocating one extra row and column removes every boundary check.

7. **Prefix techniques work for any operation that can be undone**, including sums, exclusive or, products modulo a prime, and per-character counts. They do not work for minimum or maximum, which is where segment trees and sparse tables begin.

8. **A wrapping range is either an ordinary range or the complement of one.** Check whether the empty complement is a legal choice.

9. **Negative values break sliding windows but not prefix sums.** Noticing negatives in a sum problem is a signal to work here rather than in chapter [[05 Sliding Window and Two Pointers]].

10. **When the question about earlier prefixes becomes a range query rather than a lookup, the problem has moved beyond this chapter** and needs a Fenwick tree, a merge sort, or an ordered container. LC 327 and LC 493 are exactly that step.

---

## Where people lose these problems

**Omitting the count of the empty prefix.** The answer comes out short by the number of prefixes equal to the target.

**Mishandling negative remainders.** In C++ and Java, `-7 % 3` is `-1` rather than `2`, so any hashing by remainder needs `((x % k) + k) % k`.

**Overflow.** A hundred thousand elements of up to `10^9` produce prefix sums around `10^14`, so 64-bit integers are needed as a matter of course.

**Getting the two-dimensional indices wrong.** Allocating `(n+1)` by `(m+1)` and using the convention where `P[i][j]` covers the first `i` rows and `j` columns means every formula uses the same offsets and there are no exceptions to remember.

**Forgetting the final pass over a difference array, or running it twice.**

**Building a prefix-minimum array.** There is no inverse for the minimum, so a sparse table is needed instead.

**In LC 363, missing the two-part structure.** The problem asks for the largest rectangle sum not exceeding a limit. The approach is to fix a pair of rows, reduce the rectangle to a one-dimensional array of column sums, and then solve the one-dimensional problem, which is the largest range sum not exceeding the limit and is handled with an ordered set of prefix values and a `lower_bound` call. Two separate ideas are stacked, and the row-pair reduction reappears in LC 85 and LC 1074.

**In LC 1074, over-complicating the inner problem.** It uses the same row-pair reduction with the hash-map counting pattern rather than an ordered set, which makes it the easier of the two. Solving it before LC 363 makes the harder one considerably more approachable.

---

## Working through the problem list

### Block 1 · The basic patterns

- **CSES Subarray Sums I** — *count ranges summing to x, with all values positive.* Also solvable as a sliding window, and doing it both ways shows why the window is valid here.
- **CSES Subarray Sums II** — *the same, with negative values permitted.* The window no longer applies, so the hash map is forced. This pair is the clearest illustration of the boundary between chapters 05 and 06, and they are best done consecutively.
- **CSES Subarray Divisibility** — *count ranges whose sum is divisible by n.* Remainder counting, with the negative-modulo trap.
- **CSES Range Xor Queries** — *report the exclusive or of a range.*
- **LC 1310 XOR Queries of a Subarray** — *the same in LeetCode form.*
- **LC 525 Contiguous Array** — *find the longest range with equal numbers of zeros and ones.* The `+1` and `-1` encoding.
- **LC 523 Continuous Subarray Sum** — *is there a range of length at least two whose sum is a multiple of k.* First index per remainder, with edge cases around the length requirement.
- **LC 974 Subarray Sums Divisible by K** — *count ranges whose sum is divisible by k.*

### Block 2 · Building on the basics

- **LC 918 Maximum Sum Circular Subarray** — *largest range sum where the range may wrap.* The complement approach with the all-negative guard.
- **LC 2100 Find Good Days to Rob the Bank** — *find days with enough non-increasing days before and non-decreasing days after.* Two directional passes then a combination, structurally the same as LC 135 in chapter [[03 Greedy and Exchange Arguments]].
- **LC 2381 Shifting Letters II** — *apply many range shifts to a string.* A one-dimensional difference array.
- **LC 2536 Increment Submatrices by One** — *apply many rectangle increments to a grid.* The two-dimensional difference array, worth getting right here before attempting LC 2132.
- **CSES Forest Queries** — *count trees in a rectangle of a static grid.* A two-dimensional prefix sum.
- **LC 1074 Number of Submatrices That Sum to Target** — *count rectangles summing to a target.* Row-pair reduction with hash-map counting.
- **LC 3179 Find the N-th Value After K Seconds** — *apply prefix sums repeatedly and report one entry.* Simulating works for small inputs, and noticing that repeated prefix sums produce binomial coefficients gives the closed form. This connects to chapter [[19 Combinatorics and Number Theory]].

### Block 3 · Where prefix sums meet another structure

- **LC 862 Shortest Subarray with Sum at Least K** — *shortest range with sum at least k, negatives permitted.* A monotonic deque over prefix values, holding indices with increasing prefix sums. The front is discarded while the current prefix exceeds it by at least `k`, recording the answer, and the back is discarded while its prefix is at least the current one, because a later and smaller prefix is better in every respect. Both discards need their own justification, and being able to give both is the point of the exercise. This is the single most instructive problem in section F.
- **LC 363 Max Sum of Rectangle No Larger Than K** — *the largest rectangle sum not exceeding k.* Row-pair reduction with an ordered set.
- **CSES Forest Queries II** — *the same as Forest Queries but with updates.* A two-dimensional Fenwick tree, so read chapter [[09 Fenwick Offline and Mos]] first.

### Block 4 · The harder ones

- **LC 2132 Stamping the Grid** — *can every empty cell be covered by non-overlapping-with-obstacles stamps.* Both two-dimensional structures in one problem, and best attempted after LC 2536.
- **LC 1735 Count Ways to Make Array With Product** — *count arrays of length n whose product is k.* This is a combinatorics problem: factorise `k` and distribute each prime's exponent across the positions. It is filed in section F on the sheet but belongs to chapter [[19 Combinatorics and Number Theory]], and is best attempted there.

---

**A reasonable target here is around 85% of submissions passing first time.**

These are mechanical techniques with a fixed set of traps. When accuracy is lower, the cause is nearly always the empty-prefix count, negative remainders, overflow, or a two-dimensional index error, and checking those four is quicker than looking for a conceptual mistake.

---

## Check yourself

1. Write the prefix convention and the range formula, then say which special case the convention eliminates.
2. Why is the count of the empty prefix initialised to one?
3. What transformation turns "equal numbers of zeros and ones" into a condition about sums?
4. Write the four-corner two-dimensional difference update. Why is the array allocated with an extra row and column?
5. For which operations does the prefix technique work? Name one where it does not, and say what replaces it.
6. State both cases for the maximum wrapping range, and the guard the second case needs.
7. In LC 862, justify both of the deque's discard rules.
8. What two techniques are combined in LC 2132, and which question does each one answer?
9. Given interleaved range updates and range queries, why is a difference array unsuitable, and what replaces it?
