---
tags: [dsa, guide, max-flow, min-cut, bipartite-matching]
chapter: 20
sheet-section: T
---

# Chapter 20 · Flows & Matching

> **Read this before you start the problems.** Each idea is introduced with a small example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **T** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

Almost nobody is asked to implement a maximum-flow algorithm from first principles during an assessment, and that is not really the point of this chapter. What actually happens is that a scheduling or assignment question is asked in ordinary language, and the entire content of the problem is recognising that the quantity being asked for is a maximum flow or a minimum cut, and then translating the story into a graph. Once that translation is done correctly, the algorithm itself is a fixed piece of code that does not need to be reinvented.

This means the chapter's real difficulty is modelling rather than implementation. The signal for "this is a flow problem" is usually reliable once learned, but the graph that needs to be built often requires a specific trick, such as splitting a vertex to impose a capacity on it rather than on an edge, or building a two-sided partition to represent a minimum cut, and these tricks are worth learning as a small fixed set rather than derived from scratch under time pressure.

---

## What these problems look like

The most reliable signals, in roughly descending order of how often they appear:

- **"Select at most one item per row and at most one per column."** This is always bipartite matching.
- **"Assign workers to tasks", "students to projects", or anything similar** where each side of the assignment has a capacity.
- **"The minimum number of things to remove so that some connection becomes impossible."** This is a minimum cut.
- **"The maximum number of disjoint paths between two points."** By Menger's theorem, the maximum number of edge-disjoint paths equals the minimum edge cut, which equals a maximum flow with every edge given capacity one.
- **"Partition items into two groups, paying a penalty for certain pairs split across the groups."** This is project selection, solved as a minimum cut.
- **A bipartite-looking structure with numeric capacities and a small number of vertices**, typically a few thousand at most.

**The reliable anti-signal** is a very large number of vertices, in the region of a hundred thousand or more, combined with a bipartite-looking structure. Flow algorithms are polynomial but carry real constant factors, so an assessment intending a flow-based solution will almost always keep the vertex count small; a large vertex count in a bipartite-looking problem is a much stronger signal that a greedy or matroid-style argument, rather than flow, is intended.

---

## Part 1 · Four facts worth knowing before modelling anything

**The max-flow min-cut theorem** states that the maximum amount of flow that can be routed from a source to a sink equals the smallest total capacity of a set of edges whose removal would disconnect the source from the sink. This is the reason flow is useful for minimisation questions: "the minimum cost to make some connection impossible" is a minimum cut, and a minimum cut is computed by running a maximum-flow algorithm. **Once a maximum flow has been computed, the actual cut can be recovered by running a search from the source through the residual graph**, meaning through edges that still have spare capacity; the set of vertices reachable this way, and its complement, together define the cut, and the specific edges crossing from the reachable side to the unreachable side, with no capacity left, are the cut edges themselves.

**König's theorem**, which applies specifically to bipartite graphs, states that the size of a maximum matching equals the size of a minimum vertex cover. A direct consequence is that the size of a maximum independent set equals the total number of vertices minus the size of a maximum matching, which turns "select the largest possible set of items with no two of them conflicting" into a matching problem whenever the conflict relationship happens to be bipartite.

**Hall's theorem** states that a perfect matching covering one side of a bipartite graph exists exactly when every subset of that side has at least as many neighbours, collectively, as its own size. This is used more often to *prove* that a construction works than to compute anything directly.

**Menger's theorem** states that the maximum number of edge-disjoint paths between two vertices equals the minimum number of edges whose removal disconnects them. With every edge given capacity one, a maximum flow both counts these paths and, through flow decomposition, produces them explicitly.

---

## Part 2 · Dinic's algorithm

Dinic's algorithm runs in time bounded by the square of the vertex count multiplied by the number of edges in general, and considerably faster, proportional to the number of edges times the square root of the vertex count, on graphs where every edge has capacity one, which covers every bipartite matching problem. Either bound is comfortably fast enough for anything an assessment is likely to present.

```cpp
struct Dinic {
    struct E { int to; long long cap; int rev; };
    vector<vector<E>> g; vector<int> level, iter;
    Dinic(int n) : g(n), level(n), iter(n) {}
    void addEdge(int a, int b, long long c) {
        g[a].push_back({b, c, (int)g[b].size()});
        g[b].push_back({a, 0, (int)g[a].size() - 1});     // a reverse edge, starting at zero capacity
    }
    bool bfs(int s, int t) {
        fill(level.begin(), level.end(), -1);
        queue<int> q; level[s] = 0; q.push(s);
        while (!q.empty()) {
            int v = q.front(); q.pop();
            for (auto& e : g[v]) if (e.cap > 0 && level[e.to] < 0) {
                level[e.to] = level[v] + 1; q.push(e.to);
            }
        }
        return level[t] >= 0;
    }
    long long dfs(int v, int t, long long f) {
        if (v == t) return f;
        for (int& i = iter[v]; i < (int)g[v].size(); i++) {
            E& e = g[v][i];
            if (e.cap > 0 && level[v] < level[e.to]) {
                long long d = dfs(e.to, t, min(f, e.cap));
                if (d > 0) { e.cap -= d; g[e.to][e.rev].cap += d; return d; }
            }
        }
        return 0;
    }
    long long maxflow(int s, int t) {
        long long flow = 0;
        while (bfs(s, t)) {
            fill(iter.begin(), iter.end(), 0);
            while (long long f = dfs(s, t, LLONG_MAX)) flow += f;
        }
        return flow;
    }
};
```

