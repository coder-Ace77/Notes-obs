---
tags: [dsa, guide, greedy, exchange-argument]
chapter: 3
sheet-section: C
---

# Chapter 3 · Greedy with a Proof Obligation

> **Read this before you start the problems.** Every technique here is introduced with a small worked example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **C** in [[1. Ultime DSA 2026 calibration]] · See also your existing note [[4. Exchange argument]]

---

## What makes these problems hard

Greedy algorithms are unusual in that a wrong solution behaves almost exactly like a correct one during the time you have to write it. A wrong greedy compiles, runs quickly, produces sensible-looking output, and passes the examples, because the examples were written to illustrate the problem rather than to distinguish between competing strategies. The failure only appears on the hidden tests, at which point you have no information about which of your assumptions was the faulty one.

Problem setters are aware of this. In practice, the sample cases for a greedy problem are frequently satisfied by the two or three most natural wrong orderings as well as by the correct one, so passing them tells you very little.

The consequence is that greedy is the one category where you cannot rely on testing to tell you whether you are right. You have to establish it beforehand, and the way to establish it is to show that a small local change to any solution never improves it. That argument is short — usually four or five lines of algebra — and this chapter is largely about how to produce it quickly.

---

## What these problems look like

Because you cannot rely on the samples, the useful question when you suspect a greedy is not "is this greedy?" but "which of the standard shapes is it, and can I state the supporting argument?"

There are four shapes, and nearly every greedy on this sheet is one of them.

**Sorting by a comparator, then taking in order.** The comparator is derived from the problem rather than guessed at. This covers weighted scheduling, EDPC X, and the ordering half of Course Schedule III.

**Regret, or heap-based, greedy.** You take everything optimistically and then undo the worst decision whenever the accumulated choices become infeasible.

**Stack-based greedy for lexicographic answers.** You maintain a stack together with a budget for how many more items may be discarded.

**Two-pass local repair.** You sweep left to right fixing violations, then right to left, and combine the two results.

If a problem does not fit any of the four, it is often dynamic programming presented in a way that suggests greedy, which is covered in chapter [[14 Dynamic Programming Core]].

---

## Part 1 · Producing the comparator by comparing two elements

Your existing note [[4. Exchange argument]] gives the proof structure. What follows is the working method for producing the comparator under time pressure, which is a slightly different thing.

The method is to ignore the general case entirely and consider exactly two elements. Take two items `A` and `B`, write down the cost of placing them in the order `AB`, write down the cost of `BA`, assert that `AB` is at least as good, and then rearrange the inequality until all the terms involving `A` sit on one side and all the terms involving `B` sit on the other. Whatever function of `A` ends up isolated is the sort key.

Here it is on the standard example. Each job has a processing time `t` and a weight `w`, jobs run one after another, and you want to minimise the sum of `w` times completion time.

Placing `A` first costs `w_A t_A + w_B (t_A + t_B)`, because `A` finishes at time `t_A` and `B` finishes at `t_A + t_B`. Placing `B` first costs `w_B t_B + w_A (t_A + t_B)`. Requiring the first to be no worse:

$$w_A t_A + w_B t_A + w_B t_B \le w_B t_B + w_A t_A + w_A t_B$$
$$w_B t_A \le w_A t_B$$
$$\frac{t_A}{w_A} \le \frac{t_B}{w_B}$$

So jobs should be sorted by increasing `t / w`.

The step worth paying attention to is the separation. All the terms mentioning `A` moved to one side and all the terms mentioning `B` moved to the other, which is what makes a total order possible in the first place.

That gives you a useful diagnostic. If the algebra refuses to separate, then no comparator-based greedy exists, because the right decision for a pair depends on something outside that pair. Trying the same manipulation on the knapsack problem demonstrates this: the terms will not separate, because whether an item is worth taking depends on the remaining capacity, which is a property of the whole state rather than of the two items. A failure to separate is therefore evidence that the problem is dynamic programming, which is useful information rather than a dead end.

Two further checks are worth performing once you have a comparator.

