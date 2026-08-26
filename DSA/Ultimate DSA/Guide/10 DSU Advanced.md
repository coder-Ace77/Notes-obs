---
tags: [dsa, guide, dsu, union-find, offline, rollback]
chapter: 10
sheet-section: J
---

# Chapter 10 · DSU: Rollback, Offline, Small-to-Large, Reconstruction

> **Read this before you start the problems.** Each technique is introduced with a worked example, so no prior familiarity is assumed.

Back to [[00 Guide Index]] · Sheet section **J** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

A disjoint set union structure, also called union-find, is usually taught as an introductory topic: it tracks which elements belong to the same group, and it supports merging two groups and asking whether two elements are together. Both operations run in effectively constant time.

The reason it appears in a section of a hard-problem sheet is that this simplicity comes with a strict limitation. The structure can merge groups but cannot split them. Every technique in this chapter exists to work around that limitation, by arranging matters so that the only operation ever needed is merging.

That is why these problems are harder than they look. The obvious formulation usually involves removing edges, removing cells, or answering questions at a moment when only some of the data should be present, all of which appear to require splitting. The work consists of transforming the problem so that it only ever grows: by reversing the direction of time, by sorting the queries so that the structure grows monotonically, or by decomposing the timeline so that each edge is inserted and later withdrawn in a controlled way.

---

## What these problems look like

This structure appears more often than its teaching prominence suggests. The signals are:

- **A sequence of events, with a question after each one.** Roads being built, friendships forming, servers coming online.
- **Edge weights combined with a threshold**, such as asking whether a path exists using only edges below some value.
- **Deletions.** A problem that removes things is very often a problem that adds things when the sequence is reversed.
- **A grid where cells activate over time**, such as land appearing, water rising, or bricks being knocked out.
- **A minimum spanning tree**, or something with a similar flavour, including problems where the tree is a tool rather than the goal.
- **Grouping by an equivalence relation that you discover as you process the input.**

---

## Part 1 · The structure itself

```cpp
struct DSU {
    vector<int> p, sz;
    int comps;
    DSU(int n) : p(n), sz(n, 1), comps(n) { iota(p.begin(), p.end(), 0); }
    int find(int x) { while (p[x] != x) x = p[x] = p[p[x]]; return x; }   // path halving
    bool unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return false;
        if (sz[a] < sz[b]) swap(a, b);
        p[b] = a; sz[a] += sz[b]; comps--;
        return true;
    }
};
```

Three details are worth adopting.

The merge returns whether anything actually happened. This turns out to be needed constantly, for counting components, for building a minimum spanning tree, and for deciding whether an edge was useful.

The lookup uses path halving, which reassigns each node to its grandparent as it walks upwards. It is iterative, allocates nothing, and performs as well in practice as full path compression.

Merging is done by size rather than by rank. The two are equally effective, and size is more useful because component sizes are frequently wanted for their own sake.

---

## Part 2 · Supporting undo

Some techniques require merges to be undone. Path compression makes that impossible, because it rearranges parts of the structure that were not involved in the merge you want to reverse, so there is no record of what to restore.

The solution is to give up path compression and merge only by size, which leaves lookups running in logarithmic time. That is an acceptable price for the ability to undo.

```cpp
struct RollbackDSU {
    vector<int> p, sz; vector<pair<int,int>> hist;   // (child root, parent root)
    int comps;
    RollbackDSU(int n) : p(n), sz(n, 1), comps(n) { iota(p.begin(), p.end(), 0); }
    int find(int x) { while (p[x] != x) x = p[x]; return x; }        // no compression
    bool unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return false;
        if (sz[a] < sz[b]) swap(a, b);
        hist.push_back({b, a}); p[b] = a; sz[a] += sz[b]; comps--;
        return true;
    }
    void rollback(size_t checkpoint) {
        while (hist.size() > checkpoint) {
            auto [b, a] = hist.back(); hist.pop_back();
            p[b] = b; sz[a] -= sz[b]; comps++;
        }
    }
};
```

