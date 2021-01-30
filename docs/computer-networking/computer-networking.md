---
layout: default
title: Computer networking
description: An introduction to computer networking including the hardware and basic principles.
has_children: true
has_toc: false
nav_order: 2
permalink: /computer-networking
---

<!-- prettier-ignore-start -->

# Computer networking
{:.no_toc}

Computer networking is the process of creating systems of interconnected computers. Most of these notes focus on the Internet, but there are other interesting networks that will be covered.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Hardware

Network hardware includes the host computers (**hosts**), switching devices, cables, modems, and other devices used to transmit information between hosts.

Switching devices acting at the Local Area Network (LAN) layer are called switches, while devices acting at the IP layer are called routers {% cite intro-computer-networks -l 17 %}.

Each switch/ router is responsible for the next destination of packets (the **next hop**). The ultimate destination of a packet is a host computer. You can conceptualize network nodes as either a host or a router (although some can be both) {% cite intro-computer-networks -l 27 %}.

Network nodes must be connected by a shared link, which could be a physical cable, or a wireless link. There are two main type of transmission links: broadcast links and point-to-point links.

**Point-to-point** links connect one machine to another. **Broadcast links** connect multiple machines together. A wireless network is an example of a broadcast link. Ethernet is now (mostly) an example of a point-to-point link {% cite computer-networks -l 17 %}.

On a broadcast network, packets sent by one machine are received by all others. Each machine receives every packet. If the packet is addressed to the receiving machine, the machine will process the packet {% cite computer-networks %}.

In order to send and receive data across links, the nodes must agree on how to communicate with each other. This is done by defining and following **protocols**.

## Protocols

A protocol is "an agreement between the communicating parties on how communication is to proceed" {% cite computer-networks -l 29 %}.

Computer networking is made up of multiple protocols. There are a number of organizations that standardize protocols, like IEEE (the Institute of Electrical and Electronics Engineers) and IETF (the Internet Engineering Task Force).

### Protocol hierarchies

Most networks are organized as a stack of layers that build upon each other. The number of layers and the protocols they follow differ for each network.

Each layer passes data and control information to the layer below it until the lowest layer is reached. Layer 1 is the physical layer where the data is actually transmitted.

A layer has an interface for other layers to communicate with it. These interfaces make it easy to switch out layers, as long as the replacement defines the same interface. For example, you can switch out telephone lines for satellite channels.

