---
tags: [dsa, guide, bitmask, dp, meet-in-the-middle, sos]
chapter: 15
sheet-section: O
---

# Chapter 15 · Bitmask DP, Broken Profile & Subset Convolution

> **Read this before you start the problems.** Each idea is introduced with a small example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **O** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

A bound of around twenty on some quantity in a problem is not really a constraint in the ordinary sense. It is closer to an instruction, since twenty is far too small to justify anything better than an exponential algorithm and far too large to brute-force by simply trying every possibility one at a time without structure. Once that bound is noticed, representing a subset of those twenty items as the bits of an integer, and organising a dynamic program around those integers, is close to a forced move.

The genuine difficulty in current problems is that this bound of twenty is rarely the number stated directly as the size of the input. It is far more often a smaller, hidden quantity buried inside a larger problem: the number of distinct values in an array of a hundred thousand elements, or the number of relevant skills, or the number of primes below some limit, or the width of a grid. Recognising that a bitmask applies at all is therefore a matter of scanning every number in the problem for one that happens to be small, rather than simply checking whether the input size itself is small.

---

## What these problems look like

The bound may be stated directly, such as at most twenty or twenty-two items, or at most sixteen people. Far more often it is hidden:

- "The array contains at most twenty distinct values", while the array itself has a hundred thousand entries.
- "Each job requires some subset of sixteen possible skills."
- "Values are at most seventy", which limits the relevant prime factors to a small fixed set.
- "There are at most twelve colours."
- "Grid width is at most fifteen", which signals a profile dynamic program where the mask represents one column or row.

The general habit worth building is that whenever a bound is read in a problem statement, it is worth asking whether two raised to that bound is small enough to be a plausible number of states, since if it is, that bound is very likely the one the intended solution is built around.

---

## Part 1 · Working with bits

A short list of operations is used constantly and is worth being completely fluent with:

```cpp
mask | (1 << i)          // set bit i
mask & ~(1 << i)         // clear bit i
mask ^ (1 << i)          // flip bit i
(mask >> i) & 1          // test bit i
__builtin_popcount(mask) // count of set bits (use popcountll for 64-bit masks)
__builtin_ctz(mask)      // index of the lowest set bit
mask & -mask             // isolate the lowest set bit
mask & (mask - 1)        // clear the lowest set bit
(1 << n) - 1             // a mask with the lowest n bits set
```

One detail causes real errors: writing `1 << 31` overflows a signed integer. Writing `1LL << i` whenever `i` might reach thirty-one or beyond avoids this.

---

## Part 2 · Assigning items one at a time

**AC EDPC O Matching** is the clearest introduction. Given `n` people on each side of a matching problem with a compatibility table, count the number of perfect matchings.

The observation that makes this efficient is that if a mask has `k` bits set, then exactly the first `k` people on one side must already have been assigned, since assignments happen in order. This means the count of assigned people does not need to be tracked separately as an extra dimension of state; it is recovered directly from the mask by counting its bits.

```cpp
vector<long long> dp(1 << n, 0);
dp[0] = 1;
for (int mask = 0; mask < (1 << n); mask++) {
    if (!dp[mask]) continue;
    int i = __builtin_popcount(mask);          // the person being assigned now
    if (i == n) continue;
    for (int j = 0; j < n; j++)
        if (compatible[i][j] && !(mask >> j & 1))
            dp[mask | (1 << j)] = (dp[mask | (1 << j)] + dp[mask]) % MOD;
}
// the answer is dp[(1 << n) - 1]
```

This runs in time proportional to the number of masks times `n`. The trick of deriving one dimension of state from the population count of another is worth remembering on its own, since it recurs in LC 1434, LC 1125 and any problem where items are assigned strictly in order.

---

## Part 3 · Visiting every item, tracking the current position

A second common shape tracks not just which items have been used but also which one was used last, which is needed whenever the cost of the next choice depends on where you currently are.

```cpp
for (int mask = 1; mask < (1 << n); mask++)
    for (int i = 0; i < n; i++) {
        if (!(mask >> i & 1) || dp[mask][i] == INF) continue;
        for (int j = 0; j < n; j++)
            if (!(mask >> j & 1))
                dp[mask | (1<<j)][j] = min(dp[mask | (1<<j)][j], dp[mask][i] + cost[i][j]);
    }
```

This costs time proportional to the number of masks times `n` squared, which for twenty items is around four hundred million, comfortable for eighteen items and closer to the edge at twenty.

