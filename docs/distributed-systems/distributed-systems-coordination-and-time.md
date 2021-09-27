---
layout: default
title: Coordination and time
description: Notes on coordination and time in distributed systems.
has_toc: false
nav_order: 3
parent: Distributed systems
permalink: /distributed-systems/coordination-and-time
---

<!-- prettier-ignore-start -->

# Coordination and time
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

Coordination between processes can be required for several reasons:

1. To ensure two sets of data are the same (data synchronization).
2. To ensure one process waits for another to finish its operation (process synchronization).
3. To manage the interactions and dependencies between activities in a distributed system (coordination).

{% cite distributed-systems -l 297 %}

## Clock synchronization

**Clock synchronization** is the process of coordinating independent clocks.

Clock synchronization in a distributed system is difficult since processes running on different machines will have different hardware clocks and hardware clocks suffer from clock drift. An average quartz-based hardware clock has a clock drift rate of around 31.5 seconds per year {% cite distributed-systems -l 298, 303 %}.

Clock synchronization is needed to overcome the problem of clock drift in order to provide a holistic view of a system, for example when reading logs from multiple processes.

Two important timekeeping concepts are precision and accuracy. **Precision** refers to the deviation between two clocks. **Accuracy** refers to the precision in reference to an external value, like UTC {% cite distributed-systems -l 303 %}.

**Internal clock synchronization** is the process of keeping clocks precise, whereas **external clock synchronization** is the process of making clocks accurate {% cite distributed-systems -l 303 %}.

**UTC** is the primary time standard used to regulate time. There are shortwave radio stations around the world that broadcast a pulse at the start of each UTC second, as well as satellites that offer a UTC service {% cite distributed-systems -l 302 %}.

A **UTC receiver** is hardware that can receive UTC from providers such as satellites. Other computers can sync their clocks to a computer equipped with a receiver in order to follow UTC {% cite distributed-systems -l 302 %}.

When a computer's clock is greater than the actual time, the clock must be readjusted. Setting the clock to the lower correct time can lead to problems for programs that rely on an ever-increasing time value. To solve this, the time change can be introduced gradually. One way to do this is to reduce the number of milliseconds that the computer time is incremented by on each tick until the local time has caught up to the time the computer is synchronizing with.

There are many algorithms to synchronize several computers to one machine with a UTC receiver or for keeping a set of machines as synchronized as possible.

### NTP

NTP (Network Time Protocol) is a protocol for synchronizing clocks over a network {% cite distributed-systems -l 304 %}.

In NTP, the clock offset and roundtrip delay between two machines is estimated by sending messages between them. The offset can then be used to adjust the local time {% cite distributed-systems -l 305 %}.

NTP divides servers into strata. A server with a reference clock is known as a stratum-1 server. When server $$A$$ contacts server $$B$$, it will adjust its time iff $$stratum_level(A) > stratum_level(B)$$. Once $$A$$ has synchronized, $$A$$ will change its stratum level to $$stratum_level(B) + 1$$ {% cite distributed-systems -l 306 %}.

### The Berkley algorithm

The Berkley algorithm is an internal clock synchronization algorithm. It works by having a time daemon poll other machines periodically to get their respective times. The daemon then computes an average time using the answers and tells other machines to advance/slow their clocks to the new time {% cite distributed-systems -l 306 %}.

The Berkley algorithm works well for systems that don't have a UTC receiver where internal clock synchronization is adequate {% cite distributed-systems -l 306 %}.

## History

A **history** is a sequence of events that occur in a distributed system.

Informally, a history is said to be **totally ordered** if each event occurs one after the other. A total order can be created by ordering events according to the time each event occurred. In the case of multiple events occurring at the same time (concurrent events), the tie can be broken arbitrarily (e.g. by comparing the process ID of the event's originator) {% cite Lamport:1978:TCO %}.

Informally, a history is said to be **partially ordered** if some events are not ordered relative to each other.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/distributed-systems-coordination/ordering.svg" alt="">
  <figcaption><h4>Figure: Total and partial ordering</h4></figcaption>
</figure>

_Note: For the formal definition of total ordering and partial ordering, see the [total order Wikipedia page](https://en.wikipedia.org/wiki/Total_order) and the [partial order Wikipedia page](https://en.wikipedia.org/wiki/Partially_ordered_set#Formal_definition)._

### Serializability

A history is defined as **serializable** if the outcome of executing transactions (or operations) in the history is the same as if the transactions were executed sequentially one after the other {% cite papadimitriou1979serializability -l 631 %}.

## Logical clocks

**Logical clocks** are abstract clocks that have no relation to physical time. They can be used in distributed systems to create an ordering of events {% cite Lamport:1978:TCO -l 559 %}.

### Lamport logical clocks

Leslie Lamport introduced a logical clock system in the paper _Time, Clocks, and the Ordering of Events in a Distributed System_ {% cite Lamport:1978:TCO %}.

Lamport's clock system can be used to create a totally ordered history in a system that doesn't have a precise physical time available to all processes {% cite Lamport:1978:TCO -l 559 %}.

In the paper, Lamport introduces the happened-before relation (denoted: $$ \rightarrow$$). $$a \rightarrow  b$$ means event $$a$$ occurred before event $$b$$.

Using Lamport's assumptions, there are only three ways to determine that event $$a \rightarrow b$$:

1. If $$a$$ and $$b$$ are events running in the same process and $$a$$ occurs before $$b$$.
2. If $$a$$ is the event of a message being sent by a process and $$b$$ is the event of the same message being received by a different process.
3. If $$a \rightarrow  b$$ and $$b \rightarrow  c$$ then $$a \rightarrow  c$$ (happened-before is a transitive relation).

A Lamport clock system can be used to assign a number to each event. Each process $$P_i$$ has an event counter $$C_i$$ where $$C_i(a)$$ assigns a number to event $$a$$ in that process. Globally, $$C(b) = C_j(b)$$ if $$b$$ is an event that occurred in $$P_j$$ {% cite Lamport:1978:TCO -l 559 %}.

The Clock Condition states that if $$a \rightarrow  b$$ then $$C(a) \lt C(b)$$. The Clock Condition is satisfied as long as two conditions hold:

- C1: If $$a$$ and $$b$$ are events in process $$P_i$$ and $$a$$ occurs before $$b$$ then $$C_i(a) \lt C_i(b)$$.
- C2: If $$a$$ is the sending of a message by process $$P_i$$ and $$b$$ is receipt of the message by process $$P_j$$, then $$C_i(a) \lt C_j(b)$$.

{% cite Lamport:1978:TCO -l 560 %}

This can be implemented with the following rules:

- IR1: Each process $$P_i$$ increments $$C_i$$ between any two successive events.
- IR2:
  - a) If event $$a$$ is the sending of message $$m$$ by process $$P_i$$, $$m$$ contains a timestamp $$T_m = C_i(a)$$
  - b) On receiving a message $$m$$, process $$P_j$$ sets $$C_j$$ to $$max(T_m + 1, C_j)$$

