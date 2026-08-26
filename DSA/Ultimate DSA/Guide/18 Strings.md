---
tags: [dsa, guide, strings, hashing, kmp, z-function, suffix-automaton, aho-corasick]
chapter: 18
sheet-section: R
---

# Chapter 18 · Strings: Hashing, Automata, Suffix Structures

> **Read this before you start the problems.** Each idea is introduced with a small example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **R** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

String problems are common in product assessments for a simple reason: they are easy to describe in a sentence and to give convincing sample cases for, which makes them attractive to write regardless of how hard the intended solution actually is. The consequence is that a string problem's difficulty rarely correlates with how long its statement is, and the harder versions typically need one specific structure rather than a clever observation, which makes recognition the main skill in this chapter.

There is a genuine hierarchy of cost among the available structures, running from a few lines of hashing at the cheap end to a suffix automaton at the expensive end, and the practical difficulty is choosing the cheapest structure that actually answers the question being asked, since reaching for something elaborate when a short hashing solution would do costs time without buying correctness, while reaching for something too simple produces a solution that is either wrong or too slow.

---

## What these problems look like, matched to the structure that answers them

| The question | The structure |
|---|---|
| are two substrings equal | hashing, giving a constant-time comparison |
| find every occurrence of one pattern in a text | the prefix function, or the Z-function |
| find every occurrence of many patterns in a text | Aho-Corasick |
| the longest border, or the period, of every prefix | the prefix function |
| the longest common prefix of two suffixes | the Z-function, or a suffix array with an accompanying longest-common-prefix array |
| count distinct substrings | a suffix automaton, or a suffix array, or hashing |
| the longest palindromic substring, or every palindrome | Manacher's algorithm |
| the longest substring that repeats | binary search combined with hashing, or a suffix automaton |
| the lexicographically smallest rotation | Booth's algorithm |
| whether a string has a given period | the prefix function, checking whether the string's length minus the value of the prefix function at the end divides the length |
| matching with wildcards, or counting matches at every position | hashing, or occasionally a Fourier-transform-based approach for particularly awkward cases |

---

## Part 1 · Hashing, done so that it stays correct

Polynomial hashing turns comparing two substrings into comparing two integers, computed in constant time each after a linear preprocessing pass.

$$H(s) = \sum_{i=0}^{n-1} s_i \cdot p^{\,n-1-i} \pmod M$$

```cpp
const long long M1 = 1000000007, M2 = 998244353;
long long p1 = 131, p2 = 137;
vector<long long> h1, h2, pw1, pw2;

void build(const string& s) {
    int n = s.size();
    h1.assign(n+1, 0); h2.assign(n+1, 0);
    pw1.assign(n+1, 1); pw2.assign(n+1, 1);
    for (int i = 0; i < n; i++) {
        h1[i+1] = (h1[i] * p1 + s[i]) % M1;
        h2[i+1] = (h2[i] * p2 + s[i]) % M2;
        pw1[i+1] = pw1[i] * p1 % M1;
        pw2[i+1] = pw2[i] * p2 % M2;
    }
}
pair<long long,long long> get(int l, int r) {          // covers [l, r)
    long long a = (h1[r] - h1[l] * pw1[r-l]) % M1; if (a < 0) a += M1;
    long long b = (h2[r] - h2[l] * pw2[r-l]) % M2; if (b < 0) b += M2;
    return {a, b};
}
```

Several details in this template are worth understanding rather than treating as boilerplate.

**Two independent moduli are used together.** A single modulus of ordinary size collides with a probability that grows with the square of the number of comparisons made, by the birthday bound, and with a hundred thousand comparisons that probability is uncomfortably high for a single modulus; combining two independent hashes multiplies the two collision probabilities together, making a coincidental collision on both simultaneously extremely unlikely.

**On Codeforces specifically, the base should be randomised at runtime** rather than fixed to a constant chosen in advance, since fixed, well-known bases are specifically targeted by adversarial test data designed to force collisions.

**Subtraction can go negative and needs a correction.** The expression `h[r] - h[l] * pw[r-l]` is frequently negative before taking the modulus, and adding the modulus back when the result is negative is necessary every time this pattern appears.

**Ranges are treated as half-open**, matching the convention used throughout this guide.

Hashing is usually the shortest path to a correct solution for any problem centred on comparing substrings, and it combines naturally with binary search. LC 1044 Longest Duplicate Substring is solved by binary searching on the candidate length and, for each candidate length, hashing every window of that length into a set to check for a repeat; this is around twenty lines, considerably shorter than a suffix-automaton-based solution to the same problem.

