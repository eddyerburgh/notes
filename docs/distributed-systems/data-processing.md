---
layout: default
title: Data processing
description: Notes on distributed data processing.
has_toc: false
nav_order: 7
parent: Distributed systems
permalink: /distributed-systems/data-processing
---

<!-- prettier-ignore-start -->

# Distributed data processing
{:.no_toc}

Distributed data processing frameworks enable large-scale computations in a reasonable amount of time.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Batch processing

**Batch processing** is where a computer runs batches of jobs without requiring human interaction. Batch jobs are typically short-lived, regularly scheduled, and operate on bounded sets of data.

Batch jobs can be scheduled at regular intervals as part of data pipelines using tools like Apache Airflow, which creates a DAG of tasks to execute based on task dependencies.

Batch processing is often contrasted with stream processing.

### MapReduce

MapReduce is programming pattern and system developed at Google to perform data manipulation on large datasets across distributed commodity devices.

MapReduce takes inspiration from the `map()` and `reduce()` functional programming primitives. Google realized that many of their computations involved running a `map()` operation to create intermediate values and then running a `reduce()` operation to combine the intermediate values into an output {% cite googlepub62 -l 1 %}.

The computation works by taking an input set of \<key,value\> pairs, running a user-provided map function to produce intermediate \<key,value\> pairs (which are grouped by key), and then passing the intermediate key and set of values associated with that key to a user-provided reduce function. The reduce function merges values and creates a possibly smaller set of values {% cite googlepub62 -l 2 %}.

When a user program calls the `MapReduce()` function, the following occurs:

1. The MapReduce library splits input files into $$M$$ chunks (typically 16-64MB per chunk) and starts up copies of the program on a cluster of machines.
2. One of the copies is the master, the rest are workers that run work assigned by the master. The master picks idle machines and assigns them either a map task or a reduce task.
3. A worker that’s assigned a map task reads the contents of the input chunk, parses \<key,value\> pairs and passes them to the map function. The intermediate values are then buffered in memory.
4. Periodically, buffered pairs are written to local disk and partitioned into $$R$$ regions by the partitioning function. The locations of the buffered pairs are passed to the master, which forwards these locations to a Reduce worker.
5. When a worker is notified about the locations of buffered pairs, it uses RPC to read the buffered data from the local disk of the workers. When a Reduce worker has read all the intermediate data, it sorts the keys so that occurrences of the same key are grouped together.
6. The reduce worker iterates over the sorted intermediate data and passes the data for each unique intermediate key to the reduce function. The output of reduce is then written to a final output file corresponding to the reduce partition.
7. When all map and reduce tasks are completed, the master wakes up the user program and the `MapReduce()` call in the user program returns to the user code.

{% cite googlepub62 -l 4 %}

The following `map()` and `reduce()` functions produce a \<word,count\> pair for each unique word in a document:

```py
def map(name, document):
  for word in document.split(' '):
    emit_intermediate(word, "1")

def reduce(word, partialCounts):
  sum = 0
  for count in partialCounts:
    sum += int(count)
  emit(word, str(sum))
```

The MapReduce paper was extremely influential when it was published in 2004. An open-source implementation of MapReduce called Hadoop was developed and it quickly became popular in companies that handled large datasets. MapReduce has since been replaced at Google by other systems, and Hadoop is now losing ground to newer solutions like Apache Spark.

### Spark

Spark is a distributed computing framework that can be seen as a successor to MapReduce. It's widely used for Big Data computations.

The motivation for building Spark was to be able to run computations that existing frameworks like MapReduce were not efficient at handling, specifically iterative computations and interactive data mining. Spark exposes methods for operating on distributed data that can be stored in RAM between operations so that intermediate results don't need to be written to disk (making iterative computations more efficient) {% cite usenix180560 -l 1 %}.

Spark is built around RDDs (Resilient Distributed Databases). An RDD represents a read-only collection of objects that are partitioned across a set of machines. They can either be built from a stable storage source like HDFS, or from other RDDs, and can be rebuilt in the case of failure {% cite usenix180560 -l 1,2 %}.

A Spark application runs computations on a cluster of workers that are controlled by a user-provided driver program {% cite usenix180560 -l 4 %}.

Operations that create new RDDs are called _transformations_. Examples are `map()`, `filter()`, and `join()` {% cite usenix180560 -l 2 %}. Transformations are evaluated lazily, in other words they create a lineage graph but do not perform any computations immediately. The lineage graph is then used to run computations when the driver program calls an _action_ (such as `collect()`).

Calling an _action_ will cause the Spark scheduler to build a DAG of stages, and then begin the computation by sending tasks to workers {% cite usenix180560 -l 7 %}.

Spark differentiates between wide dependencies and narrow dependencies. Narrow dependencies are dependencies where each partition of a parent RDD exists on the same partition as its child RDDs. Wide dependencies are dependencies that might exist across multiple partitions. Different operations create different dependencies. `map()` leads to narrow dependencies, whereas `join()` leads to wide dependencies. Narrow dependencies can run on the same worker machines without having to go over the network {% cite usenix180560 -l 6 %}.

Since being released, Spark has introduced another abstraction: the DataFrame. DataFrames organize data into named columns (see [Spark SQL, DataFrames and Datasets Guide](https://spark.apache.org/docs/latest/sql-programming-guide.htm)).

Spark as described in the original paper didn't work well with stream processing, but Spark Streaming was later created to add support for stream processing.

## Stream processing

**Stream processing** involves continuously processing unbounded data. Data is normally separated into time-based windows of a given length, which is then processed to produce a result.

Streaming enables low-latency results.

### Streaming windows

**Streaming windows** are finite, usually time-based, segments of data that are grouped and then processed together into a result.

Three common window patterns:

- **Sliding windows**—fixed size windows with a slide period (windows may overlap). e.g. 60m windows created every 1m.
- **Fixed windows**—fixed-sized and generally aligned (no overlapping). e.g. 30m window. Fixed windows are a special case of sliding windows.
- **Session windows**—windows that capture a period of activity. They are defined by a timeout gap. Any events that occur within the timeout of each other are grouped.

{% cite google-43864 -l 1794 %}

## References

{% bibliography --cited_in_order %}
