---
layout: default
title: Internet
description: Notes on the architecture and protocols of the Internet.
nav_order: 2
has_children: true
has_toc: false
parent: Computer networking
permalink: /computer-networking/internet
---

<!-- prettier-ignore-start -->

# The Internet
{:.no_toc}

This section is devoted to the Internet. From the architecture of the Internet, to the protocols that define it.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Internet architecture

For a customer to join the Internet, a computer (host) must be connected to an Internet Service Provider (ISP) that the customer pays for Internet access. This lets the host exchange packets to every other accessible host on the Internet {% cite intro-computer-networks -l 62 %}.

The host can be connected to the ISP through multiple technologies. For example, DSL (Digital Subscriber Line), which reuses the telephone line connected to your home to transmit digital data. The computer is connected to a DSL modem, which converts between digital packets and analog signals that can be sent across the phone line (modem is short for modulator/demodulator). At the other end of the line, a DSLAM converts between signals and packets. There are several other ways to connect to an ISP, like over a dial-up connection {% cite intro-computer-networks -l 62 %}.

Dial-up connections were slow, at 57kbps, because they were limited to a narrow bandwidth. Internet access methods like DSL are much faster than dial-up. These faster methods are known as broadband, because they operate over a broader bandwidth {% cite intro-computer-networks -l 63 %}.

The location at which packets enter an ISPs network is the ISPs POP (point of presence). ISP networks can be regional, national, or international {% cite intro-computer-networks -l 63 %}.

Typically, ISPs connect their networks to exchange traffic at Internet Service Exchanges (IXPs). IXPs are rooms of routers, with at least one router per ISP. The routers are connected over a LAN so that ISPs can exchange traffic between their backbone networks {% cite intro-computer-networks -l 63 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/internet-architecture.svg" alt="">
  <figcaption><h4>Figure: Internet architecture</h4></figcaption>
</figure>

Connected devices communicate with each other using protocols.

## Protocols

Internet protocols are defined by the Internet Engineering Task Force (IETF), which publishes specifications of the protocols, known as Request for Comments (RFCs).

Internet protocol messages are sent in 8-bit bytes known as octets. The reason octets are specified explicitly is to avoid compatibility issues with computers that don't use byte-length words {% cite intro-computer-networks -l 15 %}.

Network byte order is specified as big endian. POSIX defines helper functions to convert values to and from network byte order, like `htonl` which converts a long integer to network byte order.

### IP - Internet Protocol

The Internet Protocol was developed to allow multiple types of LANs to connect to each other. Originally IP networks were thought of as interconnected LANs, but you can conceptualize it more as a large virtual network {% cite intro-computer-networks -l 24 %}.

"IP provides a global mechanism for addressing and routing, so that packets can actually be delivered from any host to any other host". IP version 4 (IPv4) addresses are 4 bytes long "and are part of the IP header that generally follows the Ethernet header. The Ethernet header only stays with a packet for one hop; the IP header stays with the packet for its entire journey across the Internet" {% cite intro-computer-networks -l 24 %}.

IP addresses can be divided into a network part (a prefix), and a host part. Addresses are administratively assigned. Originally the network prefixes were assigned by the Internet Assigned Numbers Authority (IANA), and a network administrator would administer the host part of the IP range.

The purpose of network prefixes is to make IP scalable. In 2013 there were one billion hosts using IP, which would be very difficult to maintain in a table. Instead, because the top-level forwarding tables only hold network prefixes, there were only 300,000 entries {% cite intro-computer-networks -l 26 %}.

Hosts with the same network prefix must be on the same LAN. IP assumes that a router for the network prefix will be able to route an IP packet to the correct host. This makes it so that most routers only need to look at the network prefix in order to route a packet through multiple connected LANs {% cite intro-computer-networks -l 25 %}.

"IP is a best effort system; there are no IP-layer acknowledgments or retransmissions. We ship the packet off, and hope it gets there. Most of the time, it does." This makes IP a connectionless network. The responsibility for maintaining a connection is left to higher-level protocols, like TCP {% cite intro-computer-networks -l 25-6 %}.

Destinations at non-backbone sites generally know all hosts on their local network and have a default entry for sending all other requests, normally to their ISP {% cite intro-computer-networks -l 28 %}.

Routers build their IP forwarding tables using route sharing protocols like RIP and BGP.

### Transport

IP gets packets from one host to another, but it can't create a connection or guarantee delivery. The transport layer is the layer above IP that _can_ create connections and guarantee delivery.

The most popular transport protocol is the Transmission Control Protocol (TCP). TCP:

- Guarantees delivery by keeping track of each packet and retransmitting any that are dropped
- Creates a connection between the two hosts
- Breaks up data into packets before sending them, and reassembles received packets
- Supports port numbers
- Manages throughput

{% cite intro-computer-networks -l 30 %}

An alternative to TCP is the User Datagram Protocol (UDP). UDP supports sending messages to a port number, but it doesn't open a connection and it doesn't guarantee delivery. {% cite intro-computer-networks -l 32 %}

## References

{% bibliography --cited_in_order %}
