---
tags: [dsa, guide, trees, lca, rerooting, tree-dp, centroid]
chapter: 13
sheet-section: M
---

# Chapter 13 · Trees: LCA, Rerooting, Small-to-Large, Tree DP

> **Read this before you start the problems.** Each technique is introduced with a small example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **M** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

A tree is a graph in which there is exactly one path between any two vertices, and that single fact is what every technique in this chapter exploits in a different way. Because trees are a familiar structure, the difficulty here is rarely about recognising that you are looking at one. It is about choosing which of several available machines applies, since a tree supports at least five distinct kinds of computation and using the wrong one produces a working but far too slow solution.

The distinction that causes the most trouble in practice is between a computation that needs one pass over the tree and one that needs two. Many tree problems ask for something about each vertex's own subtree, which a single depth-first search answers directly. A smaller number ask for something about the whole tree, evaluated separately as if each vertex in turn were the root, and answering that naively means running the single-pass computation once per vertex, which is far too slow. The technique that solves the second kind in one additional pass, called rerooting, is one of the least practised ideas relative to how often it is actually needed, and it is one of the two techniques this chapter spends the most time on. The other is a family of techniques for questions about the multiset of values inside a subtree, which is a different kind of aggregation from a simple sum or count and needs its own machinery.

---

## What these problems look like

| The question | The technique |
|---|---|
| distance between two vertices, the k-th ancestor of a vertex, whether one vertex is an ancestor of another | lowest common ancestor, using binary lifting or an Euler tour |
| a value computed for each vertex from its own subtree | an ordinary tree dynamic program, one depth-first search |
| a value computed for every vertex, each time as if that vertex were the root | rerooting, two depth-first searches |
| a question about the multiset of values inside each subtree | small-to-large merging, or the closely related technique usually called union-find on a tree despite not literally using a union-find structure |
| counting or optimising over paths, as opposed to subtrees | centroid decomposition, or occasionally a more direct dynamic program |
| an update to a single vertex, and a query over a path | heavy-light decomposition, or an Euler tour combined with a Fenwick tree when the aggregate can be undone |

---

## Part 1 · The Euler tour

This is the single idea in this chapter with the widest reach, and it is simpler than its name suggests.

Run a depth-first search over the tree, recording the moment each vertex is first entered and the moment its exploration finishes:

```cpp
int timer = 0;
void dfs(int u, int p) {
    tin[u] = timer++;
    for (int v : adj[u]) if (v != p) dfs(v, u);
    tout[u] = timer;
}
```

The consequence worth internalising is that **the subtree of any vertex occupies a single contiguous range in the order these entry times are assigned**, specifically the range from that vertex's entry time up to but not including its finish time. A second consequence follows immediately: one vertex is an ancestor of another exactly when the ancestor's entry time is at or before the descendant's, and the ancestor's finish time is after the descendant's, which is a constant-time check requiring no traversal at all.

Once a subtree is understood as a contiguous range, every array-based structure from chapters [[08 Segment Trees]] and [[09 Fenwick Offline and Mos]] becomes usable on a tree. A subtree sum with point updates becomes a Fenwick tree over the Euler-tour array, which is CSES *Subtree Queries*. A sum along the path from a vertex to the root, where entire subtrees are updated at once, becomes a Fenwick tree used as a difference array: adding a value to a vertex means adding it at that vertex's entry time and subtracting it at its finish time, after which the value at any vertex is the prefix sum up to its own entry time, which is CSES *Path Queries*.

---

## Part 2 · Finding the lowest common ancestor

There are three standard ways to answer lowest-common-ancestor queries, and it is worth knowing when each is the right choice.

**Binary lifting** precomputes, for every vertex and every power of two, the ancestor reached by that many steps upward.

