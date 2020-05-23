---
layout: default
title: Distributed systems architectures
description: Notes on the architectures of distributed systems.
has_children: true
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

The **system architecture** of a distributed system is how the distributed system's software and hardware are organized.

You can make a distinction between the logical organization of distributed systems and the physical organization {% cite distributed-systems -l 55 %}.

## Architectural styles

The logical organization of a distributed system is known as its **software architecture** {% cite distributed-systems -l 56 %}.

There are different architectural styles that are formulated in terms of how different components are connected to each other.

A **component** is a "modular unit with well-defined required and provided **interfaces** that is replaceable within its environment". Components should be replaceable without requiring the system to be powered down {% cite distributed-systems -l 56 %}.

A **connector** is a mechanism that "mediates communication, coordination, or cooperation among components" {% cite distributed-systems -l 56 %}.

The most common distributed systems architectures are:

- Layered architectures
- Object-based and service-oriented architectures
- Resource-based architectures
- Event-based architectures

{% cite distributed-systems -l 56 %}

_Note: in the real-world, many systems don't follow one style exactly._

### Layered architectures

Layered architectures are where components are organized in layers. Network communication is an example of a layered architecture {% cite distributed-systems -l 57 %}.

In layered architectures, a component $$L_i$$ can make a downcall to a component at a lower-level (e.g. $$L_{i-1}$$). $$L_i$$ generally expects a response from the lower-level component {% cite distributed-systems -l 57 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/distributed-systems-architectures/layered-architectures.svg" alt="">
  <figcaption><h4>Figure: Making a downcall in a layered architecture {% cite distributed-systems -l 57 %}</h4></figcaption>
</figure>

**Layered communication protocols** are defined protocols for interacting between layers. A communication protocol "describes the rules that parties will follow in order to exchange information" {% cite distributed-systems -l 58 %}.

### Object-based and service-oriented architectures

Object-based architectures are where components (objects) are connected loosely through a (possibly remote) procedure call mechanism {% cite distributed-systems -l 62 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/distributed-systems-architectures/object-based-architecture.svg" alt="">
  <figcaption><h4>Figure: Object-based architecture {% cite distributed-systems -l 62 %}</h4></figcaption>
</figure>

In object-based architectures, each object has a defined interface that conceals implementation details {% cite distributed-systems -l 62 %}.

Because the interface and the object can be separated, it's possible to interface at one machine while the object exists on a different machine. This organization is commonly referred to as a distributed object {% cite distributed-systems -l 62 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/distributed-systems-architectures/distributed-object-system.svg" alt="">
  <figcaption><h4>Figure: Distributed object system 63</h4></figcaption>
</figure>

A **proxy** is an implementation of the object's interface. When a client binds to a distributed object, a proxy is loaded into the client's address space. A proxy marshals method invocations into messages, and unmarshals reply messages {% cite distributed-systems -l 62 %}.

In a distributed object system, an object exists on a server machine where it has the same interface as it does on the client machine. Incoming requests are passed to a server stub, known as a **skeleton** {% cite distributed-systems -l 62-3 %}.

For most distributed object systems, state is not distributed. Instead, state exists on individual machines. Only the object interfaces are made available on other machines (known as **remote objects**). In a general distributed object system, state might be distributed across physical machines, but the distribution would be hidden to a client {% cite distributed-systems -l 63 %}.

Objects provide good encapsulation, where a service is a self-contained interface. This makes it possible to organize systems using a service-oriented architecture (SOA) {% cite distributed-systems -l 63 %}.

In **service-oriented architectures** a system is constructed from multiple services that interact with each other. The services might not all be administered by the same organization {% cite distributed-systems -l 63 %}.

### Resource-based architectures

Resource-based architectures are architectures where systems are viewed as a collection of resources that are individually managed by components {% cite distributed-systems -l 64 %}.

An example is REST. RESTful architectures have four key characteristics:

- Resource are identified using a single naming scheme.
- All services offer the same interface with four methods.
- Messages sent to or from a service are fully self-described.
- Execution is stateless.

{% cite distributed-systems -l 65 %}

A RESTful API only offers four methods: `PUT` (or `PATCH`), `GET`, `POST`, and `DELETE`.

### Publish-subscribe architectures

Publish-subscribe architectures have a strong separation of processing and coordination {% cite distributed-systems -l 67 %}.

One way of conceptualizing a system is as a collection of autonomously executing processes. In this model, coordination is the communication between processes {% cite distributed-systems -l 67 %}.

[Cabri et al.](http://crystal.uta.edu/~kumar/cse6306/papers/13.pdf) created a taxonomy of coordination models that can be applied to distributed systems. Tanenbaum et al. modified this taxonomy to create a distinction between them across temporal and referential dimensions:

|                             | **Temporally coupled** | **Temporally decoupled** |
| --------------------------- | ---------------------- | ------------------------ |
| **Referentially coupled**   | Direct                 | Mailbox                  |
| **Referentially decoupled** | Event-based            | Shared data space        |

{% cite distributed-systems -l 67 %}

**Referentially coupled** means that a communicating process must address a target process directly. **Temporally coupled** means that a process must receive a message at the same time a message is sent {% cite distributed-systems -l 67 %}.

If a process is temporally and referentially coupled, communication is direct. An example is talking over cellphones {% cite distributed-systems -l 67 %}.

**Mailbox coordination** is where communication is referentially coupled but temporally decoupled, like in email {% cite distributed-systems -l 67 %}.

**Event-based communication** is temporally coupled, but not referentially coupled. An event will be published by a process without knowing what other processes are subscribed to the event {% cite distributed-systems -l 67 %}.

A **shared data space** is where a process communicates entirely in tuples, which are structured data records (like rows in a database). A process can put a tuple into the shared data space, and another process can later retrieve the tuple {% cite distributed-systems -l 68 %}.

In publish-subscribe systems, communication takes place by describing the event that took place. This makes naming of events an important consideration {% cite distributed-systems -l 70 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/distributed-systems-architectures/event-based-architecture.svg" alt="">
  <figcaption><h4>Figure: Event-based architecture 68</h4></figcaption>
</figure>

Assume that an event is described by a series of attributes. An event is published when it's available for other processes to read. A subscription must be passed to the middleware, using a name to identify the event that the subscription listens for {% cite distributed-systems -l 70 %}.

There are two common forms of publish-subscribe systems:

1. Topic-based publish-subscribe systems
2. Content-based publish-subscribe systems

In a **topic-based publish-subscribe system**, a subscriber subscribes to a named logical _channel_ (a topic). The events published to the channel typically contain <attribute, value\> pairs.

In a **content-based publish-subscribe system**, a subscription can also consist of <attribute, value\> pairs. An event is only sent to a subscriber if the event's attributes match constraints provided by the subscriber {% cite distributed-systems -l 70 %}.

If data is immediately forwarded to subscribers, the system is temporally coupled. If data must be explicitly read by subscribers, the system is temporally-decoupled. In this case, the middleware will need to store the event data {% cite distributed-systems -l 7-1 %}.

## Middleware organization

There are two important design patterns in middleware organization:

1. Wrappers
2. Interceptors

### Wrappers

A wrapper is a component that offers an interface to a client application. It solves the problem of incompatible interfaces {% cite distributed-systems -l 72 %}.

The wrapper pattern is common in object-oriented programming {% cite distributed-systems -l 72 %}.

An **object adapter** is a wrapper that provides an interface for accessing remote objects {% cite distributed-systems -l 72 %} .

### Interceptors

An interceptor is a pattern where the normal execution flow is interrupted in order to execute other code.

For example, consider an object-based distributed system. An object $$A$$ can call a method that belongs to an object $$B$$, where $$B$$ exists on a different computer than $$A$$. Remote invocation involves:

1. Object $$A$$ has same local interface as the interface offered by object $$B$$.
2. The call by $$A$$ is transformed to a generic object invocation by the middleware running on machine $$A$$.
3. The object invocation is transformed into a message that's sent over the network to $$B$$.

{% cite distributed-systems -l 73-4 %}

<!-- TODO : copy image p74 -->

A **request-level interceptor** can interrupt an invocation as the method is called on the remote interface {% cite distributed-systems -l 74 %}.

A **message-level interceptor** can interrupt an invocation just before the message is sent {% cite distributed-systems -l 74 %}.

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

Peer-to-peer systems are system architectures that support distribution across clients and servers. A client or server might be physically split into logically equivalent parts, where each part is operating on its own share of the complete data set {% cite distributed-systems -l 81 %}.

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
2. Random walks

In flooding, an issuing node, $$U$$, passes a request for an item to each of its neighbors. A request is ignored if the receiving node, $$V$$, has seen it before. If the request hasn't been seen before, $$V$$ will search locally for the item. If the item isn't available locally, $$V$$ will issue a request to each of its neighbors and then returns the response to $$U$$ if it receives one {% cite distributed-systems -l 85 %}.

Random walks involves an issuing node, $$U$$, asking one of its neighbors at random. If the neighbor cannot satisfy the request, $$U$$ asks another neighbor at random, and so on until it receives an answer {% cite distributed-systems -l 85 %}.

Locating data items can become difficult as the network grows, since there is no deterministic way to route a lookup request to a specific node. Some peer-to-peer systems use special index nodes to overcome this problem {% cite distributed-systems -l 87 %}.

An alternative is to abandon symmetric communication and use a broker node to coordinate communication between a client and a server.

Nodes that maintain an index or act as a broker are known as **super peers**. Super peers are often organized in a peer-to-peer relationship.

## Components

This section documents common components used in distributed systems.

### Load balancers

A **load balancer** is a server that distributes traffic between a cluster of backend servers. Load balancers are used to implement horizontal scaling.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/distributed-systems-architectures/load-balancer.svg" alt="">
  <figcaption><h4>Figure: Load balancer</h4></figcaption>
</figure>

A load balancer can be implemented in hardware (expensive) or as software running on a commodity machine (cheaper).

Load balancers perform health checks to determine which nodes in a cluster are available. They then distribute requests between the healthy nodes (commonly requests are distributed round-robin style) {% cite salchow2007load -l 8 %}.

The two main types of load balancer are application load balancers and network load balancers.

An **application load balancer** (also known as a layer 7 load balancer) can use application layer information to determine how to route requests. For example, they can perform HTTP path-based routing.

A **network load balancer** (also known as a layer 4 load balancer) performs NAT to forward transport packets using the IP address and port number to make load balancing decisions.

To avoid a load balancer becoming a single point of failure you can have two servers running load balancers in an active/passive configuration using a single virtual IP address. If the active server goes down, the passive server will takeover and start receiving traffic for the virtual IP.

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

An **API gateway** is a special type of reverse proxy that takes API calls from a client and routes them to the correct backend service.

## References

{% bibliography --cited_in_order %}
