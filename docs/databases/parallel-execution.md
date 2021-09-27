---
layout: default
title: Parallel execution
description: Parallel execution in RDBMSes.
has_children: true
has_toc: false
parent: Databases
nav_order: 5
permalink: /databases/parallel-execution
---

<!-- prettier-ignore-start -->

# Parallel execution
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

Parallel execution can improve DBMS performance significantly.

The main types of parallelism in DBMSes:

1. Inter-query parallelism
2. Intra-query parallelism
3. I/O parallelism

### Inter-query parallelism

**Inter-query parallelism** is where multiple queries are executed concurrently, improving throughput and reducing latency.

Inter-query parallelism doesn't require must coordination for reads, but writes require coordination between the workers.

### Intra-query parallelism

**Intra-query parallelism** is where operations of a single query are executed in parallel. This decreases latency for long-running queries.

Intra-query parallelism can be implemented with both inter-operator and intra-operator parallelism (intra-operator and inter-operator aren't mutual exclusive).

**Intra-operator parallelism** is where an operator is split into independent instances that perform the same work on different subsets of data. Generally this is implemented by inserting an Exchange operator to coalesce results from child operators.

**Inter-operator parallelism** (pipelined parallelism) is where operators are overlapped in order to pipeline data from one stage to the next without materialization. When data is calculated by an operator it's emitted and picked up by the next stage. This type of parallelism is more common in stream processing systems.

{% cite 15445-notes-14 %}

### I/O parallelism

**I/O parallelism** involves splitting a DBMS across multiple storage systems. This can be done in several ways.

**Multi-Disk Parallelism** stores the DBMS’s files across multiple storage devices (e.g. using RAID).

**File-based Partitioning** involves specifying the disk location of each individual database. The buffer pool manager maps a page to a disk location. One problem here is maintaining the log.

**Logical partitioning** is when you split a single logical table into multiple disjoint physical segments that are stored/managed separately. Ideally, partitioning is transparent to the application (e.g. in sharding you could have a router that routes a query request to the relevant shard). In horizontal logical partitioning you divide tuples into set of tables based on some partitioning key.

{% cite 15445-notes-14 %}

### Process model

A DBMS process model defines how the system handles concurrent requests from a multi-user application. In both models, the DBMS is made up of one or more workers that executes tasks on behalf of the client and returns results.

The main models are:

- Process-per-worker
- Thread-per-worker

Process-per-worker is where a single OS process corresponds to a worker.

Thread-per-worker is where a single thread corresponds to a worker. This is the most common approach. In thread-per-worker, the DBMS is responsible for scheduling workers.

{% cite 15445-notes-14 %}

## Transactions

A **transaction** is a sequence of one or more operations that executed on a shared database. They are intended to implement some higher-level application function (e.g. a transfer between bank accounts). Transactions are the basic unit of change in a DBMS.

Formally, we can model transactions as a sequence of read and write operations ($$\text{R}(\text{A})$$, $$\text{W}(\text{B})$$) on a database, where a database is a set of named data objects $$(\text{A}, \text{B}, \text{C}, ..)$$.

_Note: insertions are ignored until the Phantom reads section later._

{% cite 15445-notes-16 %}

## Concurrency control

A **concurrency control protocol** is "how the DBMS decides the interleaving of operations from multiple transactions". Interleaving transactions is done to maximize concurrency.

There are two main concurrency control protocols:

- Optimistic: assume transactions won't conflict. Only resolve if there is a conflict.
- Pessimistic: assume transactions will conflict. Resolve conflicts preemptively.

An **transaction schedule** (or history) is a sequence of database operations performed by a set of transactions. The aim of a concurrency control protocol is to generate a transaction schedule that is equivalent to some serial schedule.

A **serial schedule** is one that does not interleave the actions of different transactions.

An **equivalent schedule** is where, for a given database state, the effect of executing the first schedule is identical to the effect of executing the second schedule.

A schedule is **serializable** if it is equivalent to some serial execution of the transactions.

There are three types of conflicts that can occur between two transactions:

- Read-write conflicts ("unrepeatable reads")
- Write-read ("dirty reads")
- Write-write ("lost updates")

Given these possible conflicts, we can determine whether a schedule is conflict serializable.

Conflict serializability can be checked using a precedence graph. A precedence graph is a dependency graph where an edge from $$T_i$$ to $$T_j$$ represents an operation $$O_i$$ of $$T_i$$ that conflicts with operation $$O_j$$ of $$T_j$$ and $$O_i$$ appears earlier in the schedule than $$O_j$$. A schedule is conflict serializable iff the precedence graph is acyclic.

{% cite 15445-notes-16 %}

## Two-phase locking

**Two-phase locking** (2PL) is a pessimistic concurrency control protocol that uses locking to control access to DB objects at runtime.

In 2PL, locks are acquired and released in two phases:

1. Growing
2. Shrinking

In the Growing phase, a transaction requests the lock it needs from the lock manager. The lock manager grants/denies requests based on already-held locks.

In the Shrinking phase, locks are released and no locks are acquired.

2PL uses two types of lock:

- A **shared lock** (S) is associated with a database object by a transaction before reading.
- An **exclusive lock** (X) is associated with a database object by a transaction before writing.

Basic 2PL is sufficient to guarantee conflict serializability, but it's subject to cascading aborts. For example, in the case of a dirty read where transaction $$T_j$$ reads an uncommitted write from $$T_i$$ and then $$T_i$$ aborts, $$T_j$$ must abort too (also $$T_j$$ can't commit until $$T_i$$ commits).

