
---

#### What makes these problems hard

A segment tree works by deciding what each piece of the array stores so that the answers for two adjacent pieces can be combined into the answer for their union. Sums, maximums and greatest common divisors all combine this way. Some questions about a range do not. The number of distinct values in a range is the standard example. If the left half of a range holds 3 distinct values and the right half holds 4, the whole range holds anything from 4 to 7, because the same value can appear in both halves and the two numbers alone do not say how many are shared. No small summary stored per piece repairs this.

Mo's algorithm gives up on combining pieces. It starts from a different observation: if you already know the answer for one range, the answer for a nearby range is cheap to obtain, because adding or removing a single element at either end changes the answer by an amount that is easy to compute. The whole technique is a way of ordering the queries so that every query is close to the one before it.

Mo's algorithm applies when three things hold:

1. **All queries are known in advance.** The technique reorders them, so it cannot be used when each query depends on the answer to an earlier one or when queries must be answered as they arrive.
2. **The array does not change between queries.** The basic version assumes a fixed array. A later section extends it to updates.
3. **Adding one element to the range, or removing one element from the range, can be handled quickly**, ideally in constant time. This is where all the design work lies.

The difficulty in these problems is concentrated in three places. The first is recognising that the problem is a Mo's problem, since nothing in the statement mentions it. The second is writing the add and remove operations so that they are exact inverses of each other. The third is a handful of small implementation details, such as the order of the pointer movements, that do not crash but silently produce wrong answers.

## The idea: a window that moves

Suppose the array is `a` of length `n`, and each query is a pair `(l, r)` asking for something about the elements `a[l], a[l+1], ..., a[r]`. The direct solution scans the range for each query, which costs up to `n` per query and `n * q` in total. With `n` and `q` both equal to 100000 that is ten billion operations, which is too slow.

Mo's algorithm keeps a **window** `[L, R]` over the array, together with whatever information is needed to know the answer for the elements currently inside the window. To answer a query `(l, r)` it does not start from scratch. It moves the window one step at a time until `L = l` and `R = r`, and reads off the answer.

There are exactly four moves, and each one is a single element entering or leaving:

- `R` moves right by one: the element `a[R+1]` enters.
- `R` moves left by one: the element `a[R]` leaves.
- `L` moves left by one: the element `a[L-1]` enters.
- `L` moves right by one: the element `a[L]` leaves.

Take a concrete problem. Given an array and many queries `(l, r)`, report the number of distinct values in `a[l..r]`. The window keeps an array `cnt` where `cnt[v]` is the number of times `v` occurs inside the window, and a number `distinct`. When an element enters, increase its count, and if the count just became 1, then `distinct` goes up by one. When an element leaves, decrease its count, and if the count just became 0, then `distinct` goes down by one.

```cpp
void add(int i) { if (cnt[a[i]]++ == 0) distinct++; }
void del(int i) { if (--cnt[a[i]] == 0) distinct--; }
```

A small trace makes the mechanics concrete. Let `a = [1, 2, 1, 3]` and let the queries be `(0, 2)` followed by `(1, 3)`. The window starts empty, which is represented by `L = 0` and `R = -1`.

- For `(0, 2)`, `R` moves right three times. The elements 1, 2, 1 enter. The counts are `cnt[1] = 2` and `cnt[2] = 1`, so `distinct = 2`.
- For `(1, 3)`, `R` moves right once and the element 3 enters, so `distinct = 3` and the window is `[0, 3]`. Then `L` moves right once and the element `a[0] = 1` leaves. Its count drops from 2 to 1, which is not zero, so `distinct` stays 3. The answer is 3, which is correct since the range holds 2, 1, 3.

That second step is the reason the removal rule checks whether the count reached zero. Removing a value from the window does not remove it from the answer if another copy remains inside.

## Moving the window safely

The four moves are written as four loops, and their order matters. For a query (l,r). and current window `(L,R)`

