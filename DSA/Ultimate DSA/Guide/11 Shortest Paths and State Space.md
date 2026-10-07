
---

Read [[3. Shortest paths]] before this. The structure of the problems roughly for hard problems has three common variations as listed below - 

1. **The node is a combination rather than a single thing.** A position together with the remaining fuel, the set of keys collected, the number of flights used, or the colour of the last edge taken.
2. **The cost of moving depends on when you arrive.** Traffic lights, tides, or teleporters running on a schedule.
3. **The graph is implicit and very large, but the reachable part is small.** Transforming one string into another, or reaching a number from one using a set of permitted operations.


### Putting extra information into the node

This is the central idea of the chapter.

The procedure is to ask what you would need to know, besides your current position, in order to make the right next move. Whatever that is becomes part of the node, and the edges of the enlarged graph both move you and update that extra information.

**CSES Flight Discount** allows one coupon halving the cost of a single flight. A node becomes a city together with whether the coupon has been used. From an unused state you may travel at full price and remain unused, or travel at half price and become used. From a used state you travel at full price only. Running Dijkstra on twice as many nodes solves it.

**LC 787 Cheapest Flights Within K Stops** makes a node a city together with the number of stops used. There is also a neater formulation: running Bellman–Ford for exactly `k + 1` rounds works, because each round relaxes paths using one more edge. That second view is worth understanding because it reveals what Bellman–Ford actually computes, which is a dynamic program over the number of edges used.

**LC 864 Shortest Path to Get All Keys** makes a node a cell together with the set of keys held. With at most six keys there are sixty-four possible sets, so the state count is manageable, and since every move costs one, plain breadth-first search suffices.

**LC 847 Shortest Path Visiting All Nodes** makes a node a position together with the set of nodes already visited, starting the search from every node simultaneously. This is chapter [[15 Bitmask DP]] in the shape of a graph search.

**CF 59E Shortest Path** forbids certain triples of consecutive vertices, so the right decision depends on the previous two positions, and a node becomes a pair of consecutive vertices. This is the case where the extra information is a short window of history.

**LC 815 Bus Routes** deserves separate mention because the reframing is different in kind. Rather than enlarging the node, it replaces it: the search runs over bus routes rather than bus stops, with two routes adjacent when they share a stop. Choosing what the nodes should be is itself the problem.

Once the state is chosen, the remaining check is arithmetic. A node count multiplied by sixty-four is fine, and a node count multiplied by a million is not, so it is worth computing the product before committing.

### Broader disjktra condition

Dijkstra's algorithm requires only that extending a path never improves it, which is a broader condition than requiring the cost to be a sum of non-negative weights. This should be remebered and applied whenever possible. 

**Minimising the largest edge on a path.** Define the distance to a node as the smallest possible value of the largest edge along any path reaching it. The relaxation becomes:

```cpp
nd = max(dist[u], w(u,v));           // instead of dist[u] + w
if (nd < dist[v]) { dist[v] = nd; pq.push({nd, v}); }
```

Everything else is unchanged.  And why is it true again reason is that broarder condition requiring for disjktra to work is metric in calculation should work in such a way that extending a path never improves it. Here once you have a path and then you try to extend it via some edge it can never be improved since the smaller path has to be atleast of cost as of bigger path. 

**Maximising the smallest value on a path** is the mirror image: take the minimum during relaxation and use a maximum-heap.

The same skeleton also covers finding the widest path, the most reliable path when edges carry probabilities, and the path with the fewest colour changes.

**LC 2045 Second Minimum Time to Reach Destination** combines two variations, which is what makes it a genuine hard problem. It needs the second shortest distinct distance, which means keeping the two best values at each node and relaxing into the second when a value beats it without equalling the first. It also has costs that depend on arrival time, since arriving during a red light means waiting until it turns green. Neither variation is difficult alone.

### Traversing on the transformed graph

Many times problems force you to change the graph itself you are traversing on for example consider problem of finding shortest path with alternate colors. The problem can be solved by maintaining constrains like last node traverse color edge along with node in queue and moving forward if and only if colors are alternating. COnsider alogirithm 

```cpp
vector<vector<int>> dist(n,vector<int> (2,1e9));
queue<pair<int,int>> q;
dist[0][0]=dist[0][1]=0;

q.push({0,0});
q.push({0,1});

while(!q.empty()){
	auto [node,type]=q.front();
	q.pop();

	for(auto e:adj[node]){
		// skip unused edge
		if(e.second==type)continue;
		if(dist[e.first][e.second]>dist[node][type]+1){
			dist[e.first][e.second]=dist[node][type]+1;
			q.push({e.first,e.second});
		}
	}
}
```

Now observe the two things although this algorithm can be considered somewhat modified version of actual bfs with constraints. But to prove that it actually works it is important to visualize this as running on state graph where each node is `(node,color of last edge)`. There are edges if and only if two nodes have different colors. Now we can prove things on transformed graph - 

- Every alternate path gets exposed - Every alternate path exsits in transformed graph by definition. So our algorithm traverses everything reachable. 
- No invalid path gets explored is true due to condition where we skip invalid edges. 

Since alternating paths in the original graph and paths in the state graph correspond one-to-one with the same lengths, plain BFS on the state graph gives the shortest alternating distance.

The reason `dist[node]` alone would be wrong is that the best alternating path may need to pass through a node twice, or reach it by a longer route, to arrive with the right color. With one `visited` flag per node, the first arrival would block the second, even though the two have different futures: one can only leave on blue edges, the other only on red. Two arrivals at the same `(node, color)`, however, have identical futures, so keeping only the shortest is safe.

