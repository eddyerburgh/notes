---
layout: default
title: B-trees
description: Notes on the b-tree data structure.
has_toc: false
parent: Data structures
grand_parent: Data structures and algorithms
permalink: /data-structures-and-algorithms/data-structures/b-trees
---

<!-- prettier-ignore-start -->

# B-trees
{:.no_toc}

B+-trees (a variant of B-trees) are useful to know since relational DB indexes are often implemented using them.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

A B-tree is a self-balancing tree that maintains sorted data and allows searches, insertions, and deletions in logarithmic time.

B-trees are designed to work well when stored on secondary storage, like HDD. The size of a node is normally the same size as the system block/page size.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/b-tree/b-tree.svg" alt="">
  <figcaption><h4>Figure: B-tree containing letters A-T</h4></figcaption>
</figure>

Typical b-tree operations are:

| operation   | description                                               | time complexity (worst case) |
| ----------- | --------------------------------------------------------- | ---------------------------- |
| `insert(k)` | Inserts key $$k$$ to tree.                                | $$O(\log n)$$                |
| `delete(k)` | Deletes key $$k$$ from the tree (assuming no duplicates). | $$O(\log n)$$                |
| `find(k)`   | Return value for key $$k$$ if it exists in tree.          | $$O(\log n)$$                |

_Note: to simplify the discussion, these notes assume that satellite data associated with the key resides in the same node as the key._

## Formal definition

1. Each node $$x$$ has the following attributes:
   1. $$x.{n}$$ keys stored in node $$x$$.
   2. $$x.n$$ keys $$x.key_1, x.key_2, ..., x.key_{x.n}$$ stored in non-decreasing order.
2. Each internal node contains $$x.n + 1$$ pointers $$x.c_1, x.c_2, ..., x.c_{n + 1}$$ to its children.
3. The keys $$x.key_i$$ separate the ranges of keys stored in each subtree. If $$k_i$$ is any key stored in the subtree with root $$x.c_i$$ then: $$k_1 \le x.key_1  \le k_2  \le x.key_2  \le ...  \le x.key_x.n  \le k_x.n+1$$.
4. All leaves have the same depth.
5. The **minimum degree** ($$t$$) of a tree is used to define the minimum number of keys that can exist in a non-root node:
   1. t >= 2
   2. Non-root nodes must contain at least $$t - 1$$ keys (meaning that an internal node has at least $$t$$ children.
   3. Non-root nodes can contain at most $$2t - 1$$ keys (meaning that an internal node has at most 2t children). A node is **full** if it contains $$2t - 1$$ keys.

{% cite clrs -l 490 %}

## Find

Find requires traversing each level of the tree until the key is found. It's similar to binary search algorithm except there is a range search used to identify the correct child to visit.

## Insert

Insertion involves searching for the new leaf position and inserting the new key in sorted order. In the case that the leaf node is full, the node must be split.

Splitting involves creating two nodes from the full node $$f$$'s keys, split around the $$f$$'s median key to have $$t - 1$$ keys each. The median node is moved to $$f$$'s parent to identify the dividing point between the two new trees {% cite clrs -l 493 %}.

To avoid recursively splitting nodes up to thee root, you can preemptively split full nodes while traversing the tree {% cite clrs -l 493 %}
.

## Deletion

There are two popular B-tree deletion strategies:

1. Locate and delete the item (à la `find()`), then restructure the tree to retain its invariants.
2. Do a single pass down the tree, but before visiting a node restructure the tree so that once the key to be deleted is encountered, it can be deleted without triggering the need for any further restructuring.

Maintaining the B-tree invariants when deleting involves moving keys from sibling nodes that have $$keys >= t$$, or in the case that no immediate siblings have $$keys >= t$$, merging two nodes.

## B+-tree

B+-trees are a variant of B-trees commonly used in DBs to implement indexes (where they are often referred to as btrees).

B+-trees store all satellite information in the leaves and only stores keys and child pointers in the internal nodes {% cite clrs -l 488 %}.

## References

{% bibliography --cited_in_order %}