```cpp
while (L > l) add(--L);     // grow to the left
while (R < r) add(++R);     // grow to the right
while (L < l) del(L++);     // shrink from the left
while (R > r) del(R--);     // shrink from the right
```

**Both loops that grow the window must come before both loops that shrink it.** To see why, suppose the window is `[5, 8]` and the next query is `(1, 3)`. If the right end shrinks first, `R` walks from 8 down to 3 while `L` is still 5. After removing `a[8], a[7], a[6], a[5]` the window is empty, and the next step removes `a[4]`, an element that was never inside. Its count becomes negative or its contribution is subtracted from an answer that never included it. Nothing crashes, and the later answers are all slightly wrong. If the left end grows first, `L` walks from 5 down to 1 and adds `a[4], a[3], a[2], a[1]`, giving the window `[1, 8]`. Now shrinking the right end removes `a[8]` down to `a[4]`, and every one of those elements is genuinely inside the window.

The rule behind the order is that **the window must never be invalid**, meaning `L` must never exceed `R + 1`. Growing first guarantees that the window only ever gets bigger on its way to covering both the old and the new range, and shrinking afterwards trims it down to the new range.

## Choosing the order of the queries

Moving the window is cheap per step, so the total cost is the total distance travelled by the two ends. That distance depends entirely on the order in which the queries are processed.

If the queries are processed in the order given, the cost can be as bad as the brute force. Imagine queries that alternate between `(0, 1)` and `(n-2, n-1)`. Every query moves both ends across nearly the whole array, so the total is about `n * q` again.

Sorting by `l` alone does not help, because the right ends of consecutive queries can still be anywhere. Sorting by `l` and then by `r` does not help either, since two queries with different `l` values and the same neighbourhood of `r` values force the right end to travel back and forth between them.

The ordering that works divides the array into **blocks** of size `B` and treats the left endpoint only coarsely:

1. Group the queries by the block that their left endpoint falls into, which is `l / B`.
2. Within one group, sort the queries by `r` in increasing order.

Now count the movement.

- **The left end.** Every query in a group has its `l` inside the same block of size `B`, so between two consecutive queries the left end moves at most `B`. Over all `q` queries that is at most `q * B`. Moving from one group to the next adds at most `2B` per group, which is negligible.
- **The right end.** Within one group the queries are sorted by `r`, so the right end only moves forwards and travels at most `n` in total. There are `n / B` groups, so the right end travels at most `n * n / B` overall, counting the walk back to the start of each group.

The total is therefore about `q * B + n * n / B`. The first term grows with `B` and the second shrinks with it, and the sum is smallest when they are equal, which gives

```
B = n / sqrt(q)        total movement ≈ 2 * n * sqrt(q)
```

For `n = q = 100000` this gives `B ≈ 316` and a total of roughly `6 * 10^7` steps, against `10^10` for the direct approach. That is the entire gain.

When `n` and `q` are about the same size, `B = sqrt(n)` is the usual shortcut and is nearly as good. When the two differ a lot, the formula above is the one to use.

**The odd-even refinement.** After finishing one group the right end sits at the largest `r` of that group and then has to walk all the way back to the small `r` values of the next group. This can be avoided by sorting the groups alternately: increasing `r` in even-numbered groups and decreasing `r` in odd-numbered groups. The right end then finishes one group near where the next group begins. This roughly halves the right end's travel and is worth including. It does not change the complexity, only the constant.

## The template

```cpp
struct Query { int l, r, idx; };

int n;
vector<int> a;
// state of the window and the function-specific add / del go here

vector<long long> solve(vector<Query>& qs) {
    if (qs.empty()) return {};
    int B = max(1, (int)(n / sqrt((double)qs.size())));

    sort(qs.begin(), qs.end(), [&](const Query& x, const Query& y) {
        int bx = x.l / B, by = y.l / B;
        if (bx != by) return bx < by;
        return (bx & 1) ? x.r > y.r : x.r < y.r;     // odd-even refinement
    });

    vector<long long> ans(qs.size());
    int L = 0, R = -1;                               // empty window
    for (auto& qu : qs) {
        while (L > qu.l) add(--L);
        while (R < qu.r) add(++R);
        while (L < qu.l) del(L++);
        while (R > qu.r) del(R--);
        ans[qu.idx] = current();                     // read the answer for the window
    }
    return ans;
}
```

