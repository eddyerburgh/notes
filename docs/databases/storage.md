---
layout: default
title: Storage
description: RDBMS storage.
has_children: true
has_toc: false
parent: Databases
permalink: /databases/storage
---

<!-- prettier-ignore-start -->

# Storage
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## File organization

A DB is represented by a number of on-disk files.

A file normally includes a file header and a sequence of records, which are then mapped to disk blocks. A block can contain several records depending on the DB schema and physical organization {% cite database-system-concepts6 -l 451 %}.

A **data dictionary** stores table metadata (e.g. table name, row count). Often, data dictionaries are implemented as tables themselves {% cite database-system-concepts6 -l 463 %}.

_Note: For simplicity, these notes assume that a record is no larger than a block and that each record is entirely contained within one block, but it's possible to support larger records._

### Record structure

In **fixed-length records**, fields start and ends at the same place in each record. Insertion and deletion of fixed-length records is simple because a new record will fit in any deleted record space or at the end of the file {% cite database-system-concepts6 -l 454 %}.

Typically, deletion would involve leaving open space that was occupied by the deleted record, which is only written to when a new record is inserted. Deleted record spaces can be stored in a free list (a linked list of addresses) {% cite database-system-concepts6 -l 454 %}.

In **variable-length records**, fields can start and end at different places in different records (e.g. a `varchar` field).

One approach to handle variable-length records is to store records of the same size in the same block. The alternative is to support variable-length records in the same block {% cite database-system-concepts6 -l 452 %}.

Variable-length records usually have two parts: an initial part with fixed-length attributes, also containing \<offset, length\> data for the variable-length parts. Followed by variable-length values (sometimes records also contain a null bitmap indicating which values are null) {% cite database-system-concepts6 -l 456 %}.

The **slotted-page structure** is commonly used for storing variable-length records in a block. A block using slotted-page structure includes a header containing metadata (e.g. number of record entries, the end of free space in the block, a list containing the location and size of each record). Records are then placed contiguously in the block from the end of the block. So the free space in the block is between the final entry in the header array and the first record {% cite database-system-concepts6 -l 456 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/databases/storage/slotted-page-structure.svg" alt="">
  <figcaption><h4>Figure: Slotted page structure</h4></figcaption>
</figure>

With slotted-page structure, external record pointers usually point to the entry in the block header that contains the metadata for that record. This allows records to be moved inside the block to avoid fragmentation without needing to update external references {% cite database-system-concepts6 -l 457 %}.

Data that is larger than a block is often stored in a special file(s) instead of being stored with the other attributes of records (e.g. blob data). The record in the block then contains a logical pointer to the large object {% cite database-system-concepts6 -l 457 %}.

### Record organization in files

There are multiple ways to store records in a file:

- **Heap file organization**: any record can be placed anywhere in the file where there is space for the record.
- **Sequential file organization**: records are stored in sequential order according to the value of a search key.
- **Hashing file organization**: some attribute of each record is hashed to determine which file the record should be placed in.

{% cite database-system-concepts6 -l 458 %}

Heap file organization is probably most common.

Typically there is a single file for each table, but not always (e.g. SQLite).

## DB buffer cache

A **DB buffer cache** is a part of main memory used by the DB to cache blocks {% cite database-system-concepts6 -l 464 %}.

A **buffer manager** moves data between secondary storage and main memory. When a block is requested, the buffer manager checks to see if it's in the buffer. If it is, the buffer manager returns the block's memory address. If the block is not in the buffer then the manager must allocate space. If there are no free blocks then it must choose a block to replace. If the block has writes, then it's written back to disk. Otherwise, it can be discarded {% cite database-system-concepts6 -l 465 %}.

The buffer manager can improve on the OS virtual-memory manager by optimizing for the DB use case. For example, keeping hot blocks like the data dictionary and indexes over record blocks {% cite database-system-concepts6 -l 465-6 %}.

## Indexes

A **DB index** is a data structure that improves the speed of data retrieval operations on a database table (at the cost of additional writes and storage space to maintain the index data structure).

A **search key** is the attribute used to look up records. A **composite search key** is a search key made of multiple attributes.

A **primary index** is an index on a set of attributes that includes the primary key. A **secondary index** is an index that is not a primary index and can have duplicates.

**Ordered indexes** keep values in sorted order. There are two types of ordered indexes: dense indexes and sparse indexes.

A **dense index** contains an index entry for each search key value {% cite database-system-concepts6 -l 481 %}.

A **sparse index** only contains an index entry for some of the search key values. Sparse indexes can only be used for primary indexes and if the table is sorted in order of the search key {% cite database-system-concepts6 -l 481 %}.

Large indexes that don't fit in main memory can be optimized by converting them to a **multilevel index** where the outer index is a sparse index based on the inner index {% cite database-system-concepts6 -l 481 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/databases/storage/multilevel-index.svg" alt="">
  <figcaption><h4>Figure: Multilevel index</h4></figcaption>
</figure>

Each time a record is inserted, deleted, or if the search key column is updated, the corresponding index must be updated too.

### B+-tree indexes

Most indexes are implemented using B+-trees, which are balanced search trees with large (often block-sized) nodes. See [notes on btrees]({{ '/data-structures-and-algorithms/data-structures/b-trees' | relative_url }})).

