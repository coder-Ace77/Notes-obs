---
tags: [dsa, guide, digit-dp, probability, expectation, automata]
chapter: 16
sheet-section: P
---

# Chapter 16 · Digit DP, Automata DP, Probability & Expectation

> **Read this before you start the problems.** Each idea is introduced with a small example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **P** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

This chapter combines two topics that share nothing in common except that both are close to guaranteed to appear at some point and both are things that people with substantial practice elsewhere still get wrong under time pressure.

Digit dynamic programming is difficult mainly because its template, while short, does not stay memorised well from a single exposure. The technique itself is not conceptually hard once explained, but the specific bookkeeping, tracking whether you are still bound by the upper limit and whether you have placed a non-zero digit yet, is the kind of detail that is easy to get subtly wrong and hard to notice is wrong, because the resulting answer is usually close to correct rather than obviously broken.

Expectation and probability are difficult for a different reason. The mathematics involved is genuinely elementary, but there is a real decision to make about which direction a computation should run, forward from the start or backward from the end, and choosing the wrong one produces a recursion that either does not terminate or computes something other than what was intended. Getting this decision right consistently requires having seen the reasoning worked through carefully at least once, since it is not something that becomes obvious from formulas alone.

---

## Part 1 · Digit dynamic programming

### What these problems look like

The trigger is close to unmistakable: a request to count integers within a range, where the upper end of that range may have up to eighteen or so digits, so that no loop could ever iterate through all of them directly.

Secondary signals include constraints on the digits themselves, such as forbidding two consecutive equal digits, restricting to a particular set of allowed digits, or bounding the number of distinct digits used; constraints involving divisibility by something related to the digit sum; and questions phrased in terms of bits rather than digits, which is the same technique in base two.

The first move, in every case, is the same: reduce a two-sided range to a one-sided one, since counting valid numbers from zero up to some bound `x` is a single well-defined subproblem, and the answer for a range from `L` to `R` is obtained by computing that subproblem at `R` and subtracting the result at `L` minus one. A two-sided digit dynamic program is never the right thing to write directly.

### The template

```cpp
string S;                                     // the upper bound written as a string
vector<vector<vector<long long>>> memo;       // indexed by [position][started][state]

long long go(int pos, bool tight, bool started, int state) {
    if (pos == (int)S.size()) return accept(state, started) ? 1 : 0;
    if (!tight && memo[pos][started][state] != -1) return memo[pos][started][state];

    int lim = tight ? S[pos] - '0' : 9;
    long long res = 0;
    for (int d = 0; d <= lim; d++) {
        bool nTight   = tight && (d == lim);
        bool nStarted = started || (d > 0);
        int  nState   = nStarted ? transition(state, d) : state;
        res += go(pos + 1, nTight, nStarted, nState);
    }
    if (!tight) memo[pos][started][state] = res;
    return res;
}
```

Four parts of this are worth understanding rather than simply typing out.

**The `tight` flag** records whether every digit placed so far has exactly matched the corresponding digit of the bound. While this flag is true, the next digit placed cannot exceed the bound's own digit at that position, and the instant a smaller digit is placed, the flag becomes false and stays false for the remainder of that recursive branch, since the number being built is now guaranteed to be smaller than the bound regardless of what follows. Exactly one path through the whole recursion stays tight the entire way, namely the bound itself, which is why states where `tight` is true are never memoised: memoising them would be actively incorrect unless `tight` were included as part of the memoisation key, since the same combination of position and state means something different depending on it.

**The `started` flag** tracks whether a non-zero digit has been placed yet, which matters whenever leading zeros need to be treated differently from a genuine digit of zero appearing within the number. Whether this flag is needed at all depends on the problem: counting numbers by digit sum does not need it, since a leading zero contributes nothing to a sum regardless of whether it is "real", but counting numbers with all distinct digits does need it, since without it a leading zero would be treated as occupying the digit slot for zero even in numbers that do not actually contain a zero. Deciding this explicitly, rather than guessing, is the single most common source of digit dynamic programming bugs.

