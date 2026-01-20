
---

1. Single source short path: 
	1. Unit weigth = BFS `O(V+E)`
	2. 0/1 weight = 0/1 bfs (using deque) `O(V+E)`
	3. Multisource bfs/ dijsktra (used when multiple start or end points are given)
	4. (0,inf) weight = Dijsktra `O((V+E)logV)`
	5. (-inf,inf) = Bellman ford algorithm `O(VE)`
2. Multi source shortest path
	1. Floyd warshell `O(V^3)`
