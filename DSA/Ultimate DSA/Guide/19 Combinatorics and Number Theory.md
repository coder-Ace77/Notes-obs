---
tags: [dsa, guide, combinatorics, number-theory, modular-arithmetic, inclusion-exclusion]
chapter: 19
sheet-section: S
---

# Chapter 19 · Combinatorics, Number Theory & Modular Arithmetic

> **Read this before you start the problems.** Each idea is introduced with a small example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **S** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

Counting problems have a specific way of being difficult that is different from most of the rest of this sheet. There is rarely an algorithm to design, in the sense of choosing a data structure or a search strategy; instead, the entire difficulty is in the argument itself, deciding exactly what is being counted and confirming that every object is counted exactly once. A counting solution can be extremely short, sometimes a single formula, while still requiring real care to arrive at, because a plausible but subtly wrong counting argument produces a number that is wrong in a way that no amount of testing on small cases will necessarily reveal, since the argument can be wrong in a way that only manifests on larger inputs.

Two moves resolve a large share of these problems, and the chapter is organised around making both of them second nature. The first is counting the complement of a condition rather than the condition itself, when the complement is structurally simpler. The second is being careful about what "counting once" actually requires, which usually means either finding a canonical way to describe each object so it cannot be produced twice, or explicitly correcting for overcounting through inclusion and exclusion.

The requirement to report answers modulo a large prime adds a layer of mechanical difficulty on top of this, since ordinary division has no direct meaning modulo a number, and every formula involving division has to be rewritten in terms of modular inverses before it can be implemented.

---

## What these problems look like

The clearest signal is a request to report an answer modulo a specific large number, most often a value near a billion, attached to a question of the form "how many ways".

Beyond that direct signal, several phrasings are worth recognising: "the number of ways to arrange, choose, or partition" something is a direct counting question; "how many pairs or triples satisfy" some condition often calls for inclusion and exclusion or for counting the complement; "the number of distinct outcomes that different processes can produce" often reduces to a multinomial coefficient over a tree or a merge; a monotone condition on subsets, such as "at least one element exceeds a threshold", is frequently easier to count as its complement; grid paths that must avoid obstacles usually need inclusion and exclusion over those obstacles, taken in a specific order; and any mention of symmetry, rotation, or reflection is a signal for Burnside's lemma.

---

## Part 1 · The modular toolkit

This is worth having ready in a personal template file rather than reconstructed under pressure.

```cpp
const long long MOD = 1e9 + 7;

long long mpow(long long b, long long e, long long m = MOD) {
    long long r = 1; b %= m;
    while (e) { if (e & 1) r = r * b % m; b = b * b % m; e >>= 1; }
    return r;
}
long long minv(long long a) { return mpow(a, MOD - 2); }        // requires MOD to be prime

vector<long long> fact, ifact;
void initFact(int n) {
    fact.resize(n+1); ifact.resize(n+1);
    fact[0] = 1;
    for (int i = 1; i <= n; i++) fact[i] = fact[i-1] * i % MOD;
    ifact[n] = minv(fact[n]);
    for (int i = n; i > 0; i--) ifact[i-1] = ifact[i] * i % MOD;   // one inverse computes them all
}
long long C(int n, int k) {
    if (k < 0 || k > n || n < 0) return 0;
    return fact[n] * ifact[k] % MOD * ifact[n-k] % MOD;
}
```

Three details matter beyond simply having this code available.

**The loop computing `ifact` runs backwards from a single modular inverse.** Computing the inverse of every factorial independently would need a separate call to `mpow` for each one, costing an extra logarithmic factor; computing only the inverse of the largest factorial and then working downwards, multiplying by the next integer at each step, computes every inverse factorial needed using just one call to `mpow` in total.

**`C(n, k)` is written to return zero whenever `k` falls outside the valid range**, rather than assuming the caller only ever passes valid arguments. This single guard removes a large amount of special-case handling from formulas elsewhere, since a sum that ranges slightly beyond where its terms are meaningful can simply be summed as written, relying on the out-of-range terms to contribute zero on their own.

**Fermat's little theorem, which is what makes computing a modular inverse this way valid, requires the modulus to be prime.** The values `10^9 + 7` and `998244353`, both common choices, are prime, but a modulus that is not prime, as occurs in AC EDPC V from chapter [[13 Trees]], makes division unavailable entirely, and the correct response there is to restructure the computation using prefix and suffix products rather than attempting to invert anything.

