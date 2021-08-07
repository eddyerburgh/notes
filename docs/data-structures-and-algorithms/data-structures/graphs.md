---
layout: default
title: Graphs
description: Notes on the graph data structure.
has_toc: false
parent: Data structures
grand_parent: Data structures and algorithms
permalink: /data-structures-and-algorithms/data-structures/graphs
---

<!-- prettier-ignore-start -->

# Graphs
{:.no_toc}

Graphs can be used to represent almost any relationship: from road networks, to social relationships, to module dependencies. Many algorithmic problems can be solved by modelling data as a graph.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

A graph $$G=(V,E)$$ consists of a set of vertices $$V$$ and a set of edges $$E$$ {% cite algorithm-design-manual -l 145 %}.

A **vertex** is a node in a graph. An **edge** represents a path between two vertices.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/graphs/graph.svg" alt="">
  <figcaption><h4>Figure: A graph</h4></figcaption>
</figure>

The **degree** of a vertex is the number of edges that are incident to the vertex {% cite algorithm-design-manual -l 150 %}.

_Definition: an edge ($$u,v$$) is **incident** to the vertices $$u$$ and $$v$$._

Typical graph operations are:

| operation          | description                                                   |
| ------------------ | ------------------------------------------------------------- |
| `add_edge(x,y)`    | Add the edge $$(x,y)$$ to $$E$$.                              |
| `remove_edge(x,y)` | Remove the edge $$(x,y)$$ from $$E$$.                         |
| `has_edge(x,y)`    | Check if edge $$(x,y)\in E$$.                                 |
| `out_edges(x)`     | Return a List of all integers $$y$$ such that $$(x,y)\in E$$. |
| `in_edges(x)`      | Return a List of all integers $$y$$ such that $$(y,x)\in E$$. |

{% cite open-data-structures -l 248 %}

## Graph properties

Graph properties affect the data structure used to represent the graph and the algorithms available to interact with them.

### Undirected vs directed