```cpp
const int LOG = 20;
int up[LOG][N], depth[N];

void dfs(int u, int p) {
    up[0][u] = p;
    for (int j = 1; j < LOG; j++) up[j][u] = up[j-1][ up[j-1][u] ];
    for (int v : adj[u]) if (v != p) { depth[v] = depth[u] + 1; dfs(v, u); }
}
int lca(int u, int v) {
    if (depth[u] < depth[v]) swap(u, v);
    int d = depth[u] - depth[v];
    for (int j = 0; j < LOG; j++) if (d >> j & 1) u = up[j][u];
    if (u == v) return u;
    for (int j = LOG-1; j >= 0; j--)
        if (up[j][u] != up[j][v]) { u = up[j][u]; v = up[j][v]; }
    return up[0][u];
}
```

Preprocessing takes time proportional to the number of vertices times the logarithm of that number, and each query then takes only logarithmic time. This is the right default choice, and it also directly answers "what is the k-th ancestor of this vertex", which is what CSES *Company Queries I* and LC 1483 ask for. The second loop inside `lca` is worth reading carefully: it raises `u` and `v` together by the largest power of two that keeps them at *different* vertices, and once no such power exists, both are one step below their common ancestor. That "keep them distinct for as long as possible" idea is the same one used in the tree-descent pattern from chapter [[08 Segment Trees]].

Given two vertices, once the lowest common ancestor is known, the distance between them follows immediately:

$$\text{dist}(u, v) = \text{depth}[u] + \text{depth}[v] - 2 \cdot \text{depth}[\text{lca}(u,v)]$$

**An Euler tour combined with a sparse table** answers queries in constant time after preprocessing, at the cost of slightly more code, and is worth reaching for only when a very large number of queries makes the logarithmic factor from binary lifting too slow.

**Offline processing with a union-find structure**, due to Tarjan, answers all the queries in close to linear total time provided they are all known in advance, and while elegant it is rarely necessary in practice.

---

## Part 3 · An ordinary tree dynamic program

When a value at each vertex depends only on the values already computed for its children, one depth-first search in post-order computes every vertex's value directly.

```cpp
void dfs(int u, int p) {
    dp[u] = base(u);
    for (int v : adj[u]) if (v != p) {
        dfs(v, u);
        dp[u] = combine(dp[u], dp[v]);
    }
}
```

The design question worth asking explicitly is what a vertex's stored value needs to contain so that its parent can make correct use of it, and the answer is often more than a single number.

CSES *Subordinates* needs only the size of each subtree. CSES *Tree Matching* needs two numbers per vertex, the best matching size within its subtree when the vertex itself is left unmatched and when it is matched to one of its children, because the parent needs to know whether this vertex is still available to be matched to it. LC 337 House Robber III has exactly the same two-number structure, one number for "this vertex is included" and one for "it is not". AC EDPC P Independent Set is the same structure again with the two numbers representing a colour choice.

Recognising that these three problems are the same problem in different clothing is worth pausing on, because the underlying move generalises: **whenever a vertex's usability from its parent's perspective depends on a choice made at that vertex, the stored value needs an extra dimension for that choice.**

LC 2246 asks for the longest path along which no two adjacent characters match. The natural quantity to track at each vertex is the length of the longest downward path starting there, and the global answer is found by combining, at each vertex, its two best children, since the overall longest path may bend at that vertex rather than continuing straight down. CSES *Tree Diameter* has the same "the answer may bend at some vertex" structure, and it also has a second, quite different solution: a breadth-first search from any vertex finds a vertex that must be one endpoint of a diameter, and a second breadth-first search from that vertex finds the diameter itself. Both solutions are worth knowing, and working out why the two-search method is correct is a good proof exercise in its own right.

LC 968 Binary Tree Cameras needs three states per vertex, representing whether it still needs to be covered, is covered without holding a camera, or holds a camera itself, combined with a greedy instinct to place cameras as high in the tree as possible. The state design here takes real care and is worth working through slowly rather than looking up.

---

## Part 4 · Rerooting

**The problem this solves.** Some questions ask for a value at *every* vertex, where that value depends on the entire tree viewed with that vertex as the root. Computing this separately for each vertex by re-running a full traversal costs time proportional to the square of the number of vertices, which is too slow. Rerooting answers all of them in time proportional to the number of vertices, using exactly two passes.

**The idea in one sentence.** For each vertex, the answer splits into a part contributed by what lies below it, computed by an ordinary post-order pass, and a part contributed by what lies above it, computed by a second pass that carries information down from the parent.

