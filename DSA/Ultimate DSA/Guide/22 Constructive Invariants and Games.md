---
tags: [dsa, guide, constructive, invariants, game-theory, grundy]
chapter: 22
sheet-section: V
---

# Chapter 22 · Constructive, Invariants & Game Theory

> **Read this before you start the problems.** Each idea is introduced with a small example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **V** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

The problems in this chapter tend to produce a specific, recognisable experience: a long period of getting nowhere, followed by a solution that turns out to be very short once it is found. That pattern is not a coincidence. There is generally no algorithm to design in the ordinary sense, since the input sizes involved are frequently far too large for any search, sometimes reaching a billion or more, which is itself the clue that no exploration of the possibilities was ever intended. Instead, the entire content of the problem is a single structural fact: a quantity that never changes under any legal operation, or a short recursive rule describing who wins a game, and the difficulty is entirely in finding that fact rather than in implementing anything once it has been found.

Because there is no algorithm to lean on, the two skills this chapter actually trains are different from the rest of the sheet. The first is a habit of actively hunting for a preserved quantity whenever a problem describes a sequence of operations and asks whether one configuration is reachable from another. The second, and the one that resolves far more of these problems in practice than any theoretical insight, is a willingness to abandon theorising quickly and simply compute the answer for small cases by brute force, then look at the resulting numbers directly for a pattern. This second skill in particular is not something most preparation emphasises, and it is worth deliberately building as a habit here, since it will be the difference between an unsolved problem and a thirty-second solution more often than any specific piece of theory in this chapter.

---

## What these problems look like

Several structural signals are worth watching for: input sizes in the region of a billion attached to a request for a yes-or-no answer, which strongly suggests a formula rather than any kind of search; the phrase "both players play optimally", which signals game theory and points towards either a preserved quantity or a recursive win-loss computation; a question of the form "can state B be reached from state A using these operations", which is the classic trigger for hunting a preserved quantity; a request to "construct any valid" object, which suggests that a direct, explicit pattern exists and searching for one is more productive than reasoning abstractly; and, as a general piece of intuition, an answer that turns out to depend only on parity, or that is the same across a surprisingly wide range of inputs, which is itself a signal rather than a coincidence, since it usually means the intended solution has already been found and there is nothing more to search for.

---

## Part 1 · Hunting for a preserved quantity

**A preserved quantity, usually called an invariant, is something that every legal operation leaves unchanged.** Its value comes from a single fact: if two configurations have different values of this quantity, no sequence of legal operations can ever turn one into the other, and this can be established without searching through any of the intermediate states at all.

The practical procedure for finding one is to list the legal operations, and then, for each one, ask directly what stays the same when it is applied. A short list of things worth checking, roughly in order of how often they turn out to be the answer: the parity of some count; a sum, or a sum taken modulo some small number; an exclusive-or of some quantity across all elements; a count of inversions; an argument based on colouring the elements of the structure in two colours; or a monovariant, meaning a quantity that only ever increases, or only ever decreases, rather than staying perfectly fixed.

Once a candidate invariant has been found, computing its value on the starting configuration and on the target configuration settles the question directly: if the values differ, the target is unreachable, and if they match, the target is very often reachable, at which point the remaining task is usually to exhibit an explicit construction achieving it.

**A few classical examples are worth knowing, since recognising a disguised version of one of them is often the entire solution:**

The fifteen puzzle, where numbered tiles slide within a frame, has an invariant combining the parity of the permutation of the tiles with the parity of the blank tile's row, and roughly half of all configurations are consequently unreachable from any given starting position, regardless of how long you are willing to search.

A chessboard with two opposite corners removed cannot be tiled by dominoes, because every domino covers exactly one black and one white square under the standard chessboard colouring, and removing two corners of the same colour leaves an unequal number of black and white squares remaining, which no arrangement of dominoes could ever equalise.

An operation that flips any two adjacent bits of a binary string preserves the total parity of the string, since flipping two bits changes the count of set bits by an even amount.

An operation that replaces two numbers on a board with their difference preserves the parity of the sum of everything on the board, since replacing `a` and `b` with `a - b` changes the total sum by `2b`, which is always even.

---

## Part 2 · A worked example, showing the proof rather than just the answer

LC 810 Chalkboard XOR Game is a good illustration of how much work a correctly identified invariant can do. Players alternately erase one number from a list, and a player loses if the exclusive-or of every remaining number becomes zero at the start of their own turn.

**The answer is that the first player wins exactly when the exclusive-or of all the numbers is already zero at the very start, or when the total count of numbers is even.**

