---
tags: [dsa, guide, monotonic-stack, contribution-technique]
chapter: 7
sheet-section: G
---

# Chapter 7 · Monotonic Stacks & Deques

> **Read this before you start the problems.** Each idea is introduced with a small example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **G** in [[1. Ultime DSA 2026 calibration]] · See also your note [[Stack]]

---

## What makes these problems hard

A monotonic stack answers exactly one question: for each element of an array, where is the nearest element on the left, and on the right, that is smaller than it. That is a narrow capability, and on its own it would be a minor technique.

What makes this chapter worth a section of the sheet is that a surprising number of harder problems reduce to that question once they are reframed. The reframing is the difficult part, because the problems rarely mention anything about nearest smaller elements. They ask about rectangles, or about water, or about the sum of some quantity over every contiguous piece of the array, and the connection to the stack only becomes visible after you have changed how you are counting.

The reframing that does most of the work is described in Part 2. Instead of considering each piece of the array and asking what its minimum is, you consider each element and ask how many pieces it is the minimum of. That question is answered directly by the two nearest-smaller positions, which is what brings the stack into a problem that never mentioned one.

---

## What these problems look like

There are two families.

**Questions about nearest larger or smaller elements.** Occasionally these are stated directly, as in "how many people can each person see". More often they are disguised as:

- **Rectangles under a profile**, such as the largest rectangle fitting under a histogram, or the water trapped between bars. A bar of height `h` supports a rectangle extending exactly as far as the nearest shorter bar on each side.
- **Spans and visibility**, such as how many days until a warmer temperature, or how far a falling domino pushes.
- **Statements that some element is the minimum or maximum of a piece**, which is the most important disguise and leads to the second family.

**Sums over every contiguous piece.** The statement asks for a total over all pieces of some quantity depending on the piece's minimum or maximum, such as the sum of all minimums, or the sum of the differences between maximum and minimum. The direct approach is quadratic and the intended one is linear.

When a statement mentions both "over all subarrays" and "minimum" or "maximum", the reframing in Part 2 is almost certainly what is wanted. In current hard problems this second family has largely replaced the first, and the first now appears mainly as a subroutine.

---

## Part 1 · The template

Here is the loop that finds, for every index, the nearest strictly smaller element to its left:

```cpp
vector<int> left(n);
stack<int> st;                                    // holds indices; values increase bottom to top
for (int i = 0; i < n; i++) {
    while (!st.empty() && a[st.top()] >= a[i]) st.pop();
    left[i] = st.empty() ? -1 : st.top();
    st.push(i);
}
```

Read the loop as a sentence: discard everything on the stack that is not smaller than the current element, and whatever survives on top is the nearest smaller element to the left.

There are four variants, differing only in the direction of the scan and the comparison used:

| What you want | Scan direction | Discard while |
|---|---|---|
| nearest smaller on the left | left to right | `a[top] >= a[i]` |
| nearest smaller on the right | right to left | `a[top] >= a[i]` |
| nearest larger on the left | left to right | `a[top] <= a[i]` |
| nearest larger on the right | right to left | `a[top] <= a[i]` |

The scan is linear overall despite the inner loop, because each index is pushed exactly once and popped at most once, so the total number of operations across the whole scan is bounded by twice the array length. This is the same accounting that makes the deque linear in chapter [[05 Sliding Window and Two Pointers]] and the disjoint-interval map efficient in chapter [[02 Intervals and Sweep Line]].

There is an alternative style in which the work happens at the moment of popping, since when element `i` causes element `j` to be discarded, you have just learned that `i` is the nearest smaller element to the right of `j`. Both styles are correct. Computing the two boundary arrays explicitly is easier to reason about and easier to debug, so it is the better one to learn first, with the pop-time style as a shorter alternative once the idea is familiar.

---

## Part 2 · Counting each element's contribution

This is the central idea of the chapter.

Suppose you want the sum, over every contiguous piece of the array, of that piece's minimum value. Considering each piece in turn is quadratic. The alternative is to swap the order of the summation:

$$\sum_{\text{pieces } S} \min(S) \;=\; \sum_{i} a_i \cdot \#\{S : a_i \text{ is the minimum of } S\}$$

That is, rather than asking each piece for its minimum, ask each element how many pieces it is the minimum of.

The count is straightforward once the two boundary arrays exist. A piece has `a[i]` as its minimum exactly when it contains position `i` and does not extend past the nearest smaller element on either side. The left endpoint may therefore be anywhere strictly after `left[i]` and at or before `i`, giving `i - left[i]` choices, and the right endpoint may be anywhere at or after `i` and strictly before `right[i]`, giving `right[i] - i` choices. Multiplying the two gives the count.

