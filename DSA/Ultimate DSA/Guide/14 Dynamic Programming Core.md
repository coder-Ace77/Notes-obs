---
tags: [dsa, guide, dp, dynamic-programming, edpc]
chapter: 14
sheet-section: N
---

# Chapter 14 · Dynamic Programming: The Classic-Hard Core

> **Read this before you start the problems.** Each idea is introduced with a small example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **N** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

Dynamic programming is often presented as a technique, but it is closer to a discipline for organising an answer to one recurring question: what is the smallest amount of information about the past that is sufficient to make every future decision correctly. Once that information, called the state, has been identified correctly, the rest of the work is largely mechanical.

The difficulty in practice comes from two places. The first is that identifying the state is genuinely a creative step, and there is no substitute for practising it across a wide enough variety of problems that the common shapes become recognisable on sight. The second, more mechanical source of difficulty is that even a correct state description leaves several decisions unmade, such as exactly what a stored value means, what the base case should be, and in which order the values should be filled in, and a slip in any of these produces a solution that looks right and returns a wrong answer.

This chapter's structure follows from that. A large part of it is a catalogue of the common state shapes, since recognising a shape is most of the work of designing a state, and the sheet's advice to work through the Educational DP Contest problems in order exists because that set of twenty-six problems was deliberately constructed to cover almost all of these shapes with essentially no repetition.

---

## Why the sheet puts the Educational DP Contest first

Those twenty-six problems are worth treating as a diagnostic rather than as ordinary practice. Working through them in the given order, and briefly noting which paradigm each one represents, produces a personal map of dynamic programming after a single pass: the problems that took noticeably longer than the others identify exactly which shapes need more attention, and everything after that can be scheduled around those specific gaps rather than practised at random.

---

## Part 1 · Five questions to answer before writing any code

For any dynamic programming problem, these five questions are worth answering in writing, not merely in your head, before a single line of code is typed.

**What is the state?** A description such as "the answer after considering the first `i` items, having used `j` of the available budget."

**What does the stored value actually mean?** This needs to be stated with enough precision to remove any ambiguity, in particular whether item `i` is included in the state or not yet decided. Vagueness here is the single most common root cause of a dynamic programming bug.

**What is the transition?** For each state, which earlier states feed into it, and at what cost.

**What is the base case?** And is it zero, one, a very large number, or a very small one? Confusing a base case of zero with a base case of one is a distinctive and recurring error in counting problems, where the base case represents the single "do nothing" way of achieving an empty result.

**What is the final answer?** Sometimes it is a single entry such as `dp[n]`, and sometimes it is the best entry among several, such as the maximum of `dp[n][j]` over all `j`, which is easy to forget to take.

A sixth question is worth adding for harder problems: in what order should the states be filled in? Every state must be computed only after everything it depends on. When that order is not obvious, writing the recursion top-down with memoisation sidesteps the question entirely, since the recursive calls determine the order automatically.

---

## Part 2 · Writing it top-down first

Writing a dynamic program as a memoised recursive function, rather than as a table filled in a loop, is worth doing as a default, for several reasons. The order in which states are computed is handled automatically by the recursion. The code reads as a direct translation of the recurrence written on paper, without a separate step of working out a valid fill order. States that are never actually needed cost nothing, since the recursion never visits them. And when something goes wrong, the call sequence itself can be inspected, which is considerably easier than debugging a nested loop.

```cpp
vector<vector<long long>> memo;   // sized appropriately, filled with a sentinel such as -1
long long solve(int i, int j) {
    if (i == n) return base;
    long long& res = memo[i][j];
    if (res != -1) return res;
    res = /* combine the transitions here */;
    return res;
}
```

Taking a reference to the memo cell before computing its value, as `res` does above, means the result is written exactly once and there is no way to forget to store it.

The bottom-up form, filling a table in a loop, becomes worth converting to once the recursion depth becomes a risk, or once a rolling array is needed to reduce memory use. It is a conversion to make later rather than a starting point.