CSES *Tree Distances II* asks, for every vertex, the sum of its distances to every other vertex.

```cpp
// Pass 1, post-order: sz[u] is the subtree size, down[u] is the sum of distances within it
void dfs1(int u, int p) {
    sz[u] = 1; down[u] = 0;
    for (int v : adj[u]) if (v != p) {
        dfs1(v, u);
        sz[u] += sz[v];
        down[u] += down[v] + sz[v];
    }
}
// Pass 2, pre-order: ans[u] is the sum of distances from u to every vertex in the whole tree
void dfs2(int u, int p) {
    for (int v : adj[u]) if (v != p) {
        ans[v] = ans[u] - sz[v] + (n - sz[v]);
        dfs2(v, u);
    }
}
// call as: dfs1(root, -1); ans[root] = down[root]; dfs2(root, -1);
```

The line inside `dfs2` is worth deriving rather than memorising. Moving the root from a vertex `u` to one of its children `v` changes every distance by exactly one step: each of the `sz[v]` vertices inside `v`'s subtree becomes one step closer, and each of the remaining `n - sz[v]` vertices becomes one step further. That single observation is the whole of rerooting.

**The general recipe**, for cases where the update is not this clean:

1. Compute a post-order value `down[u]`, restricted to `u`'s own subtree.
2. Compute a pre-order value `up[u]`, representing the contribution from everything outside `u`'s subtree, derived from the parent's `up` value together with `u`'s siblings.
3. Combine the two to get the final answer at `u`.

The second step is where care is needed when the combination is a maximum or requires excluding one specific child, since computing "all children except this one" naively for every child costs time proportional to the square of the number of children. Two standard fixes exist. When the combining operation has an inverse, such as a sum, the child's own contribution can simply be subtracted out. When it does not, prefix and suffix combinations over the list of children answer "all children except this one" in constant time per child after linear preprocessing.

AC EDPC V Subtree is the problem that forces the second fix, since its combination is a product modulo a number that is not necessarily prime, meaning division is unavailable and prefix and suffix products are the only option. It is a carefully designed problem and repays the extra attention.

LC 834 Sum of Distances in Tree is CSES Tree Distances II under a different name. CSES *Tree Distances I* asks for the maximum rather than the sum, which needs the top two children tracked rather than a simple total. CF 1092F and CF 543D are weighted variants of the same idea.

**The trigger worth watching for** is the combination of "for every vertex" with "the whole tree". Seeing both phrases together is close to an explicit statement that rerooting is required.

---

## Part 5 · Questions about the multiset inside a subtree

Some problems ask, for every vertex, a question about the collection of values inside its subtree, such as how many distinct values it contains, or which value appears most often. These need more than a single running total per vertex.

**Small-to-large merging** gives each vertex its own container, typically a map, and merges a child's container into the parent's, always merging the smaller container into the larger one and reusing the larger one directly rather than copying it.

```cpp
void dfs(int u, int p) {
    mp[u] = new map<int,int>();
    (*mp[u])[col[u]]++;
    for (int v : adj[u]) if (v != p) {
        dfs(v, u);
        if (mp[u]->size() < mp[v]->size()) swap(mp[u], mp[v]);
        for (auto& [k, c] : *mp[v]) (*mp[u])[k] += c;
        mp[v]->clear();
    }
    ans[u] = mp[u]->size();
}
```

The reason this is efficient is that an element only moves into a new container when that container is at least as large as the one it came from, which means the container holding a given element at least doubles in size every time the element moves. An element can therefore move only a logarithmic number of times over the whole process, giving a total cost proportional to the number of vertices times the logarithm of that number, times the cost of a single map operation. The `swap` is what makes this bound hold, since swapping pointers is constant time while copying the larger container would defeat the whole argument.

