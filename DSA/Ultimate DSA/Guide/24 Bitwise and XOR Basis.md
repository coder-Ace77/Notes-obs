---
tags: [dsa, guide, bitwise, xor-basis, trie, linear-algebra]
chapter: 24
sheet-section: X
---

# Chapter 24 · Bitwise Tricks, XOR Basis & Tries

> **Thesis:** XOR is addition in a vector space over GF(2). Once you believe that sentence, "the set of achievable XORs of a subset" stops being a search problem and becomes linear algebra — and several impossible-looking problems collapse into fifteen lines.

Back to [[00 Guide Index]] · Sheet section **X** in [[1. Ultime DSA 2026 calibration]]

---

## Where it hides

The sheet calls XOR basis "the single highest-leverage niche topic for 2026 hards", and that's right — it's rare enough that most candidates don't know it, and mechanical enough that knowing it is a decisive advantage.

The triggers:

| Statement | Tool |
|---|---|
| "maximum XOR of a subset" / "is `x` achievable as a subset XOR?" | **XOR basis** |
| "maximum XOR of a pair" / "max XOR with a query value" | **binary trie** |
| "count subsets whose XOR is 0" (or is `x`) | **XOR basis** + `2^{n−rank}` |
| "XOR of a subarray" | **prefix XOR** (chapter [[06 Prefix Sums and Difference Arrays]]) |
| "distinct values of AND/OR over all subarrays" | **log-many-distinct-values** |
| "maximise something bit by bit" | **greedy from the high bit down** |
| "count pairs with `a & b == 0`" | **SOS DP** (chapter [[15 Bitmask DP]]) |

---

## Ground floor

### The mental model: XOR is a vector space

Treat each number as a vector of bits over the field GF(2) (where `1 + 1 = 0`). Then:

- XOR **is** vector addition.
- A set of numbers **spans** a subspace — the set of all achievable subset-XORs.
- That subspace has a **basis** of at most 30 (or 60) vectors.
- Its size is exactly `2^rank`, where `rank` is the basis size.

Every fact below is a consequence of that model. If you internalise the model, you don't have to memorise the facts.

### The XOR basis (linear basis)

```cpp
struct Basis {
    long long b[62] = {};                     // b[i] has its highest set bit at position i
    int rank = 0;

    bool insert(long long x) {
        for (int i = 61; i >= 0; i--) {
            if (!((x >> i) & 1)) continue;
            if (!b[i]) { b[i] = x; rank++; return true; }   // new independent vector
            x ^= b[i];
        }
        return false;                          // x was already representable — dependent
    }
    bool canMake(long long x) {
        for (int i = 61; i >= 0; i--) {
            if (!((x >> i) & 1)) continue;
            if (!b[i]) return false;
            x ^= b[i];
        }
        return true;
    }
    long long maxXor() {
        long long res = 0;
        for (int i = 61; i >= 0; i--) res = max(res, res ^ b[i]);
        return res;
    }
};
```

**Understand `insert`:** it's Gaussian elimination. Walk from the highest bit down; if `x` has that bit set and no basis vector owns it, `x` becomes the owner. Otherwise XOR with the owner to clear the bit and continue. Either `x` gets reduced to 0 (it was already representable — dependent) or it finds a bit to own (independent).

**Understand `maxXor`:** greedy from the high bit down. At each step, XORing with `b[i]` either raises bit `i` of the result (if it was 0) or lowers it. Take it iff it increases the result. Because `b[i]` is the *only* basis vector with bit `i` as its highest, this greedy is safe — no later choice can undo it.

**The four facts that make this useful:**

1. **Number of achievable XOR values = `2^rank`.**
2. **Number of subsets producing any *given* achievable value = `2^{n − rank}`.** (The `n − rank` dependent elements can be toggled freely, each toggle compensated by adjusting the basis.) This is what solves "count subsets with XOR = 0".
3. **`canMake(x)`** tests membership in the span.
4. **Insertion is `O(60)`**, so building a basis for `n` numbers is `O(60n)`.

**CF 895C Square Subsets** is the payoff problem: count non-empty subsets whose product is a perfect square. A product is a perfect square iff every prime's exponent is even — and since values are ≤ 70, only 19 primes matter. Map each number to a 19-bit vector of exponent parities; the question becomes "how many subsets XOR to 0?" Answer: `2^{n − rank} − 1`.

**That translation — "product is a perfect square" → "XOR of parity vectors is 0" — is the entire problem**, and it exemplifies how XOR basis problems present: the XOR is never in the statement.

### The binary trie for maximum XOR

Different problem, different tool. **"Maximum XOR of a *pair*"** doesn't need a basis — it needs a trie.

Insert every number as a 30-bit path (most significant bit first). To find the maximum XOR with a query `x`, walk down greedily **always preferring the opposite bit** — because differing bits contribute to the XOR.