The first half of this is immediate: if the exclusive-or is already zero before anyone has moved, the first player has already won without doing anything. The second half requires an actual argument. Suppose it is the first player's turn, the exclusive-or of the remaining numbers is not currently zero, and the count of numbers remaining is even. Suppose, for the sake of contradiction, that *every* possible number the first player could erase would immediately bring the exclusive-or down to zero. If removing any single element brings the total exclusive-or to zero, that means every element individually equals the exclusive-or of all the others, which forces every element in the list to be equal to the overall exclusive-or value, and hence equal to each other. But an even number of equal values, exclusive-ored together, is zero, contradicting the assumption that the current exclusive-or is not zero. So the assumption that every move loses immediately must be false, meaning at least one safe move always exists whenever the count is even and the exclusive-or is not yet zero, and the first player can always find it.

**The solution itself is two lines of code once this argument is accepted, and the argument is where essentially the entire difficulty of the problem lives.** This ratio, of a very short solution supported by a real proof, is the defining shape of this chapter, and LC 877 Stone Game has a similar character: with an even number of piles and taking from either end, the first player can commit in advance to taking either every odd-indexed pile or every even-indexed pile, whichever has the larger total, and this commitment alone is enough to guarantee a win regardless of how the second player responds. The dynamic programming solution to the same problem is also short and worth writing, but the invariant argument is the one that actually explains why the answer is always yes under the stated conditions, and doing both is more valuable than doing either alone.

---

## Part 3 · Games with identical options for both players

**A large family of two-player games has a clean theory when both players have exactly the same set of moves available to them at every position, and the player unable to move loses.** These are called impartial games under normal play, and the relevant result, the Sprague–Grundy theorem, assigns every position a single number, called its Grundy number, computed as

$$G(\text{position}) = \text{mex}\{ G(\text{next position}) \}$$

where "mex" means the smallest non-negative integer absent from the given set of values. **A position with a Grundy number of zero is a loss for whoever is about to move there**, and, most usefully, **the Grundy number of several independent games played side by side, where a move consists of playing in exactly one of them, is the exclusive-or of their individual Grundy numbers.** This last fact is what makes the theorem practically valuable: it lets a complicated combined game be broken into simple independent pieces, each analysed separately, with the results combined by a single exclusive-or at the end.

```cpp
int grundy(State s) {
    if (memo.count(s)) return memo[s];
    set<int> seen;
    for (State t : moves(s)) seen.insert(grundy(t));
    int g = 0; while (seen.count(g)) g++;      // mex
    return memo[s] = g;
}
```

**The genuinely useful working method here, worth treating as seriously as any piece of theory, is this: compute the Grundy numbers for small cases by brute force, print out the first twenty or thirty values, and look directly at the resulting sequence for a pattern.** Patterns that turn up often include straightforward periodicity, a dependence on the input taken modulo some small number, powers of two, or, occasionally, everything being the same value except for a small handful of exceptional small cases. This computational approach to spotting the pattern, followed by trusting it, is frequently faster and more reliable under time pressure than attempting to derive the pattern by pure reasoning from the outset.

**The standard reference game in this family is ordinary Nim**, where several piles of stones exist and a move removes any positive number of stones from a single pile; a single pile of size `n` has Grundy number `n`, and a position with several piles is a loss for the player to move exactly when the exclusive-or of all the pile sizes is zero. **A closely related variant, misère Nim, where the player unable to move actually wins rather than loses, agrees with ordinary Nim except in the specific case where every remaining pile has exactly one stone, in which case the outcome flips.**

CSES *Stick Game* asks about removing a number of sticks from a fixed allowed set, and the Grundy numbers here are computed by dynamic programming and then searched for periodicity by printing them out. CSES *Stair Game* has a genuinely elegant reduction: coins sit on numbered stairs and a move slides one coin down by any positive number of steps, and it turns out that **only the coins sitting on odd-numbered stairs actually matter**, because any move involving an even-numbered stair can always be mirrored by the opposing player in a way that cancels it out, which reduces the entire game to ordinary Nim played on just the odd stairs. This kind of "half the state doesn't actually matter, because it can always be mirrored away" reduction is worth watching for specifically, since it recurs across several games in this family.

---

## Part 4 · Games without identical options, and games with cycles

**When the two players have genuinely different sets of available moves, the Sprague–Grundy theorem no longer applies, and the fallback is an ordinary game-flavoured dynamic program**, in the style described in chapter [[14 Dynamic Programming Core]].

LC 913 Cat and Mouse is the hardest problem in this chapter, and understanding precisely why is worth the time, since the difficulty is instructive on its own. The graph of possible game positions contains cycles, since positions can genuinely recur, and an ordinary memoised recursive dynamic program either loops forever when it encounters such a cycle or, if implemented carelessly, silently returns an incorrect "draw" value for positions that are actually decided.