**A closely related technique**, often called small-to-large on a tree even when no explicit small-to-large merge is coded, avoids maps entirely by using one global array of counts. It identifies, at each vertex, the child whose subtree is largest, called the heavy child, processes the other children first and removes their contributions afterwards, then processes the heavy child last and keeps its contribution in place, then re-adds the other children's contributions to answer the query at this vertex, removing everything again afterwards if this vertex is itself not a heavy child of its own parent. Each vertex's contribution to the global counts is added and removed only a logarithmic number of times, because there are only a logarithmic number of "light" edges on any path from the root, and this gives the same overall cost as small-to-large merging with much less overhead per operation.

CF 600E is the standard example, asking for the sum of the most frequent colours in each subtree. CF 375D and CF 246E use the same family of techniques.

**Before reaching for either of these, it is worth checking whether an Euler tour combined with an offline sweep or with Mo's algorithm answers the same question more directly**, since a subtree is a contiguous range and the machinery from chapter [[09 Fenwick Offline and Mos]] often applies with less code.

---

## Part 6 · Centroid decomposition

Subtree techniques answer questions about vertices and the parts of the tree hanging below them. Questions about *paths* are harder, because a path can travel upward through a vertex and back down through a different branch, which does not respect the subtree structure at all.

The centroid of a tree is a vertex whose removal leaves every remaining piece with at most half the original number of vertices, and one always exists and can be found in linear time. Centroid decomposition finds the centroid, answers every question about paths passing through it, then recurses independently into each remaining piece. Because each level of recursion at least halves the size of what remains, the recursion has only a logarithmic number of levels, and every path in the original tree passes through the centroid of exactly one of these recursive pieces, which is what guarantees that nothing is double-counted and nothing is missed.

CSES *Fixed-Length Paths I* asks for the number of paths of an exact given length, and at each centroid this means collecting the depths of vertices in every child subtree and counting pairs of depths summing to the target, then subtracting the pairs that lie entirely within the same child subtree, since those do not pass through the centroid at all. That subtraction is the standard place for errors to appear.

This is the lowest-frequency technique in the chapter and worth learning after everything else here is solid. The centroid itself, independent of the full decomposition, is a useful fact on its own and is what CSES *Finding a Centroid* asks for directly.

---

## Part 7 · Path updates and path queries

When a query concerns an entire path rather than a subtree, the Euler tour alone is not enough, since a path is not generally a contiguous range.

A subtree update combined with a subtree query still works directly with the Euler tour and a segment tree supporting deferred updates. A path query combined with a point update, where the aggregate can be undone, is handled by the same difference trick used for CSES Path Queries in Part 1. A path update combined with a path query, in general, needs heavy-light decomposition, which splits the tree into chains so that any path from the root to a vertex crosses only a logarithmic number of chains, each of which is a contiguous range that a segment tree can handle. CSES *Path Queries II*, which asks for the maximum along a path with point updates, genuinely needs this and is the hardest problem in this chapter, worth attempting only once everything before it feels comfortable.

---

## The ideas worth carrying forward

1. **A subtree occupies a contiguous range in the Euler tour.** This single fact turns every array-based structure into a tree-based one.

2. **Ancestry is tested in constant time** using entry and finish times, with no traversal needed.

3. **One depth-first search answers questions about individual subtrees. Two answer questions about the whole tree from every vertex's perspective.** Recognising which of these a problem is asking for determines the entire approach.

4. **Rerooting's second pass asks what changes when the root moves one edge.** For distance sums, vertices inside the moved-to subtree get closer and everything else gets further, by exactly one step each.

5. **When a combination cannot be undone, excluding one child needs prefix and suffix combinations rather than subtraction.** AC EDPC V is the problem that forces this technique to be learned properly.

6. **When a vertex's usability depends on a choice made there, add a dimension for that choice.** Tree Matching, House Robber III and Independent Set are the same problem three times over.

7. **Small-to-large merging requires swapping containers before merging**, since copying the larger one destroys the complexity guarantee entirely.

8. **An element's container at least doubles each time it moves, bounding the number of moves to a logarithm.** The same argument underlies both small-to-large merging and the heavy-child technique.

9. **Every path passes through the centroid of exactly one piece in the recursive decomposition**, which is what makes centroid decomposition count paths correctly.

10. **Before writing small-to-large merging, check whether an Euler tour combined with an offline sweep answers the same question with less code.**