The first is that it defines a valid ordering, meaning that `cmp(a, b)` and `cmp(b, a)` are never both true. Writing the comparator as `a.t / a.w < b.t / b.w` with integer division does not define a valid ordering, and `std::sort` responds to invalid comparators by reading outside the array. Cross-multiplying to `a.t * b.w < b.t * a.w`, in 64-bit arithmetic, avoids both the division and the precision question.

The second is that comparing adjacent pairs is sufficient. This holds whenever the cost of a sequence depends only on adjacent relationships or on running totals, which is nearly always the case. The reasoning is the same as in bubble sort: if swapping any adjacent out-of-order pair never increases the cost, then repeatedly performing such swaps transforms any arrangement into the sorted one without ever making things worse.

---

## Part 2 · Regret greedy

This shape is worth more attention than the others, both because it appears frequently and because it is the one most people have not practised.

The idea is to avoid deciding whether to take each item. Instead you take everything as it arrives, keeping the committed items in a heap, and whenever the set of commitments becomes infeasible you remove the single worst item from it. Because the heap contains exactly the choices you have made, removing the worst one is cheap.

**The standard example** is LC 630 Course Schedule III. Each course has a duration `d` and a deadline `e`, courses are taken one after another starting from day 1, and you want to complete as many as possible.

```cpp
sort(courses.begin(), courses.end(), [](auto& a, auto& b){ return a[1] < b[1]; }); // by deadline
priority_queue<int> pq;      // max-heap of the durations we have committed to
long long time = 0;
for (auto& c : courses) {
    int d = c[0], e = c[1];
    time += d; pq.push(d);                          // take it optimistically
    if (time > e) { time -= pq.top(); pq.pop(); }    // over the deadline, so drop the longest
}
return pq.size();
```

There are two claims embedded in those nine lines, and both are worth being able to justify.

The first is that sorting by deadline is safe. If some set of courses can be completed in any order at all, then it can be completed in deadline order, which follows from an exchange argument on an adjacent out-of-order pair. That means it is enough to consider deadline order, and processing the timeline once becomes possible.

The second is that dropping the longest course is the right choice. At the moment the deadline is exceeded, you are holding `k + 1` courses and can keep at most `k`. Among all the ways to drop one, dropping the longest leaves the smallest total elapsed time, and a smaller elapsed time can never be worse for anything that follows, because every remaining deadline is at least as large as the current one. The choice therefore dominates all the alternatives rather than merely being plausible.

That second claim has a shape that recurs in every regret proof: among all the ways of shedding one commitment, the one you choose leaves the state at least as good as any other for everything that comes later.

The same skeleton appears in several disguises:

| Problem | What you take optimistically | What the regret step does |
|---|---|---|
| LC 630 Course Schedule III | every course, in deadline order | remove the longest duration |
| LC 1642 Furthest Building | a ladder for every climb | convert the smallest laddered climb to bricks once ladders run out |
| LC 871 Refueling Stops | drive past every station, banking its fuel in a heap | when fuel runs out, retroactively refuel from the largest banked station |

LC 871 is worth singling out because it makes the retroactive aspect explicit. You proceed as though you could travel back and refuel at a station you already passed. That is legitimate here because only the total amount of fuel matters and not when it was acquired, so the fiction never produces an invalid schedule. Once that is clear the problem becomes about eight lines.

---

## Part 3 · Stack greedy for lexicographic answers

Whenever the objective is the smallest or largest string or sequence, and you are permitted to delete items, the answer is a monotonic stack together with a deletion budget.

**LC 402 Remove K Digits** asks for the smallest number obtainable by removing `k` digits:

```cpp
string st;
for (char c : num) {
    while (!st.empty() && k > 0 && st.back() > c) { st.pop_back(); k--; }
    st.push_back(c);
}
st.resize(st.size() - k);          // budget left over, so remove from the end
```

