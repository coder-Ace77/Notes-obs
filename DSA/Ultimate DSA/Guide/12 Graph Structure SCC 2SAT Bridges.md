---
tags: [dsa, guide, graphs, scc, 2-sat, bridges, eulerian]
chapter: 12
sheet-section: L
---

# Chapter 12 · Graph Structure: SCC, 2-SAT, Bridges, Eulerian

> **Read this before you start the problems.** Each idea is introduced with a small example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **L** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

The four topics in this chapter appear far less often than the ones in earlier chapters, and that low frequency is itself a source of difficulty. Because you encounter strongly connected components or two-satisfiability only occasionally, the trigger that should make you reach for them does not stay fresh in memory, and by the time a problem needs one of them you may have forgotten what the trigger looks like.

The second source of difficulty is that these problems tend to be all or nothing. There is rarely a partial credit route where you solve half the problem with a simpler idea and pick up the rest later. If a problem needs a strongly connected components decomposition and you do not recognise that at minute five, you are unlikely to recognise it at minute thirty either, because nothing about grinding on the wrong approach reveals the missing idea.

Because of both of these, the most useful thing this chapter can give you is not depth of understanding but reliability of recognition. Each of the four topics has a distinctive trigger phrase, and learning to notice that phrase is worth as much as learning the algorithm underneath it.

---

## What these problems look like

| The phrase in the statement | The topic |
|---|---|
| "mutually reachable" or "any city can reach any city" | strongly connected components |
| "the fewest edges to add so everything can be reached" | condensing into components, then counting sources and sinks |
| "each rule involves at most two items, and each item has two choices" | two-satisfiability |
| "removing this edge disconnects the graph" or "critical connection" | bridges |
| "removing this vertex disconnects the graph" | articulation points |
| "use every edge exactly once" | an Eulerian path or circuit |
| "visit every vertex exactly once" | usually bitmask dynamic programming, from chapter [[15 Bitmask DP]], rather than anything in this chapter |
| "each vertex has exactly one outgoing edge" | a functional graph |
| "longest path" in a graph with no cycles | ordinary dynamic programming in topological order |

---

## Part 1 · Strongly connected components

Two vertices belong to the same strongly connected component when each can reach the other. Collapsing every component to a single vertex produces the condensation of the graph, and the condensation always has no cycles, regardless of how many cycles the original graph had.

**Tarjan's algorithm** finds the components with a single pass of depth-first search, and it is worth learning this one properly since it recurs.

```cpp
vector<int> disc, low, comp, stk; vector<bool> onStk;
int timer = 0, nComp = 0;

void dfs(int u) {
    disc[u] = low[u] = timer++;
    stk.push_back(u); onStk[u] = true;
    for (int v : adj[u]) {
        if (disc[v] == -1) { dfs(v); low[u] = min(low[u], low[v]); }
        else if (onStk[v])  low[u] = min(low[u], disc[v]);      // uses disc, not low
    }
    if (low[u] == disc[u]) {                                     // u roots a component
        while (true) {
            int w = stk.back(); stk.pop_back(); onStk[w] = false;
            comp[w] = nComp;
            if (w == u) break;
        }
        nComp++;
    }
}
```

Two things about this are worth understanding rather than memorising.

`low[u]` records the smallest discovery time reachable from the subtree rooted at `u`, using ordinary tree edges together with at most one back edge to a vertex still sitting on the stack. The line that updates it from a back edge uses `disc[v]` rather than `low[v]`, and it only does so when `v` is currently on the stack, because a back edge to a vertex whose component has already been finished cannot help this component reach anything new.

A useful side effect is that Tarjan's algorithm numbers the components in reverse topological order of the condensation. Iterating components from the highest index to the lowest therefore gives a valid topological order without any additional sorting, which is easy to forget and convenient when it is remembered.

Kosaraju's algorithm, which runs two depth-first searches, one on the graph and one on its reverse, is a reasonable alternative if the low-link reasoning above does not stick. It is slightly slower but easier to reconstruct from memory, and correctness matters more than elegance during an assessment.