---

## Part 3 · Reversing time

When a problem only ever removes things, running the sequence backwards converts every removal into an addition. This is the cheapest of the workarounds and it covers a good deal of ground.

**LC 803 Bricks Falling When Hit** is the standard example. Bricks are knocked out of a wall one at a time, and after each hit you report how many bricks fall. A brick falls when it is no longer connected to the top row, so this is a connectivity question, but the hits are removals and removals are what the structure cannot do.

Running it backwards works as follows. Remove all the hit bricks from the grid at the start. Build the structure on what remains, with an artificial node representing the top of the wall connected to every brick in the first row. Then process the hits in reverse order, adding each brick back. The number of bricks that fell at that hit, going forwards, equals the number that become newly connected to the top, going backwards, less one for the brick itself.

**LC 1970 Last Day Where You Can Still Cross** asks for the last day on which a path exists across a grid that is progressively flooding. Reversed, it starts fully flooded and removes water in reverse order of day, reporting the first moment the two sides connect. The same problem also yields to binary search on the day combined with a search, from chapter [[04 Binary Search on the Answer]], and it is worth noticing how frequently those two techniques are interchangeable.

The general statement is the same as in chapter [[01 Implementation and Simulation]]: when operations destroy structure, running the sequence backwards makes them build it instead.

---

## Part 4 · Sorting both the edges and the queries

This is probably the most common use of the structure in current assessments, and it is short enough to write in a few minutes.

**LC 1697** asks, for each query consisting of two nodes and a limit, whether a path exists between them using only edges strictly below that limit.

```
sort the edges by weight
sort the queries by limit
for each query in that order:
    add every edge whose weight is below the current limit
    the answer is whether the two nodes are now connected
```

Both pointers only move forwards, so every edge is added exactly once across the whole run. The only idea involved is sorting both sides so that they can be swept together.

**LC 2503** applies the same shape with a heap rather than an edge list. Sorting the queries in increasing order means the flood outwards from the origin expands monotonically, so no cell is ever visited twice.

---

## Part 5 · Handling both additions and removals

When a problem contains both, reversing time does not help, and this is where the more elaborate technique becomes necessary.

**CSES Dynamic Connectivity** adds and removes edges over time, asking for the number of connected components after each operation.

The observation is that each edge is present during one or more intervals of time. Build a segment tree over the time axis rather than over positions, and insert each edge into the logarithmically many nodes whose time ranges it fully covers, exactly as a range update would.

Then walk the tree:

```
solve(node):
    checkpoint = current size of the history
    merge every edge stored at this node
    if the node covers a single moment in time:
        record the answer for that moment
    else:
        solve(left child); solve(right child)
    roll back to the checkpoint
```

The reason this is correct is that when you reach a leaf representing time `t`, every edge alive at that time has been merged by exactly one node on the path from the root, because the edge's interval was decomposed into canonical nodes and precisely those covering `t` are ancestors of that leaf. Rolling back on the way out restores the structure for the sibling branches.

Each edge appears in logarithmically many nodes and each merge costs a logarithmic factor, so the total is comfortable for a couple of hundred thousand operations. The signal for reaching for this is deletions in a connectivity problem where reversing time is not available.

---

## Part 6 · Counting pairs as components merge

When two components of sizes `x` and `y` are merged, exactly `x` times `y` new pairs of connected nodes come into existence, because every node in one group becomes connected to every node in the other.

That observation turns several counting problems into a single pass of Kruskal's algorithm. **CF 1213G** asks how many pairs of nodes have a path whose largest edge is at most a given weight, for several weights. Processing edges in increasing order of weight and accumulating the product at each merge answers all the queries in one sweep.

---

## Part 7 · The Kruskal reconstruction tree

This structure is powerful and not widely taught. Build a minimum spanning tree with Kruskal's algorithm, and each time two components merge, create a new node whose two children are the roots of those components and whose weight is the weight of the edge that caused the merge.

