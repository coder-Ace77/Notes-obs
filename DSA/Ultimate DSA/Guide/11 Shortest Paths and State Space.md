---
tags: [dsa, guide, shortest-paths, dijkstra, bfs, state-space]
chapter: 11
sheet-section: K
---

# Chapter 11 · Shortest Paths & State-Space Search

> **Read this before you start the problems.** Each idea comes with a small example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **K** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

Shortest path algorithms are among the first things anyone learns, and they are not the difficulty here. In this block the algorithm is almost always one you already know, and the work consists of deciding what the algorithm should be run on.

The recurring pattern is that the graph described in the statement is not the graph you should search. If the right decision at a junction depends on something other than where you are, such as how much fuel remains or which keys you have collected, then position alone is an insufficient description of your situation. The search has to be run over a larger space in which each node is a combination of a position and whatever else matters, and once that space is set up correctly the algorithm is ordinary.

A second and subtler difficulty is that the quantity being minimised is not always a sum. It may be the largest edge along a path, or the number of times something changes, or the smallest value encountered. Dijkstra's algorithm does not actually require sums; it requires only that extending a path never improves it. Recognising that lets you reuse the same code with one line changed, and it connects a surprising number of problems that otherwise look unrelated.

---

## What these problems look like

The three common variations are:

1. **The node is a combination rather than a single thing.** A position together with the remaining fuel, the set of keys collected, the number of flights used, or the colour of the last edge taken.
2. **The cost of moving depends on when you arrive.** Traffic lights, tides, or teleporters running on a schedule.
3. **The graph is implicit and very large, but the reachable part is small.** Transforming one string into another, or reaching a number from one using a set of permitted operations.

And running across all three, the quantity being minimised may not be a sum.

---

## Part 1 · Choosing the algorithm from the edge weights

| Edge weights | Algorithm | Cost |
|---|---|---|
| all equal | breadth-first search | linear |
| all zero or one | breadth-first search with a deque | linear |
| small integers up to `k` | bucket queue | linear plus `k` times the node count |
| arbitrary and non-negative | Dijkstra with a heap | edges times log of nodes |
| possibly negative | Bellman–Ford | nodes times edges |
| all pairs, few nodes | Floyd–Warshall | nodes cubed |
| acyclic graph | dynamic programming in topological order | linear |

Choosing wrongly costs time in both directions. Using breadth-first search on a weighted graph gives wrong answers, and using Dijkstra where breadth-first search would do is slower and longer to write. Looking at the edge weights before deciding takes a few seconds.

---

## Part 2 · Breadth-first search with a deque

When every edge has weight zero or one, a heap is unnecessary. A deque suffices, with zero-weight neighbours pushed to the front and one-weight neighbours pushed to the back.

```cpp
deque<int> dq; dq.push_back(src); dist[src] = 0;
while (!dq.empty()) {
    int u = dq.front(); dq.pop_front();
    if (seen[u]) continue; seen[u] = true;
    for (auto [v, w] : adj[u])
        if (dist[u] + w < dist[v]) {
            dist[v] = dist[u] + w;
            if (w == 0) dq.push_front(v); else dq.push_back(v);
        }
}
```

The reason this is correct is that the deque only ever contains nodes at two distinct distances, some value and that value plus one, with all the smaller ones at the front. Pushing a zero-weight neighbour to the front keeps it in the smaller group and pushing a one-weight neighbour to the back places it in the larger group, so the deque remains sorted without any comparison being performed.

The disguise to watch for is any grid problem where some moves are free and others cost one. Changing a cell's arrow costs one while following it costs nothing, stepping onto an obstacle costs one while an empty cell costs nothing, and turning a mirror costs one while travelling straight costs nothing. Once you know this technique exists these appear regularly, and it is considerably shorter than Dijkstra.

---

## Part 3 · Putting extra information into the node

This is the central idea of the chapter.

The procedure is to ask what you would need to know, besides your current position, in order to make the right next move. Whatever that is becomes part of the node, and the edges of the enlarged graph both move you and update that extra information.

**CSES Flight Discount** allows one coupon halving the cost of a single flight. A node becomes a city together with whether the coupon has been used. From an unused state you may travel at full price and remain unused, or travel at half price and become used. From a used state you travel at full price only. Running Dijkstra on twice as many nodes solves it.