---

## Part 3 · The common shapes

This is the practical content of the chapter: a catalogue of dynamic programming shapes together with the signal that identifies each one.

**A linear sequence, where each answer depends on a small number of earlier answers.** The signal is a single dimension of state and decisions made moving left to right. AC EDPC A and B, and CSES *Removal Game*, are examples.

**A grid, where each cell depends on the cell above and the cell to the left.** The signal is a two-dimensional board with movement restricted to one direction along each axis. AC EDPC H is the straightforward version, and AC EDPC Y adds obstacles handled through inclusion and exclusion.

**A small mode carried alongside the position.** The signal is that a decision made at one position constrains what is possible at the next. AC EDPC C, and the various stock-trading problems that track whether a share is currently held, both fit this shape, as does the tree-matching pattern from chapter [[13 Trees]].

**Interval dynamic programming, where the answer for a range is built from the answers for two smaller ranges meeting at a split point.** The signal is that the objective concerns merging or combining adjacent things, or that the last operation to be undone could have occurred anywhere within the range. AC EDPC N, LC 312, LC 1039, LC 664, LC 546 and CSES *Rectangle Cutting* are all this shape. The states must be filled in order of increasing interval length, without exception. LC 312 Burst Balloons is worth singling out for its central idea: rather than deciding which balloon to burst first, which entangles the two halves of the interval, deciding which balloon to burst *last* leaves the two halves independent, because the boundaries of the interval are untouched until that final burst. Reversing the order in which a decision is imagined, so that subproblems become independent, is a general move worth remembering beyond this one problem.

**Game dynamic programming, where two players alternate and both play optimally.** The signal is exactly that description. AC EDPC K and L, CSES *Removal Game*, and LC 877 are examples. The useful device is to define the stored value as the current player's advantage, meaning their eventual score minus their opponent's, rather than tracking each player's score separately, since the recurrence then becomes a maximum over moves of the immediate gain minus the value of the resulting position for the opponent, with the alternation of turns handled entirely by that minus sign.

**Probability and expectation.** The signal is language involving "expected value" or "probability that". AC EDPC I and J, and CSES *Dice Probability*, are examples, and the topic is covered in full in chapter [[16 Digit DP and Expectation]].

**Dynamic programming over subsets, using a bitmask as the state.** The signal is a bound of around twenty on the relevant dimension. AC EDPC O and U are examples, and the topic is covered in chapter [[15 Bitmask DP]].

**Digit dynamic programming, counting numbers with a property within a range.** The signal is a request to count numbers up to some bound written with many digits. AC EDPC S is the template, covered in chapter [[16 Digit DP and Expectation]].

**Dynamic programming over a graph with no cycles, including trees.** AC EDPC G, P and V, and the material in chapters [[12 Graph Structure SCC 2SAT Bridges]] and [[13 Trees]], belong here.

**Counting dynamic programming accelerated by prefix sums.** The signal is a transition that sums over a contiguous range of the previous row. AC EDPC M and T are examples, and the fix is to maintain a running prefix sum of the previous row so that each new value is computed in constant time rather than by summing a range directly, turning a quadratic-in-the-range cost into a linear one. This is worth trying as a first response whenever a transition contains a loop over a contiguous range of earlier states, since it is the most frequently applicable speed-up in the whole topic.

**Dynamic programming accelerated by a data structure.** The signal is a transition taking a maximum, minimum or sum over previous states satisfying an inequality. AC EDPC Q, using a Fenwick tree, and AC EDPC W, using a segment tree with deferred updates, are examples, along with LC 2926 from chapter [[08 Segment Trees]].

**Matrix exponentiation.** The signal is a linear recurrence combined with an index so large, often up to a quintillion, that no loop could reach it. AC EDPC R, and CSES *Graph Paths I* and *II*, are examples: express the transition as a matrix and raise it to a high power using repeated squaring.

