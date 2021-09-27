---
layout: default
title: Distributed systems architectures
description: Notes on the architectures of distributed systems.
has_toc: false
nav_order: 1
parent: Distributed systems
permalink: /distributed-systems/distributed-systems-architectures
---

<!-- prettier-ignore-start -->

# Distributed systems architectures
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

The **system architecture** of a distributed system is how the system's software and hardware are organized.

There is a distinction between the logical organization of distributed systems (software architecture) and the physical organization {% cite distributed-systems -l 55-6 %}.

A **component** is a modular unit with a well-defined interface. Components should be replaceable without requiring the system to be powered down {% cite distributed-systems -l 56 %}.

A **connector** is a mechanism that handles communication, coordination, or cooperation between components {% cite distributed-systems -l 56 %}.

## System architectures

This section documents some common system architectures.

### Client-server architectures

In client-server architectures, processes are divided into two (sometimes overlapping) groups: servers and clients {% cite distributed-systems -l 76 %}.

A **server** is a process that implements a specific service.

A **client** is a process that uses a server's service by sending it a request and waiting for a response {% cite distributed-systems -l 76 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/distributed-systems-architectures/client-server-architecture.svg" alt="">
  <figcaption><h4>Figure: Client-server architecture</h4></figcaption>
</figure>

A client can communicate with a server using a connectionless protocol, like UDP. The downside of using a connectionless protocol is that handling transmission errors is difficult {% cite distributed-systems -l 77 %}.

One solution is to use a connection-based protocol, like TCP.

The distinction between client and server is often not clear in the client-server model {% cite distributed-systems -l 77 %}.

The client-server model is a two-tiered architecture {% cite distributed-systems -l 78 %}.

### Multitiered architectures

Many distributed applications are split into three layers:

1. User interface layer
2. Processing layer
3. Data layer

{% cite distributed-systems -l 78 %}

These layers can be distributed across different machines. An example would be a website where the browser operates at the user interface layer, a server is the processing layer, and a database operates at the data layer {% cite distributed-systems -l 78,80 %}.

The server running at the processing layer will act as a client to the database at the data layer {% cite distributed-systems -l 80 %}.

### Peer-to-peer systems

Peer-to-peer systems support distribution across clients and servers. A client or server might be physically split into logically equivalent parts, where each part is operating on its own share of the complete data set {% cite distributed-systems -l 81 %}.

In a peer-to-peer system, all processes that constitute the system are equal. Interaction between processes is symmetric: a process will act as a client and as a server at the same time {% cite distributed-systems -l 81 %}.

Peer-to-peer architectures are concerned with organizing processes in an overlay network. An **overlay network** is a network that is formed of the processes and the links that represent possible communication channels (usually TCP). An overlay network can either be structured or unstructured {% cite distributed-systems -l 81 %}.

#### Structured peer-to-peer systems

In a structured peer-to-peer system, the overlay follows a predetermined topology (e.g. a ring or a grid). The topology is used to look up data.

Generally, structured peer-to-peer systems use a semantic-free index to access data. Each data item maintained by the system is uniquely associated with a key, and the key is used as an index:

$$key(\text{data item}) = hash(\text{data item's value})$$

{% cite distributed-systems -l 82 %}

The peer-to-peer system is then responsible for storing key-value pairs. In this case, a node is assigned an identifier from all hash values, and each node is responsible for storing data associated with a subset of keys. This architecture is known as a DHT (distributed hash table) {% cite distributed-systems -l 82 %}. A DHT makes it easy for any node to find the correct node for a key:

$$\text{existing node} = lookup(key)$$

#### Unstructured peer-to-peer systems

In an unstructured peer-to-peer system, each node maintains a dynamic list of neighbors. Commonly, when a node joins an unstructured peer-to-peer system it contacts a well-known node in order to obtain a list of peers {% cite distributed-systems -l 84 %}.

Unstructured peer-to-peer systems must search for a requested value. Some examples of search methods are:

1. Flooding
2. Random walk

In flooding, an issuing node, $$U$$, passes a request for an item to each of its neighbors. A request is ignored if the receiving node, $$V$$, has seen it before. If the request hasn't been seen before, $$V$$ will search locally for the item. If the item isn't available locally, $$V$$ will issue a request to each of its neighbors and return the response to $$U$$ if it receives one {% cite distributed-systems -l 85 %}.

Random walk involves an issuing node, $$U$$, asking one of its neighbors at random. If the neighbor cannot satisfy the request, $$U$$ asks another neighbor at random, and so on until it receives an answer {% cite distributed-systems -l 85 %}.

Locating data items can become difficult as the network grows, since there is no deterministic way to route a lookup request to a specific node. Some peer-to-peer systems use special index nodes to overcome this problem {% cite distributed-systems -l 87 %}.

An alternative is to abandon symmetric communication and use a broker node to coordinate communication between a client and a server.

Nodes that maintain an index or act as a broker are known as **super peers**. Super peers are often organized in a peer-to-peer relationship.

### Pubsub systems

A pubsub system is based on asynchronous communication between publishers and subscribers.

**Publishers** publish events and relevant **subscribers** are notified of the published event.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/distributed-systems-architectures/event-based-architecture.svg" alt="">
  <figcaption><h4>Figure: Event-based architecture 68</h4></figcaption>
</figure>

There are two common forms of pubsub systems:

1. Topic-based systems
2. Content-based systems

In a **topic-based pubsub system**, a subscriber subscribes to a named _logical channel_ (a topic). Events published to the channel typically contain <attribute, value\> pairs.

In a **content-based publish-subscribe system**, a subscription can also consist of <attribute, value\> pairs. An event is only sent to a subscriber if the event's attributes match constraints provided by the subscriber {% cite distributed-systems -l 70 %}.

## Components

This section documents common components used in distributed systems.

### Load balancers

A **load balancer** is a server that distributes traffic between a cluster of backend servers. Load balancers are used to implement horizontal scaling.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/distributed-systems-architectures/load-balancer.svg" alt="">
  <figcaption><h4>Figure: Load balancer</h4></figcaption>
</figure>

A load balancer can be implemented in hardware (expensive) or as software running on a commodity machine (cheaper).

Load balancers periodically perform health checks to determine which nodes in a cluster are available. They distribute requests between the healthy nodes (commonly requests are distributed round-robin style) {% cite salchow2007load -l 8 %}.

The two main types of load balancer are application load balancers and network load balancers.

An **application load balancer** (also known as a layer 7 load balancer) can use application layer information to determine how to route requests. For example, they can perform HTTP path-based routing.

A **network load balancer** (also known as a layer 4 load balancer) performs NAT to forward transport packets using the IP address and port number to make load balancing decisions.

To avoid a load balancer becoming a single point of failure you can have two servers running load balancers in an active-passive configuration using a single virtual IP address. If the active server goes down, the passive server will take over and start receiving traffic for the virtual IP.

### Reverse proxies

A **reverse proxy** is a server that receives incoming requests and forwards them to the relevant server.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/distributed-systems-architectures/reverse-proxy.svg" alt="">
  <figcaption><h4>Figure: Reverse proxy</h4></figcaption>
</figure>

The benefits of using a reverse proxy include:

1. Load balancing
2. Controlling incoming traffic to a private network
3. Caching
4. SSL encryption/decryption

An **API gateway** is a special type of reverse proxy that takes API calls from a client and routes them to the correct backend service. API gateways often invoke multiple backend services and then aggregate the result.

## References

{% bibliography --cited_in_order %}