---

## Part 2 · The prefix function

The prefix function of a string, usually written `π`, gives at each position the length of the longest proper prefix of the string up to that position which is also a suffix of it.

```cpp
vector<int> prefixFunction(const string& s) {
    int n = s.size();
    vector<int> pi(n, 0);
    for (int i = 1; i < n; i++) {
        int j = pi[i-1];
        while (j > 0 && s[i] != s[j]) j = pi[j-1];
        if (s[i] == s[j]) j++;
        pi[i] = j;
    }
    return pi;
}
```

The inner loop reads as: having previously found a border of length `j`, if the next character does not extend it, fall back to the longest border *of that border*, and try again. Following this fallback chain, from `π[j-1]` to `π[π[j-1]-1]` and so on down to zero, enumerates every border of the current prefix, and this chain is the origin of most applications of the prefix function.

**Pattern matching** is done by running the prefix function over the pattern, a separator character guaranteed not to occur in either string, and then the text, concatenated together; any position in the combined string where the prefix function equals the length of the pattern marks a match.

**Three derived facts are worth knowing, since they are what problems actually ask for.** The set of all borders of a string is obtained by following the fallback chain from `π[n-1]` down to zero, which answers CSES *Finding Borders* directly. The smallest period of a string is `n - π[n-1]`, and the string is an exact repetition of that period precisely when that value divides `n`, which answers CSES *Finding Periods* and LC 459. Counting how many times each prefix occurs as a substring is done by initialising a count of one at every position, then processing positions from the end of the string backwards to the start, adding each position's count into the count at `π[position] - 1`, which is exactly what CF 432D asks for.

**The prefix function also produces an automaton.** Precomputing, for every combination of position and character, the state reached by extending the matched prefix with that character gives a table that turns pattern matching into a single loop with no backtracking at all, and this table is exactly what makes the digit-dynamic-programming-style problems in chapter [[16 Digit DP and Expectation]] work when the constraint being tracked is "avoid this pattern".

---

## Part 3 · The Z-function

The Z-function gives, at each position, the length of the longest common prefix between the whole string and the suffix starting at that position.

```cpp
vector<int> zFunction(const string& s) {
    int n = s.size();
    vector<int> z(n, 0); z[0] = n;
    for (int i = 1, l = 0, r = 0; i < n; i++) {
        if (i < r) z[i] = min(r - i, z[i - l]);
        while (i + z[i] < n && s[z[i]] == s[i + z[i]]) z[i]++;
        if (i + z[i] > r) { l = i; r = i + z[i]; }
    }
    return z;
}
```

The Z-function and the prefix function are interchangeable for straightforward pattern matching, and the choice between them mostly comes down to which one's output is more directly useful for a given question: the Z-function is generally easier to *use* because its values answer prefix-matching questions directly at any position, while the prefix function is generally easier to *extend*, since its fallback chain gives borders, periods and an automaton in a way the Z-function does not as naturally.

LC 214 Shortest Palindrome is a clean demonstration. Computing the prefix function, or equivalently the Z-function, of the pattern formed by the string, a separator, and the reversed string, gives at its final position the length of the longest palindromic *prefix* of the original string, and prepending the reverse of everything after that prefix produces the answer in three lines once this is seen.

---

## Part 4 · Manacher's algorithm

Manacher's algorithm finds, for every possible centre of a palindrome, the radius of the longest palindrome centred there, in linear time. Handling even-length palindromes requires inserting separator characters between every pair of original characters, turning a string such as `abc` into `^#a#b#c#$`.

```cpp
string t = "^";
for (char c : s) { t += '#'; t += c; }
t += "#$";
int n = t.size();
vector<int> p(n, 0);
for (int i = 1, c = 0, r = 0; i < n - 1; i++) {
    if (i < r) p[i] = min(r - i, p[2*c - i]);
    while (t[i + p[i] + 1] == t[i - p[i] - 1]) p[i]++;
    if (i + p[i] > r) { c = i; r = i + p[i]; }
}
```

**A candid recommendation**: Manacher's algorithm is worth having available in a personal template file but is not worth deriving from memory under time pressure, since **hashing combined with a binary search on the radius at each centre gives the same information in a factor of a logarithm more time and is considerably easier to reproduce correctly.** Reaching for the hashing-based approach by default and only using Manacher's algorithm when that extra logarithmic factor is genuinely too slow is a reasonable working policy.

---

## Part 5 · Tries and Aho-Corasick