Two details in this template are easy to get wrong. The answer is stored at `ans[qu.idx]`, the position the query had in the input, because the queries have been sorted and the output must be in the original order. And the state of the window, such as `cnt`, is created once and carried across all queries. It is never reset between queries, since the whole point is that each query starts from the previous one.

## Designing add and remove

Everything that differs between Mo's problems is the content of `add` and `del`. The method is always the same.

1. Keep `cnt[v]`, the number of times each value occurs inside the window.
2. Express the answer as a function of those counts, and keep a running value `cur` for it.
3. When one count changes from `c` to `c + 1`, work out how `cur` changes using only `c`. For removal, do the reverse.

**The rule that prevents most bugs is that `del` must undo `add` exactly, which means performing the same steps in the opposite order.** The four examples below follow this pattern, and in each the removal is the addition read backwards.

**Distinct values.** Covered above. `cur` is the number of values with a positive count.

**The sum over every value `v` of `v * cnt[v]^2`.** Suppose a query asks, for the range, for the sum over each value `v` occurring in it of `v` multiplied by the square of the number of times it occurs. When `cnt[v]` goes from `c` to `c + 1` the term changes from `v*c^2` to `v*(c+1)^2`, a difference of `v*(2c+1)`.

```cpp
void add(int i) { int v = a[i]; cur += (long long)v * (2LL * cnt[v] + 1); cnt[v]++; }
void del(int i) { int v = a[i]; cnt[v]--; cur -= (long long)v * (2LL * cnt[v] + 1); }
```

In `add` the count is read before it is increased. In `del` the count is decreased first and then read, so the amount subtracted is exactly the amount that `add` added when it went from the smaller count to the larger one. This ordering is what makes the two functions inverses. Swapping the order in `del` produces a first answer that is correct and later answers that drift.

**The number of pairs of equal elements.** Suppose a query asks how many pairs of positions `i < j` in the range have `a[i] = a[j]`. Adding an element `v` creates one new pair with each copy of `v` already inside.

```cpp
void add(int i) { cur += cnt[a[i]]; cnt[a[i]]++; }
void del(int i) { cnt[a[i]]--; cur -= cnt[a[i]]; }
```

**Subarrays with a given xor.** Suppose each query `(l, r)` asks how many subarrays inside `a[l..r]` have xor equal to `k`. This one needs a change of viewpoint before Mo's algorithm applies. Let `pre[0] = 0` and `pre[i] = a[1] xor ... xor a[i]`, using one-based positions. A subarray `a[x..y]` has xor `pre[x-1] xor pre[y]`, so it has xor `k` exactly when `pre[x-1] xor pre[y] = k`. The question therefore becomes: among the prefix values `pre[l-1], pre[l], ..., pre[r]`, count the pairs whose xor is `k`.

The window must now move over the **prefix array** `pre[0..n]`, and the range for a query `(l, r)` is `[l-1, r]`, shifted by one on the left from the range in the statement. Forgetting this shift is the usual mistake. Once the window is over `pre`, adding an element pairs it with every earlier element whose value is `pre[i] xor k`.

```cpp
void add(int i) { cur += cnt[pre[i] ^ k]; cnt[pre[i]]++; }
void del(int i) { cnt[pre[i]]--; cur -= cnt[pre[i] ^ k]; }
```

The array `cnt` needs to be indexed by every possible value of `pre[i] ^ k`, so it must be sized to the next power of two above the largest value that can occur, and not merely to the largest prefix value. When `k = 0` the two indices coincide, and the order inside `add` and `del` is what keeps an element from pairing with itself.