{% cite Lamport:1978:TCO -l 560 %}

The clock system creates a partially ordered history because two events can have the same event counter value. To create a totally ordered history, a tie between events with the same counter value can be broken arbitrarily by assigning a numeric value to a process' ID and using the ID value to compare the conflicting events {% cite Lamport:1978:TCO -l 561 %}.

### Vector clocks

A **vector clock** is a data structure for determining the partial ordering of events in a distributed system.

Vector clocks extend on logical clocks to capture causality {% cite fidge1988timestamps -l 57 %}.

With vector clocks, timestamps are represented as a vector where each index $$i$$ in the vector holds the event counter of a process $$P_i$$, e.g.: $$[c_1, c_2, ..., c_n]$$.

Let $$VC_i$$ be the vector clock of process $$P_i$$ and $$ts(m)$$ be the timestamp of message $$m$$:

1. $$VC_i[i]$$ is the number of events that have occurred so far at $$P_i$$.
2. $$VC_i[j]$$ is $$P_i$$'s knowledge of the local time at $$P_j$$.

{% cite distributed-systems -l 318 %}

Vector clocks are sent with messages according to the following rules:

1. Before executing an event $$P_i$$ increments $$VC_i[i]$$.
2. When $$P_i$$ sends a message $$m$$ to process $$P_j$$ it sets $$m$$'s vector timestamp to $$VC_i$$ after executing step 1.
3. When $$P_j$$ receives a message it adjusts $$VC_j[k]$$ to $$max(VC_j[k], ts(m)[k])$$ for each $$k$$ (equivalent to merging causal histories). It then executes the first step to record the receipt of the message.

{% cite distributed-systems -l 318 %}

This data allows a process to determine whether an event must have occurred before another event by comparing the values of each process' timestamp at the time an event occurred {% cite fidge1988timestamps -l 59 %}.

You can also use vector clocks to determine if two events are concurrent, which, depending on the event, could cause conflicts {% cite distributed-systems -l 319 %}.

## Mutual exclusion

Mutual exclusion is needed to ensure that resources cannot be accessed multiple times, since concurrent access could lead to corrupted data {% cite distributed-systems -l 321 %}.

There are two main categories of distributed mutual exclusion algorithms:

1. Token-based solutions
2. Permission-based solutions

In token-based solutions a token is passed between processes. Only processes that hold a token can access the shared resource. The downside to token-based solutions is that if the token is lost a new token must be generated (which can be a complicated procedure) {% cite distributed-systems -l 322 %}.

In permission-based solutions, a process that wishes to access a resource must gain permission from other processes first {% cite distributed-systems -l 322 %}.

## Election algorithms

Election algorithms are used to pick a node to perform a special role (e.g. a coordinator) {% cite distributed-systems -l 329 %}.

Generally, election algorithms work by deciding which reachable node has the highest ID {% cite distributed-systems -l 329 %}.

The goal of an election algorithm is to ensure that an election process ends with all processes agreeing on an elected node {% cite distributed-systems -l 330 %}.

### The bully algorithm

The bully algorithm uses the following steps to elect a new coordinator:

1. $$P_i$$ sends an ELECTION message to all processes with higher identifiers.
2. If no one responds, $$P_i$$ wins the election and becomes coordinator.
3. If one of the higher-valued nodes responds, it takes over from $$P_i$$.

A node can receive an ELECTION message from a lower-valued node at any moment. If it does, the receiver sends OK to the sender to indicate that the node is alive and taking over, and the receiver then holds an election {% cite distributed-systems -l 330 %}.

Eventually all processes give up but one, which is the newly elected node. It announces its victory by sending a message to all processes {% cite distributed-systems -l 330 %}.

If a process that was down comes back up, it holds an election {% cite distributed-systems -l 330 %}.

## References

{% bibliography --cited_in_order %}