**The correct technique, called retrograde analysis, works backwards from the known outcomes rather than forwards from the start.** Every terminal position is marked first, meaning positions where the mouse has reached its hole, which the mouse wins, or where the cat has caught the mouse, which the cat wins. From there, outcomes propagate backwards: a position is a win for whichever player is about to move if *any* move available to them leads to a position that is a loss for the opponent, and a position is a loss for the player about to move only once *every* move available to them has been confirmed to lead to a win for the opponent, which is tracked with a per-position counter of moves not yet confirmed as leading to a loss, decremented as neighbouring positions are resolved. Whatever remains undetermined once this propagation has run its course is genuinely a draw.

**This backward propagation by counting down undetermined outgoing moves is the general technique for any game whose position graph contains cycles**, and it is worth remembering independently of this specific problem, since it is the one piece of machinery in this chapter that is a genuine algorithm rather than a structural insight to be found by inspection.

---

## Part 5 · Constructing an explicit object

**Some problems ask directly for a construction, and the reasonable approach is almost never to search for one, since the input sizes involved rule that out.** Instead, work out the small cases by hand or with a short brute-force program, look specifically at *how* each successive solution is built from the previous one rather than only at what the final solution looks like, and then attempt to generalise that building process into a rule.

**A small number of cases at the very start of the sequence very often break whatever general pattern eventually emerges**, and these need to be identified and handled separately rather than assumed to fit the general rule.

CSES *Gray Code*, covered already in chapter [[01 Implementation and Simulation]], is constructed by the one-line formula `i ^ (i >> 1)`, and this is best derived by writing out the sequences for small values of `n` by hand and noticing the reflection pattern connecting consecutive lengths, rather than looked up directly. LC 899 Orderly Queue, also covered in chapter [[01 Implementation and Simulation]], combines a constructive and an invariant argument: when the allowed prefix length `k` is at least two, every permutation of the string becomes reachable, so the answer is simply the sorted string, while when `k` equals exactly one, only rotations of the string are reachable, so every one of the `n` rotations needs to be tried directly.

---

## Part 6 · Naming the brute-force-and-inspect method properly

This deserves to be treated as a real, named technique in its own right, since it is how a large share of the problems in this chapter are actually solved in practice, whatever the eventual write-up might suggest.

```cpp
for (int n = 1; n <= 30; n++) cout << n << " -> " << solveBrute(n) << "\n";
```

Looking directly at the printed output, the patterns worth having in mind as candidates include straightforward periodicity, where the answer depends only on the input taken modulo some fixed number; a dependence purely on whether the input is a multiple of some specific value; powers of two appearing directly; sequences related to the Fibonacci numbers, as happens in Wythoff's game; and, quite often, a pattern that holds uniformly except for a small handful of small exceptional cases at the very start.

**Spending twenty minutes brute-forcing small cases and staring at the resulting numbers is frequently more productive than spending two hours reasoning abstractly about the problem**, and building the habit of reaching for this quickly, rather than as a last resort after theorising has failed, is worth doing deliberately during practice so that it is available under time pressure as well.

---

## The ideas worth carrying forward

1. **A preserved quantity settles reachability without any search at all.** If two configurations differ in an invariant's value, no sequence of legal operations connects them, and this is established purely by computing the invariant twice.

2. **The checklist for finding one is short**: parity, a sum or a sum modulo something small, an exclusive-or, an inversion count, a two-colouring argument, or a monovariant that only ever moves in one direction.

3. **A colouring argument resolves tiling-impossibility questions immediately**, as in the two-missing-corners chessboard.

4. **A Grundy number of zero means a loss for the player about to move**, and Grundy numbers of independent games combine by exclusive-or, which is the entire practical value of the Sprague–Grundy theorem, since it lets a combined game be decomposed into simple independent pieces.

5. **Watch for "half the state can always be mirrored away" reductions**, as in CSES Stair Game, where only the odd-numbered stairs turn out to matter.

6. **Brute-forcing small cases and inspecting the resulting sequence directly is a legitimate first response, not a fallback**, and is frequently faster than reasoning the pattern out from first principles.

7. **Games whose position graph contains cycles need retrograde analysis**, propagating outcomes backwards from terminal positions with a counter of unresolved outgoing moves, rather than an ordinary forward or memoised recursion.

8. **A constructive problem is solved by generalising the *process* used to build small solutions**, not merely by inspecting what the small solutions happen to look like, with the smallest few cases very often needing separate handling.

9. **A suspiciously simple answer, such as one depending only on parity, is a signal that the intended solution has already been found**, not a coincidence to be second-guessed.