The justification is different from an exchange argument and worth naming separately. Lexicographic comparison is decided by the earliest position where two candidates differ, which means an improvement at an early position outweighs any possible improvement later, regardless of magnitude. Therefore, whenever a digit arrives that is smaller than the one on top of the stack, removing the larger digit is unconditionally beneficial provided the budget allows it.

**LC 316 Remove Duplicate Letters** adds a constraint, in that every distinct letter must appear exactly once in the result. The pop condition therefore gains a guard, since a letter may only be removed if it appears again later in the string:

```cpp
while (!st.empty() && st.back() > c && lastIndex[st.back()] > i) {
    inStack[st.back()] = false;
    st.pop_back();
}
```

along with a set recording which letters are already placed, so that duplicates are skipped rather than added.

**LC 2030** adds a third constraint, requiring at least a given number of copies of one particular letter and exactly `k` characters in total, which adds two more clauses to the same condition. Solving 402, then 316, then 2030 in that order means each problem is a small modification of the previous one, whereas attempting them in another order makes the last one considerably harder than it needs to be.

This overlaps with chapter [[07 Monotonic Stacks and Deques]]. The difference is that there the stack is used to compute ranges, and here it is used to construct the answer directly.

---

## Part 4 · Two-pass local repair

**LC 135 Candy** gives each child a rating and requires that any child with a higher rating than a neighbour receives more sweets than that neighbour, while minimising the total.

The constraint points in both directions along the line, so no single sweep can satisfy it. The approach is to satisfy the left-hand constraint in a left-to-right pass, satisfy the right-hand constraint in a right-to-left pass, and take the larger of the two values at each position.

```cpp
vector<int> a(n, 1);
for (int i = 1; i < n; i++) if (r[i] > r[i-1]) a[i] = a[i-1] + 1;
for (int i = n-2; i >= 0; i--) if (r[i] > r[i+1]) a[i] = max(a[i], a[i+1] + 1);
```

The reason the maximum is correct, rather than the sum, is that each pass produces a lower bound at every position: the first pass computes the smallest value that satisfies the left constraint, and the second computes the smallest that satisfies the right. Taking the larger of two lower bounds gives the tightest lower bound available. It then needs one further observation, which is that this combined value satisfies both constraints simultaneously, and it does because increasing a value never breaks a constraint requiring it to exceed a neighbour.

The general pattern is that when constraints propagate in two directions along a sequence, running one pass per direction and combining is usually the answer. The same structure appears in prefix-and-suffix dynamic programming in chapter [[06 Prefix Sums and Difference Arrays]], and in rerooting on trees in chapter [[13 Trees]], where the line is replaced by a tree.

---

## Part 5 · Trying to break your own greedy

Because the samples cannot be relied on, it is worth spending a minute actively looking for a counterexample before submitting any greedy you invented rather than derived.

The quick version is to try the degenerate inputs deliberately: all values equal, one value much larger than the rest, a case where the greedy choice consumes something a later and better choice needed, and `n = 2` with two symmetric options.

The thorough version is to write the brute force. Twenty lines using `next_permutation` over inputs with `n <= 8`, together with a random generator, will refute a wrong greedy within seconds.

```cpp
for (int iter = 0; iter < 100000; iter++) {
    auto a = randomSmallInput();
    if (greedy(a) != brute(a)) { print(a); break; }
}
```

This is worth building as a habit during practice rather than saving for the assessment itself. Testing your own greedy against an exhaustive solution teaches you which kinds of ordering fail and why, which is information that reading an editorial does not provide, because the editorial only shows you the strategy that works.

---

## The ideas worth carrying forward

1. **Derive the comparator from two elements rather than from the whole input.** Write the cost of `AB` and `BA`, require one to be no worse, and separate the variables. The isolated function of `A` is the sort key.

2. **A failure to separate means no comparator exists.** That is a positive signal that the decision depends on global state, which usually indicates dynamic programming.

3. **Cross-multiply inside comparators.** Division introduces both precision problems and invalid orderings, and an invalid ordering causes `std::sort` to read out of bounds rather than to produce a wrong answer.

