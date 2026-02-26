
---

A graph consists of nodes and edges. In this book, the variable n denotes the number of nodes in a graph, and the variable m denotes the number of edges.

A path leads from node a to node b though edges of the graph. The length of a path is the number of edges in it. A path is a cycle if the first and last node is the same. A path is simple if each node appears at most once in the path. A graph is connected if there is a path between any two nodes. A tree is a connected graph that consists of n nodes and n − 1 edges. There is a unique path between any two nodes of a tree.

A tree is a connected graph that consists of n nodes and n − 1 edges. There is a unique path between any two nodes of a tree. A tree is a connected graph that consists of n nodes and n − 1 edges. There is a unique path between any two nodes of a tree.

Two nodes are neighbors or adjacent if there is an edge between them. The degree of a node is the number of its neighbors.

The sum of degrees in a graph is always 2m, where m is the number of edges, because each edge increases the degree of exactly two nodes by one

A graph is regular if the degree of every node is a constant d. A graph is complete if the degree of every node is n − 1, i.e., the graph contains all possible edges between the nodes.

In a directed graph, the indegree of a node is the number of edges that end at the node, and the outdegree of a node is the number of dges that start at the node.

In a coloring of a graph, each node is assigned a color so that no adjacent nodes have the same color. A graph is bipartite if it is possible to color it using two colors. It turns out that a graph is bipartite exactly when it does not contain a cycle with an odd number of edges.

A graph is simple if no edge starts and ends at the same node, and there are no multiple edges between two nodes.

## Representation:

In the adjacency list representation, each node x in the graph is assigned an adjacency list that consists of nodes to which there is an edge from x. 

An adjacency matrix is a two-dimensional array that indicates which edges the graph contains. We can efficiently check from an adjacency matrix if there is an edge between two node

## Traversals

### DFS

Depth-first search (DFS) is a straightforward graph traversal technique. The algorithm begins at a starting node, and proceeds to all other nodes that are reachable from the starting node using the edges of the graph. Depth-first search always follows a single path in the graph as long as it finds new nodes. After this, it returns to previous nodes and begins to explore other parts of the graph. The algorithm keeps track of visited nodes, so that it processes each node only once.

```cpp
void dfs(int node){
	if(visited[node])return;
	visited[node]=1;
	
	for(auto child:adj[node]){
		dfs(child);
	}
}
```

### BFS

Breadth-first search (BFS) visits the nodes in increasing order of their distance from the starting node. Thus, we can calculate the distance from the starting node to all other nodes using breadth-first search

```cpp
visited[x] = true;
distance[x] = 0;
q.push(x);
while (!q.empty()) {
	int s = q.front(); q.pop();
	// process node s
	for (auto u : adj[s]) {
	if (visited[u]) continue;
	visited[u] = true;
	distance[u] = distance[s]+1;
	q.push(u);
	}
}
```

## Applications

1. Connectivity check - Do dfs/bfs check visited after that
2. Finding cycles: A graph contains a cycle if during a graph traversal, we find a node whose neighbor (other than the previous node in the current path) has already been visited.  Another way to find out whether a graph contains a cycle is to simply calculate the number of nodes and edges in every component. If a component contains c nodes and no cycle, it must contain exactly c − 1 edges (so it has to be a tree). If there are c or more edges, the component surely contains a cycle.
3. A graph is bipartite if its nodes can be colored using two colors so that there are no adjacent nodes with the same color. It is surprisingly easy to check if a graph is bipartite using graph traversal algorithms. The idea is to color the starting node blue, all its neighbors red, all their neighbors blue, and so on. If at some point of the search we notice that two adjacent nodes have the same color, this means that the graph is not bipartite. Otherwise the graph is bipartite and one coloring has been found
4. Number of connected components/size of largest component

```cpp
int mx_size = 0 , no_of_components = 0;
for(int node=0;node<n;node++){
	if(!vis[node]){
		int size = dfs(node);
		mx_size = max(mx_size , size);
		no_of_components++;
	}
}
```