```cpp
struct Trie {
    vector<array<int,2>> ch{{{0,0}}};
    void insert(int x) {
        int cur = 0;
        for (int i = 30; i >= 0; i--) {
            int b = (x >> i) & 1;
            if (!ch[cur][b]) { ch.push_back({0,0}); ch[cur][b] = ch.size() - 1; }
            cur = ch[cur][b];
        }
    }
    int query(int x) {                              // max XOR of x with anything inserted
        int cur = 0, res = 0;
        for (int i = 30; i >= 0; i--) {
            int b = (x >> i) & 1;
            if (ch[cur][b ^ 1]) { res |= 1 << i; cur = ch[cur][b ^ 1]; }
            else cur = ch[cur][b];
        }
        return res;
    }
};
```

**LC 421 Maximum XOR of Two Numbers** is exactly this. *(There's also a neat prefix-mask + hash set solution: for each bit from high to low, mask everything to that prefix and check whether two prefixes XOR to the candidate answer. Worth knowing as a shorter alternative.)*

**LC 1707 Maximum XOR With an Element From Array** adds a constraint: the chosen element must be ≤ `m_i`. **Go offline** (chapter [[09 Fenwick Offline and Mos]]): sort the array and sort the queries by `m`; insert elements into the trie as the sweep passes them. Sort-and-sweep converts a constrained query into an unconstrained one.

**LC 1938 Maximum Genetic Difference Query** puts the trie **on a tree**: for each query `(node, val)`, find the max XOR of `val` with any ancestor of `node`. Do a DFS, inserting each node's value into the trie on entry and **removing it on exit** — so the trie always contains exactly the current root-to-node path. Answer queries at their node.

**Removal from a trie** just means maintaining a count per node and decrementing; a node is "present" iff its count is positive. That's a two-line change and it's what makes the DFS-with-a-trie pattern possible. **"Maintain a structure that always represents the current root-to-node path" is a general and reusable tree technique** (see also chapter [[13 Trees]]).

### Bit-by-bit greedy

**The pattern:** build the answer one bit at a time, from the most significant down. At each bit, tentatively assume it can be 1 and check feasibility; if yes, commit.

**Why it's correct:** setting a higher bit always beats any combination of lower bits, since `2^k > 2^{k-1} + … + 2^0`. So a greedy on bits never regrets.

This is the same argument as the lexicographic greedy in chapter [[03 Greedy and Exchange Arguments]] — *earliest position dominates* — and it powers `maxXor()`, the trie query, and several ad-hoc problems.

**LC 2317 Maximum XOR After Operations.** You may replace `nums[i]` with `nums[i] & (nums[i] ^ x)` for any `x`, which lets you clear **any subset of bits** of any element. So any bit that appears in *any* element can be made to appear in the XOR. Answer: the OR of everything. **One line**, and the reasoning is entirely "what can I control per bit?"

**LC 2680 Maximum OR.** You get `k` multiply-by-2 operations. Since OR is maximised by making one number as large as possible, **apply all `k` shifts to a single element** (spreading them is never better, because doubling a number you've already doubled adds a higher bit). Try each element, combining with prefix and suffix ORs. The prefix/suffix-OR technique is the "exclude one element from an aggregate" move from chapter [[13 Trees]].

### Log-many distinct values (AND / OR over subarrays)

Already introduced in chapter [[05 Sliding Window and Two Pointers]], and it belongs here too.

**The fact:** as you extend a subarray to the left, the running AND is non-increasing *as a bitset* — bits only turn off, never on. Each change turns off at least one bit permanently, so there are at most 30 distinct values of `AND(a[l..r])` over all `l`, for fixed `r`. Same for OR (bits only turn on).

**LC 898 Bitwise ORs of Subarrays.** Maintain a set of distinct ORs of subarrays ending at `r`:

```cpp
set<int> result;
vector<int> cur;                     // ≤ 30 entries
for (int x : nums) {
    vector<int> nxt = {x};
    for (int v : cur) nxt.push_back(v | x);
    sort(nxt.begin(), nxt.end());
    nxt.erase(unique(nxt.begin(), nxt.end()), nxt.end());
    cur = nxt;
    result.insert(cur.begin(), cur.end());
}
return result.size();
```

`O(n · 30 · log)`. **LC 2419** and **LC 1521** are the same family.

### CSES Hamming Distance

`n ≤ 10^4` strings of `k ≤ 30` bits; find the minimum pairwise Hamming distance. `O(n²)` pairs = `10^8`, each comparison `O(1)` via `__builtin_popcount(a ^ b)`. Just fast enough.

The lesson: **encoding data as integers and using popcount turns an `O(n²k)` brute force into `O(n²)`.** Word-level parallelism as a legitimate optimisation, same family as the bitset trick in chapter [[15 Bitmask DP]].

---

## Aha moments

1. **XOR is vector addition over GF(2).** Subset-XOR problems are linear algebra, not search.

2. **The basis has ≤ 30 (or 60) vectors, and building it is `O(60n)`.** Insert = Gaussian elimination.

3. **Achievable XOR values: `2^rank`. Subsets producing a given one: `2^{n−rank}`.** These two facts solve most counting problems in the topic.

4. **`maxXor` is greedy from the high bit down**, safe because each basis vector uniquely owns its highest bit.

5. **The XOR is never in the statement.** "Product is a perfect square" → parity vectors → XOR = 0. Look for the translation.

6. **Max XOR of a *pair* is a trie, not a basis.** Different questions, different tools.

7. **A trie with counts supports deletion**, which lets you maintain "the trie of the current root-to-node path" during a DFS.

8. **A constrained query becomes unconstrained by sorting and sweeping.** LC 1707.

9. **A higher bit beats every lower bit combined**, so bit-by-bit greedy from the top never regrets.

10. **Running AND/OR over an extending subarray changes `O(log V)` times.** Carry the small set of distinct values forward.

11. **`__builtin_popcount(a ^ b)` is Hamming distance in one instruction.**

12. **Ask "what can I control, per bit?"** LC 2317's answer is the OR of everything, for exactly that reason.

---

## Failure modes

**`int` overflow with 60-bit values.** Use `long long` and size the basis at 62.

**Trie depth mismatched to the value range.** Values up to `10^9` need 30 bits; up to `2^{31}` need 31. Off-by-one here silently loses the top bit.

**Empty subset in counting problems.** `2^{n−rank}` includes the empty subset (XOR 0). Subtract 1 if the problem wants non-empty.

**Forgetting that the basis stores *reduced* vectors.** `b[i]` may have lower bits set; that's fine and expected. Don't "clean" it unless you need the fully-reduced (row echelon) form for lexicographic queries.

**Assuming a basis answers pair-XOR questions.** The basis spans *subset* XORs; a pair is a very specific subset. Use a trie.

**Not deleting from the trie on DFS exit** in LC 1938. You'll answer queries against nodes that aren't ancestors.

**LC 2680 specifically:** all `k` shifts go to one element. Trying to distribute them is the intuitive wrong answer.

---

## Running the list

**Warm-up:**

- **CSES Hamming Distance** — encode as integers, popcount, `O(n²)`.
- **LC 421 Maximum XOR of Two Numbers in an Array** — the trie. Build your reusable template here.
- **LC 2317 Maximum XOR After Operations** — one line, once you ask "what can I control per bit?"

**Core:**

- **LC 2680 Maximum OR** — all shifts on one element, plus prefix/suffix ORs.
- **LC 898 Bitwise ORs of Subarrays** — log-many distinct values. **The most reusable technique in this chapter after the basis itself.**
- **LC 1707 Maximum XOR With an Element From Array** — offline trie. The sort-and-sweep conversion.
- **LC 1938 Maximum Genetic Difference Query** — trie on a tree, insert on DFS entry, delete on exit. **The best problem in this block** — it combines the trie, the offline idea, and the root-to-node-path technique.

**Boss:**

- **CF 895C Square Subsets** — the XOR basis, plus the parity-vector translation, plus `2^{n−rank} − 1`. This is the problem the whole chapter exists for.

**Also worth adding** (not on the sheet, but the same core): "maximum XOR subset" (basis + `maxXor`), and "given `q` queries, is `x` an achievable subset XOR?" (basis + `canMake`). Ten minutes each, and they cement the two remaining basis operations.

**Target:** **80%+.** Small block, high leverage. The realistic goal is: **have a `Basis` struct and a `Trie` struct in your template file**, and be able to recognise the two triggers ("subset XOR" versus "pair XOR") instantly. That recognition plus those two templates is worth more than most of section X's problem count.

---

## Self-check

1. Why is XOR vector addition, and what does that make the set of subset-XORs?
2. Write `insert` for a XOR basis and explain what happens in each branch.
3. How many distinct values can be made? How many subsets make a given one? Where does `2^{n−rank}` come from?
4. Why is the greedy in `maxXor` safe?
5. In CF 895C, what is the translation from "perfect square product" to a XOR problem?
6. When do you use a trie rather than a basis?
7. How do you support deletion from a binary trie, and what does that enable on a tree?
8. In LC 1707, what does sorting the queries buy you?
9. Why does a bit-by-bit greedy from the most significant bit never need to backtrack?
10. How many distinct values does `AND(a[l..r])` take as `l` varies for fixed `r`, and why?
11. In LC 2680, why do all `k` operations go to a single element?
