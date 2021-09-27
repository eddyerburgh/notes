---
layout: default
title: Distributed systems
description: An introduction to distributed systems.
has_children: true
has_toc: false
nav_order: 5
permalink: /distributed-systems
---

<!-- prettier-ignore-start -->

# Distributed systems
{:.no_toc}

A distributed system is a collection of computers that appears to an end-user as a single system.

Most websites today are made up of multiple computers communicating over a network, so learning about distributed systems is useful for any web programmer.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

Distributed systems consist of autonomous computing elements defined as either hardware devices or software processes. Generally, these computing elements are referred to as **nodes** {% cite distributed-systems -l 2 %}.

Some examples of distributed systems include multi-tier web applications, distributed storage systems, and the Internet itself.

A fundamental principle of distributed systems is that nodes can act independently of each other. Nodes communicate by sending and receiving messages, which the nodes then use to determine how they should behave {% cite distributed-systems -l 2-3 %}.

Distributed systems are often organized as an **overlay network**—a network built on top of another network. There are two common overlay networks:

1. **Structured overlay** where each node has a well-defined set of neighbors it can communicate with.
2. **Unstructured overlay** where nodes communicate with a randomly selected set of nodes.

{% cite distributed-systems -l 3 %}

Often nodes in a distributed system run a layer of software that is placed between the applications running on a computer and the OS. This software is known as **middleware** {% cite distributed-systems -l 5 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/middleware.svg" alt="">
  <figcaption><h4>Figure: A distributed system using a middleware layer {% cite distributed-systems -l 5 %}</h4></figcaption>
</figure>

Middleware provides a similar set of services as an operating system, such as facilities for interprocess communication, accounting services, and recovery from failure {% cite distributed-systems -l 5 %}.

The two common reasons for creating distributed systems are:

1. To improve performance by splitting work across multiple computers.
2. To make a system fault tolerant.

## Performance and scalability

You can monitor the performance of a system by measuring the latency and the throughput.

**Latency** is the time it takes to perform an action. Services often make SLAs (Service Level Agreements) with their customers, which can include a maximum latency.

**Throughput** is the number of actions that are completed in a specified time period (e.g. per second).

Generally, you should aim for maximal throughput with acceptable latency.

**Scalability** is the ability of a system to scale. It's an important design goal for developers of distributed systems.

"A service is said to be scalable if when we increase the resources in a system, it results in increased performance in a manner proportional to resources added" {% cite a-word-on-scalability %}.

The **thundering herd problem** occurs when a large number of processes waiting on an event get woken up when the event happens. All woken events attempt to handle the event but (usually) only one will win. This can cause severe performance issues.

In order to remain scalable, you need to remove bottlenecks in a system. A server can become a bottleneck for multiple reasons:

1. Insufficient compute capacity.
2. Insufficient storage capacity (including the I/O transfer rate).
3. Poor network conditions.

{% cite distributed-systems -l 15-6 %}

These bottlenecks can be improved by applying scaling techniques.

### Scaling techniques

There are two main scaling techniques: vertical scaling and horizontal scaling.

**Vertical scaling** (scaling up) involves adding more resources to a machine (i.e. adding more RAM or processing power). The downside to vertical scaling is that the price of hardware grows exponentially and quickly becomes prohibitive.

**Horizontal scaling** (scaling out) involves increasing the number of nodes in a system (often these nodes use commodity hardware).

There are two common techniques for scaling out:

1. Partitioning and distribution
2. Replication

**Partitioning and distribution** involves splitting a component into smaller parts and spreading it throughout a system {% cite distributed-systems -l 21 %}.

**Replication** involves replicating components throughout a system. Replication improves availability and helps balance load {% cite distributed-systems -l 22 %}.

**Caching** is a special form of replication where a resource is copied by a client and saved for later use. The distinction here is that the decision to cache is usually made by the client, whereas the decision to replicate is made by the owner of the resource {% cite distributed-systems -l 23 %}.

The downside to caching and replication is that they can lead to problems with **consistency**, where data gets out-of-sync across replications {% cite distributed-systems -l 23 %}.

The required level of consistency depends on the system. It may be acceptable for Internet users to be served an out-of-date blog post, but it's not acceptable for a bank's customers to receive out-of-date account balances {% cite distributed-systems -l 23 %}.

Achieving strong consistency introduces new problems. Any update must be immediately propagated to all replicas and if two updates occur at the same time, they will need to be made in the same order on each replica. Solving this requires the use of a global synchronization mechanism, which can be difficult to implement in a scalable way {% cite distributed-systems -l 23 %}.

## Fault tolerance

**Fault tolerance** is the ability of systems to continue operating in the event of failure.

When a distributed system is made up of a large number of nodes, rare failures become commonplace. Therefore, distributed systems should have fault tolerance built into their design.

One aspect of fault tolerance is availability. **Availability** is "the property that a system is ready to be used immediately" {% cite distributed-systems -l 424 %}. There is always some set of failures that will mean the system is no longer available (e.g. if there are failures at multiple data centers at the same time).

Availability is measured as a percentage of uptime. Commonly, a system might guarantee 99.99% availability (known as "four nines"), meaning that the system will only be down for 52.6 minutes per year. Systems that guarantee several nines of availability are said to be highly available.

Another aspect of fault tolerance is recoverability. **Recoverability** is the ability of a system to recover after becoming unavailable due to failure. This could be achieved by saving the system state to non-volatile storage (in the form of logs or checkpoints) or by replicating data across nodes.

## Virtualization

**Virtualization** "deals with extending or replacing an existing interface so as to mimic the behavior of another system" {% cite distributed-systems -l 116 %}.

Virtualization is often used to run multiple server applications on a shared platform {% cite distributed-systems -l 117-8 %}.

In general, there are three different levels of abstraction for virtualization:

1. At the interface between hardware and software (the instruction set architecture).
2. At the system call interface.
3. As part of a library API interface.

{% cite distributed-systems -l 118 %}

One approach to virtualization is to create a runtime environment that provides an abstract instruction set that's used by executing applications {% cite distributed-systems -l 118 %}.

Another approach is to provide a system that layers the hardware but provides an identical interface as the hardware it's shielding. This is the approach used by many Virtual Machine Monitors, more commonly known as hypervisors.

A **hypervisor** is software (or hardware) that's used to run **virtual machines**—emulations of a computer system. They are often used in data centers and in cloud computing (e.g. AWS use hypervisors to run EC2 instances) {% cite distributed-systems -l 119-20 %}.

Virtual machines provide several benefits:

- Isolation.
- Portability.
- More efficient use of hardware.

{% cite distributed-systems -l 124 %}

## Servers

A server is "a process implementing a specific service on behalf of a collection of clients". Normally, servers listen for incoming requests from clients and respond to those requests in some way {% cite distributed-systems -l 129 %}.

Server architectures are either concurrent or iterative. **Concurrent servers** pass requests to a new process or thread, after which the server then waits for the next incoming request {% cite distributed-systems -l 129 %}.

**Iterative servers** handle requests in the same server process. These tend to be event-driven, like Nginx.

Stateful servers keep state about clients locally. Stateless servers don't keep state locally (or at least, service will not be disrupted if the state is lost) {% cite distributed-systems -l 131 %}.

**Session state** is state that's associated with operations of a single user. It's maintained for a period of time but not permanently {% cite distributed-systems -l 132 %}.

**Server clusters** are groups of servers connected through a network. Local-area clusters often have high bandwidth and low latency {% cite distributed-systems -l 141 %}.

Clusters are usually organized logically into three tiers:

- A logical switch.
- Application servers.
- Distributed file/data systems.

Often commodity machines are used to build clusters {% cite distributed-systems -l 142 %}.

Wide-area clusters are clusters connected across the Internet. These are becoming more common with the popularity of cloud computing, where cloud providers often have data centers distributed geographically around the world.

## The fallacies of distributed computing

Peter Deutsch codified the following false assumptions that engineers make when dealing with distributed systems, known as the **fallacies of distributed computing**:

1. The network is reliable.
2. The network is secure.
3. The network is homogeneous.
4. The topology does not change.
5. Latency is zero.
6. Bandwidth is infinite.
7. Transport cost is zero.
8. There is one administrator.

Most distributed systems principles deal directly with these assumptions {% cite distributed-systems -l 24 %}.

## References

{% bibliography --cited_in_order %}