The general lesson is that the first step in a Mo's problem is often to translate the question into the index space that the window will move over. A problem about subarrays becomes a problem about pairs of prefix positions, and the window then moves over prefix positions.

**What cannot be done directly.** Maximum, minimum and similar quantities do not fit, because removing the current maximum requires knowing the next largest element, and the window does not remember it. There is a variant for this, described in the last section.

## Practical details

**Compress large values first.** If the values are up to a billion, `cnt` cannot be an array indexed by value. A map would work but adds a logarithmic factor to every one of the roughly `10^8` add and remove calls, which is too slow. Replace each value by its rank among the distinct values, so that `cnt` is a plain array. If the answer uses the original values, as in the sum of `v * cnt[v]^2`, keep a separate array that maps each rank back to its value.

**Keep add and del tiny.** They are called around `n * sqrt(q)` times, so even a modest cost per call multiplies into seconds. A logarithmic cost per call, such as inserting into a balanced tree, usually makes the solution too slow at `n = q = 10^5`. This is also the practical test for whether Mo's algorithm applies: if you cannot write add and remove in constant time, think again.

**Use 64-bit integers for the answer.** Counting pairs or summing squares in a range of size `10^5` exceeds the range of a 32-bit integer, and the overflow is silent.

**Watch the indexing.** Statements usually number positions from 1. The window code above uses 0-based positions with an inclusive right end, so subtract one from both `l` and `r` when reading the queries, and never mix the two conventions.

**Judge feasibility from the constraints.** With `n` and `q` around `10^5` and a time limit of two to four seconds, Mo's algorithm is comfortable. At `2 * 10^5` it is workable if add and remove are very light. At `10^6` it is not an option.

## When to choose Mo's algorithm

The signals are:

- All queries are given up front, as an array of ranges.
- The array is fixed, or changes only through a small number of point updates.
- The answer depends on **how many times values occur** in the range, or on pairs and triples of equal or related elements. Words like "distinct", "pairs of equal", "how many values occur exactly k times" and "sum of squares of counts" are typical.
- The answer for a range cannot be built by combining the answers for two halves.
- The constraints are around `10^5`, which fits the cost of `n * sqrt(q)`.

Before using it, check whether something simpler works. If the answer for a range can be combined from its two halves, a segment tree from chapter [[08 Segment Trees]] is better. If the queries can be sorted by one endpoint so that a structure only ever grows, an offline sweep costing a logarithmic factor beats Mo's algorithm, which costs a square-root factor. Mo's algorithm is the fallback for when neither applies, and it earns its place because the add and remove functions are usually shorter to get right than a clever decomposition.

## Variants

**Mo's algorithm with updates.** Suppose the queries are interleaved with point assignments of the form "set `a[pos]` to `x`". The array is no longer fixed, but the idea extends by adding a third pointer `T` that counts how many updates have been applied. A query is now a triple `(l, r, t)`, where `t` is the number of updates that happen before it. To answer it, the window moves as before, and the time pointer moves forwards or backwards over the update list until it equals `t`.

Applying an update at time `T` means changing `a[pos]` from its old value to its new value. If `pos` lies inside the current window `[L, R]`, the old value must be removed from the window state and the new value added. If `pos` lies outside the window, only the array entry changes. Undoing an update swaps the two values. For this, every update records both its old and its new value, which is filled in beforehand by replaying the updates once on a copy of the array.

```cpp
void applyUpdate(int k, int L, int R, bool forward) {
    int pos  = U[k].pos;
    int from = forward ? U[k].oldv : U[k].newv;
    int to   = forward ? U[k].newv : U[k].oldv;
    if (L <= pos && pos <= R) { delValue(from); addValue(to); }
    a[pos] = to;
}
```

The queries are sorted by `(l / B, r / B, t)`. Both `l` and `r` are now treated in blocks, and within a pair of blocks the time pointer sweeps forwards. The movement is about `q * B` for each of the two window ends plus `(n / B)^2 * U` for the time pointer, where `U` is the number of updates. Balancing these gives `B ≈ n^(2/3)` and a total of about `n^(5/3)`, which is around `2 * 10^8` for `n = 10^5` and needs light add and remove functions.

