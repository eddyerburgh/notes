---
layout: default
title: Coordination and time
description: Notes on coordination and time in distributed systems.
has_children: true
has_toc: false
nav_order: 5
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

Clock synchronization in a distributed system is difficult since processes running on different machines will have different hardware clocks {% cite distributed-systems -l 298 %}.

Hardware clocks suffer from clock drift. An average quartz-based hardware clock has a clock drift rate of around 31.5 seconds per year {% cite distributed-systems -l 303 %}.

Clock synchronization is needed to overcome the problem of clock drift in order to provide a holistic view of a system, for example when reading logs from multiple processes.

Two important timekeeping concepts are precision and accuracy. **Precision** refers to the deviation between two clocks. **Accuracy** refers to the precision in reference to an external value, like UTC {% cite distributed-systems -l 303 %}.

**Internal clock synchronization** is the process of keeping clocks precise, whereas **external clock synchronization** is the process of making clocks accurate {% cite distributed-systems -l 303 %}.

**UTC** is the primary time standard used to regulate time. There are shortwave radio stations around the world that broadcast a pulse at the start of each UTC second, as well as satellites that offer a UTC service {% cite distributed-systems -l 302 %}.

A UTC receiver is hardware that can receive UTC from providers such as satellites. Other computers can sync their clocks to a computer equipped with a receiver in order to follow UTC {% cite distributed-systems -l 302 %}.

When a computer's clock is later than the actual time, the clock must be readjusted. Setting the clock to the earlier correct time can lead to problems for programs that rely on an ever-increasing time value. To solve this, the time change can be introduced gradually. One way to do this is to reduce the number of milliseconds that the computer time is incremented by on each tick until the local time has caught up to the time the computer is synchronizing with.

There are many algorithms to synchronize several computers to one machine with a UTC receiver or for keeping a set of machines as synchronized as possible.

### NTP

NTP (Network Time Protocol) is a protocol for synchronizing clocks over a network {% cite distributed-systems -l 304 %}.

In NTP, the clock offset and roundtrip delay between two machines is estimated by sending messages between them. The offset can then be used to adjust the local time {% cite distributed-systems -l 305 %}.

NTP divides servers into strata. A server with a reference clock is known as a stratum-1 server. When server $$A$$ contacts server $$B$$, it will only adjust its time if its stratum level is higher than server $$B$$. Once $$A$$ has synchronized, it will change its stratum level to $$stratum(B) + 1$$ {% cite distributed-systems -l 306 %}.

### The Berkley algorithm

The Berkley algorithm is an internal clock synchronization algorithm. It works by having a time daemon poll other machines periodically to get their respective times. The daemon then computes an average time using the answers and tells other machines to advance/slow their clocks to the new time {% cite distributed-systems -l 306 %}.

The Berkley algorithm works well for systems that don't have a UTC receiver {% cite distributed-systems -l 306 %}.

## Ordering

Ordering is the process of assigning an order to events in a distributed system.

Histories are sequences of operations. Ordered histories are used to ensure operations are executed deterministically on multiple distributed nodes.