**Sorting first, then applying dynamic programming.** The signal is that the items carry an ordering constraint the problem does not state explicitly. AC EDPC X is the clearest example: the correct sort order, by the sum of two per-item quantities, is established with an exchange argument exactly as in chapter [[03 Greedy and Exchange Arguments]], and only once the items are sorted does a knapsack-style dynamic program become valid. Greedy reasoning establishes the order, and dynamic programming makes the remaining choices, and that combination recurs often enough to be worth naming on its own.

**Optimising an already-correct but too-slow dynamic program.** The signal is an approach that is quadratic in the input size where the input size can be as large as a hundred thousand. AC EDPC Z is the introductory example, covered fully in chapter [[17 DP Optimization]].

---

## Part 4 · Reading the constraints as a hint about complexity

The stated bound on the input size is a message from whoever designed the problem, and it is worth decoding before designing the state.

| Bound on the relevant size | Complexity this suggests | Likely shape |
|---|---|---|
| around 20 | exponential, roughly `2^n` times a small factor | dynamic programming over subsets |
| around 40 | roughly `2^(n/2)` | meeting in the middle |
| around 100 | cubic or quartic | interval dynamic programming, or all-pairs shortest paths |
| around 500 | cubic | interval dynamic programming, or all-pairs shortest paths |
| around 5,000 | quadratic | an ordinary two-dimensional dynamic program |
| around 100,000 | close to linear, or linear times a logarithm | dynamic programming combined with a data structure, or an optimisation technique |
| enormous, such as a quintillion | logarithmic | matrix exponentiation, or digit dynamic programming |

Performing this check before designing the state, rather than after writing a solution that turns out to be too slow, takes only a few seconds and prevents a wasted implementation.

---

## Part 5 · Finding a genuinely difficult state

The hardest problems in this chapter are hard specifically because the state is not obvious from the shape of the problem. There are two reliable methods for finding it.

**The first method is to write the brute-force recursion first, and look at what it actually needs to pass along.** If the natural recursive solution is written as a function taking the current position, the amount of some resource remaining, the previous choice made, and a running count, then those four things are exactly the candidates for the state. From there, some of them can often be pruned away by checking whether they genuinely affect the answer or have only a small range of values. This method works reliably in every case and is what experienced problem solvers do silently rather than skip.

**The second method is to ask what information has to cross a cut in the problem.** If the problem is split at some position, whatever information the two halves need to share about each other is exactly the state.

LC 1531 String Compression II is a good illustration of both methods together. The natural recursion needs to know the current position, how many characters have already been deleted, what character the current run consists of, and how long that run currently is, because the compressed length of a run changes precisely at run lengths of one, two, ten and a hundred characters, which is not something that could be guessed from the problem's surface description without actually working through the recursion.

---

## Part 6 · Running a computation backwards

Occasionally, a dynamic program cannot be filled in the direction the problem is stated, because a decision made early on depends on information that only becomes available later.

LC 174 Dungeon Game is the clearest example. Moving forward from the top-left corner does not work, because the best path through the dungeon depends on how much health will be needed later on, which has not yet been determined at the point of choosing the path. The fix is to define the stored value as the minimum health required at entry to a cell in order to survive from that cell onward to the end, and to fill this in starting from the bottom-right corner and working backwards.

The general signal is a requirement that some running quantity never drop below zero, where the amount needed depends on what happens afterwards. When that pattern appears, filling the table in the reverse of the problem's natural direction is worth trying.

---

## The ideas worth carrying forward

1. **Write the five questions down before coding.** State, the precise meaning of the stored value, the transition, the base case, and the final answer. This takes only a couple of minutes and prevents most of the common errors.

2. **Write the recursion top-down first.** The fill order is then handled automatically, and converting to a table later is straightforward once a data structure or memory constraint requires it.

3. **Take a reference to the memoisation cell before filling it in**, so that the result cannot be computed and then accidentally left unstored.

4. **Read the constraints before designing the state.** They indicate the intended complexity and therefore the likely shape of the solution.

