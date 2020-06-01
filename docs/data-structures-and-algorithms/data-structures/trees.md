---
layout: default
title: Trees
description: Notes on trees.
has_toc: false
parent: Data structures
grand_parent: Data structures and algorithms
permalink: /data-structures-and-algorithms/data-structures/trees
---

<!-- prettier-ignore-start -->

# Trees
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

A tree is set $$T$$ of one or more nodes, where:

1. There is one designated root node of the tree, $$root(T)$$.
2. The remaining nodes are partitioned into $$m >= 0$$ disjoint sets $$T_1 ... T_m$$, where each of the sets are also trees. The trees $$T_1 ... T_m$$ are called subtrees of $$T$$.

{% cite knuth1998 -l 308 %}

Trees use the same terminology as family trees to describe relationships between nodes. A node can have a parent, a sibling, a grandparent, etc. {% cite knuth1998 -l 311 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/trees/tree.svg" alt="">
  <figcaption><h4>Figure: A tree</h4></figcaption>
</figure>

The number of subtrees of a tree is called its **degree**. A tree of degree 0 is a **leaf** {% cite knuth1998 -l 308 %}.

A **forest** is a set of of zero or more disjoint trees. Deleting the root of a tree creates a forest {% cite knuth1998 -l 309 %}.

## Representing a rooted tree

A rooted binary tree can be represented as nodes with pointers `parent`, `left`, and `right`, where `parent` points to the parent node, `left` points to the left child node and `right` points to the right child node {% cite clrs -l 246 %}.

$$x$$ is the root of the tree if `parent` $$= NULL$$.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/trees/binary-tree.svg" alt="">
  <figcaption><h4>Figure: A binary tree with parent, right, and left pointers</h4></figcaption>
</figure>

_Note: future diagrams don't show NULL pointers explicitly._

A rooted tree node with $$n$$ children (an n-ary tree) can be represented using similar pointers. Instead of a pointer to each child from the parent, each node contains a `left_child` pointer to the leftmost child (or `NULL` if not), and a `right_child` pointer to the right most child (or `NULL` if $$n \lt 2$$).

Each node then contains two pointers: `left_sibling` and `right_sibling`. `left_sibling` points to the previous sibling (`NULL` if none exists), and `right_sibling` points to its immediate right sibling (`NULL` if none exists) {% cite clrs -l 246 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/trees/n-ary-tree.svg" alt="">
  <figcaption><h4>Figure: An n-ary tree with right_sibling and left_sibling pointers</h4></figcaption>
</figure>


## References

{% bibliography --cited_in_order %}
