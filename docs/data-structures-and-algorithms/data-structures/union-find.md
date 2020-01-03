---
layout: default
title: Union-find
description: Notes on the union-find data structure.
has_toc: false
parent: Data structures
grand_parent: Data structures and algorithms
permalink: /data-structures-and-algorithms/data-structures/union-find
---

<!-- prettier-ignore-start -->

# Union-find
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

A union-find data structure (also known as a disjoint set), is used to efficiently store items in disjoint (non-overlapping) sets.

The key union-find operations are:

- `make_set(x)`: make set with `x` as a member of its set.
- `find(x)`: return the root element of `x`.
- `union(x,y)`: combine the sets of `x` and `y`.

Union-find represents sets as a **backwards tree**, where each node has a corresponding parent. The root node defines which set the node and all its children are in.

A root node has its parent set to itself, to identify that it's the root in a set {% cite algorithm-design-manual -l 199 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/union-find/union-find.svg" alt="">
  <figcaption><h4>Figure: Two sets and their representation in an array</h4></figcaption>
</figure>

Merging sets can be done by setting the parent of one set to the parent of another. In the case of a merge, the smallest subtree becomes the parent of the other {% cite algorithm-design-manual -l 199 %}.

At most $$\lg_2 n$$ doublings can occur before all sets have been merged. That means `find()` and `union()` are both $$O(\log n)$$ operations {% cite algorithm-design-manual -l 201 %}.

## Representing a union-find

A union-find can be represented as an array of nodes:

```c
typedef struct {
  int p[SET_SIZE+1]; // parent element
  int size[SET_SIZE+1]; // number of elements in subtree i
  int n; // number of elements in set
} set_union;
```

The following are implementations of the key operations:

```c

void set_union_make_set(set_union *s, int x) {
  s->p[i] = i;
  s->size[i] = 1;
}

void set_union_init(set_union *s, int n) {
  for (int i = 1; i <= n; i++) {
    set_union_make_set(s, i)
  }
  s->n = n;
}

int set_union_find(set_union *s, int x) {
  if (s->p[x] == x) {
    return(x);
  }
  return set_union_find(s, s->p[x]);
}

int set_union_union(set_union *s, int s1, int s2) {
  int r1, r2; /* roots of sets */
  r1 = find(s, s1);
  r2 = find(s, s2);
  if (r1 == r2) return;

  if (s->size[r1] >= s->size[r2]) {
    s->size[r1] = s->size[r1] + s->size[r2];
    s->p[r2] = r1;
  }
  else {
    s->size[r2] = s->size[r1] + s->size[r2];
    s->p[r1] = r2;
  }
}
```

{% cite algorithm-design-manual -l 200 %}

## References

{% bibliography --cited_in_order %}