5. **When one quantity is small and another is large, consider swapping which one indexes the state and which one is the stored value.** AC EDPC E is the clearest example.

6. **In interval dynamic programming, fill the table by increasing interval length, and think about the last operation rather than the first.** Reversing which decision is imagined first is what decouples the two halves of the interval in Burst Balloons.

7. **In game dynamic programming, store the current player's advantage rather than tracking two separate scores.** The alternation between players is then handled by a minus sign in the recurrence.

8. **A transition summing over a contiguous range of earlier states should immediately suggest maintaining a running prefix sum.** This is the single most frequently useful speed-up in the whole topic.

9. **A transition taking a maximum or minimum over earlier states satisfying an inequality should suggest a segment tree or Fenwick tree indexed by that inequality's quantity.**

10. **Sometimes an exchange argument establishes the correct order, and dynamic programming then makes the choices within that order.** Both halves are necessary, and neither alone solves the problem.

11. **When a requirement depends on what happens afterwards, fill the table backwards.** Dungeon Game is the standard example.

12. **To find a difficult state, write the brute-force recursion and look at what it actually needs to pass along.** This method never fails, even when the state cannot be guessed from the problem's description.

13. **A linear recurrence with an enormous index is a signal for matrix exponentiation.**

---

## Where people lose these problems

**Leaving the meaning of the stored value ambiguous.** Whether "the best answer using the first `i` items" includes item `i` or leaves it undecided needs to be settled and then applied consistently everywhere the value is used. This ambiguity is behind a large share of dynamic programming bugs.

**Choosing the wrong base case.** Zero is correct for sums and maximums, while one is correct for counting problems, since there is exactly one way to achieve an empty result by doing nothing. For minimisation, unreachable states need a very large sentinel value rather than zero.

**Reading a state before it has been computed**, which happens when the fill order in a bottom-up table does not respect the dependencies. Writing the recursion top-down avoids this entirely.

**Forgetting to take the best value among several final states**, rather than reading a single fixed entry, when the answer is not simply `dp[n]`.

**Applying the modulus only at the end of a computation** rather than after every addition, which allows intermediate values to overflow before the final reduction has any chance to help.

**Initialising a maximisation table to zero when negative values are possible.** A very large negative sentinel, chosen so that adding to it does not overflow, is needed instead.

**Allocating a table too large to fit in memory.** A table with a hundred thousand rows and a hundred thousand columns has ten billion cells, and encountering that size is itself the signal that either the design is wrong or a rolling array, keeping only the most recently computed row, is required.

**In a rolling-array knapsack, iterating the weight dimension in the wrong direction.** Iterating from high weight down to low permits each item to be used only once, while iterating from low to high permits an item to be reused, and this single detail is the entire difference between the two variants of the knapsack problem.

**In LC 887 Super Egg Drop, keeping the naive state.** A table indexed by the number of eggs and the number of floors, with an inner loop over which floor to test, is too slow. The fix is the same dimension-swap idea as AC EDPC E: index by the number of eggs and the number of moves made, and track the maximum number of floors distinguishable, which inverts which quantity is the index and which is the value being computed.

**In LC 546 Remove Boxes, missing the third state dimension.** The state needs not only the two ends of the interval but also the count of boxes matching the colour at the left end that have already been grouped together with it from outside the interval, and this dimension is only discoverable by writing the brute-force recursion and noticing what it needs to carry forward.

---

## Working through the problem list

### Block N1 · The Educational DP Contest, all twenty-six, in order

Working through these without skipping or reordering is the point, since the sequence was constructed to cover distinct paradigms with almost no repetition.

