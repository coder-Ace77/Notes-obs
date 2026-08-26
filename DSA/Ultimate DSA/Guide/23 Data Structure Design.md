---
tags: [dsa, guide, design, lru, lfu, data-structures]
chapter: 23
sheet-section: W
---

# Chapter 23 · Data-Structure Design Under Constraints

> **Thesis:** Design problems are graded on whether *every* operation actually hits the stated bound. The design move is almost always the same: **combine a hash map with an ordered structure, so the map gives you `O(1)` location and the structure gives you `O(1)` reordering.**

Back to [[00 Guide Index]] · Sheet section **W** in [[1. Ultime DSA 2026 calibration]]

---

## Where it hides

Unmistakable. "Design a class supporting the following operations, each in `O(1)` average time." The "design round" is standard at product companies and is often the *easiest* hard on an OA — if you've built the combinations before.

The pattern behind almost all of them:

> **A hash map from key → *iterator/node* in a second structure.**

The hash map answers "where is this thing?" in `O(1)`. The second structure answers "what's the order?" and supports `O(1)` splicing. Neither can do both alone. That's the whole idea, and once you've seen it three times you'll never miss it.

---

## Ground floor

### The composition table

| You need | Combine |
|---|---|
| `O(1)` lookup + `O(1)` reordering | `unordered_map<K, list<...>::iterator>` + `std::list` |
| `O(1)` lookup + `O(1)` random removal | `unordered_map<K, int>` (index) + `vector` (swap with last, pop) |
| `O(1)` lookup + ordered iteration | `unordered_map` + a balanced BST or a bucketed list |
| max/min with `O(1)` push/pop | a stack of `(value, runningExtreme)` pairs |
| `k`-th element with updates | BIT with descend (chapter [[09 Fenwick Offline and Mos]]) or a skip list |
| "most recent `n` items per key" | `unordered_map<K, deque>` |
| merge sorted streams | a heap of iterators |

**The `vector` + index-map trick deserves its own note** because it's the answer to "insert, delete and getRandom, all `O(1)`": store elements in a `vector` and their positions in a map; to delete, swap the target with the last element, update the map for the moved element, and `pop_back`. Random access is then just `v[rand() % v.size()]`. Elegant, and it appears constantly.

### LC 146 LRU Cache

The foundational one.

```
unordered_map<int, list<pair<int,int>>::iterator> pos;   // key → node
list<pair<int,int>> lru;                                 // front = most recent
```

- `get(k)`: look up in `pos`; if present, `lru.splice(lru.begin(), lru, pos[k])` — **`splice` moves a node in `O(1)` without invalidating the iterator.** Return the value.
- `put(k, v)`: if present, update and splice to front. Else insert at front, record the iterator, and if over capacity, erase `lru.back()` and its map entry.

**The critical property: `std::list` iterators remain valid across insertions and splices.** That's why the map can store them. A `vector` would invalidate them on every reallocation, which is exactly why you can't use one here.