**The condensation is where the actual problems live.** CSES *Coin Collector* asks for the maximum value collectible along a path, and the approach is to condense the graph and then run a longest-path dynamic program on the resulting acyclic graph, with each component's value being the sum of its vertices. CF 999E asks how many additional edges are needed so that every vertex is reachable from a given capital, and the answer is the number of components with no incoming edge from any other component, excluding the one containing the capital. CF 427C asks for both the minimum cost and the number of ways to achieve it, treating each strongly connected component independently: the minimum cost within a component is the smallest vertex cost inside it, and the count is how many vertices achieve that cost, with the answer being the product of these counts across components.

---

## Part 2 · Two-satisfiability

The trigger for this topic is structural rather than about the topic's subject matter: there are boolean variables, and every constraint involves at most two of them.

**The construction.** A clause requiring that at least one of `a` or `b` holds is logically the same as saying that if `a` is false then `b` must be true, and if `b` is false then `a` must be true:

$$(a \lor b) \equiv (\lnot a \Rightarrow b) \land (\lnot b \Rightarrow a)$$

Building a graph with two nodes per variable, one for the variable being true and one for it being false, and adding an edge for each implication, produces what is called the implication graph.

**The correctness condition.** The set of constraints is satisfiable exactly when no variable has its true node and its false node in the same strongly connected component. If they were in the same component, then `x` would imply `not x` and `not x` would imply `x`, which is a contradiction.

**Extracting an assignment.** Once the components are known, a variable is set to true when its true node's component comes later in topological order than its false node's component. Because Tarjan's algorithm already numbers components in reverse topological order, this becomes a direct comparison of component indices.

```cpp
auto neg = [](int lit) { return lit ^ 1; };
auto addImpl = [&](int a, int b) { adj[a].push_back(b); adj[neg(b)].push_back(neg(a)); };
auto addClause = [&](int a, int b) { addImpl(neg(a), b); };   // encodes (a OR b)

// after running Tarjan's algorithm:
for (int i = 0; i < n; i++) {
    if (comp[2*i] == comp[2*i+1]) return "IMPOSSIBLE";
    val[i] = comp[2*i] < comp[2*i+1];
}
```

The detail worth being careful about is that `addImpl` adds both an implication and its contrapositive. Leaving out the contrapositive is the most common error in a two-satisfiability implementation, and it produces a graph that looks plausible while being subtly incomplete, which shows up as wrong answers on a fraction of the tests rather than as an obvious failure.

CSES *Giant Pizza* is the clearest example, where each person names two toppings and wants at least one satisfied. CF 776D is a good demonstration of the trigger appearing in disguise: each door is controlled by exactly two switches, and each switch has two positions, which is exactly the "at most two binary choices per constraint" pattern once you look past the story about doors and switches.

---

## Part 3 · Bridges and articulation points

A bridge is an edge whose removal increases the number of connected components. An articulation point is a vertex with the same property.

Both are found with the same depth-first search used for strongly connected components, differing only in the final test.

```cpp
void dfs(int u, int parentEdge) {
    disc[u] = low[u] = timer++;
    int children = 0;
    for (auto [v, id] : adj[u]) {
        if (id == parentEdge) continue;                          // skip the edge just used
        if (disc[v] != -1) { low[u] = min(low[u], disc[v]); continue; }
        children++;
        dfs(v, id);
        low[u] = min(low[u], low[v]);
        if (low[v] > disc[u])  bridges.push_back(id);             // a bridge
        if (low[v] >= disc[u] && parentEdge != -1) isArt[u] = true;  // an articulation point
    }
    if (parentEdge == -1 && children > 1) isArt[u] = true;        // the root is a special case
}
```

Reading the two conditions in words explains why they differ by a single character. The bridge condition says that nothing in `v`'s subtree can reach `u` or anything above `u` except by using this particular edge, which makes the edge essential. The articulation condition says that nothing in `v`'s subtree can reach anything *above* `u`, which is a weaker requirement, since reaching `u` itself through a back edge is fine for a bridge but does not save the vertex `u` from being an articulation point.