{% cite 15445-notes-17 %}

### Strict two-phase locking

In strict two-phase locking, the transaction is not allowed to release any locks until the transaction is committed.

Strict 2PL locking avoids cascading aborts.

### Deadlock handling

2PL can lead to deadlocks. A **deadlock** is a cycle of transactions waiting for locks to be released by each other.

There are two ways to handle deadlocks in 2PL:

1. Deadlock detection
2. Deadlock prevention

#### 2PL deadlock detection

**Deadlock detection** involves detecting deadlocks as they happen and then resolving the deadlock.

Deadlock detection can be implemented with a waits-for graph that tracks the locks that each transaction is waiting to acquire. Nodes are transactions, and an edge from $$T_i$$ to $$T_j$$ means that $$T_i$$ is waiting for $$T_j$$ to release a lock. The DBMS would then periodically checks for cycles in the waits-for graph to detect deadlock.

When a deadlock is detected, the system selects a victim transaction to roll back in order to break the cycle. The victim transaction will then either restart or abort depending on how it was invoked.

{% cite 15445-notes-17 %}

#### 2PL deadlock prevention

In 2PL **deadlock prevention**, when a transaction requests a lock held by another transaction the DBMS kills one of the transactions to prevent deadlock. 2PL deadlock prevention doesn't require a waits-for graph or a detection algorithm.

To decide which transaction should be killed, transactions can be assigned a priority based on timestamps of when the transaction arrived in the system, where an older timestamp has higher priority.

The Wait-die ("Old waits for young") strategy is where if $$T_i$$ has higher priority, then $$T_i$$ waits for $$T_j$$. Otherwise, $$T_i$$ aborts.

The Wound-wait ("Young waits for old") strategy is where if $$T_i$$ has higher priority, $$T_j$$ aborts and releases the lock. Otherwise, $$T_i$$ waits.

{% cite 15445-notes-17 %}

### Multiple granularity locking

In **multiple granularity locking**, there is a lock hierarchy for different granularities of the DB. e.g. table, tuple, attribute.

**Intention locks** allow a higher-level node to be locked in either shared or exclusive mode. If a node is in an intention mode, then explicit locking is being done at a lower level in the lock tree.

There are three types of intention locks:

- **Intention-shared** (IS) indicates explicit locking at a lower level with shared locks.
- **Intention-exclusive** (IX) indicates explicit locking at a lower level with exclusive or shared locks.
- **Shared+intention-shared** (SIX) indicates that the subtree rooted by that node is locked explicitly in shared mode and explicit locking is done at a lower level with exclusive locking.

Each transaction acquires an appropriate lock at the highest level of the DB hierarchy. To get an S or IS lock on a node, the transaction must hold at least an IS lock on the parent node. To get an X, IX, or SIX on a node the transaction must hold at least an IX lock on the parent node.

Hierarchical locks minimize the number of locks a transaction must acquire to do the required work.

{% cite 15445-notes-17 %}

## Timestamp ordering

**Timestamp ordering** is a class of optimistic concurrency control protocols. Timestamp ordering protocols use timestamps assigned to transactions to determine the serializability order.

Each transaction $$T_i$$ is assigned a unique fixed timestamp that is monotonically increasing. Let $$\text{TS}(T_i)$$ be the timestamp assigned to $$T_i$$. Different schemes assign timestamps at different times during the transaction (e.g. when the transaction starts or when the transaction completes).

If $$\text{TS}(T_i) \lt \text{TS}(T_j)$$ the DBMS must ensure that the execution schedule is equivalent to a serial schedule where $$T_i$$ appears before $$T_j$$.

There are multiple implementation strategies for assigning timestamps:

- System clock
- Logical counter
- Hybrid

{% cite 15445-notes-18 %}

### Basic timestamp ordering protocol

In basic timestamp ordering protocol, every object $$X$$ is associated with timestamps for the last successful read/write of $$X$$.

$$\text{W-TS}(X)$$ is the last write timestamp on $$X$$, $$\text{R-TS}(X)$$ is the last read timestamp on $$X$$.

When the DBMS does a read/write, it first checks the read/write timestamp of the object to validate the operation. If the transaction tries to access an object from the future, the transaction is aborted.

Read protocol:

1. If $$\text{TS}(T_i) \lt \text{W-TS}(X)$$ the timestamp ordering of $$T_i$$ is violated with regard to the writer of $$X$$. In this case, abort $$T_i\$$ and restart with a newer TS.
2. Else:
   1. Allow $$T_i$$ to read $$X$$.
   2. Update $$\text{R-TS}(X)$$ to $$\text{max}(\text{TS}(T_i), \text{R-TS}(X))$$.
   3. Make a local copy of $$X$$ to ensure repeatable reads.