B+-trees are intended to minimize I/O operations. The number of I/O operations is proportional to the height of a B+-tree, but in practice many of the upper tree nodes will already be in main memory {% cite database-system-concepts6 -l 499 %}.

In a typical B+-tree index, **leaf nodes** contain pointers to the record for each key, where $$p_i$$ is a pointer corresponding to key $$k_i$$. Each leaf can hold as few as $$\lceil (n-1)/2 \rceil$$ up to $$n-1$$ values. Each key has a corresponding pointer to the record. $$p_n$$ is a special pointer that points to the next leaf node {% cite database-system-concepts6 -l 486-7 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/databases/storage/leaf-node.svg" alt="">
  <figcaption><h4>Figure: B+-tree leaf node {% cite database-system-concepts6 -l 487 %}</h4></figcaption>
</figure>

**Internal (nonleaf) nodes** form a multilevel sparse index on the leaf nodes. Internal nodes have the same structure as leaf nodes except their pointers point to tree nodes. An internal node must hold at least $$\lceil n/2 \rceil$$ pointers, the number of pointers is known as the node's **fanout** {% cite database-system-concepts6 -l 487 %}.

B+-trees can be used efficiently for range queries (finding all records with search key values in a given range) by using the next leaf node pointer {% cite database-system-concepts6 -l 490 %}.

B+-trees are used to minimize disk accesses. The number of disk accesses required for each operation is proportional to the height of the tree.

Updates can be modeled as a deletion and insertion. The programmatic complexity of insertion and deletion comes from maintaining the B+-tree invariants.

#### Insertion

Insertion involves finding the leaf node where the search-key value should be placed and inserting the entry in position so that the nodes are still in sorted order {% cite database-system-concepts6 -l 491 %}.

If the leaf node is full, then the node is split into two nodes. **Splitting** involves creating a new leaf, keeping the first ($$\lceil n/2 \rceil$$) values in the original node, and adding the remaining values to the new node. The new leaf node must then be inserted into the B+-tree by adding a new pointer in the parent node and updating the special next pointer in the leaf nodes {% cite database-system-concepts6 -l 492-3 %}.

If the parent node is full then it must also be split. Splitting is done recursively until either a parent node is not full after the split, or the split reaches the root.

The process for splitting a nonleaf node is slightly different to splitting a leaf. In this case, the child pointers are divided between the original and the newly-created nodes. The search key values that lie between the pointers moved to the right node are moved along with the pointers, while the search key values that lie between the pointers on the left are untouched {% cite database-system-concepts6 -l 493 %}.

#### Deletion

Deletion involves first locating the entry to be deleted in the leaf and then deleting the entry.

If after deletion the leaf node contains less than the minimum number of entries ($$\lceil (n-1)/2 \rceil$$) then the entries must either be merged with another node or redistributed to ensure each node meets the minimum entries invariant. Internal node pointers must also be updated {% cite database-system-concepts6 -l 498 %}.

#### Nonunique search keys

A nonunique search key occurs when multiple records can have the same search key {% cite database-system-concepts6 -l 497 %}.

Nonunique search keys make deletions less efficient in the case of duplicates because in the worst-case each duplicate would need to be checked.

One solution is to make search keys unique by creating a composite search key containing the original search key and a **uniquifier attribute** which makes the search key unique, e.g. a pointer to the record {% cite database-system-concepts6 -l 497-8 %}.

#### Indexing strings

There are two problems with indexing strings:

1. They can be of variable length.
2. They can be long, leading to low fanout (therefore increased tree height).

The fanout problem can be fixed by using prefix compression, where only a prefix of each search key value that is sufficient to distinguish between key values in the subtrees that it separates is used.

{% cite database-system-concepts6 -l 502-3 %}

## References

{% bibliography --cited_in_order %}