Two implementation details matter. The check that skips the edge just used to enter `u` must compare edge identifiers rather than vertex identifiers, because with parallel edges between the same pair of vertices, comparing vertices would incorrectly treat a second genuine edge as the one just used and report a false bridge. The root of the search is a special case, being an articulation point only when it has more than one child in the depth-first search tree, and this is a common single-test failure when overlooked.

CSES *Necessary Roads* and LC 1192 both ask directly for the list of bridges. CSES *Necessary Cities* asks for articulation points. CF 118E asks whether every edge can be given a direction so that the resulting graph is strongly connected, which is possible exactly when there are no bridges, and the construction orients tree edges away from the root and back edges towards it during the same depth-first search.

---

## Part 4 · Eulerian paths and circuits

The trigger here is explicit: a request to use every edge exactly once.

The existence conditions are worth memorising, since they cost nothing to check and immediately tell you whether a solution exists at all.

| Graph type | Circuit, same start and end | Path, different start and end |
|---|---|---|
| undirected | every vertex has even degree | exactly two vertices have odd degree |
| directed | every vertex has equal in-degree and out-degree | exactly one vertex has one more outgoing edge than incoming, exactly one has the reverse, and every other vertex is balanced |

In both cases every edge must lie in a single connected component, though isolated vertices with no edges are harmless.

**Hierholzer's algorithm** constructs the actual sequence, and it should be written iteratively, since a recursive version can exceed the call stack on graphs with a hundred thousand edges.

```cpp
vector<int> it(n, 0);              // the next untried edge at each vertex
vector<int> stk = {start}, path;
while (!stk.empty()) {
    int u = stk.back();
    if (it[u] < (int)adj[u].size()) {
        auto [v, id] = adj[u][it[u]++];
        if (used[id]) continue;
        used[id] = true;
        stk.push_back(v);
    } else { path.push_back(u); stk.pop_back(); }
}
reverse(path.begin(), path.end());
```

The algorithm walks forward until it becomes stuck, which given the existence conditions can only happen back at the starting vertex, then backs off and splices in any side trips it missed. Reversing the collected sequence at the end produces the answer in the correct order.

CSES *Mail Delivery* asks for an undirected circuit and CSES *Teleporters Path* asks for a directed path. CSES *De Bruijn Sequence* is the most elegant application: building a graph whose vertices are all strings of length `n - 1` and whose edges are all strings of length `n`, an Eulerian circuit through that graph visits every `n`-length string exactly once, which is precisely what a De Bruijn sequence requires. Here the entire difficulty of the problem is constructing the right graph, and the algorithm that runs on it is unchanged.

---

## Part 5 · Functional graphs

A functional graph is one where every vertex has exactly one outgoing edge. Its shape is always the same: each connected component consists of a cycle with trees hanging off it.

CSES *Planets Queries I* asks where you end up after a given number of steps, which is answered with binary lifting: precomputing, for each vertex and each power of two, the vertex reached after that many steps, then combining powers of two to answer any query in logarithmic time. CSES *Planets Cycles* asks how many steps until a previously visited vertex is reached, which requires first locating the cycles and then computing each vertex's depth from its own subtree down to the cycle it eventually reaches. LC 2360 asks for the longest cycle in the graph, found by recording the visiting time of each vertex along the current search path and, upon revisiting a vertex still on that path, taking the difference in visiting times as the cycle length. CSES *Round Trip II* asks for any directed cycle, found with a three-colour depth-first search in which a back edge to a vertex still being processed reveals the cycle directly.

---

## Part 6 · Topological order and acyclic graphs

CSES *Course Schedule* asks whether a valid ordering exists, which Kahn's algorithm answers by repeatedly removing vertices with no remaining incoming edges; if fewer than all the vertices are removed this way, a cycle exists. CSES *Longest Flight Route* asks for the longest path in a graph with no cycles, solved by processing vertices in topological order and taking, at each vertex, one plus the best value among its predecessors, with predecessors tracked separately so the path itself can be printed. CSES *Game Routes* asks for the number of paths rather than the longest one, using the same structure with a sum in place of a maximum.

