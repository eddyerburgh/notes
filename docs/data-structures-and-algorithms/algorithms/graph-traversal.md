---
layout: default
title: Graph traversal
description: Notes on graph traversal algorithms.
nav_order: 3
parent: Algorithms
grand_parent: Data structures and algorithms
permalink: /data-structures-and-algorithms/algorithms/graph-traversal
---

<!-- prettier-ignore-start -->

# Graph traversal
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

Traversing a [graph]({{ '/data-structures-and-algorithms/data-structures/graphs' | relative_url }}) to visit every edge and vertex is a common problem {% cite algorithm-design-manual -l 161 %}.

It's usually necessary to keep track of whether a vertex has been visited or not to avoid visiting vertices multiple times. This can be achieved by using a flag {% cite algorithm-design-manual -l 161 %}.

In this section, a vertex can exist in one of three states during traversal:

1. Undiscovered. The vertex has not been visited.
2. Discovered. The vertex has been visited but all its incident edges have not yet been visited.
3. Processed. The vertex and all its incident edges have been visited.

{% cite algorithm-design-manual -l 161 %}

To completely explore a vertex $$v$$, each edge leaving $$v$$ must be explored. Edges that go to a discovered or processed vertex are ignored {% cite algorithm-design-manual -l 161 %}.

An undirected edge will be considered twice, once from each vertex. Directed edges are only considered once {% cite algorithm-design-manual -l 161 %}.

This section will look at breadth-first search (BFS) and depth-first-search (DFS) algorithms for traversing a graph.

<!-- prettier-ignore-start -->
*Note: the code examples work on an adjacency list, as defined in the [graphs section]({{ '/data-structures-and-algorithms/data-structures/graphs' | relative_url }}).*
<!-- prettier-ignore-end -->

## Breadth-first search (BFS)

BFS (Breadth-First Search) is a common tree traversal algorithm.

When BFS is run on an undirected graph, a direction is assigned to each edge from the discoverer $$u$$ to the discovered $$v$$. Thus, $$u$$ is denoted as the parent of $$v$$ {% cite algorithm-design-manual -l 162 %}.