Entities of corresponding layers on a different machine are called **peers**. Peer processes think of communication as horizontal. They expect messages in a certain format and act as if they are communicating with each other directly. Interfaces are a way for each layer to interact with the layers above and below it {% cite computer-networks %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/protocol-peers.svg" alt="">
  <figcaption><h4>Figure: Protocol peers</h4></figcaption>
</figure>

A **network architecture** is a set of layers and protocols, and a protocol stack is the list of all protocols used by a system {% cite computer-networks -l 31 %}.

### The OSI model

The OSI model is a conceptual protocol layer model. The OSI model has 7 layers, each responsible for different parts of networking.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/osi-model.svg" alt="">
  <figcaption><h4>Figure: The OSI model</h4></figcaption>
</figure>

The **physical layer** is the lowest layer. This is the layer that transmits bits over a shared medium, like optical fiber.

The **data link layer** is responsible for reconstructing transmitted data into a line that appears free of transmission errors.

The **network layer** controls the routing of packets from one source to another. Routes can be based on static tables, or they can be built dynamically through other protocols.

The **transport layer** is responsible for chunking data it receives from the layer above into pieces to send over the network layer. It then reassembles the chunks in order at the receiving end.

The **session layer** can be used to establish sessions between machines.

The **presentation layer** ensures that the syntax and semantics of the data that’s being transmitted will be understood by other machines.

The **application layer** can contain a variety of protocols that are used by an application. HTTP is a popular application-layer protocol.

{% cite computer-networks -l 43-5 %}

## Connection-oriented vs connectionless service

A connection-oriented service is modeled after the telephone. Two nodes create a connection to each other over a network, use the connection to send information to each other, and close the connection at the end of the transaction. Another word for the connection is a circuit. This is from the days of the telephone network where a circuit was a path over cooper wire that carried a phone conversation {% cite computer-networks %}.

A connectionless service is modeled after the postal system. Each message contains a destination address. The message is then routed through intermediate nodes until it reaches its destination {% cite computer-networks %}.

## Packets

Most computer networks are packet-based networks. Packets are chunks of data that are sent as a single unit. This is opposed to telephone networks that are circuit switched. Packet-based networks were conceived in 1962 in an attempt to avoid single point of failure suffered by centrally switched protocols {% cite intro-computer-networks -l 14-5 %}.

One benefit of packet-switched networks is that you can send different packet types over a single transmission line. It’s important to keep packet sizes small to avoid congestion. A large packet will stop other packets from using the transmission line while it's transmitted. Large packets are also unstable because if any of the packet is corrupted, the entire packet must be retransmitted {% cite intro-computer-networks -l 21 %}.

Packets generally contain a header section and a data section. The header section contains delivery metadata for the packet, like the destination address, and the data section contains the packet data {% cite intro-computer-networks -l 14 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/packet.svg" alt="">
  <figcaption><h4>Figure: Packet</h4></figcaption>
</figure>

In the datagram-forwarding model of packet delivery, switches generally read the entire packet before they look at the packet header to decide where it should be sent next. This is known as store-and-forward, and it leads to a forwarding delay equal to the time it takes to read the packet {% cite intro-computer-networks -l 21 %}.

### Routing

In the datagram-forwarding model, each switch has a forwarding table that it uses to determine where to direct each packet it receives. The forwarding table is made up of \<destination, next_hop\> pairs {% cite intro-computer-networks -l 15 %}.

The way that entries are added to a forwarding table depends on the protocol. In the case of IP there are several protocols just for building the routing table {% cite intro-computer-networks -l 16 %}.

Not all switches connected to the Internet have complete tables. IP routes in small networks often have a default destination leading to another network that's connected to the Internet. This is used in case the local network doesn’t have a destination match in its routing table {% cite intro-computer-networks -l 16-17 %}.

Routing loops occur when switches continually send packets between one another in a loop so that the packet never reaches its destination. Data-forwarding protocols should have prevention built into them. For example, Ethernet disallows loops in the network topology and does not allow switches to forward a packet back to the switch that originally delivered it. IP uses a Time to Live (TTL) field that is decremented by 1 on each router hop. Once the field reaches 0, the packet is dropped {% cite intro-computer-networks -l 19-20 %}

Networks can suffer from congestion, where traffic overwhelms a switch. This can be managed with queues in the switches, but once a queue is full any future received packets are dropped by the switch. Most packet losses on the Internet are from congestion {% cite intro-computer-networks -l 20 %}.

### Packet delays

Delays in the network are inevitable. There are many different types of delay:

**Bandwidth delay**: The amount of time it takes to transmit over a link. For example, sending 16Kb at 160Kb/s will take 0.1 seconds.

**Propagation delay**: The limit imposed by the speed of light. Photons travel at the speed of light in a vacuum, but they travel slower through other material. Sending data over a 5000km cable that can only travel ~⅔ the speed of light (200km/ms) will take 25ms.

**Store-and-forward delay**: The amount of time taken to read the entire packet before deciding where to forward.

**Queuing delay**: The time spent in a router queue waiting for other packets to be processed.

{% cite intro-computer-networks -l 21 %}

## Data Rate, throughput and bandwidth

**Data rate** is the number of bits sent during a period of time.

**Throughput** is the effective transmission rate, when taking into consideration transmission overhead and protocol inefficiencies.

**Bandwidth** can refer to either data rate or throughput, although in these notes it generally refers to data rate.

Data rates are usually measured in bits per second. For example, kilobits per second (Kb/s) and megabits per second (Mb/s).

## LANs and Ethernet

A local-area network (LAN) is a system made from:

- Physical links (like fiber optics or coaxial cable)
- Interfacing hardware
- Protocols

Ethernet is the most popular type of LAN, although Wireless LANs (WLANs) are becoming increasingly popular {% cite intro-computer-networks -l 22-23 %}.

Ethernet addresses are six bytes long. Each NIC (Network Interface Card) has a unique address added by the manufacturer to ROM on the card (although the address can be spoofed in software). This is different to IP addresses, which are assigned administratively {% cite intro-computer-networks -l 23 %}.

The network interface monitors all incoming traffic, and if it sees a packet addressed to itself it will read the packet and send the packet to the CPU via an interrupt {% cite intro-computer-networks -l 23 %}.

Ethernet also has a broadcast address. Sending a packet to a broadcast address will send the packet to every node on the network. When a switch receives a packet on the broadcast address it resends the packet to every port. This is used during the Address Resolution Protocol (ARP) which is a protocol for converting IP addresses into Ethernet addresses {% cite intro-computer-networks -l 23 %}.

Ethernet switches must maintain a forwarding table in order to know where to send addresses to. The table is built dynamically using a passive learning algorithm. A host is entered in the forwarding table when a message is first received from an unrecognized host. This is in contrast to IP forwarding tables, which are learned actively {% cite intro-computer-networks -l 23-4 %}.

## WANs

A wide area network (WAN) covers a large geographical area, like a country or continent.

The Internet is an example of a WAN. Another example of a WAN is the cellular telephone network.

## References

{% bibliography --cited_in_order %}