The general principle worth keeping is that the longest path problem is intractable in a general graph but becomes straightforward, in linear time, whenever the graph has no cycles. When a problem asks for a longest path, the first question worth asking is whether the graph is acyclic, and if it is not, whether it is a tree, which chapter [[13 Trees]] handles just as easily, or whether the number of vertices is small enough for the bitmask techniques in chapter [[15 Bitmask DP]].

---

## The ideas worth carrying forward

1. **Tarjan's algorithm numbers components in reverse topological order**, which gives a topological order of the condensation without any further work.

2. **The condensation of any graph has no cycles**, so once a graph has been condensed, every technique for acyclic graphs becomes available.

3. **The trigger for two-satisfiability is structural: at most two binary choices per constraint.** It does not depend on the story the problem tells around it.

4. **A clause becomes two implications, including the contrapositive of each.** Omitting the contrapositive is the characteristic bug and produces a graph that looks complete while being wrong.

5. **Two-satisfiability is unsatisfiable exactly when a variable's true and false nodes land in the same component**, and an assignment is read off by comparing component indices.

6. **The bridge condition and the articulation condition differ by one comparison**, and reading each aloud explains why: a bridge requires that nothing below can reach `u` or above, while an articulation point requires only that nothing below can reach above `u`.

7. **Skip the parent edge by its identifier, not by the vertex it leads to**, or parallel edges cause false positives.

8. **The Eulerian existence conditions are a short table worth memorising**, since checking them costs nothing and immediately tells you whether a solution can exist.

9. **Hierholzer's algorithm needs to be iterative** for graphs with a large number of edges.

10. **In the De Bruijn sequence problem, constructing the right graph is the entire task.** The algorithm that runs on it afterwards is unchanged from any other Eulerian circuit problem.

11. **Longest path is intractable in general, straightforward in a graph with no cycles, and straightforward in a tree.** Checking which of these you actually have takes a moment and determines the entire approach.

---

## Where people lose these problems

**Writing a recursive depth-first search for a graph with a hundred thousand vertices.** This risks exceeding the call stack, and it affects Tarjan's algorithm and Hierholzer's algorithm particularly often, since both are naturally written recursively.

**Updating `low[u]` from `low[v]` on a back edge, rather than from `disc[v]`.** This is the most common error in an otherwise correct strongly connected components implementation.

**Forgetting to check whether a neighbour is still on the stack before using it to update `low`.** A back edge to a component that has already finished cannot help the current component.

**Omitting the contrapositive when building implications for two-satisfiability.**

**Forgetting that the root of a depth-first search tree is an articulation point only when it has more than one child.** This is a distinct rule from the one used for every other vertex, and it is easy to overlook.

**Checking only the degree conditions for an Eulerian circuit and forgetting connectivity.** Two disjoint cycles, each with every vertex of even degree, satisfy the degree conditions without having any Eulerian circuit at all, since the graph is not connected.

**Starting an Eulerian path from the wrong vertex.** An undirected path must start at one of the two odd-degree vertices, and a directed path must start at the vertex with one more outgoing edge than incoming. Starting elsewhere fails even on inputs where a solution genuinely exists.

**In LC 2101, assuming the reachability relation is symmetric.** One bomb's blast radius may reach a second bomb without the second bomb's radius reaching back, so the relevant graph is directed, and a union-find approach, which only makes sense for symmetric relationships, is the wrong tool. With at most a hundred vertices, running a search from every vertex individually is fast enough and avoids the mistake.

---

## Working through the problem list

### Block 1 · Topological order and cycles

- **CSES Course Schedule** — *decide whether a valid course ordering exists.* Kahn's algorithm.
- **CSES Longest Flight Route** — *find and print the longest route through a graph with no cycles.* Dynamic programming in topological order, with predecessors tracked for reconstruction.
- **CSES Game Routes** — *count the number of routes through a graph with no cycles.* The same structure with a sum instead of a maximum.
- **CSES Round Trip II** — *find any directed cycle.* Three-colour search.