A graph $$G=(V,E)$$ is undirected if an edge $$(x,y)$$ in set $$E$$ means that $$(x,y)$$ is also in set $$E$$. If this is not the case, then the graph is a **directed graph** {% cite algorithm-design-manual -l 146 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/graphs/undirected-vs-directed-graph.svg" alt="">
  <figcaption><h4>Figure: An undirected graph and a directed graph</h4></figcaption>
</figure>

Directed graphs are common when representing dependencies, which have a parent-child relationship.

### Weighted vs unweighted

In a **weighted graph** each edge is assigned a value (a weight). For example, the edge in a road network might be assigned a cost for drive time {% cite algorithm-design-manual -l 146 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/graphs/weighted-graph.svg" alt="">
  <figcaption><h4>Figure: A weighted graph</h4></figcaption>
</figure>

In an unweighted graph there is no value associated with an edge {% cite algorithm-design-manual -l 147 %}.

### Simple vs non-simple

A simple graph is a graph without self-loops or multiple edges {% cite algorithm-design-manual -l 147 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/graphs/non-simple-graph.svg" alt="">
  <figcaption><h4>Figure: A non-simple graph with a self-loop and multiple edges</h4></figcaption>
</figure>

A non-simple graph can contain these types of edges {% cite algorithm-design-manual -l 147 %}.

### Sparse vs dense

A graph is sparse if only a small number of the possible vertex pairs have edges defined between them.

A dense graph is a graph where lots of the possible vertex pairs have edges defined between them {% cite algorithm-design-manual -l 148 %}.

Dense graphs typically have a quadratic number of edges {% cite algorithm-design-manual -l 148 %}.

### Cyclic vs acyclic

Cyclic graphs contain cycles, Acyclic graphs don't {% cite algorithm-design-manual -l 148 %}.

Trees are acyclic graphs {% cite algorithm-design-manual -l 148 %}.

### Labelled vs unlabelled

In a labelled graph, each vertex is assigned a unique name. In an unlabelled graph, vertices do not have an associated name {% cite algorithm-design-manual -l 149 %}.

## Representing a graph

There are two common ways to represent a graph $$G=(V,E)$$:

- Adjacency matrix
- Adjacency list

### Adjacency matrix

An **adjacency matrix** is an $$n$$x$$n$$ matrix $$M$$, where $$n$$ is the number of vertices.

$$M[x, y] = 1$$ if $$(x, y)$$ is an edge of $$G$$, and $$0$$ if it isn't {% cite algorithm-design-manual -l 151 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/graphs/adjacency-matrix.svg" alt="">
  <figcaption><h4>Figure: A graph and its adjacency matrix</h4></figcaption>
</figure>

Common operations like `add_edge()`, `remove_edge()`, and `has_edge()` are $$O(1)$$ operations:

```c
void add_edge(int x, int y) {
  a[x][y] = 1;
}

void remove_edge(int x, int y) {
  a[x][y] = 0;
}

bool has_edge(int ix, int y) {
  return a[x][y];
}
```

Methods returning edges for a given node, e.g. `out_edges()`/`in_edges()`, are $$O(n)$$ operations. An entire row/column must be checked to find the edges.

Another problem with adjacency matrices is that they quickly grow large. For this reason they are often impractical.

### Adjacency list

An **adjacency list** is a data structure that uses an array of lists to store the edges of vertices.

Adjacency lists are more efficient at representing sparse graphs than adjacency matrices {% cite algorithm-design-manual -l 152 %}.

An adjacency list can be represented with a `graph` struct and an `edgenode` struct. Each vertex is assigned a unique identifier from 1 to `nvertices`. The edges on a vertex `x` are included as a linked list of `edgenodes` starting at the `edgenode` at `nvertices[x]` {% cite algorithm-design-manual -l 153 %}:

```c
const int MAXV = 1000;

typedef struct {
  int y;
  int weight;
  struct edgenode *next;
} edgenode;

typedef struct {
  edgenode *edges[MAXV+1];
  int degree[MAXV+1];
  int nvertices;
  bool directed;
} graph;
```

{% cite algorithm-design-manual -l 153 %}

A directed edge $$(x, y)$$ is represented as a $$y$$ `edgenode` in $$x$$'s adjacency list (at `g->edges[x]`). An undirected edge appears twice, once in $$x$$'s list as $$y$$, and once in $$y$$'s list as $$x$$ {% cite algorithm-design-manual -l 153 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/graphs/adjacency-list.svg" alt="">
  <figcaption><h4>Figure: A graph and its adjacency list</h4></figcaption>
</figure>

The following code initializes an adjacency list with $$m$$ vertices:

```c
void initialize_graph(graph *g, bool directed, int m) {
  int i;
  int x;
  int y;

  g->nvertices = n;
  g->nedges = m;
  g->directed = directed;

  for (i=1; i <= m; i++) {
    g->degree[i] = 0;
  }

  for (i=1; i <= m; i++) {
    g->edges[i] = NULL;
  }
}
```

Edges can be added from a 2D array:

```c

int example_edges[4][2] = { {1, 2}, {1, 4}, {1, 3}, {3, 4} };

void add_edges(graph *g, bool directed, int m, int edges[][2]) {
  int x;
  int y;

  for (int i = 0; i < m; i++) {
    x = edges[i][0];
    y = edges[i][1];
    insert_edge(g, x, y, directed);
  }
}

void insert_edge(graph *g, int x, int y, bool directed) {
  edgenode *p;
  p = malloc(sizeof(edgenode));
  p->weight = 0;
  p->y = y;
  p->next = g->edges[x];
  g->edges[x] = p;
  g->degree[x]++;

  if (directed == false) {
    insert_edge(g, y, x, true);
  } else {
    g->nedges++;
  }
}
```

## References

{% bibliography --cited_in_order %}