Write protocol:

1. If $$\text{TS}(T_i) \lt \text{R-TS}(X)$$ or $$\text{TS}(T_i) \lt \text{W-TS}(X)$$, abort and restart $$T_i$$ with a newer timestamp.
2. Else:
   1. Allow $$T_i$$ to write to $$X$$ and update $$\text{W-TS}(X)$$ to $$\text{TS}(T_i)$$.
   2. Make a local copy of $$X$$ to ensure reusable reads for $$T_i$$.

The basic timestamp ordering protocol generates a schedule that is conflict serializable. There are no deadlocks because a transaction never waits, but starvation can occur for long-running transactions if short transactions keep causing conflicts for the long-running transaction.

Additionally, the basic timestamp ordering protocol permits schedules that aren't _recoverable_ (a recoverable schedule is one where transactions commit only after all transactions whose changes they read have committed).

Basic timestamp ordering isn't used in many DBMSes.

{% cite 15445-notes-18 %}

### Optimistic concurrency control

The optimistic concurrency control (OCC) protocol is an optimistic timestamp-based protocol. It optimizes for the no-conflict case, so it works well when the number of conflicts is low.

In OCC, the DBMS creates a private workspace for each transaction. Any object read is copied into the workspace. Modifications are applied to the workspace. When a transaction commits, the DBMS compares the workspace write set to see whether it conflicts with other transactions. If there are no conflicts, the write set is applied to the global DB.

OCC is broken down into three phases:

1. Read Phase
2. Validation Phase
3. Write Phase

The Read phase involves tracking the read/write sets of transactions and storing their writes in a private workspace.

The Validation phase involves checking whether the commit conflicts with other active transactions. It occurs when the transaction commits. If validation fails, the DBMS aborts and restarts the transaction.

If validation succeeds, the transaction enters the Write phase and private changes are applied globally.

The Validation and Write phase are executed inside a protected critical section.

To ensure serializability, OCC maintains a global map of active transactions and their read and write sets.

$$T_i$$ checks other transactions for read-write and write-write conflicts and makes sure that all conflicts go one way (from older transactions to younger).

If $$\text{TS}(T_i)\lt \text{TS}(T_j)\$$, then one of the following three conditions must hold for a write to be safe:

1. $$T_i$$ completes all three phases before $$T_j$$ begins.
2. $$T_i$$ completes before $$T_j$$ starts its Write phase, and $$T_i$$ does not write to any object read by $$T_j$$.
3. $$T_i$$ completes its Read phase before $$T_j$$ completes its Read phase and $$T_i$$ does not write to any object that is either read or written by $$T_j$$.

OCC works well when the number of conflicts is low.

{% cite 15445-notes-18 %}

### Phantom problem

So far the notes have only considered reads and writes. Insertions introduce additional problems, e.g. the **phantom problem**, where within a transaction the same query produces different sets of tuples at different times (due to an insert occurring between the two reads).

The phantom problem can be avoided by using index locking. In index locking, a portion of the index is locked during a database access when that portion is being accessed by the transaction.

{% cite 15445-lecture-18 %}

## Isolation levels

Isolation can be relaxed to improve performance, since strong isolation can be expensive to implement when executing transactions concurrently.

There are several isolation levels in SQL that can be set for transactions.

| Isolation level  | Dirty reads | Unrepeatable reads | Phantom reads |
| ---------------- | ----------- | ------------------ | ------------- |
| SERIALIZABLE     | No          | No                 | No            |
| REPEATABLE READ  | No          | No                 | Maybe         |
| READ COMMITTED   | No          | Maybe              | Maybe         |
| READ UNCOMMITTED | Maybe       | Maybe              | Maybe         |

{% cite 15445-lecture-18 %}

#### Serializable

Serializable is the highest isolation level in SQL. In serializable mode, the execution is guaranteed to be serializable.

One strategy to implement serializable isolation in a DBMS: obtain all locks first (including index locks) and use strict 2PL.

#### Repeatable reads

In the repeatable reads isolation level, a transaction only sees data committed before the transaction began. It never sees either uncommitted data or changes committed during transaction execution by concurrent transactions, however it can suffer from phantom reads.

One strategy to implement serializable isolation in a DBMS: obtain all locks first (not including index locks) and use strict 2PL.

#### Read committed

In the read committed isolation level, a read only sees data committed before the read began. This means the same read query can return different tuples throughout the lifetime of a transaction.

One strategy to implement serializable isolation in a DBMS: obtain all locks first (not including index locks), release S locks immediately, and use strict 2PL.

#### Read uncommitted

Read uncommitted is the lowest isolation level. In this level, dirty reads are allowed, so one transaction may see not-yet-committed changes made by other transactions.

One strategy to implement serializable isolation in a DBMS: obtain all X locks first (no S locks), and use strict 2PL.

{% cite 15445-lecture-18 %}

## References

{% bibliography --cited_in_order %}