```cpp
long long ans = 0;
for (int i = 0; i < n; i++)
    ans += (long long)a[i] * (i - left[i]) * (right[i] - i);
```

That is the whole of LC 907 once the boundary arrays are available, and the same machinery covers several problems:

| Problem | How the contribution is used |
|---|---|
| LC 907 Sum of Subarray Minimums | sum of value times count |
| LC 2104 Sum of Subarray Ranges | run the whole computation twice, once for maximums and once for minimums, and subtract |
| LC 1856 Maximum Subarray Min-Product | for each element, its value times the sum over its maximal range, combined with prefix sums |
| LC 2262 Total Appeal of A String | the same reordering without a stack, counting first occurrences instead |

**LC 2104 is worth singling out.** Summing the difference between maximum and minimum over every piece appears to require handling both extremes at once, but sums distribute, so the total of the differences equals the total of the maximums minus the total of the minimums. The identical machinery runs twice with the comparisons reversed. The habit of splitting a sum of differences into a difference of sums is worth keeping, as it also unlocks several problems in chapter [[19 Combinatorics and Number Theory]].

**LC 2262 shows that the reordering is more general than minimums and maximums.** The quantity being summed is the number of distinct characters in each substring. Rather than computing that per substring, ask for each position how many substrings the character there contributes a *new* distinct value to, which is those starting after the previous occurrence of that character and ending at or after the current position. That gives a product of two counts, with no stack involved at all. The stack is a tool for a particular kind of counting, and the reordering is the idea underneath it.

---

## Part 3 · Ties, and why the comparison matters

The choice between `>=` and `>` in the discard condition makes no difference when all values are distinct. When there are ties, it determines which of several equal elements is treated as the owner of a range, and in contribution problems getting this wrong causes double counting.

The concrete failure is easy to see. Take the array `[2, 2, 2]` and the sum of minimums over all pieces. There are six pieces, each with a minimum of 2, so the answer is 12. If instead each of the three positions independently claims the whole array as a piece it is the minimum of, the total comes out as 18.

The fix is to break ties consistently in one direction by making one side's comparison strict and the other side's non-strict:

```
left[i]  = nearest index j < i with a[j] <  a[i]     (strict)
right[i] = nearest index j > i with a[j] <= a[i]     (non-strict)
```

Within a run of equal minimums this makes the leftmost one the owner of every piece spanning the run, because each of the others is blocked on its left by an equal value. Every piece therefore has exactly one owner.

Which side is made strict does not matter, provided exactly one of them is. Choosing a convention and keeping to it, then testing on `[2, 2, 2]` before submitting, resolves this permanently.

---

## Part 4 · Rectangles

**LC 84 Largest Rectangle in Histogram** asks for the largest rectangle fitting under a bar chart. The rectangle whose height is `h[i]` extends left as far as the nearest strictly smaller bar and right likewise, so its area is `h[i]` times `right[i] - left[i] - 1`. Taking the maximum over all `i` gives the answer, which makes this the contribution technique with a maximum in place of a sum.

**LC 85 Maximal Rectangle** asks for the largest rectangle of ones inside a binary grid. For each row, build a histogram where each column's height is the number of consecutive ones ending at that row, then run the previous problem on each row. The total cost is proportional to the grid size.

The transferable move here is to fix one dimension and reduce the two-dimensional problem to a one-dimensional profile. The row-pair reduction in chapter [[06 Prefix Sums and Difference Arrays]] does the same thing for LC 363 and LC 1074, and when a two-dimensional problem resists, asking what fixing one dimension buys you is a reasonable first response.

**LC 42 Trapping Rain Water** has three distinct solutions, and all three are worth knowing:

The first computes arrays of the running maximum from the left and from the right, so the water above position `i` is the smaller of the two maxima minus the height there. This is linear in both time and space, and is the easiest to get right.

The second uses two pointers moving inwards, always advancing the side with the smaller running maximum, which works because that side's answer is determined by its own maximum regardless of what lies beyond. This uses constant space, and reconstructing the reason it is safe to commit is a worthwhile exercise.

The third uses a monotonic stack, computing the water in horizontal layers as elements are popped. It is the least intuitive of the three.

---

## Part 5 · Using a stack to accelerate a recurrence

This use is less well known and worth having in mind. When a dynamic programming transition ranges over earlier positions and involves the maximum or minimum of a range, a monotonic stack often collapses it.

**LC 1130 Minimum Cost Tree From Leaf Values** asks you to build a binary tree whose leaves are the array in order, where each internal node's value is the product of the largest leaf in each of its subtrees, minimising the sum of the internal nodes. An interval dynamic program solves it in cubic time, but there is a linear solution using a decreasing stack:

```cpp
long long ans = 0;
stack<int> st;
for (int x : a) {
    while (!st.empty() && st.top() <= x) {
        int mid = st.top(); st.pop();
        ans += (long long)mid * min(st.empty() ? INT_MAX : st.top(), x);
    }
    st.push(x);
}
while (st.size() > 1) { int t = st.top(); st.pop(); ans += (long long)t * st.top(); }
```

The reasoning is that every internal node's value is the product of the two largest leaves in its two subtrees, so to minimise the total you want each small leaf consumed as cheaply as possible, which means pairing it with the smaller of its two neighbours. The stack is what locates those neighbours. It is a greedy with an exchange argument, in the sense of chapter [[03 Greedy and Exchange Arguments]], whose implementation happens to be a stack.

**LC 2818 Apply Operations to Maximize Score** is the fullest example of this chapter's techniques combined. Each element is assigned a prime score, a monotonic stack determines the range over which each element has the largest prime score, that range gives the number of pieces for which each element may be chosen, the elements are then sorted by value in descending order and a budget of operations is spent greedily, and modular exponentiation produces the final answer. Three techniques appear, none individually difficult, which is a fair representation of what a current hard problem looks like.

---

## Part 6 · Stacks and deques

The two structures follow the same discipline. The difference is that a deque also discards from the front, because it is constrained to a window, while a stack has no such constraint. Deques are covered in chapter [[05 Sliding Window and Two Pointers]], and the useful way to hold the relationship is that a monotonic stack is a monotonic deque with no expiry rule.

---

## The ideas worth carrying forward

1. **The stack answers one question**, which is where the nearest smaller or larger element sits on each side. Everything else in this chapter is a way of reducing a problem to that question.

2. **Reorder the summation.** Rather than asking each piece for its minimum, ask each element how many pieces it is the minimum of. This is the reframing that brings the stack into problems that never mention it.

3. **The count is `(i - left[i])` multiplied by `(right[i] - i)`**, which is the number of choices for the left endpoint times the number for the right.

4. **Make exactly one of the two comparisons strict.** Otherwise elements with equal values both claim the same pieces, and the total is too large. Testing on `[2, 2, 2]` detects this immediately.

5. **A sum of differences is a difference of sums**, so the total of maximum minus minimum over all pieces needs the same machinery run twice.

6. **The reordering applies beyond minimums and maximums.** LC 2262 counts how many pieces each character contributes a new distinct value to, using the same idea with no stack.

7. **Fixing one dimension reduces a two-dimensional problem to a one-dimensional profile**, which is how LC 85 reduces to LC 84.

8. **Each index being pushed once and popped once is why the scan is linear**, which is worth being able to state when the nested loop looks suspicious.

9. **Adding sentinel values removes the empty-stack checks.** A very small value placed conceptually at index `-1` and another after the end mean the stack is never empty, which is particularly convenient in histogram problems.

10. **A recurrence involving the extremum of a range often collapses with a stack**, as in LC 1130 and LC 2818.

---

## Where people lose these problems

**Storing values on the stack rather than indices.** The index is needed to compute widths, so values alone are insufficient.

**Making both comparisons strict, or neither.** Equal values then double-count, and the `[2, 2, 2]` check finds it in seconds.

**Overflow.** In LC 907 the answer is taken modulo a large prime, and the reduction has to happen before the final multiplication. In LC 1856 the product genuinely requires 128-bit arithmetic or careful modular handling.

**Leaving elements unprocessed on the stack.** In the pop-time style, anything still on the stack when the scan ends was never popped and needs its right boundary set to the array length. Sentinels handle this structurally.

**In LC 456, missing what the stack represents.** The problem asks whether there exist indices `i < j < k` with `a[k] < a[i] < a[j]`. Scanning from the right with a decreasing stack, the useful quantity is the largest value that has been popped so far, since a value is only popped when something larger appears to its right. If the current element is smaller than that quantity, the pattern exists. The aim is to make the popped value as large as possible, and the popped elements are exactly the ones that have a larger element to their right.

**In LC 962, missing the two-phase structure.** The problem asks for the largest gap between indices `i < j` with `a[i] <= a[j]`. Build a decreasing stack of candidate left endpoints in a forward pass, since only indices smaller than everything before them can usefully serve as a left endpoint, then scan from the right discarding while the current value is at least the top. The pattern of building candidates forwards and consuming them backwards is worth remembering separately.