The result is a binary tree with roughly twice as many nodes as the original graph, in which the leaves are the original vertices and the weights increase as you move from the leaves towards the root. The useful property is that the set of leaves below any node is exactly the set of vertices mutually reachable using only edges of weight at most that node's weight.

That property converts questions about threshold reachability into questions about subtrees. Asking which vertices are reachable from a given vertex using edges below some weight becomes a matter of walking upwards from that leaf, using binary lifting, to the highest ancestor whose weight is within the limit, and taking its subtree. Combined with an Euler tour from chapter [[13 Trees]], subtree questions become range questions.

CF 1416D is the standard application and CF 1213G is a gentler introduction.

---

## Part 8 · Carrying extra information

The structure can hold more than a parent pointer.

**Component aggregates** such as size, total value, or maximum can be stored at the root and updated when merging. This is straightforward and constantly useful.

**Parity**, giving a two-colouring, is stored as the parity of the path from each node to its root. Merging with the appropriate parity and detecting a contradiction tells you whether the graph can be two-coloured, which is the union-find approach to bipartiteness and solves CF 776D from chapter [[12 Graph Structure SCC 2SAT Bridges]].

**Merging containers by size**, where each component carries a set or map and the smaller container is always merged into the larger, costs a logarithmic factor per element overall. The reasoning is that an element only moves when its container merges into a strictly larger one, so its container at least doubles each time, bounding the number of moves. This is covered fully in chapter [[13 Trees]] as it applies to trees, but the principle belongs here.

---

## The ideas worth carrying forward

1. **The structure can merge but not split**, and every technique in this chapter exists to arrange that only merging is ever required.

2. **When a problem only removes, run the sequence backwards.** This is the cheapest workaround and covers a substantial share of problems.

3. **When a problem both adds and removes, decompose the timeline into a segment tree and use rollback.** Each edge lives in logarithmically many nodes, and rolling back on the way out of the recursion restores the state.

4. **Rollback requires giving up path compression**, leaving lookups logarithmic, which is an acceptable cost.

5. **Sorting the edges and the queries together is a six-line technique** covering a surprising number of problems.

6. **Merging components of sizes `x` and `y` creates `x` times `y` new connected pairs.** Accumulating this during Kruskal's algorithm answers several counting questions.

7. **In the Kruskal reconstruction tree, the leaves below a node are exactly the vertices reachable using edges at most that node's weight**, which turns threshold reachability into a subtree question.

8. **Storing parity gives bipartiteness checking** under incremental edge addition.

9. **Merging the smaller container into the larger** bounds the total work, because each element's container doubles every time it moves.

10. **Have the merge return a boolean.** It is needed in nearly every application.

---

## Where people lose these problems

**Combining path compression with rollback.** These are incompatible, and one has to be given up.

**Comparing sizes without finding the roots first.** Sizes are only meaningful at roots.

**Inserting an edge into the wrong time interval.** An edge added at one time and removed at another is alive over a half-open interval, matching the convention from chapter [[02 Intervals and Sweep Line]], and an edge never removed is alive until the end.

**Taking the checkpoint at the wrong moment.** It must be taken before merging the current node's edges and restored after the recursion returns.

**In LC 803, two specific traps.** A hit on a cell that is already empty does nothing and must not be counted, and the number of fallen bricks excludes the hit brick itself, so the count needs adjusting by one when the hit landed on an actual brick.

**In LC 924, the counting rather than the structure.** Exactly one initially infected node is removed. A component containing exactly one infected node is saved by removing it, while a component containing two or more cannot be saved by removing any single one. Ties are broken by the smallest index. The structure is easy and the accounting is where the errors are.

**In LC 2867, reaching for the wrong tool.** Union-find works, but the cleaner solution is a depth-first search in which each node combines, from its children, the counts of paths containing no prime-weighted edges and exactly one. Chapter [[13 Trees]] is the better place for it.

---

## Working through the problem list

### Block 1 · The basics