11. **The longest path in a tree may bend at some vertex**, which is why diameter-style problems combine the best two children rather than following a single branch downward.

---

## Where people lose these problems

**A recursive depth-first search on a path-shaped tree with two hundred thousand vertices.** This is the most common failure in this chapter, since a tree that happens to be a long chain produces a recursion depth equal to the number of vertices.

**Forgetting to skip the parent vertex in the recursion**, which causes infinite recursion in every tree traversal.

**Setting the number of binary-lifting levels too low.** For two hundred thousand vertices, eighteen levels are required at minimum, so using twenty is the safer choice.

**Not initialising the root's own ancestor pointer consistently.** Setting it to point to itself removes the need for a special case elsewhere in the lifting logic.

**Merging without the swap in small-to-large.** This silently removes the complexity guarantee and produces a solution that is correct but far too slow.

**Mutating shared state during the second rerooting pass before it is needed by a sibling.** Each vertex's answer should be computed from its parent's answer before descending further, and if anything is mutated in place, it needs to be restored on the way back out.

**Treating the finish time inconsistently.** Deciding once whether it represents the timer value immediately after the subtree finishes, which is the cleaner and recommended convention, or the last entry time within the subtree, and then using that convention everywhere.

**Failing to subtract same-subtree pairs in centroid decomposition.** This is the standard place for an off-by-one-like error to appear in that technique.

**In LC 2440, missing the "return zero after a cut" mechanic.** For each candidate target sum, a depth-first search that accumulates subtree sums should treat reaching exactly the target as an opportunity to cut the tree there and report zero upward, so that the parent's total does not double-count what has already been separated off.

---

## Working through the problem list

### Block M1 · Ancestry and path queries

- **CSES Subordinates** — *count the subordinates of each employee.* The simplest possible tree dynamic program.
- **CSES Company Queries I** — *find the k-th ancestor of a vertex.* Build your reusable binary-lifting template here.
- **CSES Company Queries II** — *find the lowest common ancestor of two vertices.* The same template with one additional function.
- **CSES Distance Queries** — *find the distance between two vertices.* The distance formula from Part 2.
- **LC 236 Lowest Common Ancestor of a Binary Tree** — *find the lowest common ancestor without any preprocessing.* A different and purely recursive technique, worth knowing since it is a common interview question independent of this chapter.
- **LC 1483 Kth Ancestor of a Tree Node** — *the k-th ancestor problem in LeetCode form.*
- **CSES Subtree Queries** — *sum values within a subtree, with point updates.* The problem that makes "a subtree is a range" concrete. Worth doing carefully.
- **CSES Path Queries** — *sum values along the path from a vertex to the root, with subtree updates.* The difference-array trick over the Euler tour.
- **CSES Counting Paths** — *count how many given paths cover each vertex.* A difference array built directly on the tree, adding at both endpoints and subtracting at the lowest common ancestor and its parent, then summing over each subtree. A direct callback to chapter [[06 Prefix Sums and Difference Arrays]].
- **LC 2846 Minimum Edge Weight Equilibrium Queries in a Tree** — *combines binary lifting, weight-frequency counts, and path counting via the lowest common ancestor.*
- **CSES Path Queries II** — *maximum along a path, with point updates.* Heavy-light decomposition, and the hardest problem in this block, worth leaving until last.

### Block M2 · Rerooting

- **CSES Tree Distances I** — *the maximum distance from each vertex.* Needs the top two children tracked.
- **CSES Tree Distances II** — *the sum of distances from each vertex.* The template rerooting problem, and worth deriving the transition by hand.
- **LC 834 Sum of Distances in Tree** — *the same problem under a different name.* A good check that the technique has been internalised rather than just copied.
- **LC 2581 Count Number of Possible Root Nodes** — *reroot a count of correctly guessed edge directions.* A very clean example, since the transition is a simple plus or minus one per edge.
- **CF 1092F Tree with Maximum Cost** — *a weighted version of the sum-of-distances problem.*
- **CF 543D Road Improvement** — *rerooting with a product combination and a modulus that may not be prime*, which forces prefix and suffix products.

