---
tags: [dsa, guide, binary-search, parametric-search]
chapter: 4
sheet-section: D
---

# Chapter 4 · Binary Search on the Answer & Parametric Search

> **Read this before you start the problems.** No prior familiarity is assumed; each idea is introduced with an example you can follow from scratch.

Back to [[00 Guide Index]] · Sheet section **D** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

Most people first meet binary search as a way of finding a value inside a sorted array. The technique in this chapter uses the same mechanism for something different: rather than searching a list of items, you search the range of possible answers.

The reason this works is that many optimisation questions are much easier to answer when the answer is supplied to you. Asking for the smallest ship capacity that lets you deliver all packages within a deadline is hard. Asking whether a specific capacity is sufficient is easy, because you can simply fill days greedily and count them. If checking a candidate is easy and the property behaves consistently as the candidate grows, you can find the boundary between the failing candidates and the succeeding ones in a few dozen checks.

The consequence is that the difficulty of these problems relocates. The binary search itself is six lines of boilerplate that never changes. The whole of the thinking goes into the checking function, which is usually a greedy, a counting pass, a graph search, or occasionally a dynamic program. When a problem in this block feels hard, the difficulty is nearly always in that function rather than in the search around it.

---

## What these problems look like

This structure is a convenient way for a setter to turn any easy feasibility question into a harder-looking optimisation question, which is why it appears so often. Several problems on the sheet are the same problem with different nouns.

The signals, roughly in order of how reliable they are:

- **The words "minimise the maximum" or "maximise the minimum."** Split Array Largest Sum, Minimise Max Distance, Magnetic Force, Divide Chocolate and Minimized Maximum all use this phrasing, and it is close to a direct statement of the technique.
- **A request for the smallest value with some property**, where checking a specific value is straightforward but there are far too many candidates to try individually.
- **A rate, speed or capacity that you choose**, combined with a limit on total time or total count.
- **An answer with a wide numeric range, where the property behaves consistently as that number grows.** More capacity is never worse, and more time is never worse. This consistency is the actual requirement.
- **A request for the k-th smallest of some derived quantity**, such as pairwise distances or entries of a multiplication table. Here you search over the value and count how many quantities fall at or below it.

---

## Part 1 · The consistency requirement

The technique is valid when there is a checking function whose result behaves consistently as its input grows: false for every value below some threshold and true for every value at or above it, or the mirror image of that.

Before writing anything, it is worth completing this sentence out loud:

> "If a capacity of `x` works, then a capacity of `x + 1` also works, because ..."

The reason is usually obvious once stated, since a larger ship can carry anything a smaller ship could. The value of saying it is that it catches the cases where the property does not in fact behave that way, which are easy to miss when the problem is phrased confidently.

A property that fails the test is "find `x` such that exactly `k` elements are smaller than `x`", because that is not something that switches from false to true and stays true. When you meet a condition of that kind, you generally want to search over a count and use a boundary rule rather than a plain true-or-false check.

---

## Part 2 · The template

There are many ways to write a binary search, and most of them contain an off-by-one somewhere. It is worth learning one form thoroughly and using it every time.

```cpp
// Find the SMALLEST x in [lo, hi] with check(x) == true.
// Requires: check is false below some threshold and true at and above it, and check(hi) is true.
long long lo = <smallest conceivable answer>, hi = <largest conceivable answer>;
while (lo < hi) {
    long long mid = lo + (hi - lo) / 2;      // rounds down; cannot overflow
    if (check(mid)) hi = mid;                // mid might be the answer, so keep it
    else            lo = mid + 1;            // mid is definitely not, so discard it
}
return lo;                                   // lo == hi == the boundary
```

Four properties of this particular form are worth understanding rather than memorising.

The loop condition is `lo < hi` rather than `lo <= hi`, which means the loop ends when the range has narrowed to a single value, and that value is the answer. There is no adjustment afterwards and therefore no decision about whether to return `lo` or `lo - 1`.

The true branch assigns `hi = mid` rather than `hi = mid - 1`, because `mid` satisfied the property and might be the smallest value that does.

The midpoint rounds down, which combined with `hi = mid` guarantees the range strictly shrinks on every iteration. This is why the mirrored form, which searches for the largest value satisfying a property and assigns `lo = mid`, must round the midpoint up instead. Using a rounding-down midpoint in the mirrored form produces an infinite loop, and this is the most common binary search bug there is.