*(In Java: `LinkedHashMap` with `accessOrder = true` does this for you, and overriding `removeEldestEntry` gives you eviction in one line. Know this — it's a legitimate answer in an interview and a two-minute answer in an OA.)*

### LC 460 LFU Cache

The genuinely hard one, and the reason is worth stating: you need **`O(1)` on a two-level ordering** — first by frequency, then by recency within a frequency.

The structure:

```
unordered_map<int, Node>                  key → (value, freq, iterator into its bucket)
unordered_map<int, list<int>> buckets;    freq → list of keys with that freq, MRU at front
int minFreq;                              the smallest frequency currently present
```

- **On access:** remove the key from `buckets[f]`, append to the front of `buckets[f+1]`, increment its frequency. **If `buckets[f]` is now empty and `f == minFreq`, increment `minFreq`.**
- **On eviction:** remove `buckets[minFreq].back()` — the least recently used among the least frequently used.
- **On insert:** set `minFreq = 1` and add to `buckets[1]`.

**The `minFreq` maintenance is the whole problem.** Two facts make it `O(1)`:

1. `minFreq` only ever increases by exactly 1 when a bucket empties — because the key that emptied it moved to `f+1`, so `f+1` is now non-empty.
2. On any insertion, `minFreq` resets to 1.

Convince yourself of fact (1) explicitly. It's the non-obvious step and it's what makes the whole thing constant time rather than "scan for the minimum frequency."

### LC 895 Maximum Frequency Stack

`push(x)`, and `pop()` returns the most frequent element, breaking ties by most recent.

**The trick is a stack per frequency.**

```
unordered_map<int,int> freq;              // value → its current frequency
unordered_map<int, vector<int>> group;    // frequency → stack of values that reached it
int maxFreq;
```

- `push(x)`: `f = ++freq[x]; maxFreq = max(maxFreq, f); group[f].push_back(x);`
- `pop()`: take `group[maxFreq].back()`, pop it, decrement its `freq`; if `group[maxFreq]` is now empty, `maxFreq--`.

**Why this works, and it's genuinely lovely:** when `x` is pushed and reaches frequency `f`, it's appended to `group[f]` — but its earlier copies are still sitting in `group[1..f-1]`. So when `x` is popped from `group[f]`, the copy in `group[f-1]` is still there and correctly represents "`x` at frequency `f−1`". **The value is stored once per frequency level it has attained**, and the stacks naturally give you recency tie-breaking for free.

That "store the element once per level it reaches" idea is the aha, and it's not something you'd invent quickly under pressure — which is exactly why you do this problem in practice.

### LC 355 Design Twitter

`postTweet`, `follow`, `unfollow`, `getNewsFeed` (the 10 most recent tweets from the user and everyone they follow).

- `unordered_map<int, vector<pair<int,int>>> tweets;` — user → list of `(timestamp, tweetId)`.
- `unordered_map<int, unordered_set<int>> following;`
- `getNewsFeed`: a **`k`-way merge** — push the last tweet of each followed user into a max-heap keyed by timestamp, pop 10 times, and after each pop push that user's previous tweet.

`O(f + 10 log f)` where `f` is the follow count. The lesson: **"top `k` across many sorted lists" is always a heap of list-cursors**, never a full merge.

### LC 1206 Design Skiplist

A skip list is a probabilistic balanced structure: a sorted linked list with multiple levels, where each node appears at level `i+1` with probability 1/2. Search descends from the top level, moving right while possible.

`O(log n)` expected for search, insert and erase, and it's genuinely simpler than a red-black tree.

```
Node { int val; vector<Node*> next; };     // next[i] = successor at level i

search(target):
    node = head
    for level from top down to 0:
        while node->next[level] && node->next[level]->val < target:
            node = node->next[level]
    node = node->next[0]
    return node && node->val == target
```

**Why it's worth doing once:** it's the clearest possible illustration that *randomisation can replace rebalancing logic*. Skip lists, treaps and randomised BSTs all share this idea. And when someone asks you "how would you implement an ordered map", "skip list" is a much better answer than "I'd use `std::map`".

**Practical alternative in an OA:** if you're allowed, `std::multiset` or Java's `TreeSet` does everything the skip list does. Do the problem for the understanding, not because you'll need to write one.

---

## Aha moments

1. **Hash map for location, second structure for order.** The universal design move.

2. **`std::list` iterators survive insertion and splicing** — that's what makes the LRU map-of-iterators legal. `vector` iterators do not.

3. **`list::splice` moves a node in `O(1)`** without touching its iterator.

4. **LFU's `minFreq` only ever increments by 1, or resets to 1 on insert.** That's the whole `O(1)` argument.

5. **Freq Stack: store the value once per frequency level it attains**, and the per-level stacks give you recency for free.

6. **"Top `k` across many sorted lists" = a heap of cursors**, not a merged list.

7. **`vector` + index map = `O(1)` insert, delete and random access.** Swap-with-last and pop.

8. **Min/max stack = a stack of `(value, runningExtreme)` pairs.** The extreme at every prefix is stored inline.

9. **Randomisation replaces rebalancing.** Skip lists, treaps.

10. **In an OA, use the language's ordered container if allowed.** `LinkedHashMap` solves LRU in one line. Know the built-ins.

---

## Failure modes

**An operation that isn't actually `O(1)`.** Symptom: TLE on the large test. Go through *every* method and state its complexity out loud. The one that's secretly `O(n)` is usually "find the minimum" or "remove from the middle."

**Iterator invalidation.** Storing `vector` iterators or indices into a structure that shifts.

**Capacity zero.** LRU/LFU with `capacity == 0` must accept nothing. It's a real test case.

**Forgetting to remove the map entry on eviction.** Memory grows and stale lookups succeed.

**LFU `minFreq` not updated on insert.** The most common LFU bug.

**Freq Stack decrementing `maxFreq` too eagerly.** Only decrement when the bucket is empty, and only by one.

**Not handling `get`/`update` of an existing key in `put`.** LRU's `put` on an existing key must update *and* refresh recency, not insert a duplicate.

**Timestamps colliding in Design Twitter.** Use a single global counter, never a wall clock.

---

## Running the list

Five problems. Do all five; this is a small, high-yield block and each one teaches a distinct composition.

- **LC 146 LRU Cache** — map + list + splice. **The foundational one.** Also learn the `LinkedHashMap` one-liner if you write Java.
- **LC 895 Maximum Frequency Stack** — stack per frequency. Do this second; the insight is the most surprising in the block.
- **LC 460 LFU Cache** — two-level ordering with `minFreq`. The hardest. Do it after 146 and 895.
- **LC 355 Design Twitter** — heap of cursors for a `k`-way merge.
- **LC 1206 Design Skiplist** — randomised balance.

**Worth adding if you have an extra hour** (not on the sheet, but the same family and commonly asked): LC 380 Insert Delete GetRandom O(1), LC 155 Min Stack, LC 641 Design Circular Deque, LC 981 Time Based Key-Value Store.

**Target:** **90%+.** This is the highest-accuracy block on the entire sheet and it should be — the operations are fully specified, the test harness is deterministic, and there are no hidden algorithmic insights beyond the ones above. If you're missing here, it's an unhandled edge case (capacity 0, existing-key update, empty structure), and those are enumerable.

**Time budget:** one or two sessions. Then move on.

---

## Self-check

1. State the universal design move for `O(1)` cache problems.
2. Why does the LRU map store `list` iterators rather than indices?
3. What does `list::splice` do and why does it matter here?
4. Give the two facts that make LFU's `minFreq` maintainable in `O(1)`.
5. In Max Frequency Stack, why does popping from `group[maxFreq]` leave the correct state behind?
6. How do you get the 10 most recent items across `f` sorted lists, and at what cost?
7. How do you support insert, delete and getRandom all in `O(1)`?
8. How does a min-stack work?
9. Name three edge cases that break cache implementations.
10. What Java built-in solves LRU almost for free?