**A trie** is a tree in which each edge corresponds to a character and every root-to-node path spells out a prefix present in the inserted collection of strings. On its own this is a simple structure, but it underlies several more elaborate techniques.

LC 208 is the direct template. LC 212 Word Search II uses a trie for pruning rather than for lookup: searching a grid depth-first while simultaneously walking the trie means a branch of the search can be abandoned the moment the trie shows no word could possibly continue along the letters seen so far, which is what makes the search fast enough despite trying to match many words at once. CSES *Word Combinations* combines a trie with a dynamic program, where the value at position `i` sums the values at every position `j` such that the string between `j` and `i` is a dictionary word, found efficiently by walking the trie backwards from `i`.

**Aho-Corasick** extends a trie with suffix links, producing a finite automaton that matches every inserted pattern simultaneously in one pass over the text.

The suffix link of a node representing some string `u` points to the node representing the longest proper suffix of `u` that also appears somewhere in the trie. This is exactly the prefix function generalised from a single string to an entire trie of strings, and it is built with a breadth-first pass:

```cpp
queue<int> q;
for (int c = 0; c < 26; c++) {
    if (nxt[0][c]) { link[nxt[0][c]] = 0; q.push(nxt[0][c]); }
    else nxt[0][c] = 0;
}
while (!q.empty()) {
    int v = q.front(); q.pop();
    terminal[v] |= terminal[link[v]];                 // inherit matches along the suffix link
    for (int c = 0; c < 26; c++) {
        int u = nxt[v][c];
        if (u) { link[u] = nxt[link[v]][c]; q.push(u); }
        else nxt[v][c] = nxt[link[v]][c];             // fills in the automaton directly
    }
}
```

**The final line inside the loop is the key move.** Rather than following suffix links repeatedly at the time a query is answered, the full transition for every combination of state and character is precomputed once, so matching afterwards becomes a single loop advancing through one precomputed table with no backtracking.

LC 1032 Stream of Characters is a direct application. LC 3213 combines the automaton with a dynamic program, walking it over a target string and, at each position, using the terminal information inherited along suffix links to find every pattern ending there, relaxing a dynamic programming value accordingly, which connects this chapter with chapter [[14 Dynamic Programming Core]].

---

## Part 6 · The suffix automaton, and an honest assessment of its worth

The suffix automaton of a string is the smallest possible finite automaton that accepts exactly the suffixes of that string, has a number of states proportional to the string's length, and can be built incrementally, one character at a time, in linear total time.

**What it provides cheaply, once built.** The number of distinct substrings of the whole string, computed as a single sum over its states. The number of times a given substring occurs, obtained by propagating counts upward through the tree formed by the automaton's suffix links. The k-th lexicographically smallest substring, and the longest common substring between two different strings.

**An honest assessment for assessment purposes.** A suffix automaton is around forty lines of genuinely fiddly code, and for the overwhelming majority of problems that appear in this kind of assessment, hashing combined with binary search, a suffix array, or even a plain trie, answers the same question with far less implementation risk. The specific CSES problems where a suffix automaton earns its keep, such as *String Functions*, *Substring Order I*, *Substring Distribution* and *Counting Patterns*, are exactly that: CSES problems, aimed at completeness of the CSES problem set rather than at the kind of question a hiring assessment tends to ask.

**The practical recommendation** is to make sure everything else in this chapter is fluent first, and to treat the suffix automaton as worthwhile mainly if the goal is completing the CSES set in full or competing on Codeforces at a higher level, rather than as core preparation for an assessment.

---

## The ideas worth carrying forward

1. **Hashing turns substring equality into a constant-time comparison**, and doing it safely means two independent moduli, a randomised base on Codeforces, and careful handling of negative results after subtraction.

2. **Binary search combined with hashing solves "longest repeated substring"-style problems in a short amount of code**, and this combination is the single highest-value pairing in this chapter for assessment purposes.

3. **The prefix function's fallback chain enumerates every border of a prefix.** Most applications of the prefix function are really applications of walking that chain.

4. **The smallest period of a string is its length minus the final value of its prefix function, and the string is an exact repetition of that period precisely when the period divides the length.**

5. **Pattern matching via the prefix function concatenates the pattern, a guaranteed-absent separator, and the text.**

6. **Concatenating a string, a separator, and its own reverse gives the longest palindromic prefix at the final position of the resulting prefix function.**

7. **Aho-Corasick's suffix link is the prefix function generalised from a single string to a trie**, built the same way, with a breadth-first pass in place of a simple loop.

8. **Precomputing the automaton's transitions for every state and character removes the need to follow suffix links during matching**, turning the matching pass into a single loop.