In distributed systems, a **total order** is an order where each event happens one after another. A total order can be created by ordering according to the time that an event happened and then breaking ties in the case of two events occurring at the same time (e.g., by comparing the originator's process ID) {% cite Lamport:1978:TCO %}.

A **partial order** is where some events are ordered one after but other events are not ordered relative to each other.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/distributed-systems-coordination/ordering.svg" alt="">
  <figcaption><h4>Figure: Total and partial ordering</h4></figcaption>
</figure>

### Serializability

A transaction history is defined as **serializable** if the outcome of executing the transactions in the history is the same as if the transactions were executed sequentially one after the other {% cite papadimitriou1979serializability -l 631 %}.

## Logical clocks

**Logical clocks** are abstract clocks that have no relation to physical time. They can be used in distributed systems to create an ordering of events {% cite Lamport:1978:TCO -l 559 %}.

### Lamport's logical clocks

Leslie Lamport introduced a logical clock system in the paper _Time, Clocks, and the Ordering of Events in a Distributed System_ {% cite Lamport:1978:TCO %}.

The aim of the paper was to create an ordering of events that occur in a system that doesn't have a precise physical time available to all processes {% cite Lamport:1978:TCO -l 559 %}.

In the paper, Lamport introduces the happened-before relation (denoted: $$ \rightarrow$$). $$a \rightarrow  b$$ means event $$a$$ occurred before event $$b$$.

Using Lamport's assumptions, there are only three ways to determine that event $$a$$ ocurred before event $$b$$:

1. If $$a$$ and $$b$$ are events running in the same process and $$a$$ occurs before $$b$$.
2. If $$a$$ is the event of a message being sent by a process and $$b$$ is the event of the same message being received by a different process.
3. If $$a \rightarrow  b$$ and $$b \rightarrow  c$$ then $$a \rightarrow  c$$ (happened-before is a transitive relation).

The happened-before relation can be used to create a _partial ordering_ of events. A _total ordering_ can also be created by breaking ties arbitrarily.

Lamport's logical clocks work by sending a time value (an event counter) along with messages that are sent by the processes. If the receiving process has a time value that is less than the time value sent with the message, then the receiving process' time value is incorrect (because $$a \rightarrow b$$ when $$a$$ is a message being sent and $$b$$ is the same message being received). In this case, the receiver updates it's time to be one more than the sender's time {% cite distributed-systems -l 312 %}.

In Lamport's description, each process $$P_i$$ maintains a local time value $$C_i$$. The time value is updated according to te following rules:

1. $$P_i$$ increments $$C_i$$ before executing an event.
2. When $$P_i$$ sends a message $$m$$ to process $$P_j$$ it sets $$m$$'s timestamp to $$C_i$$ after executing step 1.
3. When $$P_j$$ receives a message it adjusts $$C_j$$ to $$C_i$$ if $$C_i \lt C_j$$.

{% cite distributed-systems -l 312 %}

In the case that two events have the same event counter value, the tie is broken arbitrarily by assigning a numeric value to a process' ID and using the ID value to compare the conflicting events {% cite Lamport:1978:TCO -l 561 %}.

Let $$C(a)$$ return the time value of event $$a$$. Lamport clocks provide a total ordering of events, where event $$C(a) \lt C(b)$$ if $$a \rightarrow b$$. However, Lamport clocks can't guarantee that $$a$$ occurred before $$b$$ just because $$C(a) \lt C(b)$$, in other words Lamport's clock can't guarantee event causality.

One solution to determine causality is to use vector clocks.

### Vector clocks

Vector clocks extend on logical clocks to help capture causal ordering {% cite fidge1988timestamps -l 57 %}
.

With vector clocks, timestamps are represented as a vector where each index $$i$$ in the vector holds the event counter of a process $$P_i$$, e.g.,: $$[c_1, c_2, c_n]$$.

Let $$VC_i$$ be the vector clock of process $$P_i$$, and $$vc(m)$$ be the vector clock of message $$m$$:

1. $$VC_i[i]$$ is the number of events that have occurred so far at $$P_i$$.
2. $$VC_i[j]$$ is $$P_i$$'s knowledge of the number of events that have happened at $$P_j$$.

{% cite distributed-systems -l 318 %}

Vector clocks are sent with messages according to the following rules:

1. Before executing an event, $$P_i$$ increments $$VC_i[i]$$.
2. When $$P_i$$ sends a message $$m$$ to process $$P_j$$ it sets $$m$$'s vector timestamp to $$VC_i$$ after executing step 1.
3. When $$P_j$$ receives a message it adjusts $$VC_j[k]$$ to $$max(VC_j[k], vc(m)[k])$$ for each $$k$$ (equivalent to merging causal histories). (what about incrementing?)

{% cite distributed-systems -l 318 %}

This data allows a process to determine whether an event must have occurred before another event by comparing the values of each process' timestamp at the time an event occurred {% cite fidge1988timestamps -l 59 %}. You can also use vector clocks to determine if two events are concurrent, which, depending on the event, could cause conflicts {% cite distributed-systems -l 319 %}.

## Mutual exclusion

Mutual exclusion is needed to ensure that resources cannot be accessed multiple times, since concurrent access could lead to corrupted data {% cite distributed-systems -l 321 %}.

There are two main categories of distributed mutual exclusion algorithms:

1. Token-based solutions.
2. Permission-based solutions.

In token-based solutions, a token is passed between processes. Only processes that hold a token can access the shared resource. The downside to token-based solutions is that if the token is lost a new token must be generated (which can be a complicated procedure) {% cite distributed-systems -l 322 %}.

In permission-based solutions, a process that wishes to access a resource must gain permission from other processes first {% cite distributed-systems -l 322 %}.

## Election algorithms

Election algorithms are used to pick a node to perform a special role (e.g., a coordinator) {% cite distributed-systems -l 329 %}.

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

<!--

Partial ordering .

A single process is defined to











a -> b means it’s possible for a to causally affect b (but not guaranteed). Two events that cannot causally affect each other are said to be concurrent (event though one event probably
occurred before the other)  L 559





Leslie Lamport defined the happened-before relationship (denoted: $$ \rightarrow$$) to help synchronize logical clocks. $$a \rightarrow  b$$ means event $$a$$ happened before event $$b$$. Happened-before is a transitive relationship 310.





There

Logical clocks can be implemented with counters without a timing mechanism 559.



Lamport discusses how a partial ordering can be created using happened-before, and how lamport timestamps can be used to create a total ordering (558).

Partial ordering .

A single process is


https://lamport.azurewebsites.net/pubs/time-clocks.pdf

Logical clocks use event counters as a mechanism for measuring the order of events in distributed systems 310.

An event is either causally related or concurrent. Two events are causally related if event a caused event b or event b caused event a. Two events are concurrent if ... . Logical clocks can tell you

Total ordering is , whereas partial ordering is .

In Time, Clocks, and the Ordering of Events in a Distributed System, Leslie Lamport

### Lamport timestamps

Happened before creates a partial ordering.
Logical clocks use event counters as a mechanism for measuring the order of events in distributed systems 310.

Leslie Lamport defined the happened-before relationship (denoted: $$ \rightarrow$$) to help synchronize logical clocks. $$a \rightarrow  b$$ means event $$a$$ happened before event $$b$$. Happened-before is a transitive relationship 310.

There are two cases where happened-before can be directly observed:

1. If $$a$$ and $$b$$ are events running in the same process and $$a$$ occurs before $$b$$.
2. If $$a$$ is the event of a message being sent by a process and $$b$$ is an event of the same message being received by a process.

310

If two events $$x$$ and $$y$$ happen in different processes that don't exchange messages, then the two events are said to be concurrent ($$x \rightarrow  y$$ is not true and $$y \rightarrow  x$$ is also not true) 310-1.

Let $$C({\text a })$$ be the time value of event $$a$$ that is agreed by all communicating processes. The time value should have the property that $$C({\text a }) \lt C({\text b })$$ if $$a \rightarrow  b$$. As an additional requirement, $$C$$ must always go forward, never backward 311.

Lamport's logical clocks work by sending a time value (which is an event counter) along with any messages. If a receiving process has a local time value that is greater than the time value that sent with with the message, then everything is fine. If the receiving process has a time value that is less than the time value sent with the message, then the receiving process' time value is incorrect (because $$a \rightarrow  b$$ when a is a message being sent and b is the same message being received). The solution is to update the receiver's time to be one more than the sender's time 312.

In the implementation, each local process ($$P_i$$) maintains a local counter ($$C_i$$) that is used for the time value. The counter is updated accordingly:

1. $$P_i$$ increments $$C_i$$ before executing an event.
2. When $$P_i$$ sends a message $$m$$ to process $$P_j$$ it sets $$m$$'s timestamp to $$C_i$$ after executing step 1.
3. When $$P_j$$ receives a message it adjusts $$C_j$$ to $$C_i + 1$$ if $$C_i \lt C_j$$.

312

This provides a partial ordering. In order to create a total ordering, ties in the case of events occurring at the same time, the process identifier is sent in a tuple along with the counter value 313.

Lamport clocks provide a total ordering, where event $$a$$ is guaranteed to be before $$b$$ if $$a \rightarrow. However, Lamport clocks can't guarantee that events causally affected each other. One solution to determine causality is vector clocks.

<!-- Another way of putting this is that Lamport clocks don't capture causality 316-7. -->

<!-- Vector clocks were developed separately by Fidge and ... in 1988. The problem statement for Fidge was to determine whether to events $$a$$ and $$b$$ happened before each other in all valid variations of the distributed computation. In other words, Fidge wanted to determine whether $$a$$ causally affected event $$b$$ 57.

Lamport's clock return the total ordering of events but lost key information.

Vector clocks extend on logical clocks to help capture causal ordering, in other words to capture what event happened another 317. -->

<!-- Lamport clocks can lead to a situation where all events in a system are totally ordered, but you can't know for certain that event $$a$$ happened before event $$b$$ by simply comparing their time value. Lamport clocks don't capture causality 316-7. -->

<!-- You can capture causality by assigning each event a unique name (using the process ID and a locally incrementing counter). Capturing causal history is more difficult (is it?) 317

If two local events happen at process $$P$$ then $$H(p2)$$ is the causal history of event $$p2$$ ($${p1, p2}$$). If P sends an event to Q (recording the event as $$pk$$) and at the time of arrival the most recent event for q was q1, P sends its most recent causal history {p1,p2, p3}. Process Q records the event of receiving the message (q2) and merges th two causal histories into a new causal history: {p1,p2,p3,q1,q2} 317-8. An event p causally preceeds event q if H(p) \subset H(q).

This representation of causal histories is not efficient (since it includes every event). Rather than keeping track of all events, you can simply keep track of the last event. Each process can be assigned an index (used in a vector). Causality can then be captured in these vector clocks, where the entry at index j represents the number of events that have happened at Pj 318. -->
<!--
The properties of a vector clock are:

1. VC_i[i] is the number of events that have occurred so far at P_i (VC_i[i] is the logical clock of p[i).
2. VC_i[j] is P_i's knowledge of the number of events that have happened at P_j.

318

Vectors are then sent with messages according to the following rules:

1. Before executing an event, P_i increments VC_i[i]
2. When $$P_i$$ sends a message $$m$$ to process $$P_j$$ it sets $$m$$'s vector timestamp to $$VC_i$$ after executing step 1.
3. When $$P_j$$ receives a message it adjusts $$VC_j[k]$$ to $$max(VC_j[k], ts(m)[k])$$ for each k (equivalent to merging causal histories).

318 -->