**The `state`** is whatever the specific problem's constraint requires: a running digit sum, a remainder modulo some number, a bitmask of digits used so far, the previous digit placed, or anything else needed to decide validity at the end.

**The memoisation is indexed by position, `started`, and `state`, deliberately excluding `tight`.** Reusing the same memoised table across the two calls needed for `f(R)` and `f(L - 1)` is incorrect, since the memoisation was built against a different bound; the table must either be cleared between the two calls or the bound must itself be part of the key.

### A catalogue of states

| Problem | The state | Is `started` needed |
|---|---|---|
| AC EDPC S Digit Sum | the digit sum, taken modulo the divisor | no |
| LC 233 Number of Digit One | the count of ones placed so far | no |
| LC 600 No Consecutive Ones, base two | the previous bit | no |
| LC 902 Numbers At Most N Given Digit Set | none beyond the allowed digit set | yes, since shorter numbers must also be counted |
| LC 1012 Numbers With Repeated Digits | a mask of digits used so far | yes |
| LC 2376 Count Special Integers | a mask of digits used so far | yes |
| CF 628D Magic Numbers | position parity together with a remainder | no |
| CF 55D Beautiful numbers | a value modulo 2520, together with the least common multiple of the digits used | no |

**CF 55D Beautiful numbers**, which asks for numbers divisible by every non-zero digit they contain, is worth working through in detail, because its compression trick generalises well beyond this one problem. Being divisible by every digit present is the same as being divisible by the least common multiple of those digits, and any such least common multiple must itself divide the least common multiple of one through nine, which is 2520. This means the running value only ever needs to be tracked modulo 2520, and the running least common multiple only ever takes one of the forty-eight divisors of 2520, which can be precomputed and indexed directly. The state therefore becomes a value modulo 2520 combined with one of forty-eight possible least-common-multiple values, giving a total state count in the low millions, which is entirely manageable. **The underlying idea, that you never need to track a quantity more precisely than the largest thing you will ever need to divide by, is worth remembering on its own.**

### When a direct formula is faster than the template

Some digit problems have a closed-form or purely combinatorial answer that is shorter to derive than the full template.

LC 233 Number of Digit One can be solved by considering each digit position in turn and counting, using a split into the digits above, at, and below that position, how many numbers up to the bound have a one there. This is the same "count in bands" idea used for CSES *Digit Queries* in chapter [[01 Implementation and Simulation]], applied to a different quantity, and it is worth deriving once even though the digit dynamic programming template would also work.

LC 357 Count Numbers with Unique Digits is pure counting, since the number of `k`-digit numbers with all distinct digits is a simple product of decreasing choices.

LC 1012 Numbers With Repeated Digits is most easily solved by counting the complement, meaning numbers with entirely distinct digits, using the combinatorial method above, and subtracting from the total. Counting the complement of a condition rather than the condition itself is a move that recurs constantly and is the central idea of chapter [[19 Combinatorics and Number Theory]].

### Automata dynamic programming

Digit dynamic programming is really dynamic programming over an automaton whose alphabet happens to be the ten digits and whose state happens to track a numeric constraint. Replacing the alphabet and the automaton with something else produces a whole family of related techniques.

CSES *Required Substring* asks for the count of strings of a given length, over some alphabet, that avoid containing a specified pattern. Building the failure automaton for that pattern, using the technique from chapter [[18 Strings]], and then running a dynamic program where the state is the current automaton state reached after each character, counting only paths that never reach the pattern's accepting state, answers this directly. **Combining a string-matching automaton with a dynamic program is a genuinely recurring technique**, and it is also how LC 3213, which combines an Aho-Corasick automaton with a dynamic program, works one level further.

---

## Part 2 · Probability and expectation

### The two laws that do almost all the work