| Letter | Problem | Paradigm | Note |
|---|---|---|---|
| A | Frog 1 | a linear sequence | the warm-up |
| B | Frog 2 | a linear sequence with several possible jump lengths | |
| C | Vacation | a small mode carried alongside the position | |
| D | Knapsack 1 | the ordinary zero-one knapsack | good practice for the rolling array |
| E | Knapsack 2 | the dimension swap | the value becomes the index |
| F | LCS | a two-dimensional string comparison, plus reconstructing the answer | practise the reconstruction |
| G | Longest Path | dynamic programming over a graph with no cycles | processed in topological order |
| H | Grid 1 | counting paths through a grid | |
| I | Coins | probability | |
| J | Sushi | expectation, with a state that counts identical items rather than tracking each one | a significant realisation once seen |
| K | Stones | game dynamic programming, win or lose | |
| L | Deque | interval-based game dynamic programming | using the advantage formulation |
| M | Candies | counting accelerated by prefix sums | the key technique of this block |
| N | Slimes | interval dynamic programming | the archetype |
| O | Matching | dynamic programming over subsets | computing a permanent |
| P | Independent Set | dynamic programming on a tree | |
| Q | Flowers | dynamic programming accelerated by a Fenwick tree | the first structure-accelerated example |
| R | Walk | matrix exponentiation | |
| S | Digit Sum | digit dynamic programming | the template used throughout chapter 16 |
| T | Permutation | counting with a genuinely tricky state, accelerated by prefix sums | |
| U | Grouping | summing over all subsets of a subset | an exponential-time enumeration |
| V | Subtree | rerooting combined with prefix and suffix products | see chapter [[13 Trees]] |
| W | Intervals | dynamic programming accelerated by a segment tree with deferred updates | the hardest of the twenty-six |
| X | Tower | an exchange argument followed by a knapsack | greedy combined with dynamic programming |
| Y | Grid 2 | inclusion and exclusion over obstacles | see chapter [[19 Combinatorics and Number Theory]] |
| Z | Frog 3 | the convex hull trick | see chapter [[17 DP Optimization]] |

The useful habit while working through these is to do one or two per sitting, answer the five questions in writing for each, and then write a single line afterwards naming the paradigm it turned out to be. By the end this produces a personal twenty-six-line map of dynamic programming, and the handful that took noticeably longer than the rest identify exactly what to schedule more practice around.

### Block N2 · Harder problems with an added twist

**Interval dynamic programming, best done as a group, since each reinforces the others:**

- **LC 312 Burst Balloons** — *maximise coins from bursting balloons, where bursting one changes its neighbours.* Think about the last balloon burst, not the first.
- **LC 1039 Minimum Score Triangulation** — *triangulate a polygon at minimum cost.* The same "think about what happens last" idea applied to a different shape.
- **LC 664 Strange Printer** — *print a string using the fewest strokes of a printer that always prints a contiguous run.* Interval dynamic programming with a special case when the two ends match.
- **LC 546 Remove Boxes** — *the three-dimensional interval problem from Part 5.* The most demanding interval dynamic program in this chapter.
- **CSES Rectangle Cutting** — *cut a rectangle into squares using the fewest cuts.* A two-dimensional interval dynamic program, gentler than the above.
- **CSES Removal Game** — *two players alternately remove from either end of a row of values.* Game-flavoured interval dynamic programming.
- **CF 1132F Clear the String** — *remove matching adjacent-ish characters at minimum cost.* Nearly identical in structure to Strange Printer, and a useful additional repetition.
- **LC 375 Guess Number Higher or Lower II** — *minimise the worst-case cost of guessing a number correctly.* A minimum over choices of a maximum over outcomes, which is the structure worth internalising here.

**String dynamic programming:**

- **LC 115 Distinct Subsequences** — *count how many times one string appears as a subsequence of another.* Watch the base case carefully, since matching an empty target should count as exactly one way.
- **LC 87 Scramble String** — *decide whether one string can be produced from another by recursively swapping substrings.* A good exercise in writing the brute-force recursion and then memoising it directly.
- **LC 1531 String Compression II** — *the state-design problem from Part 5.*
- **LC 2565 Subsequence With the Minimum Score** — *remove characters from one string to leave it as a subsequence of another with minimum disruption.* Not a table-based dynamic program at all; the approach uses arrays tracking the longest matching prefix and suffix, then combines them with a two-pointer scan.
- **CF 10D LCIS** — *the longest common increasing subsequence of two arrays.* Combines the ideas behind longest common subsequence and longest increasing subsequence, with an elegant quadratic solution using a running best value.