Apart from the root node, each node has one parent. This defines a tree of the vertices in a graph {% cite algorithm-design-manual -l 162 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/algorithms/graph-traversal/bfs-tree.svg" alt="">
  <figcaption><h4>Figure: Undirected graph and BFS tree {% cite algorithm-design-manual -l 162 %}</h4></figcaption>
</figure>

The following is the algorithm for BFS in pseudocode:

```
BFS(G,s)
  for each vertex u ∈ V [G] − {s} do
    state[u] = “undiscovered”
    p[u] = nil, i.e. no parent is in the BFS tree
  state[s] = “discovered”
  p[s] = nil
  Q = {s}
  while Q ≠ ∅ do
    u = dequeue[Q]
    process vertex u as desired
    for each v ∈ Adj[u] do
      process edge (u,v) as desired
      if state[v] = “undiscovered” then
        state[v] = “discovered”
        p[v] = u
        enqueue[Q,v]
    state[u] = “processed”
```

The following BFS implementation uses two arrays (`processed` and `discovered`) to store information about the state of a vertex (initially set to `false`). As well as a `parent` array used to store the parent of a vertex:

```c
bool processed[MAXV+1];
bool discovered[MAXV+1];
int parent[MAXV+1];

void initialize_search(graph *g) {
  for (int i = 1; i <= g->nvertices; i++) {
    processed[i] = discovered[i] = FALSE;
    parent[i] = -1;
  }
}
```

{% cite algorithm-design-manual -l 163 %}

In BFS, a vertex is placed on a queue when it's discovered. This means the oldest vertices (the ones closest to the root) are expanded first {% cite algorithm-design-manual -l 163 %}.

```c
void bfs(graph *g, int start) {
  queue q;
  int v;
  int y;
  edgenode *p;

  init_queue(&q);
  enqueue(&q,start);
  discovered[start] = TRUE;

  while (empty_queue(&q) == FALSE) {
    v = dequeue(&q);
    process_vertex_early(v);
    processed[v] = TRUE;
    p = g->edges[v];

    while (p != NULL) {
      y = p->y;
      if ((processed[y] == FALSE) || g->directed) {
        process_edge(v, y);
      }
      if (discovered[y] == FALSE) {
        enqueue(&q, y);
        discovered[y] = TRUE;
        parent[y] = v;
      }
      p = p->next;
    }
    process_vertex_late(v);
  }
}
```

The behavior of BFS is defined by the functions `process_vertex_early()`, `process_edge()`, and `process_vertex_late()` {% cite algorithm-design-manual -l 164 %}.

The vertex that discovered vertex `i` is at `parent[i]`. Because vertices are discovered in order of increasing distance from the root node, the tree path from each node $$x\in V$$ uses the minimum number of edges on a root-to-x path {% cite algorithm-design-manual -l 165 %}.

The path from a vertex can be reconstructed by following the chain of parents:

```c
void find_path(int start, int end, int parents[]) {
  if ((start == end) || (end == -1)) {
    printf("\n%d",start);
  } else {
    find_path(start, parents[end], parents);
    printf(" %d",end);
  }
}
```

{% cite algorithm-design-manual -l 165 %}

- The BFS shortest path tree from $$x$$ to $$y$$ is only useful if $$x$$ is the root of the tree.
- BFS only gives shortest path if path is unweighted.

{% cite algorithm-design-manual -l 166 %}

BFS runs in $$O(n + m)$$ time on a graph of $$n$$ vertices and $$m$$ edges {% cite algorithm-design-manual -l 166 %}.

### Connected components

A connected component of an undirected graph is a set of vertices where there is a path between each pair of vertices {% cite algorithm-design-manual -l 166 %}.

Components are separate pieces of a graph if there is no connection between the pieces {% cite algorithm-design-manual -l 166 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/algorithms/graph-traversal/connected-components.svg" alt="">
  <figcaption><h4>Figure: Connected components</h4></figcaption>
</figure>

Many problems can be reduced to finding or counting connected components. Connected components can be found with BFS {% cite algorithm-design-manual -l 166 %}:

```c
void connected_components(graph *g) {
  int c; /* component number */
  int i; /* counter */

  initialize_search(g);
  c = 0;

  for (i=1; i<=g->nvertices; i++) {
    if (discovered[i] == FALSE) {
      c = c+1;
      printf("Component %d:",c);
      bfs(g,i);
      printf("\n");
    }
  }
}
```

### Two-Coloring Graphs

The vertex-coloring problem seeks to assign a label (color) to each vertex of a graph so that no edge links two vertices of the same color. The goal is to use as few colors as possible {% cite algorithm-design-manual -l 167 %}.

Vertex coloring often occurs in scheduling applications, like register allocation in a compiler {% cite algorithm-design-manual -l 167 %}.

A graph is **bipartite** if it can be scheduled with only two colors {% cite algorithm-design-manual -l 168 %}.

BFS can be updated to determine whether a graph is bipartite. It works by coloring a newly discovered vertex the opposite of its parent. If any non-discovery edge links two vertices of the same color, the graph is not bipartite. If there are no conflicts during BFS, then the graph is bipartite {% cite algorithm-design-manual -l 168 %}:

```c
void twocolor(graph *g) {
  int i;

  for (i = 1; i <= (g->nvertices); i++) {
    color[i] = UNCOLORED;
  }

  bipartite = TRUE;

  initialize_search(&g);

  for (i = 1; i <= (g->nvertices); i++) {
    if (discovered[i] == FALSE) {
      color[i] = WHITE;
      bfs(g,i);
    }
  }
}

void process_edge(int x, int y) {
  if (color[x] == color[y]) {
    bipartite = FALSE;
    printf("Warning: not bipartite due to (%d,%d)\n",x,y);
  }
  color[y] = complement(color[x]);
}

void complement(int color) {
  if (color == WHITE) {
    return(BLACK);
  }
  if (color == BLACK) {
    return(WHITE);
  }
  return(UNCOLORED);
}
```

## Depth-first Search (DFS)

DFS (Depth-first search) is an alternative method for visiting a graph.

The difference between DFS and BFS is the order that they visit nodes in. DFS visits all children in a path, before backing up to previous nodes {% cite algorithm-design-manual -l 169 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/algorithms/graph-traversal/dfs-tree.svg" alt="">
  <figcaption><h4>Figure: Undirected graph and DFS tree {% cite algorithm-design-manual -l 171 %}</h4></figcaption>
</figure>

DFS uses a stack to store discovered nodes that need to be processed (instead of a queue like BFS) {% cite algorithm-design-manual -l 169 %}.

DFS can also be defined recursively (implicitly using the language call stack), which eliminates the use of an explicit stack {% cite algorithm-design-manual -l 169 %}:

```
DFS(G,u)
  state[u] = “discovered”
  process vertex u if desired
  entry[u] = time
  time = time + 1
  for each v ∈ Adj[u] do
    process edge (u,v) if desired
    if state[v] = “undiscovered” then
      p[v] = u
      DFS(G,v)
  state[u] = “processed”
  exit[u] = time
  time = time + 1
```

{% cite algorithm-design-manual -l 169-70 %}

Time intervals in DFS are useful for:

- Determining an ancestor. An ancestor will have a lower entry time than its descendants {% cite algorithm-design-manual -l 170 %}.
- Determining number of descendants. The difference in entry and exit time determines how many descendants a vertex has {% cite algorithm-design-manual -l 170 %}.

A DFS partitions edges of an undirected graph into tree edges and back edges. **Tree edges** discover new vertices, and are encoded in the `parent` relation {% cite algorithm-design-manual -l 170 %}.

**Back edges** are edges whose other endpoint is an ancestor of the vertex being expanded {% cite algorithm-design-manual -l 170 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/algorithms/graph-traversal/tree-edge-and-back-edge.svg" alt="">
  <figcaption><h4>Figure: DFS tree with tree edges and back edge</h4></figcaption>
</figure>

The following implementation uses recursion rather than a user-defined stack:

```c
void dfs(graph *g, int v) {
  edgenode *p;
  int y;

  if (finished) {
    return;
  }

  discovered[v] = TRUE;
  time = time + 1;
  entry_time[v] = time;

  process_vertex_early(v);

  p = g->edges[v];
  while (p != NULL) {
    y = p->y;
    if (discovered[y] == FALSE) {
      parent[y] = v;
      process_edge(v,y);
      dfs(g,y);
    } else if ((!processed[y]) || (g->directed)) {
      process_edge(v,y);
      if (finished) {
        return;
      }
      p = p->next;
    }
  }

  process_vertex_late(v);

  time = time + 1;
  exit_time[v] = time;

  processed[v] = TRUE;
}
```

{% cite algorithm-design-manual -l 171-2 %}

The behavior of DFS depends on when the vertex is processed. It can be either processed before outgoing edges have been traversed (`process_vertex_early()`) or after they have been processed (`process_vertex_late()`) {% cite algorithm-design-manual -l 172 %}.

In undirected graphs, each edge $$(x,y)$$ is in the adjacency list of $$x$$ and $$y$$. This means there are two times to process each edge, once when exploring $$x$$ and once when exploring $$y$$. Normally the first time you see an edge is the time to do processing, but sometimes you may want to perform an action when you see an edge for the second time {% cite algorithm-design-manual -l 172 %}.

You can determine whether an edge $$(x,y)$$ has not been traversed yet if $$y$$ is undiscovered. You can determine whether an edge _has_ been traversed before if $$y$$ is discovered but not processed. If $$y$$ is discovered and is an ancestor of $$x$$ (and thus in a discovered state) then it is the first time it has been traversed, unless $$y$$ is an immediate ancestor of $$x$$ {% cite algorithm-design-manual -l 173 %}.

### Finding cycles

If there is a back edge from $$(x, y)$$ then a cycle exists between $$x$$ and $$y$$. If there are no back edges, then there are no cycles {% cite algorithm-design-manual -l 173 %}.

You can check for cycles with `dfs()` by modifying `process_edge()`:

```c
void process_edge(int x, int y) {
  if (parent[x] != y) {
    printf("Cycle from %d to %d:",y,x);
    find_path(y,x,parent);
    printf("\n\n");
    finished = TRUE;
  }
}
```

{% cite algorithm-design-manual -l 173 %}

The cycle detection algorithm is only correct if each undirected edge is processed exactly once {% cite algorithm-design-manual -l 173 %}.

### Articulation Vertices

An **articulation vertex** is a node that will disconnect a connected component of a graph if it is deleted {% cite algorithm-design-manual -l 174 %}.

The **connectivity** of a graph is the smallest number of vertices whose deletion will disconnect the graph. If a graph has an articulation vertex then the connectivity of the graph is 1 {% cite algorithm-design-manual -l 174 %}.

A **bridge** is a single edge whose deletion would disconnect the graph. A graph without a bridge edge is **edge-biconnected** {% cite algorithm-design-manual -l 177 %}.

You can use DFS to check for articulation vertices in linear time. Note that a back edge from $$x$$ to $$y$$ ensures that none of the vertices on the path from $$x$$ to $$y$$ are articulation vertices. Finding articulation vertices requires maintaining the extent to which back edges link chunks of the DFS tree back to ancestor nodes {% cite algorithm-design-manual -l 174-5 %}.

Let `reachable_ancestors[v]` denote the earliest reachable ancestor of vertex v. Initially, `reachable_ancestors[v] = v`:

```c
int reachable_ancestor[MAXV+1];
int tree_out_degree[MAXV+1];

void process_vertex_early(int v) {
  reachable_ancestor[v] = v;
}
```

{% cite algorithm-design-manual -l 175 %}

`reachable_ancestors[v]` is updated whenever a back edge is encountered that takes it to an earlier ancestor (the age of ancestors can be calculated from their `entry_time`) {% cite algorithm-design-manual -l 175 %}.

```c
void process_edge(int x, int y) {
  int class = edge_classification(x, y);

  if (class == TREE) {
    tree_out_degree[x] = tree_out_degree[x] + 1;
  }

  if ((class == BACK) && (parent[x] != y)) {
    if (entry_time[y] < entry_time[reachable_ancestor[x]] )
     reachable_ancestor[x] = y;
    }
}
```

There are three cases of articulation vertices:

1. Root cut-nodes. If the root of a DFS tree has two or more children then it must be an articulation vertex.
2. Bridge cut-nodes. If the earliest reachable vertex from $$v$$ is $$v$$, then deleting the edge $$(parent[v], v)$$ would disconnect the graph.
3. Parent cut-nodes. If the earliest reachable vertex from $$v$$ is the parent of $$v$$, then the parent of $$v$$ is an articulation vertex (unless the parent of $$v$$ is the root).

{% cite algorithm-design-manual -l 176 %}

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/algorithms/graph-traversal/cutnodes.svg" alt="">
  <figcaption><h4>Figure: Different cases of articulation vertices {% cite algorithm-design-manual -l 176 %}</h4></figcaption>
</figure>

The following code checks for each of these cut-nodes after a vertex's edges have been processed:

```c
void process_vertex_late(int v) {
  bool root; /* is the vertex the root of the DFS tree? */
  int time_v; /* earliest reachable time for v */
  int time_parent; /* earliest reachable time for parent[v] */

  if (parent[v] < 1) {
    if (tree_out_degree[v] > 1) {
      printf("root articulation vertex: %d \n",v);
    }
    return;
  }

  root = (parent[parent[v]] < 1); /* is parent[v] the root? */
  if ((reachable_ancestor[v] == parent[v]) && (!root)) {
    printf("parent articulation vertex: %d \n",parent[v]);
  }

  if (reachable_ancestor[v] == v) {
    printf("bridge articulation vertex: %d \n",parent[v]);
    if (tree_out_degree[v] > 0) {
      printf("bridge articulation vertex: %d \n",v);
    }
  }

  time_v = entry_time[reachable_ancestor[v]];
  time_parent = entry_time[reachable_ancestor[parent[v]]];

  if (time_v < time_parent) {
    reachable_ancestor[parent[v]] = reachable_ancestor[v];
  }
}
```

{% cite algorithm-design-manual -l 165 %}

## DFS on directed graphs

In an undirected graph, every edge is either a tree edge or a back edge {% cite algorithm-design-manual -l 178 %}.

In directed graphs, there are more possibilities for edge labelling:

- Tree edge
- Forward edge
- Back edge
- Cross edge

{% cite algorithm-design-manual -l 178 %}

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/algorithms/graph-traversal/edge-cases.svg" alt="">
  <figcaption><h4>Figure: Edge cases for BFS/DFS traversal {% cite algorithm-design-manual -l 178 %}</h4></figcaption>
</figure>

The labelling can be calculated from the discovery time, state, and parent of each vertex:

```c
int edge_classification(int x, int y) {
  if (parent[y] == x) {
    return(TREE);
  }
  if (discovered[y] && !processed[y]) {
    return(BACK);
  }
  if (processed[y] && (entry_time[y]>entry_time[x])) {
    return(FORWARD);
  }
  if (processed[y] && (entry_time[y]<entry_time[x])) {
    return(CROSS);
  }
  printf("Warning: unclassified edge (%d,%d)\n",x,y);
}
```

### Topological sorting

Topological sorting works on DAGs (Directed Acyclic Graphs). Topological sorting orders vertices so that "all directed edges go from left to right" {% cite algorithm-design-manual -l 179 %}.

Every DAG has at least one topological sort {% cite algorithm-design-manual -l 179 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/algorithms/graph-traversal/dag.svg" alt="">
  <figcaption><h4>Figure: DAG with one topological sort (G,A,B,C,F,E,D) {% cite algorithm-design-manual -l 179 %}</h4></figcaption>
</figure>

Topological sort is useful because it creates an order that can be used to process a vertex before any of its successors {% cite algorithm-design-manual -l 179 %}.

You can find the topological order of a DAG during DFS by labelling vertices in the reverse order that they are marked as processed {% cite algorithm-design-manual -l 180 %}.

The following implements a topological sort:

```c
void process_vertex_late(int v) {
  push(&sorted,v);
}

void process_edge(int x, int y) {
  int class;

  class = edge_classification(x,y);

  if (class == BACK) {
    printf("Warning: directed cycle found, not a DAG\n");
  }
}

void topsort(graph *g) {
  int i;
  init_stack(&sorted);

  for (i = 1; i <= g->nvertices; i++) {
    if (discovered[i] == FALSE) {
      dfs(g,i);
    }
  }

  print_stack(&sorted);
}
```

### Strongly Connected Components

A directed graph is strongly connected if there is a path between all pairs of vertices {% cite algorithm-design-manual -l 181 %}.

Graphs that aren't strongly connected can be partitioned into **strongly connected components** {% cite algorithm-design-manual -l 181 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/algorithms/graph-traversal/strongly-connected-components.svg" alt="">
  <figcaption><h4>Figure: The strongly-connected components of a graph, with the associated DFS tree {% cite algorithm-design-manual -l 182 %}</h4></figcaption>
</figure>

You can calculate strongly connected components in linear time with [Tarjan's strongly connected components algorithm](https://en.wikipedia.org/wiki/Tarjan%27s_strongly_connected_components_algorithm), which uses DFS.

(see a [video demonstration of Tarjan's strongly connected components algorithm](https://www.youtube.com/watch?v=TyWtx7q2D7Y&feature=youtu.be&t=389)).

## References

{% bibliography --cited_in_order %}