4. **Regret greedy means committing to everything and undoing the worst decision when the commitments become infeasible.** The heap holds the commitments, and the proof obligation is always that the chosen removal leaves the state at least as good as any alternative removal would.

5. **Retroactive resources are legitimate when only the total matters.** Refuelling Stops works because fuel acquired at any earlier point is interchangeable.

6. **Lexicographic objectives lead to a monotonic stack with a deletion budget**, because an improvement at an earlier position outweighs any improvement later regardless of size.

7. **Constraints pointing in two directions call for two passes combined by taking the maximum**, since each pass produces a lower bound and the larger of two lower bounds is the tightest one available.

8. **Sorting in descending order of some quantity is common when that quantity represents a cost paid later.** Both LC 2136 and EDPC X reduce to two-element algebra of this kind.

9. **Passing the samples provides almost no evidence for a greedy.** They are usually satisfied by the plausible wrong strategies as well, which is why the argument has to come first.

---

## Where people lose these problems

**Writing code before writing the inequality.** This is the underlying cause of most failures in this block. If your notes contain no algebra, nothing has been established.

**Using an invalid comparator.** Floating-point or integer-division keys produce crashes or inconsistent results on large inputs while behaving correctly on small ones. Cross-multiplying in 64-bit arithmetic resolves it.

**Applying greedy where the quantity being maximised and the quantity being limited are both interesting.** That combination is the boundary between Course Schedule III and knapsack, and it usually indicates dynamic programming.

**Leaving the unused budget unspent.** In LC 402 the loop can finish with `k` still positive, which happens when the input is non-decreasing, so the remaining removals must be taken from the end.

**Producing leading zeros or an empty result.** In LC 402 the input `"10"` with `k = 2` produces an empty string, which should be printed as `"0"`.

**Overflow in accumulated totals.** The elapsed time in LC 630 and the fuel total in LC 871 both exceed 32 bits at the stated limits.

**In LC 321 Create Maximum Number, comparing only the current characters when merging.** The problem consists of three parts: selecting the best subsequence of a given length from one array using the stack greedy, merging two sequences to produce the largest result, and trying every way of splitting `k` between the two arrays. The merge step must break ties by comparing the entire remaining suffixes rather than the current characters, so the comparison is `a.substr(i) > b.substr(j)`.

---

## Working through the problem list

### Block 1 · Comparators you can derive in three minutes

- **CSES Stick Lengths** — *make all sticks the same length at minimum total cost.* The answer is the median. Establish it by considering what happens to the total when the target moves slightly, and counting how many sticks get closer against how many get further.
- **CSES Tasks and Deadlines** — *order tasks to maximise total reward, where reward decreases with completion time.* A direct application of the two-element algebra from Part 1.
- **CSES Towers** — *stack cubes into towers where each cube must be smaller than the one below.* A multiset with `upper_bound`. The choice of which existing tower to extend needs a one-line argument.
- **CSES Reading Books** — *two people read all books, never the same book simultaneously; find the minimum time.* The answer is the larger of twice the longest book and the total, which is worth deriving rather than looking up.
- **LC 1005 Maximize Sum After K Negations** — *flip the sign of exactly k elements to maximise the sum.* Sort, flip the negatives, then handle any leftover flips on the smallest absolute value.
- **LC 1953 Maximum Number of Weeks** — *work on projects without repeating two weeks running.* The same structure as Reading Books, being the larger of a bottleneck and a total.
- **CF 763A Timofey and a tree** — *is there a vertex such that removing it leaves all subtrees single-coloured?* A checking problem with a clean observation behind it.

### Block 2 · Regret and heap greedies

