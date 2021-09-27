---
layout: default
title: Communication
description: Notes on communication in distributed systems.
has_toc: false
nav_order: 2
parent: Distributed systems
permalink: /distributed-systems/communication
---

<!-- prettier-ignore-start -->

# Communication
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

Processes in a distributed system must communicate with each other in order to work together. There are several different types of communication that processes can use.

With **persistent communication** a message is stored by the communication middleware for as long as it takes to deliver the message to the receiver {% cite distributed-systems -l 172 %}.

With **transient communication** a message is stored by the communication middleware for only as long as the sending and receiving applications are executing {% cite distributed-systems -l 172 %}.

With **asynchronous communication** a sender continues immediately after it has submitted its message for transmission {% cite distributed-systems -l 173 %}.

With **synchronous communication** a sender is blocked until it receives a confirmation from the receiver {% cite distributed-systems -l 173 %}.

## RPCs

RPCs (Remote Procedure Calls) are a way for processes to call procedures located in a different address space (normally on a different machine). RPCs are a form of transient communication that mask the fact that you might be communicating over a network {% cite distributed-systems -l 173 %}.

Consider a call from machine A to add data to a list on machine B:

```
newlist = append(data, db_list)
```

`append()` would be implemented by a client stub on machine A. The client stub would marshall the parameters into messages that can be sent to machine B. The client stub would then make a request to machine B and block itself until it receives a response {% cite distributed-systems -l 175-6 %}.

At machine B, the message would be passed by the OS to a server stub that would unmarshall the parameters and then call the server procedure. The server stub would then marshall the result and send it back to Machine A {% cite distributed-systems -l 176 %}.

When the response is received by machine A, the client stub would become unblocked, unmarshall the result and return it to the caller {% cite distributed-systems -l 176 %}.

One problem with RPCs is passing pointers between machines, since pointers only make sense in the address space in which they are used. The solution is to copy the entire data structure and send it to the remote machine, replacing copy-by-reference with copy-by-value {% cite distributed-systems -l 179-80 %}.

Interfaces for RPCs are often defined using an IDL (Interface Definition Language). An interface is defined in an IDL which is then compiled into a client stub and a server stub along with any compile-time (or run-time) interfaces {% cite distributed-systems -l 184 %}.

The synchronous nature of RPCs is not always appropriate. For example, if it can't be guaranteed that a server will be able to respond quickly {% cite distributed-systems -l 193 %}.

## Message-oriented communication

**Message-oriented communication** involves communicating with other processes by sending messages.

One way to send messages is over the transport layer (using the socket API) {% cite distributed-systems -l 193 %}.

### HTTP

HTTP is a popular mechanism for sending messages between two processes.

HTTP is a form of transient communication, since both the sender and the receiver need to be active in order for requests to complete successfully.

### Message queuing systems

**Message queuing systems** support asynchronous persistent communication. They store messages for an intermediate time without requiring the sender or receiver to be active {% cite distributed-systems -l 206 %}.

Message queuing systems allow applications to communicate by sending messages to a queue. Multiple applications can share a single queue {% cite distributed-systems -l 206 %}.

Message queues cannot guarantee when a message will be read. They only guarantee that the message will be sent to the recipient queue {% cite distributed-systems -l 206 %}.

Queues are managed by queue managers. A **queue manager** is a separate process, or part of a library, that is responsible for ensuring that a queue message reaches its destination {% cite distributed-systems -l 208 %}.

Addressing is done by providing a system-wide unique name for the message queue . A queue name is associated with a \<host, port\> pair, which a queue manager can use to send messages to the correct destination queue {% cite distributed-systems -l 207-8 %}.

**Message brokers** are special nodes in a message queuing system that are responsible for converting incoming messages from the messaging protocol of the sender to the messaging protocol of the destination queue manager {% cite distributed-systems -l 210 %}.

## Multicast communication

Multicast communication is when data is sent to multiple receivers at once {% cite distributed-systems -l 221 %}.

Multicast communication generally refers to sending data to a group of nodes, whereas broadcast communication involves sending data to every node in a network {% cite distributed-systems -l 225 %}.

In application-level tree-based multicasting, nodes are organized into an overlay network which is used to disseminate information to its members {% cite distributed-systems -l 221 %}.

When nodes are organized into a tree, there is a unique overlay path between each node. When nodes are organized into a mesh network, it's likely that there are multiple paths between each node. Mesh networks have better redundancy than a tree network {% cite distributed-systems -l 221-2 %}.

Flooding-based multicasting is where each node forwards a message to each of its neighbors {% cite distributed-systems -l 226 %}.

**Gossip protocols** (also known as epidemic protocols) are protocols for spreading information between nodes using only local information (i.e. there is no central control node for determining communication). One propagation model is the anti-entropy model where node $$P$$ picks a node $$Q$$ at random and then exchanges updates with $$Q$$ {% cite distributed-systems -l 229-30 %}.

## References

{% bibliography --cited_in_order %}