**Mo's algorithm on a tree.** Suppose each query gives two nodes `u` and `v` of a tree and asks about the nodes on the path between them. Run a depth-first search that records each node twice, once when it is entered at position `tin[u]` and once when it is left at position `tout[u]`, producing a sequence of length `2n`. Take `tin[u] <= tin[v]` after swapping if needed, and let `w` be the lowest common ancestor.

- If `w = u`, the path corresponds to the range `[tin[u], tin[v]]` of the sequence.
- Otherwise it corresponds to the range `[tout[u], tin[v]]`, and the node `w` must additionally be included by hand while answering this query.

In either range, a node on the path appears **exactly once**, while a node that is off the path and has both of its entries inside the range appears twice. So the window keeps a flag `inPath[node]` and, whenever a position of the sequence enters or leaves the window, **toggles** the node: if the flag is set, the node is removed from the state, and otherwise it is added. The rest of the algorithm, including the sorting, is unchanged, applied to a sequence of length `2n`.

```cpp
void toggle(int v) {
    if (inPath[v]) del(v); else add(v);
    inPath[v] = !inPath[v];
}
```

**When removal is impossible.** For a quantity such as the maximum in a range, an element cannot be taken out of the state. The fix is a version of the algorithm that only ever adds. The groups of queries are formed as before, and for each group the right end moves strictly forwards, adding elements as usual. For each individual query, the left end starts from the end of its block, moves left to `l` while adding elements, answers the query, and then **rolls back** every change made during that walk by restoring the saved values. Queries whose range lies entirely inside one block are answered by brute force. The state therefore needs a way to undo the last additions, which is straightforward for a maximum since the previous value can be saved before each change.

## Where people lose these problems

**Shrinking before growing.** The window briefly becomes invalid and subtracts elements that were never added. The answers are wrong with no obvious cause.

**An add and a del that are not exact inverses.** In the squared-count example, reading the count before decrementing in `del` makes the subtracted amount differ from the added amount. The first answer is correct and later ones drift.

**Resetting the state between queries.** This turns the algorithm back into the brute force, and often also corrupts the counts.

**Forgetting the prefix shift.** In problems about subarrays that are solved through prefix values, the window moves over the prefix array and the left end of each query is `l - 1` rather than `l`.

**Choosing the block size badly.** A constant such as 500 regardless of the input ruins the balance between the two terms. Use `n / sqrt(q)`, or `sqrt(n)` when the two sizes are similar, and make sure the block size is at least 1.

**Using a map for the counts.** The solution is correct and times out. Compress the values and use an array.

**Writing answers in sorted order.** The queries were reordered, so the answer must be stored at the query's original index.

**A 32-bit answer.** Counting pairs in a range of `10^5` elements gives values near `5 * 10^9`.

---

## Quick questions

1. What three conditions must hold for Mo's algorithm to apply?
2. Why can the number of distinct values in a range not be obtained by combining the answers for two halves?
3. Write the four pointer loops in the correct order and give a concrete example where a different order breaks.
4. Why does sorting the queries by `l` alone, or by `l` then `r`, fail to reduce the total movement?
5. Derive the cost `q * B + n * n / B` and the block size that minimises it.
6. What does the odd-even refinement change, and what does it leave unchanged?
7. Write `add` and `del` for the number of equal pairs in a range, and explain why `del` performs its steps in the opposite order.
8. In the xor-subarray problem, why does the window move over `pre[0..n]`, and what is the range for a query `(l, r)`?
9. Why is a map a bad choice for the counts, and what replaces it?
10. How does the time pointer extend the algorithm to updates, and what block size does that require?
11. In Mo's algorithm on a tree, why does a node on the path appear exactly once in the range of the sequence?
12. Why can a plain Mo's algorithm not maintain a range maximum, and what does the add-only variant do instead?
