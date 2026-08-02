# Graph Algorithms — The Complete Revision Guide 🧠

## Table of Contents

| # | Topic | Jump |
|---|---|---|
| 1 | [Graph Representations](#1-graph-representations) | Adj List / Matrix / Edge List |
| 2 | [DFS](#2-depth-first-search-dfs) | Stack / Recursion |
| 3 | [BFS](#3-breadth-first-search-bfs) | Queue / Grid / Multi-source |
| 4 | [Topological Sort](#4-topological-sort) | Kahn's / DFS-based |
| 5 | [Shortest Path on DAG](#5-shortestlongest-path-on-dag) | Topo-order relaxation |
| 6 | [Dijkstra's](#6-dijkstras-algorithm) | Greedy SSSP |
| 7 | [Bellman-Ford](#7-bellman-ford-algorithm) | Negative weights SSSP |
| 8 | [Floyd-Warshall](#8-floyd-warshall-all-pairs-shortest-path) | All-pairs shortest path |
| 9 | [Bridges & Articulation Points](#9-bridges--articulation-points) | Cut edges / vertices |
| 10 | [Tarjan's SCC](#10-tarjans-strongly-connected-components) | Strongly connected components |
| 11 | [Eulerian Paths & Circuits](#11-eulerian-paths--circuits) | Visit every edge once |
| 12 | [Prim's MST](#12-prims-minimum-spanning-tree) | Lazy + Eager |
| 13 | [TSP — Travelling Salesman](#13-travelling-salesman-problem-dp--bitmask) | DP + Bitmask |
| 14 | [Network Flow](#14-network-flow) | Ford-Fulkerson / Edmonds-Karp / Dinic's |
| 15 | [Master Cheat Sheet](#15-master-decision-cheat-sheet) | When to use what |

---

## 1. Graph Representations

### Adjacency List (⭐ Most Common)
```java
// Unweighted
List<List<Integer>> graph = new ArrayList<>();
for (int i = 0; i < n; i++) graph.add(new ArrayList<>());
graph.get(u).add(v);  // directed edge u → v

// Weighted
List<List<int[]>> graph = new ArrayList<>();
for (int i = 0; i < n; i++) graph.add(new ArrayList<>());
graph.get(u).add(new int[]{v, weight});
```

### Adjacency Matrix
```java
int[][] graph = new int[n][n];  // 0 = no edge, value = weight
graph[u][v] = weight;

// Good for: Dense graphs, Floyd-Warshall, checking if edge exists in O(1)
```

### Edge List
```java
int[][] edges = {{u1,v1,w1}, {u2,v2,w2}, ...};

// Good for: Bellman-Ford, Kruskal's MST
```

### When to Use What

| Representation | Space | Check Edge | Iterate Neighbors | Best For |
|---|---|---|---|---|
| Adj List | O(V+E) | O(degree) | O(degree) | Sparse graphs, BFS/DFS |
| Adj Matrix | O(V²) | O(1) | O(V) | Dense graphs, Floyd-Warshall |
| Edge List | O(E) | O(E) | O(E) | Bellman-Ford, Kruskal |

---

## 2. Depth First Search (DFS)

> **Mental model**: "Go as deep as possible, then backtrack."

### Template — Recursive
```java
boolean[] visited = new boolean[n];

void dfs(int node) {
    visited[node] = true;
    // process node here

    for (int neighbor : graph.get(node)) {
        if (!visited[neighbor]) {
            dfs(neighbor);
        }
    }
    // post-processing here (useful for topo sort, SCC, etc.)
}
```

### Template — Iterative (Stack)
```java
void dfs(int start) {
    boolean[] visited = new boolean[n];
    Stack<Integer> stack = new Stack<>();
    stack.push(start);

    while (!stack.isEmpty()) {
        int node = stack.pop();
        if (visited[node]) continue;  // ← key: skip already visited
        visited[node] = true;

        // process node here

        for (int neighbor : graph.get(node)) {
            if (!visited[neighbor]) {
                stack.push(neighbor);
            }
        }
    }
}
```

### When to Use DFS
- Connected components
- Cycle detection
- Topological sorting
- Finding bridges / articulation points
- Strongly connected components
- Path finding (not shortest path)
- Flood fill

### Complexity
- **Time**: O(V + E)
- **Space**: O(V) for visited + O(V) recursion stack

> [!WARNING]
> For large graphs (V > 10⁴), use iterative DFS to avoid stack overflow.

---

## 3. Breadth First Search (BFS)

> **Mental model**: "Explore layer by layer, like ripples in water."

### Template — Standard BFS
```java
int[] bfs(int start) {
    int[] dist = new int[n];
    Arrays.fill(dist, -1);
    Queue<Integer> queue = new LinkedList<>();

    dist[start] = 0;
    queue.offer(start);

    while (!queue.isEmpty()) {
        int node = queue.poll();

        for (int neighbor : graph.get(node)) {
            if (dist[neighbor] == -1) {          // not visited
                dist[neighbor] = dist[node] + 1;
                queue.offer(neighbor);
            }
        }
    }

    return dist;  // dist[i] = shortest distance from start to i
}
```

### Template — BFS on Grid (Shortest Path)
```java
int bfsGrid(int[][] grid, int sr, int sc, int er, int ec) {
    int rows = grid.length, cols = grid[0].length;
    boolean[][] visited = new boolean[rows][cols];
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};  // R, L, D, U

    Queue<int[]> queue = new LinkedList<>();
    queue.offer(new int[]{sr, sc, 0});  // row, col, distance
    visited[sr][sc] = true;

    while (!queue.isEmpty()) {
        int[] curr = queue.poll();
        int r = curr[0], c = curr[1], d = curr[2];

        if (r == er && c == ec) return d;  // reached target

        for (int[] dir : dirs) {
            int nr = r + dir[0], nc = c + dir[1];
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols
                && !visited[nr][nc] && grid[nr][nc] != 1) {  // 1 = wall
                visited[nr][nc] = true;    // ← mark BEFORE adding to queue
                queue.offer(new int[]{nr, nc, d + 1});
            }
        }
    }

    return -1;  // no path found
}
```

> [!TIP]
> **Mark visited BEFORE adding to queue**, not when popping. This prevents duplicate entries and saves memory.

### Template — Multi-Source BFS ⭐
> Start BFS from **multiple sources simultaneously**. All sources are at distance 0.

```java
// Example: Find distance of every cell from the nearest source
int[][] multiSourceBFS(int[][] grid, List<int[]> sources) {
    int rows = grid.length, cols = grid[0].length;
    int[][] dist = new int[rows][cols];
    for (int[] row : dist) Arrays.fill(row, -1);
    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};

    Queue<int[]> queue = new LinkedList<>();

    // ⭐ Add ALL sources to queue at once with distance 0
    for (int[] src : sources) {
        dist[src[0]][src[1]] = 0;
        queue.offer(new int[]{src[0], src[1]});
    }

    while (!queue.isEmpty()) {
        int[] curr = queue.poll();
        int r = curr[0], c = curr[1];

        for (int[] dir : dirs) {
            int nr = r + dir[0], nc = c + dir[1];
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols
                && dist[nr][nc] == -1) {
                dist[nr][nc] = dist[r][c] + 1;
                queue.offer(new int[]{nr, nc});
            }
        }
    }

    return dist;
}
```

> [!IMPORTANT]
> **Multi-source BFS trick**: Instead of running BFS from each source separately (O(k × V)), add all sources to the queue upfront. The BFS naturally computes the shortest distance from the **nearest** source for every cell. Same O(V + E) complexity!

**LeetCode problems using Multi-Source BFS**:
- LC #542 — 01 Matrix (distance to nearest 0)
- LC #994 — Rotting Oranges (all rotten oranges spread simultaneously)
- LC #1162 — As Far from Land as Possible
- LC #286 — Walls and Gates

### When to Use BFS vs DFS

| Use BFS when | Use DFS when |
|---|---|
| Shortest path (unweighted) | Detecting cycles |
| Level-order traversal | Topological sort |
| Nearest neighbor problems | Connected components |
| Multi-source distance | Path existence (not shortest) |
| Minimum steps / moves | Backtracking problems |

---

## 4. Topological Sort

> **Mental model**: "Order tasks so that every dependency comes before the task that depends on it."
> Only works on **Directed Acyclic Graphs (DAGs)**.

### Template A — Kahn's Algorithm (BFS-based) ⭐ (Preferred)
```java
List<Integer> topoSort(int n, List<List<Integer>> graph) {
    int[] indegree = new int[n];

    // Step 1: Compute in-degrees
    for (int u = 0; u < n; u++)
        for (int v : graph.get(u))
            indegree[v]++;

    // Step 2: Add all nodes with in-degree 0 to queue
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < n; i++)
        if (indegree[i] == 0) queue.offer(i);

    // Step 3: Process
    List<Integer> order = new ArrayList<>();
    while (!queue.isEmpty()) {
        int node = queue.poll();
        order.add(node);

        for (int neighbor : graph.get(node)) {
            indegree[neighbor]--;
            if (indegree[neighbor] == 0)  // all dependencies met
                queue.offer(neighbor);
        }
    }

    // Step 4: Cycle detection
    if (order.size() != n) return null;  // ⚠️ cycle exists! Not a DAG.
    return order;
}
```

### Template B — DFS-based
```java
List<Integer> topoSortDFS(int n, List<List<Integer>> graph) {
    boolean[] visited = new boolean[n];
    boolean[] inStack = new boolean[n];      // for cycle detection
    LinkedList<Integer> order = new LinkedList<>();

    for (int i = 0; i < n; i++)
        if (!visited[i])
            if (hasCycle(i, graph, visited, inStack, order))
                return null;  // cycle detected

    return order;
}

boolean hasCycle(int node, List<List<Integer>> graph,
                 boolean[] visited, boolean[] inStack, LinkedList<Integer> order) {
    visited[node] = true;
    inStack[node] = true;

    for (int neighbor : graph.get(node)) {
        if (inStack[neighbor]) return true;     // cycle!
        if (!visited[neighbor])
            if (hasCycle(neighbor, graph, visited, inStack, order))
                return true;
    }

    inStack[node] = false;
    order.addFirst(node);  // ← add to FRONT after all descendants processed
    return false;
}
```

> [!TIP]
> **Prefer Kahn's** for interviews — it's iterative, naturally detects cycles (order.size() != n), and is easier to reason about.

**LeetCode**: LC #207 Course Schedule, LC #210 Course Schedule II, LC #269 Alien Dictionary

---

## 5. Shortest/Longest Path on DAG

> **Key insight**: On a DAG, you can find shortest/longest path in **O(V + E)** — faster than Dijkstra! Process nodes in topological order and relax edges.

### Template
```java
int[] shortestPathDAG(int n, List<List<int[]>> graph, int source) {
    // Step 1: Get topological order
    List<Integer> topoOrder = topoSort(n, graph);

    // Step 2: Initialize distances
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[source] = 0;

    // Step 3: Relax edges in topological order
    for (int u : topoOrder) {
        if (dist[u] == Integer.MAX_VALUE) continue;  // unreachable

        for (int[] edge : graph.get(u)) {
            int v = edge[0], w = edge[1];
            if (dist[u] + w < dist[v]) {          // relaxation
                dist[v] = dist[u] + w;
            }
        }
    }

    return dist;
}

// For LONGEST path: just negate all weights, or flip the comparison:
// if (dist[u] + w > dist[v]) dist[v] = dist[u] + w;
// Initialize dist to Integer.MIN_VALUE, dist[source] = 0
```

### When to Use
- Shortest path in a DAG (faster than Dijkstra)
- Longest path in a DAG (NP-hard in general graphs, but **linear** on DAGs!)
- Critical path in project scheduling

**Complexity**: O(V + E)

---

## 6. Dijkstra's Algorithm

> **Mental model**: "Always expand the closest unvisited node. Like a growing circle."
> Works on graphs with **non-negative weights only**.

### Template (Priority Queue)
```java
int[] dijkstra(int n, List<List<int[]>> graph, int source) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[source] = 0;

    // Min-heap: {distance, node}
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
    pq.offer(new int[]{0, source});

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int d = curr[0], u = curr[1];

        if (d > dist[u]) continue;  // ⭐ stale entry, skip (lazy deletion)

        for (int[] edge : graph.get(u)) {
            int v = edge[0], w = edge[1];
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.offer(new int[]{dist[v], v});
            }
        }
    }

    return dist;
}
```

### The Key Line: `if (d > dist[u]) continue;`
This is **lazy deletion**. We don't remove old entries from the heap — we just skip them when popped. This is crucial for correctness with Java's PriorityQueue (which doesn't support decrease-key).

### When to Use Dijkstra vs Others

| Algorithm | When to Use | Complexity |
|---|---|---|
| BFS | Unweighted graphs | O(V + E) |
| DAG relaxation | DAGs (even negative weights) | O(V + E) |
| **Dijkstra** | **Non-negative weights** | **O((V+E) log V)** |
| Bellman-Ford | Negative weights / detect neg cycles | O(V × E) |
| Floyd-Warshall | All-pairs shortest path | O(V³) |

> [!CAUTION]
> **Dijkstra FAILS with negative weights.** Once a node is finalized, it's never revisited. A negative edge could later provide a shorter path, but Dijkstra won't catch it.

**LeetCode**: LC #743 Network Delay Time, LC #787 Cheapest Flights, LC #1514 Path with Maximum Probability

---

## 7. Bellman-Ford Algorithm

> **Mental model**: "Relax ALL edges V-1 times. If anything still relaxes on the Vth pass, there's a negative cycle."

### Template
```java
int[] bellmanFord(int n, int[][] edges, int source) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[source] = 0;

    // Step 1: Relax all edges V-1 times
    for (int i = 0; i < n - 1; i++) {
        for (int[] edge : edges) {  // edge = {u, v, weight}
            int u = edge[0], v = edge[1], w = edge[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
            }
        }
    }

    // Step 2: Detect negative cycles (one more pass)
    for (int[] edge : edges) {
        int u = edge[0], v = edge[1], w = edge[2];
        if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
            // Negative cycle detected!
            // Mark reachable nodes from this cycle as -∞
            dist[v] = Integer.MIN_VALUE;
        }
    }

    return dist;
}
```

### Why V-1 iterations?
The shortest path between any two nodes has **at most V-1 edges**. Each iteration guarantees at least one more node gets its correct shortest distance. So after V-1 iterations, all shortest paths are found.

### When to Use
- Graph has **negative weight edges**
- Need to **detect negative weight cycles**
- LC #787 Cheapest Flights Within K Stops (modified: limit iterations to K+1)

**Complexity**: O(V × E) — slower than Dijkstra, but handles negative weights.

---

## 8. Floyd-Warshall (All-Pairs Shortest Path)

> **Mental model**: "For every pair (i, j), try every possible intermediate node k. Is going through k shorter?"

### Template
```java
int[][] floydWarshall(int n, int[][] graph) {
    // graph[i][j] = weight of edge i→j, or INF if no edge
    int[][] dist = new int[n][n];
    int INF = (int) 1e9;

    // Step 1: Initialize
    for (int i = 0; i < n; i++) {
        Arrays.fill(dist[i], INF);
        dist[i][i] = 0;  // distance to self is 0
    }
    // Copy known edges
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            if (graph[i][j] != 0) dist[i][j] = graph[i][j];

    // Step 2: DP — try each intermediate node k
    for (int k = 0; k < n; k++) {           // ⚠️ k MUST be the outermost loop
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (dist[i][k] != INF && dist[k][j] != INF) {
                    dist[i][j] = Math.min(dist[i][j], dist[i][k] + dist[k][j]);
                }
            }
        }
    }

    // Step 3: Detect negative cycles (optional)
    for (int k = 0; k < n; k++)
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                if (dist[i][k] != INF && dist[k][j] != INF && dist[i][k] + dist[k][j] < dist[i][j])
                    dist[i][j] = -INF;  // affected by negative cycle

    return dist;
}
```

> [!CAUTION]
> **k MUST be the outermost loop.** The DP recurrence is:
> `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])` — "best path from i to j using nodes {0..k} as intermediates."
> If k isn't outermost, the DP ordering is wrong and you get incorrect results.

### When to Use
- You need shortest path between **ALL pairs** of nodes
- Graph is **small** (V ≤ 400, since O(V³))
- Detecting negative cycles in dense graphs
- Transitive closure (can node i reach node j?)

**Complexity**: O(V³) time, O(V²) space

---

## 9. Bridges & Articulation Points

> **Bridge** = an edge whose removal disconnects the graph.
> **Articulation Point** = a node whose removal disconnects the graph.

### Template — Finding Bridges
```java
int timer = 0;
int[] disc, low;
List<int[]> bridges = new ArrayList<>();

void findBridges(int n, List<List<Integer>> graph) {
    disc = new int[n];
    low = new int[n];
    Arrays.fill(disc, -1);

    for (int i = 0; i < n; i++)
        if (disc[i] == -1)
            dfs(i, -1, graph);
}

void dfs(int u, int parent, List<List<Integer>> graph) {
    disc[u] = low[u] = timer++;

    for (int v : graph.get(u)) {
        if (v == parent) continue;       // don't go back to parent

        if (disc[v] == -1) {             // unvisited — tree edge
            dfs(v, u, graph);
            low[u] = Math.min(low[u], low[v]);  // pull up low from child

            if (low[v] > disc[u]) {      // ⭐ BRIDGE condition
                bridges.add(new int[]{u, v});
            }
        } else {                         // visited — back edge
            low[u] = Math.min(low[u], disc[v]);  // pull up low from ancestor
        }
    }
}
```

### Template — Finding Articulation Points
```java
// Same DFS, but the condition changes:
void dfs(int u, int parent, List<List<Integer>> graph) {
    disc[u] = low[u] = timer++;
    int childCount = 0;

    for (int v : graph.get(u)) {
        if (v == parent) continue;

        if (disc[v] == -1) {
            childCount++;
            dfs(v, u, graph);
            low[u] = Math.min(low[u], low[v]);

            // ⭐ Articulation point conditions:
            if (parent == -1 && childCount > 1)   // root with 2+ children
                isAP[u] = true;
            if (parent != -1 && low[v] >= disc[u]) // non-root: child can't reach above u
                isAP[u] = true;
        } else {
            low[u] = Math.min(low[u], disc[v]);
        }
    }
}
```

### Key Concepts

| Term | Meaning |
|---|---|
| `disc[u]` | Discovery time of node u |
| `low[u]` | Lowest disc time reachable from subtree of u via back edges |
| **Bridge** | Edge (u,v) where `low[v] > disc[u]` — subtree of v can't reach u or above |
| **Articulation Point** | Node u where `low[v] >= disc[u]` — removing u cuts off v's subtree |

**LeetCode**: LC #1192 Critical Connections in a Network

**Complexity**: O(V + E)

---

## 10. Tarjan's Strongly Connected Components

> **SCC** = a maximal set of vertices where every vertex is reachable from every other vertex (in a **directed** graph).

### Template
```java
int timer = 0, sccCount = 0;
int[] disc, low, sccId;
boolean[] onStack;
Stack<Integer> stack = new Stack<>();

void tarjanSCC(int n, List<List<Integer>> graph) {
    disc = new int[n]; low = new int[n]; sccId = new int[n];
    onStack = new boolean[n];
    Arrays.fill(disc, -1);

    for (int i = 0; i < n; i++)
        if (disc[i] == -1) dfs(i, graph);
}

void dfs(int u, List<List<Integer>> graph) {
    disc[u] = low[u] = timer++;
    stack.push(u);
    onStack[u] = true;

    for (int v : graph.get(u)) {
        if (disc[v] == -1) {
            dfs(v, graph);
            low[u] = Math.min(low[u], low[v]);
        } else if (onStack[v]) {       // ⭐ only update from nodes ON the stack
            low[u] = Math.min(low[u], disc[v]);
        }
    }

    // If u is a root of an SCC
    if (low[u] == disc[u]) {           // ⭐ SCC root: low == disc
        while (true) {
            int v = stack.pop();
            onStack[v] = false;
            sccId[v] = sccCount;
            if (v == u) break;
        }
        sccCount++;
    }
}
```

### Key Insight
- `low[u] == disc[u]` means u is the **root** of an SCC. Everything above u on the stack belongs to this SCC.
- Only update `low[u]` from nodes that are **on the stack** (not from already-finished SCCs).

### When to Use
- Finding SCCs (obviously)
- **2-SAT** problems
- Condensing a directed graph into a DAG of SCCs

**Complexity**: O(V + E)

---

## 11. Eulerian Paths & Circuits

> **Eulerian Path** = visits every **edge** exactly once.
> **Eulerian Circuit** = Eulerian path that starts and ends at the same node.

### Existence Conditions

| Type | Undirected Graph | Directed Graph |
|---|---|---|
| **Eulerian Circuit** | Every node has even degree | Every node: in-degree = out-degree |
| **Eulerian Path** | Exactly 0 or 2 nodes with odd degree | Exactly one node: out - in = 1 (start), exactly one: in - out = 1 (end), rest equal |
| **Neither** | More than 2 odd-degree nodes | Anything else |

### Template — Hierholzer's Algorithm
```java
LinkedList<Integer> eulerianPath(int n, List<List<Integer>> graph) {
    // Step 1: Check existence & find start node (omitted for brevity)
    int start = findStartNode();

    // Step 2: Hierholzer's Algorithm
    LinkedList<Integer> path = new LinkedList<>();
    Stack<Integer> stack = new Stack<>();
    stack.push(start);

    while (!stack.isEmpty()) {
        int u = stack.peek();
        if (!graph.get(u).isEmpty()) {
            int v = graph.get(u).remove(graph.get(u).size() - 1);  // take an edge
            stack.push(v);
        } else {
            path.addFirst(stack.pop());  // ⭐ add to FRONT when backtracking
        }
    }

    return path;  // verify path.size() == totalEdges + 1
}
```

> [!TIP]
> **Hierholzer's trick**: When you're stuck (no more edges from current node), add to the FRONT of the result and backtrack. This naturally handles branches.

**LeetCode**: LC #332 Reconstruct Itinerary, LC #753 Cracking the Safe

**Complexity**: O(V + E)

---

## 12. Prim's Minimum Spanning Tree

> **Mental model**: "Grow the MST one edge at a time. Always pick the cheapest edge that connects a new node to the tree."

### Template — Lazy Prim's (with Priority Queue)
```java
int primMST(int n, List<List<int[]>> graph) {
    boolean[] inMST = new boolean[n];
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]); // {weight, node}
    int mstCost = 0, edgesUsed = 0;

    // Start from node 0
    inMST[0] = true;
    for (int[] edge : graph.get(0))
        pq.offer(new int[]{edge[1], edge[0]});  // {weight, neighbor}

    while (!pq.isEmpty() && edgesUsed < n - 1) {
        int[] curr = pq.poll();
        int w = curr[0], u = curr[1];

        if (inMST[u]) continue;         // ⭐ lazy deletion — skip stale edges

        inMST[u] = true;
        mstCost += w;
        edgesUsed++;

        for (int[] edge : graph.get(u)) {
            if (!inMST[edge[0]])
                pq.offer(new int[]{edge[1], edge[0]});
        }
    }

    return (edgesUsed == n - 1) ? mstCost : -1;  // -1 if graph not connected
}
```

### Lazy vs Eager Prim's

| | Lazy Prim's | Eager Prim's |
|---|---|---|
| **Heap stores** | All candidate edges (may have stale ones) | Best edge per node (uses indexed PQ) |
| **Deletion** | Lazy — skip stale entries on pop | Eager — decrease-key updates in-place |
| **Complexity** | O(E log E) | O(E log V) |
| **Implementation** | Simple with standard PQ | Needs an Indexed Priority Queue |
| **Use in interviews** | ⭐ Preferred (simpler) | Only if asked specifically |

> [!TIP]
> For interviews, **Lazy Prim's** is almost always sufficient. Eager Prim's is mainly a theoretical improvement.

### Prim's vs Kruskal's

| | Prim's | Kruskal's |
|---|---|---|
| Approach | Grow tree from a node | Sort edges, add if no cycle |
| Data structure | Priority Queue | Union-Find |
| Better for | Dense graphs | Sparse graphs, edge list input |
| Complexity | O(E log V) | O(E log E) |

**LeetCode**: LC #1584 Min Cost to Connect All Points, LC #1135 Connecting Cities With Minimum Cost

---

## 13. Travelling Salesman Problem (DP + Bitmask)

> **Problem**: Visit every node exactly once and return to start. Minimize total cost.
> **NP-hard** in general, but solvable in O(n² × 2ⁿ) with DP + bitmask for small n (≤ 20).

### Template
```java
int tsp(int n, int[][] dist) {
    int FULL_MASK = (1 << n) - 1;
    int[][] dp = new int[1 << n][n];  // dp[mask][i] = min cost to visit nodes in mask, ending at i
    for (int[] row : dp) Arrays.fill(row, Integer.MAX_VALUE);

    dp[1][0] = 0;  // start at node 0, only node 0 visited

    for (int mask = 1; mask <= FULL_MASK; mask++) {
        for (int u = 0; u < n; u++) {
            if (dp[mask][u] == Integer.MAX_VALUE) continue;
            if ((mask & (1 << u)) == 0) continue;  // u not in mask

            // Try extending to each unvisited node
            for (int v = 0; v < n; v++) {
                if ((mask & (1 << v)) != 0) continue;   // v already visited
                if (dist[u][v] == Integer.MAX_VALUE) continue;

                int newMask = mask | (1 << v);
                dp[newMask][v] = Math.min(dp[newMask][v], dp[mask][u] + dist[u][v]);
            }
        }
    }

    // Find min cost to visit all nodes and return to start
    int ans = Integer.MAX_VALUE;
    for (int u = 0; u < n; u++) {
        if (dp[FULL_MASK][u] != Integer.MAX_VALUE && dist[u][0] != Integer.MAX_VALUE) {
            ans = Math.min(ans, dp[FULL_MASK][u] + dist[u][0]);
        }
    }

    return ans;
}
```

### Bitmask Cheat Sheet

| Operation | Code | Meaning |
|---|---|---|
| Check if bit i is set | `(mask >> i) & 1` or `mask & (1 << i)` | Is node i visited? |
| Set bit i | `mask \| (1 << i)` | Mark node i as visited |
| Clear bit i | `mask & ~(1 << i)` | Unvisit node i |
| Count set bits | `Integer.bitCount(mask)` | How many nodes visited |
| All bits set | `(1 << n) - 1` | All nodes visited |

**Complexity**: O(n² × 2ⁿ) — feasible for n ≤ 20

**LeetCode**: LC #943 Find the Shortest Superstring, LC #847 Shortest Path Visiting All Nodes

---

## 14. Network Flow

### 14.1 Core Concepts

| Term | Meaning |
|---|---|
| **Flow Network** | Directed graph with source s, sink t, and edge capacities |
| **Max Flow** | Maximum amount of "stuff" that can flow from s to t |
| **Residual Graph** | Graph showing remaining capacity + reverse edges for flow cancellation |
| **Augmenting Path** | Path from s to t in the residual graph with available capacity |
| **Min Cut** | Minimum total capacity of edges to remove to disconnect s from t |
| **Max-Flow Min-Cut Theorem** | Max Flow = Min Cut (always!) |

---

### 14.2 Ford-Fulkerson Method

> **Mental model**: "Keep finding augmenting paths and pushing flow through them until no more paths exist."

```java
// Conceptual framework — specific implementations below
maxFlow = 0
while (augmenting path P exists from s to t in residual graph):
    bottleneck = min capacity along P
    push bottleneck flow along P
    update residual graph (decrease forward, increase reverse)
    maxFlow += bottleneck
return maxFlow
```

> [!IMPORTANT]
> Ford-Fulkerson is a **method**, not an algorithm. It doesn't specify HOW to find augmenting paths. The choice of path-finding strategy gives us different algorithms:
> - BFS → **Edmonds-Karp** (O(VE²))
> - BFS + scaling → **Capacity Scaling** (O(E² log C))
> - BFS for level graph + DFS → **Dinic's** (O(V²E))

---

### 14.3 Edmonds-Karp Algorithm

> Ford-Fulkerson + **BFS** for augmenting paths.

```java
int edmondsKarp(int n, int[][] capacity, int s, int t) {
    int[][] residual = new int[n][n];
    for (int i = 0; i < n; i++)
        System.arraycopy(capacity[i], 0, residual[i], 0, n);

    int maxFlow = 0;

    while (true) {
        // BFS to find augmenting path
        int[] parent = new int[n];
        Arrays.fill(parent, -1);
        parent[s] = s;
        Queue<int[]> queue = new LinkedList<>();
        queue.offer(new int[]{s, Integer.MAX_VALUE});

        while (!queue.isEmpty()) {
            int[] curr = queue.poll();
            int u = curr[0], flow = curr[1];

            for (int v = 0; v < n; v++) {
                if (parent[v] == -1 && residual[u][v] > 0) {
                    parent[v] = u;
                    int newFlow = Math.min(flow, residual[u][v]);
                    if (v == t) {
                        // Found path to sink — update residual graph
                        maxFlow += newFlow;
                        int node = t;
                        while (node != s) {
                            int prev = parent[node];
                            residual[prev][node] -= newFlow;  // forward edge
                            residual[node][prev] += newFlow;  // reverse edge ⭐
                            node = prev;
                        }
                        break;
                    }
                    queue.offer(new int[]{v, newFlow});
                }
            }
            if (parent[t] != -1) break;  // path found, restart BFS
        }

        if (parent[t] == -1) break;  // no more augmenting paths
    }

    return maxFlow;
}
```

**Complexity**: O(V × E²)

---

### 14.4 Dinic's Algorithm (Fastest for interviews)

> **Mental model**: "Build level graph with BFS, then push multiple flows with DFS. Repeat."

```java
// Using adjacency list with Edge objects for efficiency
class Edge {
    int to, rev;  // rev = index of reverse edge in adj[to]
    int cap;
    Edge(int to, int rev, int cap) {
        this.to = to; this.rev = rev; this.cap = cap;
    }
}

List<List<Edge>> graph;
int[] level, iter;

void addEdge(int from, int to, int cap) {
    graph.get(from).add(new Edge(to, graph.get(to).size(), cap));
    graph.get(to).add(new Edge(from, graph.get(from).size() - 1, 0)); // reverse edge
}

boolean bfs(int s, int t) {
    level = new int[graph.size()];
    Arrays.fill(level, -1);
    Queue<Integer> queue = new LinkedList<>();
    level[s] = 0;
    queue.offer(s);

    while (!queue.isEmpty()) {
        int u = queue.poll();
        for (Edge e : graph.get(u)) {
            if (e.cap > 0 && level[e.to] < 0) {  // has capacity & unvisited
                level[e.to] = level[u] + 1;
                queue.offer(e.to);
            }
        }
    }
    return level[t] >= 0;  // can reach sink?
}

int dfs(int u, int t, int pushed) {
    if (u == t) return pushed;

    for (; iter[u] < graph.get(u).size(); iter[u]++) {
        Edge e = graph.get(u).get(iter[u]);
        if (e.cap > 0 && level[e.to] == level[u] + 1) {  // only go to next level
            int d = dfs(e.to, t, Math.min(pushed, e.cap));
            if (d > 0) {
                e.cap -= d;
                graph.get(e.to).get(e.rev).cap += d;  // reverse edge
                return d;
            }
        }
    }
    return 0;
}

int maxFlow(int s, int t) {
    int flow = 0;
    while (bfs(s, t)) {           // build level graph
        iter = new int[graph.size()];  // reset DFS iterators
        int d;
        while ((d = dfs(s, t, Integer.MAX_VALUE)) > 0) {  // push all blocking flows
            flow += d;
        }
    }
    return flow;
}
```

**Complexity**: O(V² × E) — in practice much faster, especially on unit-capacity graphs: O(E√V)

---

### 14.5 Capacity Scaling

> **Idea**: Only consider "big" augmenting paths first. Start with paths that can push at least Δ flow, then halve Δ.

```
Δ = largest power of 2 ≤ max capacity
while Δ ≥ 1:
    while augmenting path with bottleneck ≥ Δ exists:
        push flow along path
    Δ = Δ / 2
```

**Complexity**: O(E² log C) where C = max capacity

---

### 14.6 Bipartite Matching via Network Flow

> **Problem**: Given a bipartite graph, find the maximum matching.

### Reduction to Max Flow
```
1. Create source s and sink t
2. s → every node in left set (capacity 1)
3. Every node in right set → t (capacity 1)
4. Original edges: left → right (capacity 1)
5. Max flow = maximum matching
```

```java
// If left = {0..m-1}, right = {m..m+n-1}, source = m+n, sink = m+n+1
int bipartiteMatching(int m, int n, List<int[]> edges) {
    int S = m + n, T = m + n + 1;
    // Build flow network
    for (int i = 0; i < m; i++) addEdge(S, i, 1);
    for (int j = 0; j < n; j++) addEdge(m + j, T, 1);
    for (int[] e : edges) addEdge(e[0], m + e[1], 1);

    return maxFlow(S, T);  // using Dinic's or Edmonds-Karp
}
```

> [!TIP]
> For simple bipartite matching, you can also use the **Hungarian Algorithm** or **Hopcroft-Karp** (O(E√V)), but the flow reduction works perfectly and is easier to code in interviews.

### When to Use Which Flow Algorithm

| Algorithm | Complexity | Best For |
|---|---|---|
| Edmonds-Karp | O(VE²) | Simple to implement, small graphs |
| Capacity Scaling | O(E² log C) | Large capacities |
| **Dinic's** | O(V²E) | **Best general-purpose**, unit-capacity graphs |
| Hopcroft-Karp | O(E√V) | Bipartite matching specifically |

---

## 15. Master Decision Cheat Sheet

### "I have a graph problem. What algorithm do I use?"

```
Is it a SHORTEST PATH problem?
├── Unweighted? → BFS
├── DAG? → Topological Sort + relaxation (O(V+E))
├── Non-negative weights?
│   ├── Single source → Dijkstra (O((V+E) log V))
│   └── All pairs → Run Dijkstra from each node or Floyd-Warshall
├── Negative weights?
│   ├── Single source → Bellman-Ford (O(VE))
│   ├── Detect negative cycles → Bellman-Ford (Vth iteration check)
│   └── All pairs → Floyd-Warshall (O(V³))
│
Is it a CONNECTIVITY problem?
├── Connected components? → DFS/BFS or Union-Find
├── Strongly connected components? → Tarjan's SCC
├── Bridges / cut edges? → Tarjan's bridge algorithm
├── Articulation points / cut vertices? → Modified Tarjan's
│
Is it an ORDERING problem?
├── Task scheduling / dependencies? → Topological Sort (Kahn's)
├── Cycle detection in directed graph? → Topo sort (fails) or DFS coloring
│
Is it a SPANNING TREE problem?
├── Minimum spanning tree? → Prim's (dense) or Kruskal's (sparse)
│
Is it a PATH/CIRCUIT problem?
├── Visit every EDGE once? → Eulerian Path (check conditions + Hierholzer's)
├── Visit every NODE once? → TSP / Hamiltonian (DP + bitmask, n ≤ 20)
│
Is it a FLOW / MATCHING problem?
├── Maximum flow? → Dinic's algorithm
├── Bipartite matching? → Max flow reduction or Hopcroft-Karp
├── Min cut? → Max flow (they're equal!)
│
GRID problem?
├── Shortest path? → BFS
├── Distance from nearest X? → Multi-source BFS
├── Flood fill / connected regions? → DFS or BFS
```

### Complexity Quick Reference

| Algorithm | Time | Space |
|---|---|---|
| DFS / BFS | O(V + E) | O(V) |
| Multi-source BFS | O(V + E) | O(V) |
| Topological Sort | O(V + E) | O(V) |
| Dijkstra (min-heap) | O((V+E) log V) | O(V) |
| Bellman-Ford | O(V × E) | O(V) |
| Floyd-Warshall | O(V³) | O(V²) |
| Bridges / Art. Points | O(V + E) | O(V) |
| Tarjan's SCC | O(V + E) | O(V) |
| Eulerian Path | O(V + E) | O(V + E) |
| Prim's MST | O(E log V) | O(V) |
| Kruskal's MST | O(E log E) | O(V) |
| TSP (DP + bitmask) | O(n² × 2ⁿ) | O(n × 2ⁿ) |
| Edmonds-Karp | O(V × E²) | O(V²) |
| Dinic's | O(V² × E) | O(V + E) |

---

## TL;DR — The 5 Graph Commandments

1. **Unweighted shortest path?** → BFS. Always.
2. **Weighted, no negative edges?** → Dijkstra. Don't forget `if (d > dist[u]) continue;`
3. **Negative edges?** → Bellman-Ford. V-1 iterations, Vth detects cycles.
4. **Need ordering / dependencies?** → Topo Sort (Kahn's). If `order.size() != n`, there's a cycle.
5. **Flow / matching?** → Dinic's. Model it as source → left → right → sink.

Master these five, and you can derive everything else. 🎯