The midpoint is computed as `lo + (hi - lo) / 2` rather than `(lo + hi) / 2`, which matters once the bounds approach `10^18`.

Because of the infinite-loop hazard in the mirrored form, it is worth always phrasing the problem as "find the smallest `x` such that something is true." When a problem asks for the largest value with a property, search instead for the smallest value that fails the property and subtract one. Reframing costs a few seconds of thought, whereas maintaining two templates costs debugging time on the occasions you reach for the wrong one.

---

## Part 3 · The checking function

Since the search never changes, the checking function is where the problem actually lives. It is usually one of four things.

**A greedy simulation.** Shipping packages with a capacity of `c` means walking the array and starting a new day whenever adding the next package would exceed the capacity, then comparing the day count against the limit. Split Array Largest Sum and Divide Chocolate are the same procedure with different wording, and Minimum Days to Make Bouquets counts consecutive runs instead.

```cpp
auto check = [&](long long cap) {
    int days = 1; long long cur = 0;
    for (long long w : weights) {
        if (cur + w > cap) { days++; cur = 0; }
        cur += w;
    }
    return days <= D;
};
```

**A counting pass.** To answer how many pairs have a distance no greater than `x`, sort the array and use two pointers, which is linear. To count entries of an `m` by `n` multiplication table that are at most `x`, sum `min(n, x / i)` over the rows. Then binary search for the smallest `x` whose count reaches `k`. This is LC 719 and LC 668.

**A graph search.** In Swim in Rising Water, checking a value `t` means asking whether a path exists using only cells with elevation at most `t`, which is one breadth-first search. Path With Minimum Effort and Find the Safest Path are structured the same way.

These three deserve a note. Each of them also has a Dijkstra solution in which the notion of distance is redefined so that the cost of a path is the maximum edge on it rather than the sum. Solving one of them both ways, as the sheet suggests for LC 778, is among the more valuable exercises in this block, because it shows that binary search combined with reachability and Dijkstra with a modified relaxation rule are two views of the same computation. Which is faster depends on the numbers, since one costs a factor of `log` in the number of edges and the other a factor of `log` in the range of weights.

**A dynamic program.** This is less common, but the check in LC 2560 amounts to asking whether `k` non-adjacent elements can all be kept below a threshold, which sits close to a dynamic program even though a greedy suffices.

---

## Part 4 · Real-valued answers

Occasionally the answer is not an integer, as in Minimize Max Distance to Gas Station. In that case:

```cpp
double lo = 0, hi = 1e9;
for (int iter = 0; iter < 100; iter++) {     // fixed iteration count
    double mid = (lo + hi) / 2;
    if (check(mid)) hi = mid; else lo = mid;
}
```

A fixed iteration count is preferable to looping until the interval is smaller than some tolerance. One hundred halvings reduce a range of `10^9` by a factor of `2^100`, which is far beyond the precision a `double` can represent, and it takes a negligible amount of time. Looping on a tolerance instead can fail to terminate because of floating-point representation, and choosing the tolerance is an additional thing to get wrong.

Where the answer is a rational number with a bounded denominator, multiplying through and searching over integers is usually cleaner still.

---

## Part 5 · Finding the k-th smallest of an implicit set

A common variation asks for the k-th smallest value of some quantity defined over a space too large to enumerate, such as all pairwise distances in an array.

The approach has three parts. Search over the value rather than over the items. Define the check on a candidate `v` as asking whether at least `k` of the quantities are less than or equal to `v`. The answer is then the smallest `v` for which that holds.

It is worth convincing yourself once that the result is guaranteed to be a value that actually occurs, since the search ranges over all numbers rather than over achievable ones. The count of quantities at or below `v` only increases at values that are themselves achievable, so the smallest `v` at which the count first reaches `k` must be one of them. Knowing this avoids adding unnecessary correction steps afterwards.

This structure covers LC 719 and LC 668 on the sheet, and it appears frequently in assessments more generally.

---

## Part 6 · A note on the Aliens trick

There is a deeper version of this idea in which you search not over the answer but over a penalty parameter. A problem asking you to split an array into exactly `k` parts at minimum cost can be transformed into one where each split costs an additional `λ` and the number of splits is unconstrained, which removes a dimension from the dynamic program. You then search over `λ` until the unconstrained optimum happens to use exactly `k` splits.

