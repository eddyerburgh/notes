---
layout: default
title: Query processing
description: Query processing in RDBMSes.
has_children: true
has_toc: false
parent: Databases
nav_order: 4
permalink: /databases/query-processing
---

<!-- prettier-ignore-start -->

# Query processing
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

Query processing involves parsing an SQL statement, transforming it into an optimized query plan, then processing the query plan.

A **query plan** is a tree of operators. Data flows from the leaves toward the root and the output of the root node is the query result.

<figure>
  <img src="{{site.baseurl}}/assets/img/databases/query-processing/query-plan.svg" alt="">
  <figcaption><h4>Figure: Query plan represented as tree</h4></figcaption>
</figure>

## Processing Models

A DBMS processing model defines how the system executes a query plan. There are different models with various tradeoffs for different workloads.

It also specifies what data is moved from one operator to the next (e.g. single tuple/batch of tuples).

{% cite 15445-notes-10 %}

### Iterator Model

In the **iterator model** (also known as the pipeline model) each plan operator implements a `next()` function which returns either a single tuple or a null marker if there are no tuples remaining. This is the most common processing model.

The iterator model is a top-down approach that starts at the root.

The iterator model allows for pipelining where the DBMS can process a tuple through as many operators as possible before having to retrieve the next tuple.

{% cite 15445-notes-10 %}

### Batch Model

The batch model is similar to the iterator model except each operator emits a batch of tuples instead of a single tuple.

### Materialization Model

In the **materialization model** each operator produces all its tuples before emitting them as a single result to be used by the parent operator.

The materialization model is a bottom-up approach, starting at the leaf nodes before moving up the tree.

{% cite 15445-notes-10 %}

## Selection

Selection is commonly implemented with either a file scan or an index scan.

A **file scan** is a search algorithm that retrieves files meeting a given selection predicate. File scans are slow since every disk block must be read {% cite database-system-concepts -l 541 %}.

An **index scan** is a search algorithm that uses an index to speed up the scan {% cite database-system-concepts -l 542 %}. Indexes can only be exploited if an index is available, although sometimes a temporary index may be constructed.

Heuristic: do selection as early as possible to minimize intermediate results.

## Projection

Projection is straightforward to implement: for every input tuple, create an output tuple with only the specified attributes.

Heuristic: do projection as late as possible to do it on less rows.

## Sorting

Sorting is required both for queries that specify an order as well as for efficiently implementing certain operations (like joins and group bys) {% cite database-system-concepts -l 546 %}.

Heuristic: do sorting as late as possible (although sometimes ordering would make other plan nodes more efficient, e.g. joins, group by).

If data can fit in memory, use a sorting algorithm like quicksort or timsort.

If data doesn't fit in memory, use an external sorting algorithm like 2-way external merge sort.

## Joins

### Nested-loop join

A nested-join of two relations $$r$$ and $$s$$ examines each pair of tuples by iterating over them in a nested-loop fashion.

Relation $$r$$ is the outer relation, and relation $$s$$ is the inner relation:

```
for tuple_r in relation_r:
  for tuple_s in relation_s:
    if join_condition(tuple_r, tuple_s):
      result.add(concat_tuple(tuple_r, tuple_s))
```

Nested-join loop is the fallback algorithm for implementing joins. It doesn't use indexes.

The smaller relation should be used as the inner relation, since if it fits in main memory it could be read from secondary storage only once {% cite database-system-concepts -l 551 %}.

There are multiple variants that improve on nested-loop join in certain cases.

### Block nested-loop join

In **block nested-loop join** each block of the inner relation is paired with every block of the outer relation, thus requiring less block transfers than the nested-loop-join {% cite database-system-concepts -l 551 %}.

```
for block_r in relation_r:
  for block_s in relation_s:
    for tuple_r in block_r:
      for tuple_s in block_s:
        if join_condition(tuple_r, tuple_s):
          result.add(concat_tuple(tuple_r, tuple_s))
```

Block nested-loop join is useful if neither relation fits in memory.

### Indexed nested-loop join

An **indexed nested-loop join** uses the index of an inner relation to speed up lookup of the inner relation and thus to speed up equijoins.

This can be done either by using an existing index or creating a temporary index {% cite database-system-concepts -l 553 %}.

```
for tuple_r in relation_r:
  for tuple_s in get_from_index(table_s, index_s, join_value(tuple_r)):
    result.add(concat_tuple(tuple_r, tuple_s))
```

### Sort-merge join

Sort-merge join can be used to compute equijoins on relations sorted by the join attributes.

The algorithm maintains one pointer for each sorted relation. It iterates through the relations. When both pointers have the same value of the join attribute, a row is generated using those rows, and the pointers are advanced. Otherwise, the pointer of the relation with the lower value is advanced.

See [Sort-merge join wiki page](https://en.wikipedia.org/wiki/Sort-merge_join) for more details.

### Hash join

Hash joins can be used to compute equijoins. In a hash join, a hash table is built for one or both relations and then used to compare tuples.

Generally, one relation (the build relation) is used to construct a hash table where the key is `hash(possibly_composite_join_attribute)`. The second relation (the probe relation) is then scanned and compared with the corresponding relations in the hash table.

For large relations where the probe table doesn't fit in memory, the two relations can first be partitioned using the hash function and the algorithm can run against the matching partitions.

### Outer joins

Outer joins are more complex to implement than inner joins.

The block nested-loop join algorithm can be extended to support left outer joins—add each tuple in the outer relation that doesn't meet the join condition with any of the inner tuples to the final result. Right joins can be implemented by converting them to left joins {% cite database-system-concepts -l 565 %}.

Sort-merge join is the only join algorithm that easily supports full outer joins.

## Aggregation

Aggregation can be implemented by grouping tuples by either sorting or hashing based on the grouping attribute, then applying the the aggregate function on each group to get the result.

An optimization is to run the aggregation function on the tuples as they are grouped.

## Query optimization

**Query optimization** is the process of selecting the most efficient query plan from the possible strategies.

There are two types of query optimization strategies:

1. Heuristic/rule-based rewriting
2. Cost-based (using a cost model)

### Rule-based query optimization

Rule-based query optimization (query rewriting) involves rewriting a query plan based on some known equivalence rules.

Examples include:

- Predicate push-down: perform predicate filtering before a join to reduce the size of the join.
- Projection push-down: perform projections early to create smaller tuples and intermediate results.

{% cite 15445-notes-13 %}

### Cost-based query optimization

Cost-based query optimization involves generating multiple plans, assigning an estimated cost to each plan based on a cost model, and then selecting the cheapest plan.

The cost estimation uses various statistics about tables, such as number of rows, to estimate cost.

### Statistics

**Statistics** about tables are kept in a system catalog to help with cost estimation of query plans.

Some statistics for relation $$R$$ might include:

- $$N_r$$—the number of tuples in $$R$$
- $$V(A,R)$$—the number of distinct values for attribute $$A$$ in $$R$$

Generally, there is a background collector that will scan data periodically, derive statistics, and store them in an internal system catalog.

The **selectivity** of a given predicate is the number of tuples that qualify for a predicate. The formula to estimate this depends on the type of predicate (e.g. equality, range, negation, conjunction, disjunction). To improve the accuracy of selectivity, you need to estimate the number of tuples for a given value attribute.

One approach to calculate selectivity is to create a smaller temporary table by selecting a sample from the source table. You can then calculate selectivity by doing a sequential scan on the temporary table and counting the occurrences of the value.

{% cite 15445-notes-13 %}

## References

{% bibliography --cited_in_order %}