CSES *Hamiltonian Flights* is the counting version of this shape. LC 943 Find the Shortest Superstring uses it with the cost between two items being how much overlap merging them saves, and it additionally requires reconstructing the actual path, which needs a parent array recording, for each state, which choice produced it, followed by walking that array backwards from the best final state. Practising this reconstruction here, rather than skipping it, is worthwhile because it is easy to be unable to do under pressure if it has never actually been written before.

---

## Part 4 · Partitioning into groups

Some problems ask you to split a set of items into several groups, where each group's contribution to the total score can be computed independently once you know which items are in it.

**Enumerating submasks of a mask** is the idiom that makes this efficient:

```cpp
for (int sub = mask; sub; sub = (sub - 1) & mask) {
    // sub ranges over every non-empty submask of mask
}
// the empty submask, if needed, must be handled separately
```

The total work of enumerating every submask of every mask, across all masks, is proportional to three raised to the power `n` rather than four raised to that power, because each of the `n` items independently falls into one of three categories relative to a given mask and submask pair: outside the mask entirely, inside the mask but not the submask, or inside both. For twenty items, three to the twentieth power is a little over three billion, which is at the edge of being fast enough, while for sixteen items it is around forty-three million, comfortably fast.

**AC EDPC U Grouping** is the clean statement of this pattern: the best score for a set is the best, over every way of choosing one group to peel off, of that group's own score plus the best score for whatever remains. To avoid considering the same partition of the set multiple times over in different orders, the standard fix is to always require that the lowest set bit of the current mask belongs to the chosen group, which forces a unique sequence in which groups are peeled off and is essential when the operation being combined is a count rather than a maximum, since a maximum would tolerate the redundancy while a count would not.

---

## Part 5 · Profile dynamic programming on a grid

The trigger here is a grid where one dimension is small, around fifteen or fewer, combined with a tiling or placement task with adjacency constraints.

**CSES Counting Tilings** asks for the number of ways to tile a grid with dominoes. The state processes cells one at a time in reading order, and the mask represents which of the next several cells are already filled by a domino placed earlier that sticks into them. At each cell the choices are to skip it if it is already filled, to place a vertical domino, or to place a horizontal domino.

This is genuinely fiddly to implement and the direct return on practising it for assessment purposes is modest. What is worth taking away regardless is the underlying idea: when the boundary between the part of a grid already decided and the part not yet decided is narrow, that boundary can be encoded as a mask, even when the state it represents is more elaborate than a simple set membership.

LC 1815 Maximum Number of Groups Getting Fresh Donuts is a more assessment-friendly instance of a related idea, where only the remainders of certain counts modulo a batch size matter, giving a small number of relevant values that are tracked as a tuple of counts and hashed, rather than encoded as a bitmask in the strict sense. Recognising that the state can sometimes be "a small multiset of counts" rather than "a set of flags" is a useful generalisation of the bitmask idea.

---

## Part 6 · Summing over all submasks efficiently

Occasionally a problem needs, for every mask, an aggregate over all of its submasks, and computing each one by explicit submask enumeration costs the three-to-the-power-`n` total from Part 4, which is sometimes too slow. A different technique computes all of these aggregates together in time proportional to the number of masks times `n`.

```cpp
for (int i = 0; i < n; i++)
    for (int mask = 0; mask < (1 << n); mask++)
        if (mask >> i & 1) f[mask] += f[mask ^ (1 << i)];
```

The way to read this is that it processes one bit position at a time, and after processing bit `i`, the value at each mask has accumulated the values of every submask that differs from it only in the bits at position `i` or lower. This is exactly the same idea as a running prefix sum from chapter [[06 Prefix Sums and Difference Arrays]], applied independently along each of the `n` dimensions of the space of subsets rather than along a single array.

This technique appears less often in assessment-style problems than the others in this chapter, but when it is needed there is usually no alternative that runs fast enough.

---

## Part 7 · When the bound is around forty

A bound of around forty, rather than twenty, is a distinct signal, since two raised to the fortieth power is far too large but two raised to the twentieth power, computed twice, is entirely manageable.

The approach is to split the items into two halves, enumerate every subset of each half separately, and then combine the two halves using sorting and either binary search or a two-pointer scan, or occasionally a hash map.

LC 2035 splits the items into two halves and, within each half, groups the subsets by their size, since a chosen subset of one half of a particular size needs to be paired with a subset of the other half of a complementary size whose sum is as close as possible to a target. Sorting each size group and binary searching across halves finds the best pairing.

CSES *Meet in the Middle* is the direct template: count subsets summing to a target by enumerating both halves, sorting one, and binary searching the other.

