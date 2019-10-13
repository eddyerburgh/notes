---
layout: default
title: Distributed systems
description: An introduction to distributed systems.
has_children: true
has_toc: false
nav_order: 4
permalink: /distributed-systems
---

<!-- prettier-ignore-start -->

# Distribtued systems
{:.no_toc}


## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

"A distributed system is a collection of autonomous computing elements that appears to its users as a single coherent system" {% cite distributed-systems -l 2 %}.

## Introduction

Distributed systems consist of autonomous computing elements, defined as either hardware devices or software processes. Generally, these computing elements are referred to as **nodes** {% cite distributed-systems -l 2 %}.

A fundamental principle of distributed systems is that nodes can act independently of each other. They communicate by sending and receiving messages and then use these messages to determine how they should behave {% cite distributed-systems -l 2-3 %}.

In practice, distributed systems are often organized as an **overlay network** — a network built on top of another network. There are two common overlay networks:

1. Structured overlay, where each node has a well-defined set of neighbors it can communicate with.
2. Unstructured overlay, where nodes communicate with a randomly selected set of nodes.

{% cite distributed-systems -l 3 %}

Distributed systems are often organized with a layer of software that is placed between the applications running on a computer, and the operating system. This software is often known as **middleware** {% cite distributed-systems -l 5 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/middleware.svg" alt="">
  <figcaption><h4>Figure: A distributed system using a middleware layer {% cite distributed-systems -l 5 %}</h4></figcaption>
</figure>

Middleware acts as an operating system for nodes in a distributed system. It provides a similar set of services as an operating system (facilities for interprocess communication, accounting services, recovery from failure). The difference is that the services are provided in a networked environments {% cite distributed-systems -l 5 %}.

## Design goals

The design goals of distributed systems include:

1. Supporting resource sharing.
2. Making distribution transparent.
3. Being scalable.

{% cite distributed-systems -l 8 %}

Distributed systems provide transparency, which hides the fact that the resources are distributed. There are different types of transparency:

| Transparency | Description                                                             |
| ------------ | ----------------------------------------------------------------------- |
| Access       | Hide differences in data representation and how a resource is accessed. |
| Location     | Hide where a resource is located.                                       |
| Migration    | Hide that a resource may have moved locations.                          |
| Replication  | Hide that a resource is replicated.                                     |
| Concurrency  | Hide that a resource may be shared by several users.                    |
| Failure      | Hide failure and recovery of resources.                                 |

{% cite distributed-systems -l 8 %}

## Scalability

Scalability is an important design goal for developers of distributed systems.

Scalability can be described in three dimensions:

- Size scalability
- Geographical scalability
- Administrative scalability

Many systems rely on centralized servers, or a centralized cluster of servers acting as a single service. This can become a bottleneck as traffic grows in size {% cite distributed-systems -l 15 %}.

There are three root causes for bottlenecks:

1. Computational capacity.
2. Storage capacity, including the I/O transfer rate.
3. The network between the user and the centralized service.

{% cite distributed-systems -l 15-6 %}

Geographical scaling also comes with problems. Interprocess communication across large geographical regions can take hundreds of milliseconds, and wide-area networks are inherently unreliable {% cite distributed-systems -l 17-8 %}.

### Scaling techniques

If performance problems are caused by capacity, then increasing the size of a machines resources (**scaling up**) can solve the problem. The downside to scaling up is that the price of hardware grows exponentially.

The other approach is to **scale out** by increasing the number of nodes in a system. There are two common techniques for scaling out:

1. Partitioning and distribution.
2. Replication.

**Partitioning** and **distribution** involves splitting a component into smaller parts and spreading it throughout a system {% cite distributed-systems -l 21 %}.

**Replication** involves replicating components throughout a distributed system. Replication improves availability and helps balance load {% cite distributed-systems -l 22 %}.

**Caching** is a special form of replication, where a resource is copied by a client and saved for later use. The distinction here is that the decision to cache is made by a client, whereas the decision to replicate is made by the owner of a resource {% cite distributed-systems -l 23 %}.

The downside of caching and replication is that they can lead to problems with **consistency**, where resources get out-of-date {% cite distributed-systems -l 23 %}.

The required level of consistency depends on the system. It may be acceptable for Internet users to be served an out-of-date blog post, but it's not acceptable for a bank's customers to receive out-of-date account balances {% cite distributed-systems -l 23 %}.

Strong consistency requirements introduces new problems. Any update must be immediately propagated to all replicas and if two updates occur at the same time, they will need to be made in the same order on each replica. Solving this requires the use of a global synchronization mechanism, which can be difficult to implement in a scalable way {% cite distributed-systems -l 23 %}.

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

Most distributed systems principals deal directly with these assumptions {% cite distributed-systems -l 24 %}.

## References

{% bibliography --cited_in_order %}