---

## Part 2 · Counting the complement

This is the single highest-frequency idea in the whole chapter.

**When a condition phrased as "at least one" is difficult to count directly, and the condition "none" is easy, counting the complement and subtracting from the total is usually the fastest route to a solution.**

LC 2518 asks for the number of ways to split a set into two groups such that both groups have a sum of at least some threshold `k`. Counting this directly is awkward, but the complement, meaning splits where at least one of the two groups has a sum below `k`, is countable with a subset-sum dynamic program over sums strictly less than `k`, since a sum that small is easy to enumerate directly. LC 1012 similarly counts numbers containing at least one repeated digit by counting numbers with entirely distinct digits, which is a short combinatorial computation, and subtracting.

More generally, "at least one of several conditions holds" is inclusion and exclusion applied to counting a union, which is the complement idea taken further, covered in the next part.

**The habit worth building is to write down what "none" or "neither" looks like before attempting anything else**, whenever a statement contains the phrase "at least one". This takes a matter of seconds and resolves a meaningful fraction of these problems outright.

---

## Part 3 · Inclusion and exclusion

The general formula for the size of a union of several sets is

$$\left|\bigcup_i A_i\right| = \sum |A_i| - \sum |A_i \cap A_j| + \sum |A_i \cap A_j \cap A_k| - \cdots$$

**When the number of sets involved is small, around twenty or fewer,** every subset of them can be enumerated directly with a bitmask, in the style of chapter [[15 Bitmask DP]], applying the alternating sign according to the size of the subset.

**When the sets are more numerous but carry additional structure,** that structure often gives a shortcut. CF 451E asks for the number of solutions to an equation of the form `x_1 + x_2 + ... + x_n = s` where each `x_i` is bounded above by some `c_i`. Without any upper bounds, this is a standard stars-and-bars count of `C(s + n - 1, n - 1)`. With upper bounds, the count needs correcting for the cases where one or more variables exceed their bound, and substituting `x_i' = x_i - (c_i + 1)` for a variable that has exceeded its bound converts that violation into another, shifted stars-and-bars count; with at most twenty variables, enumerating which subset of them are assumed to violate their bound, with alternating signs, applies inclusion and exclusion directly.

The two stars-and-bars formulas worth having memorised, since re-deriving them from scratch is a waste of time under pressure, are that the number of non-negative integer solutions to `x_1 + ... + x_n = s` is `C(s + n - 1, n - 1)`, and the number of strictly positive solutions is `C(s - 1, n - 1)`.

**CF 559C asks for the number of grid paths from a start to an end point avoiding a given set of blocked cells.** Sorting the blocked cells appropriately and defining `dp[i]` as the number of paths reaching blocked cell `i` while avoiding every earlier blocked cell, the value is the total number of unrestricted paths to cell `i` minus, for every earlier blocked cell `j`, the number of paths from `j` to `i` multiplied by `dp[j]`. This runs in time proportional to the square of the number of blocked cells and is a form of inclusion and exclusion organised as a dynamic program by classifying each invalid path according to the *first* blocked cell it touches. **AC EDPC Y is the same technique applied to a slightly different obstacle-avoidance setting**, and doing both problems back to back makes the shared idea obvious.

**The general lesson worth extracting from this pair of problems is that classifying invalid objects by the first violation they commit makes the resulting classes disjoint**, which allows the objects to simply be summed rather than combined with inclusion-and-exclusion signs, and this reorganisation is considerably more powerful than it might first appear, recurring well beyond grid-path problems.

---

## Part 4 · Counting each object exactly once

The other common failure in a counting problem, alongside miscounting a union, is counting a single object more than once through different routes to it.

LC 1569 asks how many distinct orderings of a sequence produce the same binary search tree when inserted one at a time. The root of the resulting tree is fixed, being whichever element was inserted first, and everything in its left subtree and everything in its right subtree can be interleaved with each other in any order without changing which tree results, so

$$f(\text{node}) = \binom{L + R}{L} \cdot f(\text{left}) \cdot f(\text{right})$$

