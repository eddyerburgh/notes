---
layout: default
title: Data structures
description: Notes on data structures.
nav_order: 2
has_children: true
has_toc: false
parent: Data structures and algorithms
permalink: /data-structures-and-algorithms/data-structures
---

<!-- prettier-ignore-start -->

# Data structures
{:.no_toc}

Data structures affect how long it takes to access and update data. Selecting appropriate data structures for a problem is in an important skill for software engineers to develop.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

A data structure is a representation of data and a collection of associated operations {% cite open-data-structures -l 5 %}.

Data structures are implementations of abstract data types {% cite open-data-structures -l 4 %}.

### Abstract data types

ADTs (Abstract Data Types) define the set of operations supported by a data structure {% cite open-data-structures -l 4 %}.

An ADT is the interface of a data structure, it doesn't define how a data structure implements the operations {% cite open-data-structures -l 4 %}.

There can be many implementations of an ADT {% cite open-data-structures -l 4 %}.

## Contiguous vs Linked data structures

Data structures can be either **contiguous** in memory, or **linked** with pointers.

Contiguously allocated structures are built from a single slab of memory. They include arrays, matrices, and heaps {% cite algorithm-design-manual -l 66 %}.

Linked data structures are built from chunks of memory linked together via pointers. They include lists, trees, and graph adjacency lists {% cite algorithm-design-manual -l 66 %}.

### Arrays

Arrays are the fundamental contiguously-allocated data structure. Arrays are fixed-size, and their items can be accessed using an index.

The advantages of contiguously allocated arrays are:

- **Constant time access**. As long as you have the index, you can access an array item in constant time.
- **Space efficiency**. Arrays only consist of data. No space is spent on pointers or other formatting information.
- **Memory locality**. It's common to iterate over all items of a data structure. Arrays are very good for this because they exhibit good **memory locality**: each item exists directly after the previous item in memory. This works well with the cache system used in modern computer architectures.

{% cite algorithm-design-manual -l 66 %}

There are two types of array:

1. Static arrays
2. Dynamic arrays

Static arrays can't have their size modified during execution.

Dynamic arrays can increase and decrease in size during execution. A simple implementation is to initialize an array with a size of 1. Whenever the array runs out of space, its size is doubled by allocating double the previous size in memory and copying over the old array items to the lower half of the new array {% cite algorithm-design-manual -l 67 %}.

### Pointers and linked structures

A pointer "represents the address of a location in memory". Pointers connect linked pieces of a data structure together {% cite algorithm-design-manual -l 67 %}.

Linked lists are the simplest linked data structure. A linked list is made up of nodes that contain data. As well as data, each node contains a pointer to the next next node in the list.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/singly-linked-list.svg" alt="">
  <figcaption><h4>Figure: Singly linked list {% cite lkd -l 86 %}</h4></figcaption>
</figure>

A generic linked list definition includes a `next` pointer, and a `data` value that contains some data:

```c
typedef struct list {
  DATA data;
  struct list *next;
} list;
```

Generally, linked data structures share common properties:

- Each node contains one or more data fields.
- Each node contains at least one pointer to another node (although the pointer can be empty). This can mean that much of the data space of linked structures is taken up with pointers, not data.

{% cite algorithm-design-manual -l 68 %}

### Comparison

The advantages of arrays include:

- Random (constant) access.
- Better memory locality.
- Better space efficieny (since they don't need to store pointers).

{% cite algorithm-design-manual -l 71 %}

The advantages of linked structures over static arrays include:

- No overflows unless memory is full.
- Insertions and deletions are simpler than for contiguous arrays.

{% cite algorithm-design-manual -l 71-2 %}

Both of these data types can be thought of as recursive objects. Removing the first element from a linked list leaves a smaller linked list, and splitting elements from an array creates two smaller arrays. Divide-and-conquer algorithms work well on these recursive data structures {% cite algorithm-design-manual -l 71 %}.

## Stacks and queues

Stacks and queues are both container data structures that allow you to add and retrieve data.

Stacks support retrieval based on **LIFO** (last-in-first-out). That is, the last item added to a stack is the first item to be removed. LIFO is often compared to a pile of cafeteria trays. A worker adds more trays to a pile by placing them on top of the existing trays and a customer takes their tray from the top of the pile.

The _put_ and _get_ operations for stacks are usually called `push()` and `pop()` {% cite algorithm-design-manual -l 71 %}.

Queues support retrieval based on **FIFO** (first-in-first-out). They work the same way as real-world queues.

The _put_ and _get_ operations for queues are usually called `enqueue()` and `dequeue()` {% cite algorithm-design-manual -l 71 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/queue.svg" alt="">
  <figcaption><h4>Figure: A queue {% cite lkd -l 86 %}</h4></figcaption>
</figure>

Stacks and queues are commonly implemented with either arrays or linked lists {% cite algorithm-design-manual -l 71 %}.

## Dictionaries

A dictionary data type enables access to data items by their content, or by a key value.

Dictionaries generally support the following operations:

- `search(k)`
- `insert(k, v)`
- `delete(k)`

Some dictionary data structures include other useful operations:

- `max()` or `min()`.
- `predecessor(k)` or `successor(k)`.

{% cite algorithm-design-manual -l 72-3 %}

### Hash tables

Hash tables are an efficient way of maintaining a dictionary.

A hash table works by using a hashing function to map a key to an integer. The integer is used as an index to store and retrieve an item from an array (or from a list stored in an array) {% cite algorithm-design-manual -l 89 %}.

Normally the integer produced by the hashing function ($$H$$) is larger than the number of slots available in the hash table ($$m$$). The large integer can be converted to an integer in the range of the hash table slots by calculating the remainder of $$H(K)/m$$ using the modulo (`%`) operator {% cite algorithm-design-manual -l 89 %}.

Hash tables often suffer from **collisions**, where multiple distinct keys hash to the same value. There are different strategies that can be used in the case of collisions {% cite algorithm-design-manual -l 89 %}.

One approach is **chaining**. In chaining the hash table is represented as an array of linked lists. Each time an item is added to the hash table, it's inserted as a list node {% cite algorithm-design-manual -l 89 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/hash-table.svg" alt="">
  <figcaption><h4>Figure: Hash table using chaining {% cite lkd -l 86 %}</h4></figcaption>
</figure>

## Binary Search Trees

Binary search trees are a data structure that enable fast search.

Binary search trees are built on **rooted binary trees**. A rooted binary tree is recursively defined as either empty, or a node (the root) with two child rooted binary trees, known as the left subtree and right subtree {% cite algorithm-design-manual -l 77 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/binary-tree.svg" alt="">
  <figcaption><h4>Figure: Binary tree {% cite lkd -l 86 %}</h4></figcaption>
</figure>

A binary search tree is a rooted binary tree where all nodes in the left subtree have a value < the root, and all nodes in the right subtree have trees > the root {% cite algorithm-design-manual -l 77 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/data-structures-and-algorithms/data-structures/binary-search-tree.svg" alt="">
  <figcaption><h4>Figure: Binary search tree {% cite lkd -l 86 %}</h4></figcaption>
</figure>

Binary search trees offer $$O(h)$$ search, insertion, and deletion, where $$h$$ is the height of the tree. If the search tree is _balanced_ (the difference between the depth of the bottom subtrees is at most 1), then this is $$\lg n$$, where $$n$$ is the number of items. The problem is that binary search trees will not always be naturally balanced {% cite algorithm-design-manual -l 81-2 %}.

<!-- TODO: link to red-black trees and splay trees section-->

Balanced binary search trees are data structures that maintain a balanced tree by doing extra work during insertion and deletion. For example, red-black trees and splay trees {% cite algorithm-design-manual -l 82 %}.

## References

{% bibliography --cited_in_order %}