Two implementation details are worth understanding rather than treating as fixed boilerplate.

**Every forward edge is paired with a reverse edge starting at zero capacity.** This reverse edge is what allows the algorithm to undo an earlier routing decision by sending flow backwards along it, and this ability to undo is what lets the algorithm escape a suboptimal set of choices rather than getting permanently stuck with them. The reverse edge's index into its partner's list is recorded at the moment it is created, before the forward edge itself is appended, so that both edges correctly point at one another.

**The `iter` array, sometimes called the current-arc optimisation, ensures that once an edge has been found to be useless within the current phase, it is never examined again during that phase.** Taking `iter[v]` by reference inside the loop is what makes this work. Without this optimisation, the algorithm's running time degrades substantially, since the same exhausted edges would otherwise be re-examined repeatedly.

**Whether to memorise this implementation is worth answering directly: it is worth keeping as a tested, saved file rather than reconstructing from memory.** What is worth being able to reconstruct under pressure is the modelling, not the forty lines of algorithm underneath it.

---

## Part 3 · Building the graph

This is where the actual thinking in this chapter happens.

**Bipartite matching.** Connect a source to every vertex on the left side with capacity one, connect every compatible pair between the two sides with capacity one, and connect every vertex on the right side to a sink with capacity one. The resulting maximum flow equals the size of the maximum matching, and after computing it, any left-to-right edge whose capacity has been fully used, meaning its remaining capacity is zero, identifies a matched pair. CSES *School Dance* is exactly this.

**Weighted capacities rather than unit matching.** The same graph works when a worker can take on several tasks and a task needs several workers, simply by setting the source-to-left and right-to-sink capacities to the actual limits rather than to one each.

**Imposing a capacity on a vertex rather than an edge.** Flow naturally limits edges, not vertices, so limiting a *vertex* to some capacity requires splitting it into an incoming half and an outgoing half, joined by a single edge carrying that capacity, with every original incoming edge directed into the incoming half and every original outgoing edge leaving from the outgoing half. This is the standard way to model "at most `k` paths may pass through this location" or to force paths to be vertex-disjoint, in the latter case by giving the internal edge a capacity of exactly one.

**Minimum cut as a two-way partition.** Every vertex ends up assigned to either the source's side or the sink's side of the eventual cut. An edge from the source to a vertex, with some capacity, represents a cost incurred if that vertex ends up on the sink's side. An edge from a vertex to the sink, with some capacity, represents a cost incurred if that vertex ends up on the source's side. An edge between two ordinary vertices with effectively infinite capacity encodes a hard implication: if the first vertex is on the source's side, the second must be too, since otherwise that infinite-capacity edge would need to be cut, which is never optimal.

**This partition-based view of minimum cut is the single idea in this chapter most worth internalising**, since it is what turns real optimisation questions, such as choosing which projects to undertake given shared costs and individual profits, into a flow computation. CSES *Police Chase* is the pure version of this idea: the fewest edges needed to disconnect two vertices is a minimum cut computed with every edge given capacity one, with the actual cut edges recovered afterwards through a search over the residual graph.

**Minimum cost maximum flow.** When each unit of flow along an edge also carries a cost, and the goal is the cheapest way to achieve the maximum possible flow, or occasionally a specified flow value at minimum cost, the standard approach repeatedly finds the cheapest augmenting path using a shortest-path algorithm that tolerates negative edges, such as Bellman–Ford or SPFA, or Dijkstra combined with a potential function once the graph has non-negative reduced costs. **This is needed when an assignment problem carries genuine values rather than plain compatibility**, and the Hungarian algorithm, which runs in time proportional to the cube of the vertex count, is a reasonable and often simpler alternative specifically for square assignment problems. AC ACL Practice E is the standard place to build a template for this; it appears rarely enough in assessments that knowing it exists is more valuable than deep fluency with it.

**Path decomposition.** CSES *Distinct Routes* asks not only for the maximum number of edge-disjoint paths between two vertices but for the paths themselves. Running the maximum-flow algorithm with every edge given capacity one, and then repeatedly walking from the source to the sink along edges that have been saturated, removing one unit of saturation as each edge of a walk is used, extracts one path per walk.

---

## Part 4 · A short checklist for recognising a flow problem

When a problem seems like it might involve flow, four questions help confirm it.

**Is there a natural source and sink, or could one be invented** by connecting a single artificial source to every genuine starting point? **Is the quantity being optimised a count of routed units, or a cut**, rather than something else entirely? **Are the constraints all phrased as "at most `c` of these"**, since that phrasing is exactly what a capacity represents? **Is the vertex count small enough** that a polynomial but not especially fast algorithm is clearly intended?