**LC 787 Cheapest Flights Within K Stops** makes a node a city together with the number of stops used. There is also a neater formulation: running Bellman–Ford for exactly `k + 1` rounds works, because each round relaxes paths using one more edge. That second view is worth understanding because it reveals what Bellman–Ford actually computes, which is a dynamic program over the number of edges used.

**LC 864 Shortest Path to Get All Keys** makes a node a cell together with the set of keys held. With at most six keys there are sixty-four possible sets, so the state count is manageable, and since every move costs one, plain breadth-first search suffices.

**LC 847 Shortest Path Visiting All Nodes** makes a node a position together with the set of nodes already visited, starting the search from every node simultaneously. This is chapter [[15 Bitmask DP]] in the shape of a graph search.

**CF 59E Shortest Path** forbids certain triples of consecutive vertices, so the right decision depends on the previous two positions, and a node becomes a pair of consecutive vertices. This is the case where the extra information is a short window of history.

**LC 815 Bus Routes** deserves separate mention because the reframing is different in kind. Rather than enlarging the node, it replaces it: the search runs over bus routes rather than bus stops, with two routes adjacent when they share a stop. Choosing what the nodes should be is itself the problem.

Once the state is chosen, the remaining check is arithmetic. A node count multiplied by sixty-four is fine, and a node count multiplied by a million is not, so it is worth computing the product before committing.

---

## Part 4 · Changing what is being minimised

Dijkstra's algorithm requires only that extending a path never improves it, which is a broader condition than requiring the cost to be a sum of non-negative weights.

**Minimising the largest edge on a path.** Define the distance to a node as the smallest possible value of the largest edge along any path reaching it. The relaxation becomes:

```cpp
nd = max(dist[u], w(u,v));           // instead of dist[u] + w
if (nd < dist[v]) { dist[v] = nd; pq.push({nd, v}); }
```

Everything else is unchanged. Chapter [[04 Binary Search on the Answer]] solves the same problems by searching over the threshold and testing reachability, and chapter [[10 DSU Advanced]] solves them by adding edges in sorted order until the endpoints connect. Three techniques address one class of problem, and spending a few minutes on why they agree is worthwhile.

**Maximising the smallest value on a path** is the mirror image: take the minimum during relaxation and use a maximum-heap.

The same skeleton also covers finding the widest path, the most reliable path when edges carry probabilities, and the path with the fewest colour changes.

**LC 2045 Second Minimum Time to Reach Destination** combines two variations, which is what makes it a genuine hard problem. It needs the second shortest distinct distance, which means keeping the two best values at each node and relaxing into the second when a value beats it without equalling the first. It also has costs that depend on arrival time, since arriving during a red light means waiting until it turns green. Neither variation is difficult alone.

---

## Part 5 · Heap-driven floods

**LC 407 Trapping Rain Water II** is the clearest example of the algorithm being used for something other than a shortest path.

Push every boundary cell into a minimum-heap keyed by height. Repeatedly remove the lowest cell and examine its unvisited neighbours. The water level at a neighbour is the larger of the current cell's level and the neighbour's own height, the water held there is that level minus its height, and the neighbour is then pushed with its computed level.

The reason this works is that the lowest boundary cell is the point at which water escapes, so processing cells in increasing order of the level at which they become reachable from outside is exactly the minimising-the-largest-edge relaxation from Part 4. It is the same computation with a different question asked at the end.

CSES *Monsters* uses the multi-source form of the idea: run a search from every monster at once to determine when each cell becomes dangerous, then run a second search from the player through cells reachable strictly earlier. Starting a search from many sources requires only pushing all of them with distance zero before the loop begins, which is a small change with wide application.

---

## Part 6 · Negative weights and counting

**Bellman–Ford** relaxes every edge as many times as there are nodes, less one. If a further round still improves something, a negative cycle exists.

**CSES High Score** asks for the maximum score, so weights are negated and a negative cycle indicates an unbounded answer. The part that is easy to miss is that the cycle only matters if it can be reached from the start and can itself reach the destination, so a forward search from the source and a backward search from the sink are needed to filter the candidates.

**CSES Cycle Finding** asks you to print a negative cycle. Tracking predecessors and then, from a node relaxed in the final round, walking backwards as many steps as there are nodes ensures you are on the cycle rather than merely approaching it, after which following predecessors traces it out.

**CSES Investigation** runs one Dijkstra while tracking four quantities at each node: the distance, the number of shortest paths modulo a prime, the fewest edges on a shortest path, and the most. All four are updated together, and the pattern is the same in every path-counting problem:

```cpp
if (nd < dist[v]) { dist[v] = nd; cnt[v] = cnt[u]; mn[v] = mn[u]+1; mx[v] = mx[u]+1; push; }
else if (nd == dist[v]) { cnt[v] += cnt[u]; mn[v] = min(mn[v], mn[u]+1); mx[v] = max(mx[v], mx[u]+1); }
```

A strictly better distance replaces the accompanying values, and an equal distance combines them. Internalising that split makes every counting variant routine.

---

## Part 7 · The k shortest path lengths

**CSES Flight Routes** asks for the `k` shortest path lengths to a destination, with repetition permitted. The solution is simpler than expected: run Dijkstra but allow each node to be finalised up to `k` times, keeping a count per node, skipping a node that has already been finalised `k` times, and otherwise recording the distance and relaxing onwards.

This works because of a generalisation of the ordering property that makes Dijkstra correct in the first place: the `i`-th time a node is removed from the heap, the value removed is its `i`-th shortest distance.

---

## The ideas worth carrying forward

1. **Look at the edge weights before choosing an algorithm.** The choice between breadth-first search, its deque variant, Dijkstra, Bellman–Ford and topological dynamic programming follows directly from them.

2. **A deque handles weights of zero and one**, because the queue only ever holds two distinct distances and pushing to the correct end keeps it sorted.

3. **Ask what you would need to know besides your position, and put that into the node.** This is the most important idea in the chapter.

4. **Dijkstra works for any relaxation where extending a path never improves it**, which covers taking the largest edge, the smallest value, or a product of probabilities.

5. **Minimising the largest edge is solved by search with a threshold, by Dijkstra with a modified relaxation, and by adding edges in sorted order.** Three chapters address one class of problem.

6. **Starting from many sources requires only pushing all of them before the loop begins.**

7. **Bellman–Ford is a dynamic program over the number of edges used**, which is why running it a fixed number of rounds solves problems with a limit on stops.

8. **When counting shortest paths, a strictly better distance replaces and an equal distance combines.** The same rule applies to accompanying minimum and maximum edge counts.

9. **The `i`-th removal of a node from the heap gives its `i`-th shortest distance**, which turns Dijkstra into a k-shortest-paths algorithm with three extra lines.

10. **Time-dependent costs make the arrival time part of the state**, either explicitly in the node or implicitly through a waiting calculation applied during relaxation.

11. **Use the distance array as the visited marker.** Checking whether the value removed from the heap exceeds the recorded distance, and skipping if so, removes the need for a separate set and one more thing to keep synchronised.

---

## Where people lose these problems

**Using breadth-first search on a weighted graph.** It works on the samples and fails on tests with varied weights.

**Using Dijkstra with negative weights.** The result is silently wrong. Bellman–Ford is needed when negatives are possible.

**Not skipping stale heap entries.** Checking whether the removed distance is worse than the recorded one, immediately after removal, is necessary. Without it nodes are expanded repeatedly and the running time degrades badly.

**State explosion.** A node count multiplied by a million will not fit, so the product is worth computing before committing to a design.

**Sizing the distance array for the original graph.** It has to be sized for the enlarged one, and indexing it as a single flat array is usually cleaner than a nested container when performance matters.

**Overflow in the distance values.** The value used for infinity should be large enough to dominate any real distance but small enough that adding an edge weight does not wrap around, so around a quintillion is a reasonable choice.

**In LC 1368, inverting the sense of the cost.** Following the existing arrow is the free move and every other direction costs one, which is easy to reverse by accident.

**In CSES High Score, skipping a reachability filter.** A negative cycle must be reachable from the source and able to reach the sink, and omitting either check produces wrong answers on the tests written for exactly that purpose.

**In CF 20C, the routine details.** Distances reach around a hundred trillion, so 64-bit values are required, and the path itself must be printed, so predecessors have to be tracked. It is a test of whether your implementation is complete rather than of any idea.

---

## Working through the problem list

### Block 1 · The four algorithms

- **CSES Message Route** — *find a shortest route and print it.* Breadth-first search with path reconstruction.
- **CSES Labyrinth** — *find a shortest path through a grid and print the moves.* Reconstruction is the point, and worth practising.
- **CSES Shortest Routes I** — *single-source shortest paths.* Plain Dijkstra.
- **CSES Shortest Routes II** — *all-pairs shortest paths.* Floyd–Warshall, with the node count of at most five hundred being the signal.
- **CF 20C Dijkstra?** — *shortest path with the route printed.*
- **CSES Monsters** — *escape a maze while monsters chase you.* Multi-source search followed by a constrained one.