**In LC 2419, over-engineering.** The largest possible AND of any piece is the largest single element, because combining values with AND can only remove bits. The problem therefore reduces to finding the longest run of the maximum element, which is a short answer behind an intimidating statement.

---

## Working through the problem list

### Block 1 · Making the template automatic

- **CSES Nearest Smaller Values** — *for each element, report the nearest smaller element to its left.* The template exactly. If this takes more than five minutes it is worth repeating until it does not.
- **LC 1944 Number of Visible People in a Queue** — *for each person, count how many people to their right they can see over the heads of shorter ones.* The answer for each position is the number of discards it causes, plus one if the stack is still non-empty. A clean demonstration of the pop-time style.
- **LC 42 Trapping Rain Water** — *compute the water trapped between bars of varying height.* Worth doing all three ways described in Part 4.
- **LC 84 Largest Rectangle in Histogram** — *the largest rectangle fitting under a bar chart.* The foundational problem of the chapter, and a good place to adopt sentinels.
- **LC 85 Maximal Rectangle** — *the largest rectangle of ones in a binary grid.* Row-by-row reduction onto the previous problem.

### Block 2 · Contribution counting

- **LC 907 Sum of Subarray Minimums** — *sum the minimum of every contiguous piece.* The template contribution problem. Test on `[2, 2, 2]` before submitting.
- **LC 2104 Sum of Subarray Ranges** — *sum, over every piece, the difference between its largest and smallest values.* About ten minutes once 907 is understood.
- **LC 1856 Maximum Subarray Min-Product** — *maximise the minimum of a piece multiplied by its sum.* Contribution ranges combined with prefix sums, with overflow needing real attention.
- **LC 2262 Total Appeal of A String** — *sum the number of distinct characters over every substring.* The reordering without a stack, best attempted immediately after 907 so the generalisation is visible.
- **LC 3113 Find the Number of Subarrays Where Boundary Elements Are Maximum** — *count pieces whose first and last elements are both equal to the maximum.* Group equal values and use the stack to bound each group's span.
- **LC 2334 Subarray With Elements Greater Than Varying Threshold** — *find a piece whose every element exceeds a threshold divided by its length.* Each element's maximal range as the minimum gives a candidate, so the contribution ranges are used as candidates rather than as counts.

### Block 3 · Stacks used for other purposes

- **LC 456 132 Pattern** — *detect the index pattern described above.*
- **LC 962 Maximum Width Ramp** — *the largest gap between two indices with the earlier value no larger.*
- **LC 1063 Number of Valid Subarrays** — *count pieces whose first element is the minimum.* This is a direct sum over the nearest-smaller-to-the-right array, which makes it a useful confidence check once the machinery exists.
- **LC 1130 Minimum Cost Tree From Leaf Values** — *the tree-building problem from Part 5.*
- **LC 316 Remove Duplicate Letters** — *the lexicographic problem from chapter [[03 Greedy and Exchange Arguments]].* The sheet asks you to revisit it here, and the reason is worthwhile: reading it as a monotonic stack rather than as a greedy shows that the two descriptions refer to the same mechanism.
- **LC 2419 Longest Subarray With Maximum Bitwise AND** — *the reduction described above.* Read it, notice the reduction, and move on.

### Block 4 · The harder ones

- **LC 2818 Apply Operations to Maximize Score** — *the combined problem from Part 5.* Best left until last.
- **CF 1288D Minimax Problem** — *choose two rows of a matrix maximising the minimum of their elementwise maximum.* This is binary search combined with bitmasks, from chapters [[04 Binary Search on the Answer]] and [[15 Bitmask DP]], rather than a stack problem, and the column count of at most eight is the signal. It is filed in section G on the sheet, so recognising what it actually is saves time.

---

**A reasonable target here is around 80% of submissions passing first time.**

There is essentially one template, and the ways to go wrong are ties, overflow, and storing values instead of indices. All three can be checked in under a minute before submitting.

---

## Check yourself

1. Write the nearest-smaller-on-the-left template from memory, and give the settings for the other three variants.
2. Why is the scan linear despite the inner loop?
3. State the contribution count and explain both factors in terms of endpoint choices.
4. Why must exactly one of the two comparisons be strict? Demonstrate the failure on `[2, 2, 2]`.
5. How do you obtain the sum of maximum minus minimum over all pieces from machinery that only computes the sum of minimums?
6. LC 2262 uses the reordering without a stack. What is being counted at each position?
7. How does LC 85 reduce to LC 84? Name another problem on this sheet using the same dimension-fixing move.
8. What do sentinels remove from the histogram template?
9. In LC 456, what does the tracked popped value represent, and why are popped elements the right candidates?