This requires the cost to behave convexly as a function of `k`, which is the substantive condition. It is covered properly in chapter [[17 DP Optimization]], and the sheet's instruction to revisit LC 410 with the Aliens trick is a deliberate callback. There is no need to attempt it now, but it is worth knowing that the same instinct extends one level further.

---

## The ideas worth carrying forward

1. **State the consistency sentence before writing code.** Completing "if `x` works then `x + 1` works, because ..." takes ten seconds and catches the cases where the technique does not apply.

2. **Use one template, which finds the smallest value satisfying the property.** When a problem asks for the largest, reframe it as the smallest value failing the property and subtract one.

3. **The form is `lo < hi`, `hi = mid`, `lo = mid + 1`, midpoint rounded down, return `lo`.** No adjustment is needed after the loop.

4. **The mirrored form requires the midpoint to round up**, and using a rounding-down midpoint there produces an infinite loop rather than a wrong answer. This is the reason for keeping to a single template.

5. **The search is boilerplate and the checking function is the problem.** A hard problem in this block is asking you a greedy, counting or reachability question wrapped in a search.

6. **The phrase "minimise the maximum" is close to an explicit statement of the technique**, as is "maximise the minimum."

7. **A request for the k-th smallest of an implicit set becomes a search over values combined with a count of how many fall below.** The counting pass is usually two pointers or a per-row formula.

8. **Binary search with reachability and Dijkstra with a modified relaxation solve the same class of problem.** Understanding this connection makes chapter [[11 Shortest Paths and State Space]] considerably easier.

9. **For real-valued answers, run a fixed hundred iterations** rather than looping on a tolerance, since a hundred halvings exceed the available precision anyway.

10. **Choose the upper bound so that it is provably large enough** rather than hoping. The total weight for a capacity problem, or the worst-case serial time for a time problem. An extra factor of two in the iteration count costs nothing, whereas an upper bound that is too small produces a wrong answer that looks like a logic error.

---

## Where people lose these problems

**Using the mirrored template with a rounding-down midpoint.** The symptom is a timeout with no apparent cause, and the fix is to use the single template above.

**Setting the upper bound too low.** The solution is correct on the samples and wrong on a large test. Deriving the bound rather than guessing at it prevents this.

**Setting the lower bound too high.** For problems where a single element cannot be split, the smallest sensible capacity is the largest element rather than zero. The safer approach is to allow a low starting bound and make sure the checking function correctly rejects impossible values.

**Overflow inside the check.** In LC 2141 the check involves summing values capped at `t`, and both sides of the comparison reach around `10^14`.

**A property that does not behave consistently.** The search converges on a value that then fails the check. This usually indicates a bug in the checking function, but it can indicate that the property genuinely does not behave consistently. Evaluating the check at every value on a small input and printing the results reveals which, since a valid property produces a clean run of failures followed by a clean run of successes.

**Confusing "at least k" with "exactly k" in a counting check.** For a k-th smallest problem the correct condition is that the count reaches `k`, since requiring it to equal `k` does not behave consistently when there are duplicate values.

**In LC 2141, summing the batteries without capping them.** The check is not whether a given computer can run for `t` minutes but whether the total available charge, with each battery capped at `t`, is at least `n` times `t`. The cap is necessary because a single computer runs for at most `t` minutes in total, so charge beyond that in one battery cannot be used. Writing the sum without the cap is the natural first attempt and is incorrect.

**In LC 2513, mishandling the inclusion and exclusion.** Given a limit, the check must count how many numbers up to that limit are divisible by neither divisor, by only the first, and by only the second, and the four-way accounting is most of the work.

---

## Working through the problem list

### Block 1 · The repeated family

These are the same problem several times over. Solving three of them establishes the pattern; solving all seven consecutively is not a good use of time, so the rest are better saved for a review pass.

