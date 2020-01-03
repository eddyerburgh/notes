---
layout: default
title: Weighted graphs
description: Notes on weighted graph algorithms.
nav_order: 4
parent: Algorithms
grand_parent: Data structures and algorithms
permalink: /data-structures-and-algorithms/algorithms/weighted-graphs
---

<!-- prettier-ignore-start -->

# Weighted graphs
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

# Weighted Graph Algorithms

In a **weighted graph**, each edge is assigned a value (weight). For example, the edge in a road network might be assigned a value for drive time {% cite algorithm-design-manual -l 146 %}.

Weighted graphs are useful for modelling real-world problems where different paths have an associated cost, but they introduce extra complexity compared to unweighted graphs {% cite algorithm-design-manual -l 191 %}.

## Minimum Spanning Trees

A **spanning tree** of a graph $$G=(V,E)$$ is a subset of edges that form a tree connecting all vertices in $$V$$. A **minimum spanning tree** is a spanning tree with the lowest possible sum of all edges {% cite algorithm-design-manual -l 192 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/algorithms/weighted-graphs/spanning-tree.svg" alt="">
  <figcaption><h4>Figure: Spanning tree (edges in red) {% cite algorithm-design-manual -l 162 %}</h4></figcaption>
</figure>

Minimum spanning trees are useful for problems where you want to connect points together using the least amount of material. For example, connecting homes by the least amount of pipe {% cite algorithm-design-manual -l 192 %}.

### Prim's algorithm

Prim's algorithm is a greedy algorithm that starts from a single vertex and grows the rest of the tree one edge at a time until all vertices are included in the tree. At each step, Prim's algorithm chooses the lowest-weight edge available from the current tree to an unvisited vertex {% cite algorithm-design-manual -l 192 %}.

_Note: A **greedy algorithm** chooses its next move by making the optimal decision at each step {% cite algorithm-design-manual -l 192 %}._

See the pseudocode for Prim's algorithm:

```
Prim-MST(G)
  Select an arbitrary vertex s to start the tree from.
  While (there are still nontree vertices)
    Select the edge of minimum weight between a tree and nontree vertex
    Add the selected edge and vertex to the tree Tprim.
```

One implementation of Prim's algorithm is to keep track of which vertices are in the tree (`intree` in the following code), and to keep track of the minimum distance from the tree for each vertex not in the tree (`distance`):

```c
void prim(graph *g, int start) {
  int i; /* counter */
  edgenode *p; /* temporary pointer */
  bool intree[MAXV+1]; /* is the vertex in the tree yet? */
  int distance[MAXV+1]; /* cost of adding to tree */
  int v; /* current vertex to process */
  int w; /* candidate next vertex */
  int weight; /* edge weight */
  int dist; /* best current distance from start */

  // Reset value of each vertex
  for (i = 1; i <= g->nvertices; i++) {
    intree[i] = FALSE;
    distance[i] = MAX_INT;
    parent[i] = -1;
  }

  // Distance from start is 0, v is start
  distance[start] = 0;
  v = start;

  while (intree[v] == FALSE) {
    intree[v] = TRUE;
    p = g->edges[v];

    // Loop over each edge node (y) for current vertex
    // If the weight of the edge is less than the current distance[v]
    // set the parent of y to be v, set the distance of y to be the weight
    while (p != NULL) {
      w = p->y;
      weight = p->weight;
      if ((distance[w] > weight) && (intree[w] == FALSE)) {
        distance[w] = weight;
        parent[w] = v;
      }
      p = p->next;
    }

    v = 1;
    dist = MAXINT;

    for (i = 1; i <= g->nvertices; i++) {
      if ((intree[i] == FALSE) && (dist > distance[i])) {
        dist = distance[i];
        v = i;
      }
    }
  }
}
```

An improved implementation of Prim's algorithm uses a priority queue.

### Kruskal’s Algorithm

Kruskal’s algorithm is another greedy algorithm to find the minimum spanning tree.

The algorithm works by grouping vertices in connected components. Each vertex begins as its own connected component. The algorithm first sorts the edges by weight. It then iterates over each edge starting from the lowest weight, and tests whether the vertices of the edge are in the same connected component. If they aren't, then the edge can be added. The two connected components are then merged into one {% cite algorithm-design-manual -l 196 %}.

```
Kruskal-MST(G)
  Put the edges in a priority queue ordered by weight.
  count = 0
  while (count < n − 1) do
    get next edge (v,w)
    if (component (v) ≠ component(w))
      add to Tkruskal
      merge component(v) and component(w)
```

Checking that vertices are in the same components can be done in $$O(\log n)$$ by using a [union-find]({{ '/data-structures-and-algorithms/data-structures/union-find' | relative_url }}) data structure. This means the running time depends on the sort. If the sort is $$O(n\log n)$$ then the algorithm is $$O(m\log m)$$ (where $$m$$ is the number of edges) {% cite algorithm-design-manual -l 197 %}.

The following implementation uses a union-find:

```c
bool same_component(set_union *s, int s1, int s2) {
  return find(s,s1) == find(s,s2);
}
```

{% cite algorithm-design-manual -l 200 %}

```c
void kruskal(graph *g) {
  int i; /* counter */
  set_union s; /* set union data structure */
  edge_pair e[MAXV+1]; /* array of edges data structure */
  bool weight_compare();

  set_union_init(&s, g->nvertices);

  to_edge_array(g, e); /* sort edges by increasing cost */
  qsort(&e,g->nedges, sizeof(edge_pair), weight_compare);

  for (i = 0; i < g->nedges; i++) {
    if (!same_component(s, e[i].x, e[i].y)) {
      printf("edge (%d,%d) in MST\n", e[i].x, e[i].y);
      union_sets(&s, e[i].x, e[i].y);
    }
  }
}
```

{% cite algorithm-design-manual -l 198 %}

### Minimum spanning tree variations

There are many variations of minimum spanning tree:

- **Maximum spanning tree**: creates the maximum value path {% cite algorithm-design-manual -l 201 %}.

- **Minimum product spanning tree**: the minimum spanning tree when multiplying edge weights. This can be determined by running minimum weight spanning tree algorithms on the log of each path (since $$\lg(a\cdot b)=\lg(a)+\lg(b)$$) {% cite algorithm-design-manual -l 201 %}.

- **Minimum bottleneck spanning tree**: a tree that minimizes the maximum edge weight. Every minimum spanning tree has this property. {% cite algorithm-design-manual -l 201 %}

## Shortest paths

The shortest path problem is the problem of finding the shortest path between two vertices $$(x,y)$$ so that the sum of the edge weights is the minimum possible.

### Dijkstra's algorithm

Dijkstra's algorithm is a pathfinding algorithm.

The algorithm works by picking a new path from one of the discovered vertices to a new vertex. The new vertex is selected based on the total cost of the path to the new vertex {% cite algorithm-design-manual -l 207 %}.

Dijkstra's algorithm in pseudocode:

```
ShortestPath-Dijkstra(G,s,t)
  known = {s}
  for i = 1 to n, dist[i] = ∞
  for each edge (s,v), dist[v] = w(s,v)
  last = s
  while (last != t)
    select vnext, the unknown vertex minimizing dist[v]
    for each edge (vnext,x), dist[x] = min[dist[x],dist[vnext] + w(vnext,x)]
    last = vnext
    known = known ∪ {vnext}
```

{% cite algorithm-design-manual -l 207 %}

Dijkstra's algorithm is very similar to [Prim's algorithm](#prims-algorithm). In Prim's algorithm, only the cost of the next edge is considered. In Dijkstra's, it is the combined cost of the next edge and the cost of the path up to that vertex that is considered. {% cite algorithm-design-manual -l 207 %}

```c
dijkstra(graph *g, int start) {
  int i; /* counter */
  edgenode *p; /* temporary pointer */
  bool intree[MAXV+1]; /* is the vertex in the tree yet? */
  int distance[MAXV+1]; /* distance vertex is from start */
  int v; /* current vertex to process */
  int w; /* candidate next vertex */
  int weight; /* edge weight */
  int dist; /* best current distance from start */

  for (i=1; i<=g->nvertices; i++) {
    intree[i] = FALSE;
    distance[i] = MAXINT;
    parent[i] = -1;
  }

  distance[start] = 0;
  v = start;
  while (intree[v] == FALSE) {
    intree[v] = TRUE;
    p = g->edges[v];
    while (p != NULL) {
      w = p->y;
      weight = p->weight;
      if (distance[w] > (distance[v]+weight)) {
        distance[w] = distance[v]+weight;
        parent[w] = v;
      }
      p = p->next;
    }
    v = 1;
    dist = MAXINT;
    for (i=1; i<=g->nvertices; i++) {
      if ((intree[i] == FALSE) && (dist > distance[i])) {
        dist = distance[i];
        v = i;
      }
    }
  }
}
```

{% cite algorithm-design-manual -l 208 %}

The time complexity of Dijkstra's algorithm is $$O(n^2)$$.

_Note: Dijkstra's algorithm is only correct when run on graphs with non-negative edges {% cite algorithm-design-manual -l 210 %}._

You could run Dijkstra's algorithm on a graph with weighted vertices by converting the vertex costs to edge costs, before running an unmodified Dijkstra's over the new graph {% cite algorithm-design-manual -l 210 %}.

## All-pairs shortest path problem

The all-pairs shortest path problem involves determining the shortest path between each pair of vertices in a graph.

This could be solved by running Dijkstra's algorithm $$n$$ times. An alternative is the Floyd–Warshall algorithm.

### Floyd–Warshall algorithm

The Floyd–Warshall algorithm uses dynamic programming to calculate the shortest path between each pair of vertices in a graph. Unlike Dijkstra's algorithm, negative edges are allowed {% cite algorithm-design-manual -l 210 %}.

The algorithm works best on an adjacency matrix {% cite algorithm-design-manual -l 210 %}.

The adjacency matrix can be represented as a struct:

```c
typedef struct {
  int weight[MAXV+1][MAXV+1];
  int nvertices;
} adjacency_matrix;
```

For unweighted graphs, an edge between two vertices $$(x,y)$$ is often represented as a 1 in `weight[x][y]` and non-edges are represented as a 1.

For weighted graphs, an edge $$(x,y)$$ can be represented as the weight of the edge at `weight[x][y]`, and non-edges as infinity {% cite algorithm-design-manual -l 210 %}.

The Floyd–Warshall algorithm works by storing the cost from edge $$(x,y)$$ in `weight[x][y]`. The algorithm compares all possible paths through a graph between each edge by iterating over them. On each iteration, it checks the value of `weight[x][y]` with `weight[x][k] + weight[k][y]`. If the combined value of the edges $$(x,k)$$ and $$(k,y)$$ are lower than $$(x,y)$$, then the value stored at $$(x,y)$$ is replaced with the path from $$(x,k)$$ to $$(k,y)$$.

You can see an implementation of the algorithm:

```c
void floyd(adjacency_matrix *g) {
  int x,y;
  int k;
  int through_k;
  for (k = 1; k <= g->nvertices; k++) {
    for (x = 1; x <= g->nvertices; x++) {
      for (y = 1; y <= g->nvertices; y++) {
        through_k = g->weight[x][k] + g->weight[k][y];
        if (through_k < g->weight[x][y]) {
          g->weight[x][y] = through_k;
        }
      }
    }
  }
}
```

{% cite algorithm-design-manual -l 211 %}

The Floyd–Warshall algorithm runs in $$O(n^3)$$, the same as running Dijkstra's algorithm on each node. But Floyd's often has better performance than Dijkstra's in practice because the loops are so tight {% cite algorithm-design-manual -l 211 %}.

See a [video demonstration of the Floyd–Warshall algorithm](https://www.youtube.com/watch?v=4OQeCuLYj-4).

## References

{% bibliography --cited_in_order %}