where `L` and `R` are the sizes of the two subtrees, and the binomial coefficient counts the number of ways to interleave two sequences of those two lengths while preserving the relative order within each. **This interleaving binomial coefficient is worth recognising as a standalone idea**, since it also appears in counting topological orderings and in counting certain families of lattice paths.

LC 2514 counts anagrams of each word in a list, using the standard formula for permutations of a multiset, being the factorial of the word's length divided by the product of the factorials of each letter's count, and multiplying these counts together across the whole list.

LC 920 builds playlists of a given length using a given number of distinct songs, with the rule that a song may not be repeated until at least a fixed number of other songs have played since its last use. The dynamic program tracks the playlist length and the number of distinct songs used so far, and at each step either introduces a brand new song, giving a number of choices equal to the total number of songs minus those already used, or replays an old song that is far enough back, giving a number of choices equal to the number of already-used songs minus the required gap. **That subtraction of the required gap from the count of already-used songs is the entire content of the problem**, once the state itself has been identified.

---

## Part 5 · Number theory that comes up often enough to know cold

**A sieve computing the smallest prime factor of every number up to some bound** gives logarithmic-time factorisation of any number in that range afterwards:

```cpp
vector<int> spf(n+1);
for (int i = 2; i <= n; i++) if (!spf[i])
    for (int j = i; j <= n; j += i) if (!spf[j]) spf[j] = i;
```

**Counting and summing divisors** follows directly from a number's prime factorisation: if `n` equals the product of `p_i` raised to `e_i` over its distinct prime factors, the count of divisors is the product of `(e_i + 1)` over those factors, and the sum of divisors is the product, over those factors, of `(p_i^{e_i+1} - 1) / (p_i - 1)`.

**CSES Divisor Analysis is exactly these two formulas, with one genuine complication**: computing the sum-of-divisors formula requires dividing by `p_i - 1` modulo the chosen prime, which fails specifically when `p_i - 1` is itself a multiple of that modulus, since then no modular inverse exists. The correct handling in that specific case is to compute the geometric series `1 + p + p^2 + ... + p^e` directly rather than through the closed-form division. **This is a useful lesson in its own right: a formula can have an edge case that is invisible until you check whether every division inside it is actually valid.**

**Fermat's little theorem also applies to exponents that are themselves enormous.** To compute `a` raised to the power `b` raised to the power `c`, modulo a prime `p`, the exponent itself only needs to be tracked modulo `p - 1`, so computing `e = b^c mod (p - 1)` first and then `a^e mod p` gives the answer, which is what CSES *Exponentiation II* asks for. Some care is needed when `a` is itself a multiple of `p`.

**Möbius-style inclusion and exclusion over divisors** answers CSES *Counting Coprime Pairs*, which asks for the number of pairs in an array whose greatest common divisor is exactly one. Letting `f[d]` be the number of array elements divisible by `d`, the number of pairs where `d` divides the greatest common divisor of the pair is `C(f[d], 2)`, and Möbius inversion converts this into the number of pairs whose greatest common divisor is exactly one via a signed sum, `Σ μ(d) · C(f[d], 2)`, where the values `f[d]` are computed efficiently by iterating over multiples of each `d`.

**The extended Euclidean algorithm, the Chinese remainder theorem, and Euler's totient function** are all worth knowing exist and having a working implementation of, without investing heavily beyond that, since they appear comparatively rarely on this sheet.

---

## Part 6 · Named counting sequences

**Catalan numbers**, given by `C(2n, n) / (n + 1)`, count several structurally different things that turn out to be equivalent: balanced sequences of brackets, binary trees with a given number of nodes, triangulations of a polygon, and certain monotone lattice paths that never cross a diagonal boundary. CSES *Bracket Sequences I* asks for this directly, and the reflection-principle proof, which pairs every path crossing the forbidden boundary with a path to a reflected endpoint, giving the formula `C(2n, n) - C(2n, n+1)`, is worth seeing once even if it is not re-derived every time.

**Derangements**, permutations with no element left in its original position, satisfy the recurrence `D_n = (n - 1)(D_{n-1} + D_{n-2})`, with `D_0 = 1` and `D_1 = 0`, and CSES *Christmas Party* asks for exactly this quantity.

**Burnside's lemma** states that the number of genuinely distinct objects, once symmetries that make some of them equivalent are accounted for, equals the average, over every symmetry in the relevant group, of the number of objects that symmetry leaves entirely unchanged:

$$|X/G| = \frac{1}{|G|}\sum_{g \in G} |\text{Fix}(g)|$$

CSES *Counting Necklaces* asks how many distinct necklaces can be made from a fixed number of beads and colours, treating rotations of the same necklace as identical. A rotation by `k` positions leaves a colouring fixed exactly when every cycle induced by that rotation is monochromatic, and there are `gcd(n, k)` such cycles, so that rotation fixes `m` raised to the power `gcd(n, k)` colourings, where `m` is the number of available colours; averaging this quantity over every rotation from zero to `n - 1` gives the answer.

This is a low-frequency topic on this sheet, and its value is mostly in recognition: a problem explicitly about symmetry is close to unsolvable without the lemma and close to mechanical once it is applied.

---

## The ideas worth carrying forward

1. **When a condition is phrased as "at least one", write down what "none" looks like before doing anything else.** Counting the complement resolves a meaningful share of these problems outright.

2. **`C(n, k)` should return zero for out-of-range `k`.** This single guard removes special-case handling from every formula built on top of it.

3. **Compute every inverse factorial from a single modular inverse, working backwards.** This costs one call to fast exponentiation rather than one per factorial.

4. **Fermat's little theorem, and therefore this style of modular inverse, requires a prime modulus.** A non-prime modulus rules out division entirely and calls for prefix and suffix products instead.

5. **Classifying invalid objects by the *first* violation they commit makes the resulting classes disjoint**, which turns what looks like an inclusion-and-exclusion problem into a straightforward dynamic program that simply sums contributions. CF 559C and AC EDPC Y both rely on this.

6. **Stars and bars gives `C(s + n - 1, n - 1)` for non-negative solutions and `C(s - 1, n - 1)` for strictly positive ones**, with upper bounds handled by inclusion and exclusion over which variables are assumed to violate them.

7. **Interleaving two sequences while preserving order within each is counted by a single binomial coefficient**, which is the mechanism behind LC 1569 and behind counting topological orderings more generally.

8. **Burnside's lemma averages the count of fixed objects over every symmetry**, and for rotations specifically, a rotation by `k` positions fixes `m` raised to `gcd(n, k)` colourings.

9. **A sieve of smallest prime factors gives fast factorisation for every number up to its bound**, and is worth building once and reusing.

10. **An enormous exponent is reduced modulo `p - 1` before the base is exponentiated modulo `p`**, by Fermat's little theorem, whenever the exponent itself is too large to use directly.

---

## Where people lose these problems

**Forgetting to reduce modulo the chosen number after every multiplication**, rather than only at the end. Two values each under a billion multiply to something that fits safely in a 64-bit integer, but a third multiplication on top of that does not, so reducing after every single multiplication is the safe habit.

**Leaving a subtraction negative.** The correction `(a - b + MOD) % MOD` is needed wherever a subtraction modulo the chosen number occurs.

**Attempting to divide directly rather than multiplying by a modular inverse.** There is no meaning to ordinary division under a modulus.

**Using a modular inverse computed via Fermat's little theorem when the modulus is not prime.** This produces a value that looks like an answer and is not one.

**Overcounting due to an unaccounted symmetry.** If ordered pairs are counted but the problem wants unordered pairs, dividing by two is correct only when no pair can be equal to its own reverse; when that is possible, those cases need separate handling.

**Off-by-one errors in the bounds passed to `C(n, k)`**, which the zero-guard resolves for most cases but not for a fundamentally wrong summation range.

**Sizing the factorial array for the smallest test case seen rather than the largest possible input.**

**In CSES Divisor Analysis, missing the case where `p - 1` shares a factor with the modulus.** The direct geometric-series computation is the fallback for that specific case.

**In LC 1977, underestimating the difficulty.** This problem needs a dynamic program tracking, for each position, where the most recent number in a decomposition started and ended, with the transition requiring a constant-time comparison of two substrings, answered by a precomputed longest-common-prefix table, combined with prefix sums over the transition to keep the whole computation quadratic rather than cubic. It draws on chapters [[06 Prefix Sums and Difference Arrays]], [[14 Dynamic Programming Core]] and [[18 Strings]] simultaneously and is one of the more demanding problems on the entire sheet, worth treating as a stretch goal rather than routine practice.