- **LC 502 IPO** — *complete at most k projects, each requiring capital and yielding profit, to maximise final capital.* Two heaps, one ordered by capital feeding one ordered by profit.
- **LC 630 Course Schedule III** — *take as many courses as possible, each with a duration and a deadline.* The template from Part 2.
- **LC 1642 Furthest Building You Can Reach** — *cross buildings using a limited number of ladders and bricks.* Regret applied to ladders. It also yields to binary search on the answer, from chapter [[04 Binary Search on the Answer]], and comparing the two is instructive.
- **LC 871 Minimum Number of Refueling Stops** — *reach a target with the fewest stops, given fuel stations along the way.* Retroactive refuelling.
- **CSES Factory Machines** — *given machines of differing speeds, produce k items in minimum time.* This is binary search on the answer rather than greedy, and it is included here deliberately because it feels like a greedy and is not.
- **LC 2136 Earliest Possible Day of Full Bloom** — *plant and grow seeds, one planting at a time, minimising the day all have bloomed.* Sort by growing time descending, and write the two-element swap that establishes it.
- **LC 2589 Minimum Time to Complete All Tasks** — *run tasks within their windows, sharing computer time where possible.* Sort by deadline, then place the required time as late as possible. The instinct to schedule as late as possible is worth keeping.
- **CF 985C Liebig's Barrels** — *build barrels from planks, maximising the total of their minimum planks.* Sort, then reason about which planks can serve as group minima.
- **CF 1401D Maximum Distributed Tree** — *distribute prime factors onto tree edges to maximise a weighted sum.* Subtree sizes give each edge a multiplier, and the largest factors belong on the largest multipliers.

### Block 3 · Stack greedies, strictly in order

- **LC 402 Remove K Digits** — *remove k digits to leave the smallest possible number.*
- **LC 316 Remove Duplicate Letters** — *produce the lexicographically smallest string containing each letter exactly once.*
- **LC 2030 Smallest K-Length Subsequence With Occurrences of a Letter** — *the same, with a length requirement and a minimum count of one letter.*
- **LC 321 Create Maximum Number** — *pick k digits from two arrays, preserving order within each, to form the largest number.* The most involved of the four.

### Block 4 · Structural and counting greedies

- **LC 135 Candy** — *the sweets distribution problem from Part 4.*
- **LC 330 Patching Array** — *add the fewest numbers so that every value up to n is representable as a subset sum.* The invariant is that you can currently form every value below some reach, and when the next available number exceeds that reach, adding the reach itself doubles the coverage. The reasoning generalises to other representability questions.
- **LC 767 Reorganize String** — *rearrange a string so no two adjacent characters match.* Place the most frequent character first. The feasibility condition is worth deriving by placing that character in alternating positions and counting the slots, rather than memorising.
- **LC 2311 Longest Binary Subsequence ≤ K** — *find the longest subsequence whose binary value is at most k.* Every zero can be taken without cost, after which ones are taken greedily from the right.
- **CF 1042C Array Product** — *use operations to maximise the product of an array.* Case analysis on the number of negatives and the presence of zeros. Tedious, and a realistic representation of what assessment greedies often look like.
- **LC 2856 Minimum Array Length After Pair Removals** — *repeatedly remove pairs of unequal elements; find the shortest possible remainder.* The answer depends only on the largest frequency compared with the rest, which is the third problem in this block with that shape.

---

**A reasonable target here is around 70% first-time accuracy**, though the more informative measure is different.

The sheet schedules this block in the third pass because the 2026 hard problems do not concentrate here. Its value is that it builds the habit of establishing a claim before coding, which pays off across the rest of the sheet. The metric worth tracking is therefore what fraction of the greedies you solved have a written exchange argument in your notes, and that number is more useful than the accuracy figure.

---

## Check yourself

1. Given two jobs described by weight and processing time, derive the sort key for minimising weighted completion time, showing the separation step.
2. What does it mean when the two-element algebra will not separate, and what should you do next?
3. Why must comparators cross-multiply rather than divide?
4. State the two claims that make LC 630 correct, and justify the second one.
5. Why is Refuelling Stops allowed to refuel at a station that has already been passed?
6. Why does LC 402 remove a larger digit unconditionally, while LC 316 needs a guard? What is the guard and what does it protect against?
7. In LC 135, why is the elementwise maximum the correct combination rather than the sum?
8. Describe the invariant in LC 330 in one sentence.
9. You have invented a greedy and it passes both samples. What do you do before submitting?