**Sequence and structural dynamic programming:**

- **LC 174 Dungeon Game** — *the backwards-filled dynamic program from Part 6.*
- **LC 887 Super Egg Drop** — *the reformulation described above.*
- **LC 1959 Minimum Total Space Wasted With K Resizing Operations** — *partition an array into segments minimising wasted space, using a limited number of segments.* An ordinary partition dynamic program, and a reasonable bridge into chapter [[17 DP Optimization]].
- **LC 3122 Minimum Number of Operations to Satisfy Conditions** — *a grid-colouring problem.* A small state once identified, indexed by column and by a colour choice.
- **CSES Increasing Subsequence** — *the longest increasing subsequence, in close to linear time.* Worth knowing both the patience-sorting formulation and the Fenwick-tree formulation.
- **CSES Projects** — *weighted interval scheduling.* The same dynamic program as in chapter [[02 Intervals and Sweep Line]], presented here in CSES form.
- **CSES Array Description** — *count arrays consistent with a partially specified sequence and a bound on adjacent differences.* A straightforward two-dimensional dynamic program.
- **CSES Counting Towers** — *count ways to build a tower of a given height from blocks of width one and two.* A deceptively fiddly case analysis, worth deriving the transitions on paper rather than guessing at them.
- **CF 4D Mysterious Present** — *find the longest chain of nested envelopes.* Sort first, then apply a two-dimensional variant of longest increasing subsequence.
- **CF 1114D Flood Fill** — *minimise the operations needed to make an array uniformly coloured.* Interval dynamic programming over a run-length-compressed version of the array, where the compression step is the key realisation.
- **LC 1866 Number of Ways to Rearrange Sticks With K Sticks Visible** — *count arrangements where exactly k sticks are visible from one end.* This computes unsigned Stirling numbers of the first kind, and deriving the recurrence by asking where the shortest stick can go is a satisfying exercise in combinatorial reasoning applied to a dynamic program.
- **LC 1235, revisited as dynamic programming with binary search** — *worth revisiting deliberately after having solved it in chapter [[02 Intervals and Sweep Line]], to see the same problem described as a dynamic program rather than as interval scheduling.*
- **LC 322 Coin Change** — *the baseline coin-change problem.* Skip if it is already comfortable, as the sheet suggests.

---

**For block N1, the target is completion rather than accuracy, since its purpose is diagnostic.**

**For block N2, a reasonable target is around 70% of submissions passing first time.** These problems are genuinely difficult specifically because of state design, and when the difficulty is in finding the state rather than in implementing it, the response that helps is writing the brute-force recursion first every time, until identifying the state becomes close to automatic.

---

## Check yourself

1. List the five questions to answer before writing any dynamic programming code.
2. Why write the recursion top-down before converting it to a table?
3. A subset problem has at most forty relevant items. What technique does that suggest, and why?
4. Explain the dimension swap in AC EDPC E. Name another problem on this sheet using the same idea.
5. In Burst Balloons, why does considering the last balloon burst decouple the two halves of the interval?
6. Write the game-dynamic-programming recurrence using the advantage formulation, and explain why it removes the need for a separate turn indicator.
7. A transition sums `dp[i-1][j-k]` for `k` from 0 to `m`. How is this made constant time per state?
8. A transition takes a maximum over all `j` less than `i` with `a[j]` less than `a[i]`. What structure handles this, and what should it be indexed by?
9. What is the state in LC 1531, and what method would have revealed the need for the run-length dimension?
10. When should a dynamic program be filled backwards? Give the standard example and the reason.
11. In a rolling-array knapsack, what does the direction of the loop over the weight dimension determine?