- **LC 875 Koko Eating Bananas** — *choose an eating speed so that all piles are finished within h hours.* The standard first example.
- **LC 410 Split Array Largest Sum** — *split an array into k parts minimising the largest part sum.* The same problem again. It also has a classic quadratic dynamic program, which is worth writing because it is the version the Aliens trick later optimises.
- **LC 1482 Minimum Days to Make m Bouquets** — *wait until enough adjacent flowers have bloomed.* The check counts consecutive runs.
- **LC 1231 Divide Chocolate** — *cut a chocolate bar into k+1 pieces maximising the smallest piece.*
- **LC 2064**, **LC 2226**, **LC 1760** — *three further restatements.* Solve one, read the other two, and return to them later.

### Block 2 · Where the check becomes interesting

- **LC 1552 Magnetic Force Between Two Balls** — *place m balls in baskets maximising the minimum gap.* The check places balls as early as possible.
- **LC 2141 Maximum Running Time of N Computers** — *run n computers simultaneously for as long as possible using a set of batteries.* The capping rule described above, and the clearest example of the check carrying the whole problem.
- **LC 2560 House Robber IV** — *choose k non-adjacent houses minimising the largest amount taken.* The check scans left to right taking any eligible house not adjacent to the last one taken.
- **LC 1898 Maximum Number of Removable Characters** — *how many scheduled removals can be applied before p stops being a subsequence of s.* The check is a two-pointer subsequence test against a boolean mask.
- **LC 2439 Minimize Maximum of Array** — *move value leftwards between adjacent elements to minimise the maximum.* The check is a condition on prefix averages. There is also a direct linear solution, and comparing the two is worthwhile.
- **CSES Factory Machines** — *k items across machines of different speeds, in minimum time.* Watch for overflow, since the candidate time reaches `10^18` and the sum needs capping.
- **LC 774 Minimize Max Distance to Gas Station** — *add k stations to minimise the largest gap.* Real-valued, or made integral by scaling.
- **LC 2861 Maximum Number of Alloys** — *how many alloys can be produced given stock, costs and a budget.* The check loops over machines and computes the cost of the shortfall.

### Block 3 · Searching over values

- **LC 668 Kth Smallest in Multiplication Table** — *the k-th smallest entry of an m by n multiplication table.* The count is a sum of `min(n, v / i)` across the rows.
- **LC 719 Find K-th Smallest Pair Distance** — *the k-th smallest distance among all pairs.* Sort, then count with two pointers.

### Block 4 · Searching over graphs

- **LC 778 Swim in Rising Water** — *find the earliest time at which a path exists across a grid as water rises.* Worth doing both ways, first as a search with reachability and then as Dijkstra where the cost of a path is its largest cell. Spending a few minutes on why the two agree pays off later.
- **LC 1631 Path With Minimum Effort** — *find a path minimising the largest height difference between consecutive cells.* The same duality, and it also yields to a union-find approach from chapter [[10 DSU Advanced]].
- **LC 2812 Find the Safest Path in a Grid** — *maximise the minimum distance from any thief along a path.* A multi-source breadth-first search to build the distance field, followed by either a search with reachability or a maximising Dijkstra.

### Block 5 · The harder ones

- **LC 2513 Minimize the Maximum of Two Arrays** — *fill two arrays with distinct integers avoiding given divisors, minimising the largest number used.* The accounting inside the check is the difficulty.
- **LC 1889 Minimum Space Wasted From Packaging** — *choose a supplier of boxes minimising wasted space.* Sorting plus a binary search per box size. The naive approach is quadratic and does not fit the constraints.

---

**A reasonable target here is around 85% of submissions passing first time**, which is the highest of any block on the sheet.

That is appropriate because the template is fixed, the signals are unambiguous, and the ways to go wrong form a short and checkable list. Accuracy below that level is most often caused by overflow or by an upper bound that was guessed rather than derived, both of which can be corrected in a single sitting.

---

## Check yourself

1. Write the template from memory, then explain why the true branch assigns `hi = mid` and why the loop condition uses `<`.
2. Why does the mirrored form loop forever with a rounding-down midpoint?
3. State the consistency sentence for LC 875.
4. For the k-th smallest pair distance, what is the check, and why is the result guaranteed to be a distance that actually occurs?
5. Write the check for LC 2141, and explain why summing the batteries without capping is wrong.
6. Name two problems here where the check is a reachability query, and describe the Dijkstra formulation of the same problem.
7. Why run exactly a hundred iterations for a real-valued search rather than looping until the interval is small?
8. Your search returns a value that fails the check. What are the two most likely causes?