### Block 2 · Strongly connected components

- **CSES Planets and Kingdoms** — *label each vertex with its component.* The template problem, and the place to get Tarjan's algorithm, or Kosaraju's, working cleanly.
- **CSES Flight Routes Check** — *decide whether the whole graph is one component, or provide a counterexample pair.*
- **CF 427C Checkposts** — *minimise total cost and count the number of ways, per component.* A gentle application, good for building confidence.
- **CSES Coin Collector** — *maximise value collected along a path in a graph that may contain cycles.* Condense, then run dynamic programming on the acyclic result. The most representative strongly-connected-components problem here.
- **CF 999E Reachability from the Capital** — *find the fewest edges to add so every vertex is reachable from the capital.* Condense and count components with no incoming edge.
- **AC ACL Practice G · SCC** — *a direct rep of the template.*
- **LC 2360 Longest Cycle in a Graph** — *the longest cycle in a functional graph.* Simpler than the full strongly-connected-components machinery, and a useful contrast.

### Block 3 · Two-satisfiability

- **CSES Giant Pizza** — *the archetypal two-satisfiability problem.* Build this as your reusable template.
- **CF 776D The Door Problem** — *recognise the two-choices-per-constraint trigger, then apply the template mechanically.* Also solvable with a parity-tracking union-find structure from chapter [[10 DSU Advanced]], and doing both is worthwhile, since the union-find version is much shorter and seeing why they agree is instructive.
- **AC ACL Practice H · Two SAT** — *a direct rep.*

### Block 4 · Bridges and articulation points

- **CSES Necessary Roads** — *find all bridges.*
- **LC 1192 Critical Connections** — *the same problem in LeetCode form.* Do whichever you prefer and skim the other.
- **CSES Necessary Cities** — *find all articulation points*, including the root's special case.
- **CF 118E Bertown roads** — *orient every edge so the graph becomes strongly connected.* The constructive consequence of having no bridges.

### Block 5 · Eulerian paths and circuits

- **CSES Mail Delivery** — *an undirected circuit.*
- **CSES Teleporters Path** — *a directed path.*
- **CSES De Bruijn Sequence** — *construct a sequence containing every string of a given length exactly once.* The most satisfying problem in this block, since constructing the graph is the entire challenge.

### Block 6 · Functional graphs

- **CSES Planets Queries I** — *where do you end up after k steps.* Binary lifting, and a template that also serves lowest-common-ancestor queries in chapter [[13 Trees]], so it is worth writing carefully once.
- **CSES Planets Cycles** — *steps until a previously visited vertex recurs.*

### Odd one out

- **LC 2101 Detonate the Maximum Bombs** — *maximise the bombs detonated by a chain reaction.* Directed reachability with brute-force search from each vertex, included here specifically to train the habit of checking whether a relationship is symmetric before reaching for union-find.

---

**This block rewards recognition more than accuracy.** The realistic goal is to name the correct topic within a couple of minutes of reading a statement, and to have a tested implementation of Tarjan's algorithm, two-satisfiability, bridge-finding, and Hierholzer's algorithm saved and ready to use. A reasonable target for submissions is around 70%, but the more useful measure of progress is whether your saved templates work correctly the first time they are pulled out during practice.

---

## Check yourself

1. In Tarjan's algorithm, why does the back-edge update use `disc[v]` rather than `low[v]`, and what does checking whether `v` is still on the stack protect against?
2. What ordering do Tarjan's component indices give you, and what does that save you from computing separately?
3. Translate the clause "a or not b" into implications, including both contrapositives.
4. State the satisfiability condition for two-satisfiability and the rule for reading off an assignment.
5. State the bridge condition and the articulation condition, and explain in words why they differ.
6. Why must the parent edge be skipped by its identifier rather than by the vertex it leads to?
7. Give all four Eulerian existence conditions, plus the connectivity requirement that is easy to forget.
8. What are the vertices and what are the edges in the graph built for the De Bruijn sequence problem?
9. When is the longest path problem easy, and when is it hard?
10. Why is LC 2101 not a good fit for union-find?
