---
layout: default
title: Data processing
description: Notes on distributed data processing.
has_children: true
has_toc: false
nav_order: 5
parent: Distributed systems
permalink: /distributed-systems/data-processing
---

<!-- prettier-ignore-start -->

# Data processing
{:.no_toc}


## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction



## MapReduce

MapReduce is programming pattern and system developed at Google to perform data manipulation on large datasets across distributed commodity devices.

MapReduce takes inspiration from the `map()` and `reduce()` functional programming primitives. Google realized that many of their computations involved running a map operation to create intermediate values and then a reduce operation to combine the intermediate values into an output  {% cite googlepub62 -l 1 %}.

The computation works by taking an input set of \<key,value\> pairs, running a user-provided map function to produce intermediate \<key,value\> pairs (which are grouped by key), and then passing the intermediate key and set of values associated with that key to a reduce function. The reduce function merges values and creates a possibly smaller set of values {% cite googlepub62 -l 2 %}.

When a user program calls the `MapReduce()` function, the following occurs:

1. The MapReduce library splits input files into $$M$$ chunks (typically 16-64MB per chunk) and starts up copies of the program on a cluster of machines.
2. One of the copies is the master, the rest are workers that run work assigned by the master. The master picks idle machines and assigns them either a map task or a reduce task.
3. A worker that’s assigned a map task reads the contents of the input chunk, parses key/value pairs and passes them to the Map function. The intermediate values are then buffered in memory.
4. Periodically, buffered pairs are written to local disk and partitioned into $$R$$ regions by the partitioning function. The locations of the buffered pairs are passed to the master, which forwards these locations to a Reduce worker.
5. When a worker is notified about the locations of buffered pairs, it uses RPC to read the buffered data from the local disk of the workers. When a Reduce worker has read all the intermediate data, it sorts the keys so that occurrences of the same key are grouped together.
6. The reduce worker iterates over the sorted intermediate data and passes the data for each unique intermediate key to the Reduce function. The output of Reduce is then written to a final output file corresponding to the Reduce partition.
7. When all map and reduce tasks are completed, the master wakes up the user program and the `MapReduce()` call in the user program returns to the user code.

{% cite googlepub62 -l 4 %}

MapReduce was extremely influential when it was published by google and an open-source version implementation (Hadoop) became incredibly popular. MapReduce has since been replaced at Google by other systems, and Hadoop is losing ground to newer solutions like Apache Spark.

## Spark

Spark can be seen as a successor to MapReduce. It's widely used for Big Data computations.

The motivation for Spark was to handle computations that MapReduce/Hadoop were not efficient at handling: iterative computations and interactive data mining. Spark exposes methods for operating on distributed data that's stored in-memory data so that intermediate results don't need to be written to disk, which works well for iterative algorithms like PageRank {% cite usenix180560 -l 1 %}.

When Spark was initially developed, it was built around RDDs (Resilient Distributed Databases). An RDD represents a read-only collection of objects that are partitioned across a set of machines, which can be rebuilt in the case of failure. They can either be built from a stable storage source like HDFS, or from other RDDs {% cite usenix180560 -l 1,2 %}.

Spark is made up of a user-provided driver program which will then cause Spark to run computations on a cluster of workers {% cite usenix180560 -l 4 %}.

Operations that create new RDDs are called _transformations_. Examples are `map()`, `filter()`, and `join()` {% cite usenix180560 -l 2 %}.

Transformations are evaluated lazily, in other words they create a lineage graph but do not perform any computations immediately. The lineage graph is then used to run computations when the driver program calls an _action_ (such as `collect()`). Spark can use the lineage graph to create optimizations, such as non-duplicate partitions (find reference)

Calling an action will cause the Spark scheduler to build a DAG of stages, and then begin the computation by sending tasks to workers 7.

Spark differentiates between wide dependencies and narrow dependencies. Narrow dependencies are dependencies that exist on the same partition as their parent dependencies, and wide dependencies are dependencies that might be depended on by multiple child partitions. `map()` leads to narrow dependencies, `join()` leads to wide dependencies. Narrow dependencies can run on the same worker machines without having to go over the network {% cite usenix180560 -l 6 %}.

Spark as described in the original paper didn't work well with stream processing, but Spark Streaming was created to add support for stream processing.







<!-- 
Spark generalizes data computation

Spark creates a dataflow graph which allows it to make optimizations. The dataflow graphs are a way to describe computations.

It supports iterative applications that loop over data better than map.

PageRank doesn't run well in MapReduce, it's often used as an example of something that doesn't run well because it includes iterations, meaning that MapReduce would require multiple MapReduce calls, which would mean lots of file I/O. Spark works much better for PageRank.

Data is spread out over workers. distinct would need to move data do that data with the same keys end on the same machine. This will require communication over machines. This could involve hashing to determine the machine that the key should exist on.

Calling `cache()` will persist data in memory.

The computer that runs the Scala program that starts the transformation is called the driver.

The code generates a lineage graph, until `collect()` is called.

map is a local operation so it can happen on each worker, without going over the network.

Multiple map phases can string together in memory, whereas MapReduce would need to write to disk for each map operation.

Narrow transformations are transformations that can happen locally without contacting other nodes.

groupByKey, distinct, and other transformations are wide transformations, they might need to look at all partitions to read keys and move the data between partitions. This can be done by hashing the key and then modding by the number of workers. Wide transformations can be very expensive.

Because Spark creates a lineage graph before running any operations, it can make some optimizations.

Spark is a data processing engine that was designed for large-scale non-acyclic jobs. For example, machine learning jobs that apply a function repeatedly to the same dataset, or interactive analysis where data is loaded into memory and queried repeatedly 1.

The basic abstraction in Spark is the RDD (Resilient Distributed Database). An RDD represents a read-only collection of objects that are partitioned across a set of machines, which can be rebuilt in the case of failure 1.

An RDD can be constructed in four ways:

1. From a file in a shared file system (e.g., HDFS).
2. By dividing an array into multiple slices.
3. By transforming an existing RDD using flatMap, filter, or map operations.
4. By changing the persistence of an existing RDD with either the cache or the save action.
2
There are several operations that can run on RDDs:
- reduce: “Combines dataset elements using an associative function to produce a result at the driver program.”
- collect: “Sends all elements of the dataset to the driver program”
- foreach: Passes each element through a user provided function. -->


## The Dataflow

https://medium.com/stream-processing/what-is-stream-processing-1eadfca11b97