If all four answers are yes, building the graph is the next step. When there is genuine uncertainty between flow and a greedy approach, it is worth remembering that many small flow problems also admit a greedy solution, but the reverse is far less often true, so reaching for flow when time allows is the safer choice.

---

## The ideas worth carrying forward

1. **Maximum flow equals minimum cut**, which is the reason a minimum-cut question is answered by running a maximum-flow algorithm.

2. **The actual cut is recovered by searching from the source through the residual graph**, meaning through edges with capacity remaining; the reachable set is one side of the cut.

3. **König's theorem gives maximum matching equal to minimum vertex cover in a bipartite graph, and therefore maximum independent set equal to the vertex count minus the maximum matching.**

4. **A vertex capacity is imposed by splitting the vertex** into an incoming half and an outgoing half joined by an edge carrying that capacity.

5. **A minimum cut is a two-way partition of the vertices**, and the graph is built so that source-to-vertex and vertex-to-sink edges encode costs for each side, while very-high-capacity edges between ordinary vertices encode hard implications.

6. **The reverse edge with zero starting capacity is what allows an earlier routing decision to be undone**, which is essential to the algorithm's correctness.

7. **The current-arc optimisation is not optional**; without it, the algorithm's running time degrades substantially due to repeatedly re-examining exhausted edges.

8. **A graph with every edge of capacity one runs Dinic's algorithm considerably faster than the general bound**, which is why bipartite matching on large graphs is still practical.

9. **"Select at most one per row and per column" is always bipartite matching**, regardless of how the problem's story disguises it.

10. **A very large vertex count in what looks like a bipartite structure is a signal that a greedy approach, not flow, was intended**, since flow constraints in an assessment tend to be kept deliberately small.

---

## Where people lose these problems

**Omitting the reverse edge.** The algorithm then silently returns a flow that is smaller than the true maximum.

**Recording a reverse edge's index after appending the forward edge rather than before.** This points the two edges at each other incorrectly.

**Resetting the current-arc array at the wrong point.** It is reset once per phase, immediately after the breadth-first search that establishes the level graph, not once per individual augmenting path.

**Using a capacity value intended to represent infinity that is large enough to cause overflow when summed with other capacities.** A large but finite value, chosen with headroom, avoids this while still functioning as an effectively unbreakable edge.

**Modelling a vertex capacity as though it were an edge capacity.** Splitting the vertex is required instead.

**Attempting to apply straightforward maximum-flow matching to a graph that is not bipartite.** General graph matching requires the blossom algorithm, which is not a reasonable thing to implement under assessment time pressure, and a graph that turns out not to be bipartite calls for a different approach entirely.

**In CSES Download Speed, discarding what appear to be duplicate edges between the same pair of vertices.** Genuine parallel edges each need their own capacity, or their capacities need to be summed deliberately, rather than being silently collapsed into one.

---

## Working through the problem list

This is the smallest chapter on the sheet, at six problems, which is proportionate to how often flow genuinely appears in assessments. The goal is to work through them once, keep a tested implementation, and move on.

- **AC ACL Practice D · Maxflow** — *compute a maximum flow on a given graph.* Get Dinic's algorithm working and saved here first.
- **CSES Download Speed** — *find the maximum flow from one network node to another.* A direct application, confirming the template behaves correctly on realistic constraints, including any parallel edges.
- **CSES School Dance** — *pair dancers from two groups who are willing to dance with each other.* Bipartite matching via flow, and the shape most likely to appear directly in an assessment.
- **CSES Police Chase** — *find the fewest roads to close to disconnect two cities, and report them.* Minimum cut, with **cut recovery** as the part worth practising deliberately, since it is easy to have never actually implemented before.
- **CSES Distinct Routes** — *find the maximum number of edge-disjoint routes between two cities, and print them.* Path decomposition, and a good test of whether the residual graph is genuinely understood rather than merely used.
- **AC ACL Practice E · MinCostFlow** — *compute a minimum-cost maximum flow.* Worth knowing this exists and having a basic implementation, without deep investment beyond that.

---

**Completion, together with a working saved template, is a more meaningful goal here than a numerical accuracy target,** since six problems is too small a sample for a percentage to be informative. The realistic aim is that an assignment-flavoured problem in an assessment is recognised within a couple of minutes, and that a tested Dinic's implementation exists in a file ready to be reused immediately.

---

## Check yourself

1. State the max-flow min-cut theorem, and explain why it makes computing a minimum cut possible.
2. How is the actual set of cut edges recovered after running a maximum-flow algorithm?
3. State König's theorem, and derive the maximum-independent-set consequence from it.
4. How is a capacity imposed on a vertex rather than an edge?
5. In a minimum-cut model, what does an edge from the source to a vertex represent? What does a very-high-capacity edge between two ordinary vertices represent?
6. What is the reverse edge with zero starting capacity for?
7. What does the current-arc array do in Dinic's algorithm, and what happens without it?
8. A problem asks to select at most one cell per row and per column, maximising the total selected. What is the model?
9. A problem looks like bipartite matching but has a hundred thousand vertices. What should be suspected instead?
10. After running a matching-flow computation, how are the actual matched pairs extracted?