CSES *Beautiful Subgrids* uses a related but distinct idea: representing each row of a large grid as a bitset of columns and comparing pairs of rows using bitwise operations processes many comparisons at once at the level of machine words, which is a form of exploiting small state through hardware-level parallelism rather than through halving a search space, and it is worth knowing as a separate tool from meeting in the middle.

---

## The ideas worth carrying forward

1. **A bound of around twenty suggests a bitmask, and a bound of around forty suggests meeting in the middle.** Both are close to explicit instructions once recognised.

2. **The small dimension is often hidden.** It is worth checking every number in a problem statement, not only the size of the main input, for one that is small enough to raise two to its power comfortably.

3. **The population count of a mask gives a dimension of state for free**, whenever items are being assigned strictly in order.

4. **Enumerating submasks costs three to the power `n` in total across all masks**, because each item independently falls into one of three categories with respect to a mask and its submask.

5. **When partitioning into groups, forcing the lowest set bit into the chosen group** ensures every partition is counted exactly once rather than once per ordering of its groups.

6. **Summing over all submasks efficiently is a prefix sum performed independently along each bit position.**

7. **`1 << 31` overflows.** Writing `1LL << i` avoids this whenever the shift amount could reach that far.

8. **Reconstructing an actual path from a bitmask dynamic program needs a parent array**, and it is worth practising this rather than encountering it for the first time under pressure.

9. **Sometimes the state is a small multiset of counts rather than a set of flags.** LC 1815 is the example worth remembering for this.

10. **Bitsets provide a substantial constant-factor speed-up on brute-force comparisons over boolean data**, independent of the bitmask techniques above.

11. **Multiply out the complexity, such as two to the power `n` times `n` squared, and check it against a hundred million before committing to a design.** A bound of twenty with a squared inner loop is often too slow, while eighteen is comfortable, and doing this arithmetic first avoids writing a correct but too-slow solution.

---

## Where people lose these problems

**Overflow from `1 << i` on large shift amounts.** Use `1LL << i` when `i` could reach thirty-one or more.

**Iterating masks in an order incompatible with the dependency structure.** For a forward-filling dynamic program moving from a mask to a larger one obtained by adding a bit, iterating masks in increasing order is both necessary and sufficient, since a mask with an added bit is always numerically larger.

**Forgetting whether the empty submask is included.** The standard submask enumeration loop excludes it, and it must be added back explicitly if the problem needs it.

**Counting the same partition multiple times.** The lowest-set-bit rule from Part 4 is the fix.

**Allocating more memory than is available.** A table with two to the twentieth rows and twenty columns of 64-bit values uses over a hundred and fifty megabytes, and it is worth computing this before allocating, using a smaller integer type or a different structure if it does not fit.

**Not pruning unreachable states.** Skipping a mask whose stored value is still the initial "unreachable" sentinel, at the very top of the loop, is often a substantial and free speed-up.

**In LC 691 Stickers to Spell Word, masking the wrong thing.** The mask represents which characters of the *target* string, not which stickers, have been covered, and the standard pruning is always to attempt covering the lowest-indexed uncovered character next, trying every sticker containing it, which mirrors the lowest-set-bit idea from Part 4 and is what keeps the search fast.

**In LC 464 Can I Win, missing the early exits.** If the largest choosable number alone reaches the target, the first player wins immediately, and if the sum of every available number is less than the target, no one can ever win, and omitting the second check risks an unbounded search.

**In LC 473 Matchsticks to Square, reaching for a bitmask when backtracking with pruning is the better fit.** Sorting the sticks in decreasing order, skipping a length that has already failed at the same position, and abandoning a side early once it cannot possibly be completed are together faster and simpler than encoding the state as a mask.

---

## Working through the problem list

### Block 1 · Building the core idioms

- **LC 464 Can I Win** — *decide whether the first player can force a win by picking numbers to reach a target total.* A mask over the numbers used, combined with the two early exits described above.
- **LC 473 Matchsticks to Square** — *decide whether sticks can be arranged into a square.* Backtracking with pruning, worth contrasting against a bitmask solution to feel the difference.
- **CSES Elevator Rides** — *split a group into elevator trips of limited capacity, minimising the number of trips.* The stored value is a pair, the number of trips and the current trip's used capacity, compared lexicographically, which is a good demonstration that a dynamic programming value need not be a single number.
- **AC EDPC O · Matching** — *the population-count problem from Part 2.* Attempt this before anything else in the chapter.

### Block 2 · Assignment and covering