9. **Terminal flags must be inherited along suffix links**, or patterns that are themselves suffixes of other patterns will be missed entirely.

10. **Manacher's algorithm is faster than hashing with binary search by one logarithmic factor, and considerably harder to reproduce correctly.** Reaching for hashing by default is a reasonable policy.

11. **Per-character prefix counts**, giving how many times each character appears in a prefix, answer range-count questions per character in constant time and are worth remembering even though they are simple.

12. **A trie's real value in search problems is pruning a search early**, not merely fast lookup.

---

## Where people lose these problems

**Using a single modulus on Codeforces.** Adversarial test data specifically targets common single-modulus setups.

**Using a fixed, well-known base on Codeforces.** Randomising it at runtime avoids this.

**Forgetting to correct a negative hash value after subtraction.**

**Overflow in the product `h[l] * pw[r-l]`.** Both factors can approach the size of the modulus, so the product can approach the square of that, which fits into a 64-bit integer only when the modulus itself is kept to a moderate size; a modulus much larger than around a billion requires extended-precision arithmetic for this multiplication.

**Choosing a separator character that can actually appear in the input.** The pattern-plus-separator-plus-text trick for the prefix function relies entirely on the separator never occurring naturally, and this needs to be verified rather than assumed.

**Off-by-one errors mapping Manacher's output back to positions in the original string.** Writing the mapping down as a comment once, and trusting it thereafter, avoids re-deriving it incorrectly under pressure.

**Forgetting to inherit terminal flags along suffix links in Aho-Corasick.** Patterns that are suffixes of other inserted patterns are then silently missed.

**In LC 336, over-engineering the solution.** For each word and each way of splitting it into two halves, checking whether one half is a palindrome and whether the reverse of the other half exists in the dictionary, using a plain hash map rather than a trie, solves this directly; the traps are handling the empty string, which pairs with any palindrome, and counting each ordered pair exactly once.

**In LC 3008, mishandling the window between two sets of positions.** After finding every occurrence of both patterns with the prefix function or the Z-function, a two-pointer scan or binary search over the two sorted lists of positions is needed, and the distance constraint between the two occurrences is where the careful handling belongs.

---

## Working through the problem list

### Block 1 · Hashing and the prefix function

**Warm-up:**

- **CSES String Matching** — *count occurrences of one string within another.* Solve it with the prefix function, then again with hashing, and compare how long each takes to write.
- **LC 28 Find the Index of the First Occurrence in a String** — *find the first occurrence of a pattern.* The sheet specifically asks for an actual implementation of the prefix function or a similar matching algorithm here, rather than a built-in search function.
- **CSES Finding Borders** — *list every border of a string.* The fallback chain.
- **CSES Finding Periods** — *list every period of every prefix.* The period formula, applied throughout.
- **LC 459 Repeated Substring Pattern** — *decide whether a string is built from repeating a shorter one.* The period test, in three lines once known.

**Core:**

- **LC 214 Shortest Palindrome** — *find the shortest palindrome formable by adding characters to the front.* The string-separator-reverse trick.
- **LC 1044 Longest Duplicate Substring** — *find the longest substring that occurs more than once.* Binary search combined with hashing. **The single most valuable problem in this chapter for assessment purposes.** Worth doing carefully and keeping the resulting template.
- **CSES Repeating Substring** — *the same technique, in CSES form.*
- **LC 1147 Longest Chunked Palindrome Decomposition** — *split a string into the most pieces forming a palindrome when read together.* A greedy approach combined with hashing to check equality quickly.
- **CSES Longest Palindrome** — *the longest palindromic substring of a string.* Manacher's algorithm, or hashing combined with a binary search per candidate centre.
- **CSES Palindrome Queries** — *answer palindrome queries on a string that is also being updated.* Two Fenwick-tree-backed hashes, one over the string read forwards and one read backwards, supporting point updates. This combines chapter [[09 Fenwick Offline and Mos]] with the present one, and building a hash that tolerates updates is a genuinely useful idea worth the extra time.
- **CSES Minimal Rotation** — *find the lexicographically smallest rotation of a string.* Booth's algorithm, or the simpler approach of concatenating the string with itself and comparing candidate starting points with a two-pointer scan, which is worth learning first.
- **CSES Distinct Substrings** — *count distinct substrings of a string.* A suffix automaton gives this in one line, a suffix array with a longest-common-prefix array gives it with a short formula, and hashing every substring length separately with binary search also works. Any one of these is a reasonable choice.
- **CF 471D MUH and Cube Walls** — *match a silhouette of building heights against a wall profile.* The reframing, where differences between adjacent heights convert "match a shape" into "match a pattern of relative changes", is what turns this into an ordinary prefix-function matching problem, and is an excellent example of transforming an input so a standard tool becomes applicable.
- **CF 432D Prefixes and Suffixes** — *count how many times each prefix of a string occurs as a substring, restricted to those prefixes which are also suffixes.* The border chain combined with the occurrence-counting technique.
- **CF 526D Om Nom and Necklace** — *count prefixes satisfying a repetition condition involving a ratio of two integers.* The prefix function combined with a divisibility argument for each prefix. A demanding problem.
- **CF 985F Isomorphic Strings** — *decide whether two substrings are isomorphic to each other.* Hash the set of positions at which each individual character occurs within the substring, separately per character, then compare the multiset of these twenty-six position-hashes between the two substrings. A useful reminder that hashing can be applied to a derived structure rather than only to the raw string.
- **LC 3008 Find Beautiful Indices in the Given Array II** — *find positions satisfying a proximity condition between occurrences of two patterns.* Two applications of the prefix function or Z-function, followed by a two-pointer scan.