### Block M3 · Ordinary tree dynamic programming

- **CSES Tree Matching** — *the maximum matching size within each subtree.*
- **AC EDPC P Independent Set** — *the same two-state structure applied to a different problem.* Do this right after Tree Matching to see the repetition.
- **LC 337 House Robber III** — *the same structure a third time.* If the repetition has not become obvious by now, it is worth revisiting all three together.
- **CSES Tree Diameter** — *the longest path in a tree.* Worth solving both ways described in Part 3.
- **CSES Finding a Centroid** — *find a vertex whose removal leaves balanced pieces.* One depth-first search checking the largest remaining piece at each vertex.
- **LC 2246 Longest Path With Different Adjacent Characters** — *the bending-path structure with the top two children.*
- **LC 968 Binary Tree Cameras** — *the three-state greedy dynamic program.*
- **LC 2440 Create Components With Same Value** — *split a tree into equal-sum pieces.* Enumerate divisors of the total sum, then check each with a depth-first search using the cut-and-return-zero mechanic.
- **CF 274B Zero Tree** — *track how much needs to be added and how much needs to be subtracted separately within each subtree*, combining the two totals at the root.
- **AC EDPC V Subtree** — *the rerooting problem forcing prefix and suffix products.* The most instructive problem in this chapter, and worth attempting right after Tree Distances II.
- **LC 1547 Minimum Cost to Cut a Stick** — *not a tree problem at all, despite its position on the sheet.* This is interval dynamic programming from chapter [[14 Dynamic Programming Core]], and recognising the misfile saves time.

### Block M4 · Small-to-large and centroid decomposition

- **CSES Distinct Colors** — *count distinct colours within each subtree.* The gentlest introduction to small-to-large merging.
- **CF 600E Lomsat gelral** — *the sum of the most frequent colours within each subtree.* The canonical example, worth doing first with plain small-to-large merging and then again with the heavy-child technique to feel the difference.
- **CF 375D Tree and Queries** — *count colours appearing at least a given number of times within each subtree.* Worth also attempting with an Euler tour and an offline sweep, which is often shorter.
- **CF 208E Blood Cousins** and **CF 246E Blood Cousins Return** — *questions about vertices at the same depth sharing a common ancestor.* Both admit a straightforward offline solution using an array indexed by depth, maintained as the depth-first search enters and leaves each vertex, which is worth learning as its own reusable idea.
- **CF 161D Distance in Tree** — *count paths of exactly a given length, with the length bounded by five hundred.* A dynamic program tracking, for each vertex, the count of descendants at each depth is much simpler than centroid decomposition here, and is a good example of a small parameter belonging in the state rather than requiring heavier machinery.
- **CSES Fixed-Length Paths I** and **Fixed-Length Paths II** — *centroid decomposition.* The last topic to attempt in this chapter.

---

**The sheet is right to describe this block as verification plus filling gaps.** Blocks M1 and M3 should move quickly if they are familiar, and the time saved there is best spent on M2, rerooting, and M4, small-to-large merging, since these are the two techniques most commonly missing and the two most likely to appear in a genuinely hard problem.

---

## Check yourself

1. What range does a subtree occupy in the Euler tour, and how is ancestry tested in constant time?
2. Explain the second loop in the binary-lifting lowest-common-ancestor function. What invariant does it maintain?
3. Write the distance formula in terms of depths and the lowest common ancestor.
4. What distinguishes a problem needing one depth-first search from one needing two?
5. Derive the rerooting transition for CSES Tree Distances II from scratch.
6. Why does AC EDPC V require prefix and suffix products rather than a direct subtraction?
7. Why is small-to-large merging bounded by a logarithmic number of moves per element, and what breaks if the swap step is omitted?
8. Describe the heavy-child rule and explain why it gives the same complexity as small-to-large merging.
9. What guarantee does centroid decomposition give about every path in the tree?
10. Name three problems on this sheet that all reduce to "add a dimension for the choice made at this vertex."
11. Given a question about the multiset inside each subtree, what should be checked before writing small-to-large merging?