- **LC 1434 Number of Ways to Wear Different Hats to Each Other** — *count assignments of hats to people, where each person likes a subset of hats.* The mask is over people, who number at most ten, not over hats, which number up to forty. Good training for noticing which of two candidate dimensions is actually the small one.
- **LC 1125 Smallest Sufficient Team** — *choose the fewest people covering every required skill.* The mask is over the skills, with reconstruction of which people were chosen.
- **LC 691 Stickers to Spell Word** — *the target-mask problem with lowest-bit pruning.*
- **LC 1799 Maximize Score After N Operations** — *pair up numbers to maximise a sum of greatest-common-divisor terms.* At most seven pairs, meaning fourteen numbers, so the mask is straightforward once identified.
- **AC EDPC U · Grouping** — *the submask partition problem from Part 4.* This is where the lowest-set-bit rule should be learned properly.

### Block 3 · Visiting items with a tracked current position

- **CSES Hamiltonian Flights** — *count Hamiltonian paths through a directed graph.*
- **LC 943 Find the Shortest Superstring** — *find the shortest string containing every given string as a substring.* Practise the path reconstruction properly here rather than skipping it.
- **LC 3149 Find the Minimum Cost Array Permutation** — *find the minimum-cost, and lexicographically smallest, permutation under a given cost function.* The same shape with a lexicographic tie-break requiring careful reconstruction.
- **CSES Knight's Tour** — *find a path visiting every square of a chessboard.* This is Warnsdorff's heuristic combined with backtracking rather than dynamic programming, since the board is too large for a mask, and it is included here to make the point that "visit every cell" does not always imply a bitmask solution. **LC 2664 The Knight's Tour** is the small-board version where backtracking alone is sufficient.

### Block 4 · Partitioning

- **LC 1723 Find Minimum Time to Finish All Jobs** — *distribute jobs among workers minimising the maximum workload.* Solvable with a submask-enumeration dynamic program, and also with binary search on the answer combined with a feasibility check from chapter [[04 Binary Search on the Answer]]. Worth doing both, since the binary search version is often faster in practice.
- **LC 1655 Distribute Repeating Integers** — *satisfy customer order quantities using limited stock of each distinct value.* At most fifty quantities but far fewer distinct values, with the mask over the customers, who number at most ten.
- **LC 1815 Maximum Number of Groups Getting Fresh Donuts** — *the remainder-count state from Part 5.*

### Block 5 · Meeting in the middle

- **CSES Meet in the Middle** — *count subsets summing to a target.* The template.
- **LC 2035 Partition Array Into Two Arrays to Minimize Sum Difference** — *split an array into two equal-sized halves minimising the difference of their sums.* Grouped by subset size, with a binary search across the two halves. The most demanding meeting-in-the-middle problem here.
- **CSES Beautiful Subgrids** — *count pairs of rows sharing many columns of a particular value.* Bitset-accelerated brute force.

### Block 6 · Profile dynamic programming and further examples

- **CSES Counting Tilings** — *count domino tilings of a grid.* The broken-profile technique. Optional for assessment purposes, worth attempting if the idea itself is of interest.
- **CF 580D Kefa and Dishes** — *choose an ordered sequence of dishes maximising satisfaction, including pairwise bonuses for adjacent dishes.* A clean example of the "mask plus current position" shape from Part 3.
- **CF 8C Looking for Order** — *collect items in trips of at most two, minimising total travel distance.* The state always picks up the lowest-indexed remaining item next, another instance of the lowest-bit pruning idea, with reconstruction required.

---

**A reasonable target for this block is around 80% of submissions passing first time.**

Recognition is close to free once the habit of scanning for a hidden small dimension is established, and the implementations themselves tend to be short. The failures that do occur are almost always either masking the wrong dimension of the problem, or failing to check the arithmetic of two to the power `n` times `n` squared against the time limit before committing to an approach.

---

## Check yourself

1. What do a bound of around twenty and a bound of around forty each tell you about the intended approach?
2. Name four ways the small dimension of a problem can be disguised in its statement.
3. Why does the population count of a mask provide a free dimension of state in assignment problems?
4. Write the submask enumeration idiom. Why does the total work across all masks come to three to the power `n`?
5. When partitioning into groups, what rule prevents the same partition from being counted more than once?
6. Write the summing-over-submasks technique in three lines and describe what it computes.
7. In LC 1434, which quantity is masked, and why is it not the more obvious candidate?
8. How is the actual path reconstructed from a bitmask dynamic program solving a travelling-salesman-style problem?
9. For twenty items with a dynamic program costing two to the power `n` times `n` squared, is that fast enough? Work out the arithmetic.
10. What is the dynamic programming value in CSES Elevator Rides, and why is it a pair rather than a single number?