- **AC ACL Practice A · Disjoint Set Union** — *the structure itself.*
- **CSES Road Reparation** — *connect all cities at minimum total cost.* Kruskal's algorithm, with the disconnected case to handle.
- **CSES Road Construction** — *after each road is built, report the number of components and the size of the largest.* Both quantities maintained at the roots.
- **CF 25D Roads not only in Berland** — *rebuild roads so that all cities are connected.* Edges that would create a cycle are redundant and can be reused to join separate components, which shows the structure identifying useless edges.
- **LC 1101 The Earliest Moment When Everyone Become Friends** — *find when everyone first becomes connected.* Sort by time and merge until one component remains.

### Block 2 · Sweeps and reversals

- **CSES New Roads Queries** — *for each pair of cities, when did they first become connected.* Solvable with a reconstruction tree, or with small-to-large merging of query lists. The reconstruction tree is worth using because the structure pays off later.
- **LC 1697 Checking Existence of Edge Length Limited Paths** — *the sorted-sweep problem from Part 4.*
- **LC 2503 Maximum Number of Points From Grid Queries** — *how many cells are reachable from the origin below each query value.* Sorted queries with a heap-driven flood.
- **LC 803 Bricks Falling When Hit** — *the reversed-time problem from Part 3.*
- **LC 1970 Last Day Where You Can Still Cross** — *the flooding grid.* Worth doing both by reversal and by binary search.
- **LC 3235 Check if the Rectangle Corner Is Reachable** — *can you cross a rectangle without touching any circle.* The reframing is the whole problem: rather than looking for a path, look for a chain of overlapping circles that blocks every path, by merging intersecting circles and checking whether any group touches both opposing pairs of walls. Thinking about the obstacle rather than the route is a move worth remembering.
- **LC 924 Minimize Malware Spread** — *remove one infected node to minimise the eventual spread.* Component sizes plus careful counting.

### Block 3 · The heavier techniques

- **CF 1213G Path Queries** — *count pairs of nodes whose path maximum is at most each given weight.* Kruskal with the pair counting from Part 6, and the gentlest introduction to reconstruction-tree thinking.
- **CF 891C Envy** — *for each set of edges, can they all belong to some minimum spanning tree simultaneously.* The theory is that for each distinct weight, the edges an MST uses at that weight are determined by the component structure formed by all lighter edges. So the query edges are grouped by weight, and for each group the structure is advanced through all lighter edges before attempting the group's edges, with rollback used so that the next group starts from a clean state. This is the problem that makes rollback feel necessary rather than academic.
- **CSES Dynamic Connectivity** — *report the number of components after each addition or removal.* The timeline decomposition from Part 5, and the most substantial problem in the chapter. It is worth a full session, since afterwards offline problems with both additions and removals stop being intimidating.
- **CF 1416D Graph and Queries** — *repeatedly report and remove the largest value in a component, with edges being deleted.* A reconstruction tree with an Euler tour and a segment tree, combining chapters 08, 10 and 13. Genuinely hard and reasonable to treat as a stretch problem.

---

**A reasonable target here is around 75% of submissions passing first time.**

The number of ideas is small but each carries weight. If only three things stay with you, the most valuable are reversing time, sorting both sides and sweeping, and counting pairs at each merge.

---

## Check yourself

1. Write the structure with merging by size and path halving. Why does the merge return a boolean?
2. Why is path compression incompatible with undo? What is given up, and what does a lookup then cost?
3. Describe the timeline decomposition in four sentences. Why does each edge appear in logarithmically many nodes?
4. Name three problems where reversing time converts removals into additions.
5. Write the sorted-sweep solution to LC 1697 in six lines of pseudocode.
6. Two components of sizes 7 and 4 are merged. How many new connected pairs appear, and which problem uses this?
7. State the defining property of the Kruskal reconstruction tree and one query type it makes easy.
8. How is the structure made to detect that a graph cannot be two-coloured?
9. Why is merging the smaller container into the larger bounded by a logarithmic factor per element?
10. In LC 3235, what reframing makes the problem tractable?