### Block 2 · Tries, Aho-Corasick, and the suffix automaton

- **LC 208 Implement Trie** — *the trie template.*
- **LC 212 Word Search II** — *find every word from a dictionary present in a grid.* A trie used for pruning.
- **CSES Word Combinations** — *count the ways to build a string by concatenating dictionary words.* A trie combined with a dynamic program.
- **LC 336 Palindrome Pairs** — *find every pair of words whose concatenation is a palindrome.* A hash map together with a check at every split point, as described above.
- **LC 1032 Stream of Characters** — *detect, as characters arrive one at a time, whether any suffix of what has been seen matches a dictionary word.* Aho-Corasick, directly. Build a reusable template here.
- **CF 271D Good Substrings** — *count substrings satisfying a per-character goodness condition, within a length bound that keeps the total number of relevant substrings manageable.* A trie of substrings, once the bound is noticed.
- **CF 963D Frequency of String** — *find the shortest substring in which a given pattern occurs at least a given number of times.* Aho-Corasick combined with the occurrence positions of each pattern and a sliding window over them, combining this chapter with chapter [[05 Sliding Window and Two Pointers]].
- **LC 3213 Construct String with Minimum Cost** — *build a target string from a set of available words at minimum cost.* Aho-Corasick combined with a dynamic program, and the best demonstration in this chapter of two techniques genuinely working together.
- **AC ACL Practice I · Number of Substrings** — *count distinct substrings.* A suffix array with a longest-common-prefix array, or a suffix automaton, serving as the entry point into the more advanced suffix-structure problems.
- **CSES String Functions** — *print the prefix function and Z-function of a string.* Direct once both are implemented.
- **CSES Counting Patterns**, **CSES Pattern Positions**, **CSES Substring Order I**, **CSES Substring Distribution** — *the suffix automaton and suffix array problems.* **Genuinely optional for assessment purposes.** Worth doing if the goal also includes completing the CSES problem set or competing at a higher Codeforces rating.

---

**A reasonable target for the first block is around 75% of submissions passing first time, and the second block is best treated as opportunistic rather than mandatory.**

The realistic payoff in this chapter is concentrated in four techniques: hashing on its own, hashing combined with binary search, the prefix function or Z-function for pattern matching, and Aho-Corasick for matching many patterns at once. Together these four cover the large majority of what genuinely appears in assessments, and everything past them in Block 2 is worth pursuing according to separate goals rather than assessment preparation specifically.

---

## Check yourself

1. Write the polynomial hash and the formula for extracting a substring's hash. Name the three safety measures described above.
2. What does the prefix function's fallback chain enumerate?
3. Give the formula for a string's smallest period, and the condition under which the string is an exact repetition of it.
4. How is the prefix function used for pattern matching, and what property must the separator character have?
5. What does the prefix function of the pattern formed by a string, a separator, and its own reverse compute? Which problem does this solve?
6. What is Aho-Corasick's suffix link a generalisation of?
7. Why is the automaton's transition table precomputed for every combination of state and character, rather than following suffix links at query time?
8. What goes wrong if terminal flags are not inherited along suffix links?
9. When would Manacher's algorithm be preferred over hashing combined with binary search?
10. In CF 471D, what transformation turns the problem into an ordinary pattern-matching problem?
11. Which four string techniques give the strongest return for assessment purposes, and which techniques in this chapter are better thought of as completeness goals?