---

## Working through the problem list

### Block 1 · Building the toolkit

- **CSES Exponentiation II** — *compute a number raised to a tower of exponents, modulo a prime.* Fermat's little theorem applied to the exponent itself. Also the place to write and save a personal `mpow` implementation.
- **CSES Bracket Sequences I** — *count balanced bracket sequences of a given length.* The Catalan number formula, worth deriving from the reflection principle rather than looked up.
- **CSES Christmas Party** — *count seatings where nobody sits in their assigned seat.* Derangements.
- **LC 2514 Count Anagrams** — *count the distinct rearrangements of each word in a sentence and multiply them together.* Straightforward once `initFact` is available.

### Block 2 · The core techniques

- **CSES Divisor Analysis** — *compute the count and sum of divisors of a large number given its prime factorisation.* The two formulas, and the edge case in the sum-of-divisors computation.
- **CSES Counting Necklaces** — *count distinct necklaces up to rotation.* Burnside's lemma.
- **CSES Counting Coprime Pairs** — *count pairs of array elements whose greatest common divisor is one.* The Möbius-style inclusion-and-exclusion sum over divisors.
- **CF 559C Gerald and Giant Chess** — *count grid paths avoiding a set of blocked cells.* The "classify by first violation" dynamic program. **The single most instructive problem in this block.**
- **AC EDPC Y · Grid 2** — *the same technique applied to a related obstacle-avoidance setting.* Attempt this immediately after CF 559C to see that they are the same idea.
- **CF 451E Devu and Flowers** — *count ways to select flowers from bouquets with individual upper bounds.* Stars and bars combined with inclusion and exclusion over which bounds are violated.
- **LC 1569 Number of Ways to Reorder Array to Get Same BST** — *count orderings producing the same binary search tree.* The interleaving binomial coefficient, applied recursively.
- **LC 920 Number of Music Playlists** — *count playlists satisfying a no-repeat-too-soon rule.* The dynamic program with the repeat-gap subtraction.
- **LC 2518 Number of Great Partitions** — *count partitions where both parts meet a sum threshold.* Counting the complement via a bounded subset-sum dynamic program.
- **CF 300C Beautiful Numbers** — *count numbers with a bounded quantity of a particular digit whose resulting digit sum is itself well-behaved.* A clean instance of enumerating a small parameter directly and counting everything else combinatorially.
- **LC 1735 Count Ways to Make Array With Product** — *count arrays of a given length whose product equals a target.* Factorise the target and distribute each prime's exponent among the positions using stars and bars, multiplying the results across primes. This problem is filed under a different section on the sheet but belongs here.

### Boss problem

- **LC 1977 Number of Ways to Separate Numbers** — *count ways to split a digit string into a non-decreasing sequence of numbers with no leading zeros.* The dynamic program combining a longest-common-prefix table with prefix sums, described above. Genuinely one of the hardest problems on the whole sheet, and worth saving for last.

---

**A reasonable target here is around 75% of submissions passing first time.**

The toolkit itself is fixed and short, so once `mpow`, `initFact`, `C(n, k)` and the smallest-prime-factor sieve exist in a template file that has already been tested, most remaining failures are conceptual, usually overcounting or an incorrectly chosen complement, and the fix for those is writing the counting argument out in full sentences before writing any code, rather than trusting an argument that merely feels right.

---

## Check yourself

1. Write `initFact` and explain why the loop computing `ifact` runs backwards.
2. Why does `C(n, k)` guard against out-of-range values of `k`, and what does that guard save elsewhere?
3. When is Fermat's little theorem unusable for computing a modular inverse, and what should replace it in that case?
4. State the stars-and-bars formulas for non-negative and for strictly positive integer solutions.
5. How are upper bounds handled within a stars-and-bars count?
6. Explain the "classify by first violation" technique, and name two problems that rely on it.
7. Where does the binomial coefficient in LC 1569 come from?
8. State Burnside's lemma, and apply it to counting distinct necklaces under rotation.
9. In LC 920, where does the subtraction of the repeat gap come from in the transition?
10. Given `a` raised to the power `b` raised to the power `c`, modulo a prime `p`, with `b` and `c` both very large, what is the approach?