**Linearity of expectation** states that the expected value of a sum of random quantities equals the sum of their individual expected values, and critically, this holds regardless of whether those quantities are independent of one another. This unconditional validity is what gives the law its power: a quantity that would be extremely difficult to reason about directly, because of complicated dependencies between its parts, can often be broken into simple pieces whose individual expectations are easy, added together, with the dependencies simply never needing to be resolved.

CSES *Moving Robots* demonstrates this clearly. Several robots move randomly and independently on a board for a fixed number of steps, and the question asks for the expected number of squares left empty at the end. The positions of different robots are not independent of each other in any simple way, and reasoning about the joint distribution of all of them together would be difficult. But the expected number of empty squares can be written as a sum, over every square, of the probability that no robot ends up there, and that probability factors as a product, over the robots, of the probability that each individual robot avoids that square, since each robot moves independently of the others. Each robot's own distribution is a simple dynamic program over its own steps, computed entirely separately from the others. The correlation between different robots never had to be understood, because linearity never asked for it.

The general template worth writing down whenever a problem asks for "the expected number of things with some property" is that this quantity equals the sum, over every candidate thing, of the probability that the candidate has the property.

**The law of total expectation** states that the expected value of a quantity equals a weighted average, over every possible state you might currently be in, of the probability of being in that state times the expected value of the quantity given that state. This is what allows an expectation to be computed by a dynamic program at all, since it expresses the overall answer in terms of answers to smaller, more specific subproblems.

### Deciding which direction the computation runs

This is the point where people go wrong most often, so it is worth being explicit about it.

A **forward** dynamic program tracks the probability of being in each state, pushing that probability ahead as the process unfolds. This is natural when the goal is to know the distribution at some fixed later time.

A **backward** dynamic program tracks the expected remaining cost or number of steps starting from each state, pulling information back from states that occur later. This is natural when the goal is a total expected cost or a total expected number of steps, because "expected remaining amount" satisfies a clean self-referential equation:

$$E[s] = \text{cost}(s) + \sum_{s'} P(s \to s') \cdot E[s']$$

with the expected value at any terminal state set to zero.

**Expectation dynamic programming almost always runs backward**, and the reason is that "expected total accumulated so far" does not lead to a usable recursive equation in the way that "expected amount still remaining" does, since the remaining-amount formulation is a genuine fixed point that can be solved for directly.

AC EDPC J Sushi is the standard hard example, and working through its state design is worth doing slowly. There are `n` plates of sushi, each holding one, two or three pieces, and at each step a plate is chosen uniformly at random and, if it is not already empty, one piece is eaten from it; the question asks for the expected number of picks needed until every plate is empty.

The state that seems natural at first, namely which plates currently hold which amounts, is exponential and unusable. The insight is that only the *counts* matter, not which specific plate holds which amount, so the state becomes a triple of counts: how many plates currently hold exactly one piece, how many hold exactly two, and how many hold exactly three. For a few hundred plates this gives a state space in the low tens of millions, which is entirely manageable.

$$E[a,b,c] = 1 + \frac{n-a-b-c}{n}E[a,b,c] + \frac{a}{n}E[a{-}1,b,c] + \frac{b}{n}E[a{+}1,b{-}1,c] + \frac{c}{n}E[a,b{+}1,c{-}1]$$

The term involving `E[a, b, c]` on the right-hand side, representing the probability of picking an already-empty plate and returning to exactly the same state, cannot be handled by ordinary recursion, since it would call itself. **The fix is to move that term to the left-hand side algebraically and solve for `E[a, b, c]` directly**, which produces a closed expression for this state's expected value purely in terms of states with strictly smaller totals. This kind of self-loop is the single most common trap in expectation dynamic programming, and it appears in essentially every "keep trying until something new happens" problem, including the classic coupon collector setup. The rule worth remembering is simply that whenever a state can transition back to itself, that term needs to be isolated and solved for algebraically before any code is written.

### A transition summed over a sliding window

LC 837 New 21 Game draws numbers uniformly from a fixed range while a running total stays below some threshold, and asks for the probability the final total does not exceed a given limit. The dynamic program defining `dp[i]` as the probability of ever reaching total `i` exactly has a transition that sums `dp[j]` over the previous several values of `j`, specifically a window whose width is the size of the range being drawn from. Maintaining a running sum of that window incrementally, exactly as in chapter [[06 Prefix Sums and Difference Arrays]] and in AC EDPC M from chapter [[14 Dynamic Programming Core]], turns a transition that would otherwise cost the width of the window into one costing constant time, and this speed-up is worth trying as a first response whenever a probability transition involves summing over a contiguous range of earlier states.

### Working modulo a prime

Many competitive problems ask for a probability expressed modulo a large prime rather than as a decimal, since a probability is generally a fraction and fractions do not reduce cleanly modulo a prime by ordinary division. The standard convention represents a fraction `p` over `q` as `p` multiplied by the modular inverse of `q`, computed using Fermat's little theorem as `q` raised to the power of the modulus minus two.

Every division encountered during such a computation is replaced by multiplication by the appropriate modular inverse, and if many different inverses will be needed, precomputing all of them for the small integers from one up to some bound is more efficient than computing each one separately:

```cpp
inv[1] = 1;
for (int i = 2; i <= n; i++) inv[i] = (MOD - (MOD/i) * inv[MOD % i] % MOD) % MOD;
```

CSES problems in this area typically want a floating-point answer, while Codeforces problems typically want a modular one, and it is worth checking which is expected before starting, since converting a solution from one convention to the other partway through is unpleasant.

### Probability problems that are not dynamic programming at all

CF 442B Andrey and Problem is a useful reminder that not every probability problem needs a recursive structure. Given several friends, each independently able to solve a problem with some stated probability, the task is to choose a subset maximising the probability that exactly one of them solves it. The correct approach sorts the friends by their probability in decreasing order and greedily adds them while doing so keeps improving the objective, which can be justified with an exchange argument in the style of chapter [[03 Greedy and Exchange Arguments]], since the objective as a function of how many friends are included turns out to have a single peak. This problem sits in the probability section of the sheet and is solved entirely by greedy reasoning, which is worth keeping in mind before assuming every probability problem requires a table.

---

## The ideas worth carrying forward

**On digit dynamic programming:**

1. **Reduce every range query to a one-sided one**, computing a count up to the upper bound and subtracting a count up to one less than the lower bound. A two-sided version is never the right thing to write directly.

2. **States where `tight` is true are never memoised**, since exactly one path through the recursion stays tight, and memoising it without including `tight` as part of the key would be incorrect.

3. **Decide explicitly whether `started` is needed.** Constraints about which digits are used need it; constraints purely about a digit sum usually do not.

4. **A running quantity never needs to be tracked more precisely than the largest number you will ever need to divide by.** This is the idea behind CF 55D's compression to a value modulo 2520.

5. **Some digit problems have a direct combinatorial answer, particularly by counting the complement**, which is worth checking for before writing a full digit dynamic program.

6. **Digit dynamic programming generalises to any automaton.** A string-matching automaton combined with a dynamic program counts strings avoiding a pattern in exactly the same way.

**On probability and expectation:**

7. **Linearity of expectation holds regardless of dependence**, which is precisely what makes it useful: a correlated quantity can be decomposed into simple independent pieces and their expectations simply added.

8. **"Expected number of things with a property" equals a sum, over every candidate, of the probability that candidate has the property.** Writing this down explicitly is worth doing every time the phrase "expected number of" appears.

9. **Expectation dynamic programming runs backward, from the terminal states**, computing the expected amount still remaining, because that quantity satisfies a genuine fixed-point equation while an accumulated total does not.

10. **A state that can transition to itself needs that term isolated and solved for algebraically**, rather than left inside an ordinary recursive call. This is the most common trap in the whole topic.

11. **When the state describes "a small number of identical items in various conditions", collapse it to counts of each condition** rather than tracking every item individually.

12. **A transition summing over a sliding window of earlier states should immediately suggest maintaining a running sum**, exactly as in ordinary dynamic programming.

13. **Fractions modulo a prime are handled by multiplying by a modular inverse**, computed with Fermat's little theorem, and it is worth checking early whether a problem wants a floating-point or a modular answer.

14. **Not every problem in the probability section is dynamic programming.** CF 442B is solved entirely by a greedy argument.

---

## Where people lose these problems

**Memoising a state where `tight` is true, without including `tight` in the memoisation key.** The result looks close to correct and is not.

**Reusing a memoisation table between the two calls needed for a range query.** The table must be cleared, or the bound included in the key, between computing the count up to the upper bound and the count up to one below the lower bound.

**Getting leading zeros wrong.** Testing against a small range where the answer can be counted by hand is the most reliable way to catch this.

**Forgetting to guard the lower end of a range at zero**, where subtracting one produces a value requiring special handling.

**Overflow in the final count.** A range with up to a quintillion numbers in it can produce a count exceeding the range of a standard integer, so a wider integer type or a modular reduction is needed depending on what the problem asks for.

**Leaving a self-loop unresolved in an expectation recursion.** This produces either infinite recursion or, if silently dropped, a wrong answer that can be difficult to notice is wrong.

**Attempting to divide directly in modular arithmetic.** Division has no direct meaning modulo a number; multiplying by the modular inverse is required instead.

**Accumulating floating-point error over a very large number of steps.** This is usually fine with standard double-precision arithmetic, but if a problem demands several decimal places of precision over a million or more accumulated terms, extended precision is worth considering.

**In CSES Candy Lottery, missing the tail-sum identity.** For a non-negative integer-valued random variable, the expected value equals the sum, over every positive threshold, of the probability that the variable is at least that threshold. This identity converts an expectation of a maximum or minimum into a sum of simple probabilities and is worth having memorised.

**In CSES Throwing Dice, treating it as a probability problem rather than what it actually is.** With an index reaching into the quintillions, this is matrix exponentiation from chapter [[14 Dynamic Programming Core]], despite its position in this section of the sheet.

---

## Working through the problem list

### Block 1 · Digit dynamic programming, in order

- **AC EDPC S · Digit Sum** — *count numbers up to a bound whose digit sum is divisible by a given value.* The template. Write it from scratch, since this is the version you will reuse.
- **LC 233 Number of Digit One** — *count occurrences of the digit one among all numbers up to a bound.* Worth doing both with the template and with the direct closed-form counting method.
- **LC 600 Non-negative Integers without Consecutive Ones** — *count numbers up to a bound with no two consecutive set bits.* Base two. There is also a Fibonacci-based closed form, worth finding after the template version works.
- **LC 902 Numbers At Most N Given Digit Set** — *count numbers formable from a restricted digit set that do not exceed a bound.* The `started` flag matters here, since shorter numbers must also be counted.
- **LC 2376 Count Special Integers** — *count numbers up to a bound with all distinct digits.* A mask of used digits, combined with `started`.
- **LC 1012 Numbers With Repeated Digits** — *count numbers up to a bound containing at least one repeated digit.* Counting the complement using the same machinery as the previous problem, then subtracting.
- **LC 2719 Count of Integers** — *count numbers in a range whose digit sum falls within given bounds.* A two-sided range reduced to the one-sided form, with a subtlety in how the lower bound is handled.
- **LC 357 Count Numbers with Unique Digits** — *count numbers with a given number of digits, all distinct.* Pure combinatorics, worth a few minutes rather than a full digit dynamic program.
- **CSES Counting Numbers** — *count numbers up to a large bound with no two adjacent equal digits.* Needs `started`.
- **CF 628D Magic Numbers** — *count numbers up to a bound matching a fixed alternating-digit pattern and divisible by a given value.* The state is a running remainder, and the parity rule for which positions must equal the fixed digit is worth writing out clearly.
- **CF 55D Beautiful numbers** — *the 2520-compression problem worked through above.* The most demanding problem in this block.
- **LC 3007, revisited as digit dynamic programming** — *having solved the binary search half of this problem in chapter [[04 Binary Search on the Answer]], now write the digit-counting function it depends on, either as a digit dynamic program or as a closed form.*

### Block 2 · Probability and expectation, in order

- **CSES Dice Probability** — *the probability distribution of a sum of dice rolls.* A forward probability dynamic program, and a good warm-up.
- **AC EDPC I · Coins** — *the probability that more coins land heads than tails.* Another forward dynamic program.
- **CSES Candy Lottery** — *the expected value of the maximum of several random draws.* The tail-sum identity.
- **CSES Moving Robots** — *the expected number of empty squares after several robots move randomly.* The clearest possible demonstration of linearity of expectation, and the most important problem in this half of the chapter.
- **LC 837 New 21 Game** — *the probability a running total, built from random draws, does not exceed a limit.* The sliding-window probability dynamic program.
- **AC EDPC J · Sushi** — *the expected number of picks to empty a set of plates, worked through above.* The most demanding problem in this half of the chapter, and worth deriving the self-loop algebra on paper before writing any code.
- **CF 442B Andrey and Problem** — *choose a subset of friends maximising the probability exactly one solves a problem.* Greedy, with an exchange argument, despite its position among probability problems.
- **LC 1467 Probability of a Two Boxes Having The Same Number of Distinct Balls** — *a probability computed via multinomial counting over at most eight colours.* More combinatorics than probability, and comparatively low priority.
- **CSES Throwing Dice** — *a linear recurrence with an enormous index.* Matrix exponentiation, not probability, despite its placement.
- **CSES Graph Paths I** and **Graph Paths II** — *count, and then minimise the cost of, walks of a given length through a graph.* Matrix exponentiation over an adjacency matrix, with the second problem using the minimum-plus-sum semiring in place of ordinary addition and multiplication, which is a genuinely elegant demonstration that matrix exponentiation works over structures other than ordinary arithmetic.
- **CSES Required Substring** — *count strings avoiding a given pattern.* The automaton dynamic program from Part 1, best attempted after chapter [[18 Strings]].

---

**A reasonable target here is around 75% of submissions passing first time.**

Digit dynamic programming becomes mechanical after the template has been written a handful of times, and its failures are almost entirely the `started` flag and memoisation across the two calls of a range query, both of which are checkable in a fixed way. Expectation failures tend to be conceptual, involving the direction of the computation or an unresolved self-loop, and the response that helps is deriving the recurrence fully on paper before writing any code, every time, rather than trusting an approach that merely feels right.

---

## Check yourself

1. What is the first reduction to apply to any "count within a range" problem?
2. Why are states where `tight` is true never memoised?
3. Give one problem needing the `started` flag and one that does not. What causes the difference?
4. In CF 55D, why is tracking a value modulo 2520 sufficient, and how many distinct least-common-multiple states are there?
5. State linearity of expectation, and explain precisely why holding regardless of dependence is what makes it powerful.
6. Rewrite "the expected number of empty squares" as a sum, and say what the correlation between robots would otherwise have cost you.
7. In which direction does expectation dynamic programming usually run, and what does the stored value at each state represent?
8. A state transitions back to itself with some probability. Write out how to solve for its value directly.
9. What is the state in AC EDPC J, and why is it not "which plate holds which amount"?
10. State the tail-sum identity for the expected value of a non-negative integer-valued random variable.
11. How do you divide by seven when working modulo a large prime?
