---
layout: default
title: Heap
description: Notes on the heap data structure.
has_toc: false
parent: Data structures
grand_parent: Data structures and algorithms
permalink: /data-structures-and-algorithms/data-structures/heap
---

<!-- prettier-ignore-start -->

# Heap
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

A heap is a data structure for supporting the priority queue operations `insert()` and `extract_min()`. Heaps maintain a partial order of elements that is weaker than sorted order, but stronger than random order {% cite algorithm-design-manual -l 109 %}.

A heap-labelled tree is a binary tree where the key labelling of each node _dominates_ each of its children. In a min-heap, a node dominates its children by holding a smaller key than its children. In a max-heap tree, a node dominates its children by holding a larger key than its children {% cite algorithm-design-manual -l 110 %}.

In a min-heap tree, the root node is always the node with the lowest value.

## Representing a heap

A heap can be represented as an array, rather than using pointers like a typical binary tree. The position of keys in the array is used to determine the position of keys in the tree {% cite algorithm-design-manual -l 110 %}.

The root of the tree is the first position in the array. Its left and right sub children are stored at position 2 and 3 respectively {% cite algorithm-design-manual -l 110 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/heap/min-heap.svg" alt="">
  <figcaption><h4>Figure: A min-heap with its representation as an array {% cite algorithm-design-manual -l 110 %}</h4></figcaption>
</figure>

the $$2^l$$ keys of the $$l$$th level will be stored at $$2^{l−1}$$ to $$2^l - 1$$ respectively {% cite algorithm-design-manual -l 110 %}.

_Note: The index starts at 1 to simplify calculations._

A heap can be represented as a structure with an array and a number value:

```c
typedef struct {
  item_type q[PQ_SIZE+1];
  int n;
} priority_queue;
```

{% cite algorithm-design-manual -l 110 %}

The good thing about this format is that the position of the children at a key position $$k$$ can be worked out easily. The left child of $$k$$ sits at $$2k$$, the right at $$2k + 1$$. The parent of $$k$$ holds position $$n/2$$:

```c
int pq_parent(int n) {
  if (n == 1) {
    return -1;
  } else {
    return n / 2;
  }
}

int pq_young_child(int n) {
  return(2 * n);
}
```

{% cite algorithm-design-manual -l 111 %}

## Constructing a heap

Heaps can be built by inserting each element into the left-most open spot in the array ($$n + 1$$). This ensures a tree remains balanced, but it does not guarantee the dominance order of the keys {% cite algorithm-design-manual -l 112 %}.

The solution for ensuring a dominance relation is to swap any unsatisfied element with its parent. The other child is still dominated, because the new parent's value is even higher/ lower than the previous element's value {% cite algorithm-design-manual -l 112 %}.

The new element might still dominate its new parent after being moved. To ensure that it doesn't, the same process is followed with the element and its new parent. This process _bubbles up_ until the element is satisfied {% cite algorithm-design-manual -l 112 %}.

```c
void pq_insert(priority_queue *q, item_type x) {
  if (q->n >= PQ_SIZE) {
    printf("Warning: priority queue overflow insert x=%d\n", x);
  } else {
    q->n = (q->n) + 1;
    q->q[q->n] = x;
    bubble_up(q, q->n);
  }
}

void bubble_up(priority_queue *q, int p) {
  if (pq_parent(p) == -1) {
    return; /* at root of heap, no parent */
  }
  if (q->q[pq_parent(p)] > q->q[p]) {
    pq_swap(q,p,pq_parent(p));
    bubble_up(q, pq_parent(p));
  }
}
```

{% cite algorithm-design-manual -l 112 %}

The swap takes constant time at each level. Since the height of a heap is $$\lg n$$, the max insert will take $$O(\log n)$$. In other words, a heap of $$n$$ elements can be constructed in $$O(n\log n)$$ time:

```c
void make_heap(priority_queue *q, item_type s[], int n) {
  int i;

  q->n = 0;

  for (i = 0; i < n; i++) {
    pq_insert(q, s[i]);
  }
}
```

{% cite algorithm-design-manual -l 113 %}

## Extracting the minimum

In a min-heap, the minimum element is always at the top of the heap {% cite algorithm-design-manual -l 113 %}.

Removing an element requires rearranging the heap. The hole can be filled by moving the rightmost leaf (in the nth position of the array) into the first position {% cite algorithm-design-manual -l 113 %}.

After moving the rightmost element, the labelling of the nodes might be incorrect. If the new root is dominant, then the heap is maintained. If not, the dominant child of the root should be swapped with the root. This process _bubbles down_ until either the element either dominates all its children, or the end of the heap is reached {% cite algorithm-design-manual -l 113 %}.

```c
int item_type extract_min(priority_queue *q) {
  int min = -1; /* minimum value */

  if (q->n <= 0) {
    printf("Warning: empty priority queue.\n");
  } else {
    min = q->q[1];
    q->q[1] = q->q[q->n];
    q->n = q->n - 1;
    bubble_down(q, 1);
  }
  return(min);
}

void bubble_down(priority_queue *q, int p) {
  int c;
  int min_index;

  c = pq_young_child(p);
  min_index = p;

  for (int i = 0; i <= 1; i++) {
    if ((c + i) <= q->n) {
      if (q->q[min_index] > q->q[c + i]) {
        min_index = c + i;
      }
    }
  }

  if (min_index != p) {
    pq_swap(q, p, min_index);
    bubble_down(q, min_index);
  }
}
```

{% cite algorithm-design-manual -l 113 %}

This process takes in the worst case $$O(\log n)$$.

## References

{% bibliography --cited_in_order %}