### Block 2 · Enlarging the node

- **CSES Flight Discount** — *use one discount coupon to halve a single flight.* The cleanest possible example, and worth doing first.
- **LC 787 Cheapest Flights Within K Stops** — *cheapest route using at most k stops.* Worth doing both as an enlarged search and as a fixed number of Bellman–Ford rounds.
- **LC 1129 Shortest Path with Alternating Colors** — *paths must alternate between red and blue edges.* The node carries the colour of the last edge.
- **LC 1293 Grid with Obstacles Elimination** — *cross a grid removing at most k obstacles.* The node carries the number of removals used.
- **LC 864 Shortest Path to Get All Keys** — *collect all keys, with doors requiring matching keys.* The node carries the set of keys.
- **LC 847 Shortest Path Visiting All Nodes** — *visit every node in the shortest walk.* The node carries the visited set, with all starting points seeded.
- **CF 59E Shortest Path** — *shortest path avoiding forbidden triples.* The node is a pair of consecutive vertices.
- **LC 1928 Minimum Cost to Reach Destination in Time** — *cheapest route completing within a time limit.* The node carries elapsed time.
- **LC 815 Bus Routes** — *fewest buses to travel between two stops.* The reframing described in Part 3.

### Block 3 · Weights of zero and one

- **LC 1368 Minimum Cost to Make at Least One Valid Path** — *change the fewest arrows to create a path across a grid.*
- **LC 2290 Minimum Obstacle Removal to Reach Corner** — *cross a grid removing as few obstacles as possible.*
- **CF 173B Chamber of Secrets** — *place mirrors so light reaches the far wall.* The modelling step, in which rows and columns become nodes, is where the difficulty sits.
- **CF 1063B Labyrinth** — *reach cells subject to limits on leftward and rightward moves.* Harder than it appears, since the two limits are linked and tracking one is sufficient.

### Block 4 · Changing the relaxation

- **LC 778 Swim in Rising Water** — *the rising-water grid.* Worth doing both ways, as the sheet suggests.
- **LC 1631 Path With Minimum Effort** — *minimise the largest height change along a route.* The same duality, and also solvable with union-find.
- **LC 2812 Find the Safest Path in a Grid** — *maximise the minimum distance from any thief.* Multi-source search followed by a maximising Dijkstra.
- **LC 407 Trapping Rain Water II** — *water trapped in a two-dimensional height map.* The heap-driven flood from Part 5, and the most elegant problem in this block.
- **LC 2577 Minimum Time to Visit a Cell In a Grid** — *cells become available at given times.* Waiting is permitted by stepping back and forth, so the parity of the wait matters, and handling that is the trick.

### Block 5 · Negative weights and counting

- **CSES Investigation** — *count shortest paths and report the fewest and most edges among them.* Four quantities relaxed together.
- **CSES High Score** — *maximise score, possibly unbounded.* Bellman–Ford with the reachability filter.
- **CSES Cycle Finding** — *find and print a negative cycle.*
- **CSES Flight Routes** — *the k shortest route lengths.*
- **LC 2045 Second Minimum Time to Reach Destination** — *the combined problem from Part 4.*

---

**A reasonable target here is around 75% of submissions passing first time.**

This is a large block with a great deal of surface area and comparatively few underlying ideas. When accuracy is low, the useful diagnostic question is whether you are choosing the wrong algorithm or describing the node wrongly, and it is nearly always the second.

---

## Check yourself

1. Edge weights are all zero or one. Which algorithm, and what makes it correct?
2. What question do you ask yourself to discover the extra information a node needs to carry?
3. Write the relaxation line for minimising the largest edge on a path, and name three techniques that solve that class of problem.
4. How does Bellman–Ford solve a limit on the number of stops, and what does that reveal about what it computes?
5. Write the rule for updating path counts when a better distance is found and when an equal one is found.
6. How do you obtain the k shortest path lengths from Dijkstra?
7. Why is the heap in LC 407 keyed by height, and what does removing the minimum represent?
8. In LC 815, what are the nodes, and why is that better than the obvious choice?
9. What check replaces a visited array in Dijkstra, and where does it go?
10. In CSES High Score, which two conditions must a negative cycle satisfy before it counts?