---

## Where people lose these problems

**Attempting to search when the input size has already ruled it out.** A bound in the region of a billion is a direct indication that no exploration of the state space was ever intended, and the response should be to look for structure rather than to search harder.

**Using an ordinary memoised recursion on a game whose position graph contains cycles.** This either fails to terminate or silently returns an incorrect draw value, and retrograde analysis is required instead.

**Assuming misère play behaves identically to normal play.** It generally does, with the specific and important exception of the all-piles-of-size-one case in Nim, and the exact wording of the losing condition needs to be checked carefully rather than assumed.

**Combining Grundy numbers from sub-games that are not actually independent**, for instance because they secretly share a resource. The exclusive-or combination rule only applies when the sub-games are genuinely independent of one another.

**Failing to special-case the smallest few inputs in a constructive solution.** These almost always exist and are easy to overlook once a general pattern has been found for larger inputs.

**Not verifying that a claimed invariant is actually preserved by every single legal operation**, including operations that seem too obviously harmless to bother checking.

**In LC 913 specifically, mismodelling the state.** The state needs to include whose turn it currently is, in addition to the positions of both the cat and the mouse, and the handling of repeated positions needs to be resolved through the retrograde-analysis machinery described above rather than through an ad hoc rule.

---

## Working through the problem list

### Block 1 · Invariants

- **LC 877 Stone Game** — *decide whether the first player can always win a pile-taking game.* Solve it first with dynamic programming, in a few minutes, and then find the parity-based invariant argument described above. **The contrast between the two approaches is the actual point of this problem.**
- **LC 2038 Remove Colored Pieces if Both Neighbors are the Same Color** — *decide the winner of a piece-removal game.* A pure counting argument over the available moves for each player, with no search required.
- **LC 2029 Stone Game IX** — *decide the winner given counts of values by their remainder modulo three.* A case analysis over those three counts. Fiddly, and worth taking slowly.
- **LC 810 Chalkboard XOR Game** — *the exclusive-or game worked through in full above.* A very short solution supported by a real proof.

### Block 2 · Grundy numbers

- **CSES Nim Game II** — *a Nim variant.* Establishes the baseline theory.
- **CSES Stick Game** — *remove sticks from an allowed set of quantities.* Compute Grundy numbers by dynamic programming, then find and exploit the resulting period. Treat this as your first real exercise in the brute-force-and-inspect method.
- **CSES Stair Game** — *coins slide down a numbered staircase.* The odd-stairs reduction described above. The best problem in this block.
- **CSES Grundy's Game** — *split a pile into two unequal non-empty piles.* Build a table of Grundy numbers and look for structure.

### Block 3 · Search and construction

- **CSES Chessboard and Queens** — *place queens on a board so that none attacks another, respecting some pre-placed queens.* Backtracking with the standard diagonal-conflict tracking arrays, included as a reminder that "constructive" occasionally just means "search a genuinely small space carefully" rather than requiring a closed-form pattern.

### Boss problem

- **LC 913 Cat and Mouse** — *decide the outcome of a pursuit game on a graph, under optimal play from both sides.* Retrograde analysis with a countdown of unresolved moves, worked through in Part 4. Genuinely difficult, and worth doing once for the technique itself, since it is the general answer for any game whose positions can recur.

---

**Progress in this chapter is better measured by a habit than by a percentage.**

The habit worth tracking directly is how long you spend theorising before switching to brute-forcing small cases and inspecting the results. If that switch consistently takes twenty minutes rather than five, that delay is itself the finding, and the fix is a simple rule: allow five minutes of pure thought, and if nothing has emerged by then, write the brute-force version immediately rather than continuing to theorise.

This chapter sits in the third pass of the sheet because problems of this kind appear infrequently in practice. What it rewards, more than any specific piece of theory, is a habit that is cheap to build and that pays off disproportionately whenever a problem of this flavour does appear.

---

## Check yourself

1. What is a preserved quantity, and what can be concluded from it without searching any intermediate states?
2. List six things worth checking when hunting for one.
3. Why can a chessboard missing two opposite corners not be tiled by dominoes?
4. State the three parts of the Sprague–Grundy theorem. Which one is what makes it practically useful?
5. Compute the Grundy number for a game where one to three stones may be removed from a single pile. What pattern emerges?
6. Why do only the odd-numbered stairs matter in CSES Stair Game?
7. Give the invariant-based argument for LC 877, distinct from its dynamic programming solution.
8. Why does an ordinary memoised recursion fail on LC 913, and what replaces it?
9. Describe the brute-force-and-inspect method in four steps.
10. What should always be checked separately when generalising a constructive solution from small cases?
