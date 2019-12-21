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

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Internet architecture

For a customer to join the Internet, a computer (host) must be connected to an ISP (Internet Service Provider) that the customer pays for Internet access. This lets the host exchange packets with every other accessible host on the Internet {% cite intro-computer-networks -l 62 %}.

The host can be connected to the ISP through multiple technologies. For example, DSL (Digital Subscriber Line), which reuses the telephone line connected to your home to transmit digital data. For DSL, the computer is connected to a DSL modem, which converts between digital packets and analog signals that can be sent across a phone line. At the other end of the line, a DSLAM converts between signals and packets {% cite intro-computer-networks -l 62 %}.

There are several other ways to connect to an ISP, an older example being dial-up Internet access. Dial-up connections are slow, at 57kb/s, because they are limited to a narrow bandwidth. Faster methods, like DSL, are known as broadband, because they operate over a broader bandwidth {% cite intro-computer-networks -l 63 %}.

The location where packets enter an ISP's network is the ISP's POP (point of presence). ISP networks can be regional, national, or international {% cite intro-computer-networks -l 63 %}.

Typically, ISPs connect their networks to exchange traffic at IXPs (Internet Service Exchanges). IXPs are rooms of routers, with at least one router per ISP. The routers are connected over a LAN so that ISPs can exchange traffic between their backbone networks {% cite intro-computer-networks -l 63 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/internet-architecture.svg" alt="">
  <figcaption><h4>Figure: Internet architecture</h4></figcaption>
</figure>

Connected devices communicate with each other using protocols.

## Protocols

Internet protocols are defined by the IETF (Internet Engineering Task Force), which publishes protocol specifications known as RFCs.

Internet protocol messages are sent in 8-bit bytes known as **octets**. The reason octets are specified explicitly is to avoid compatibility issues with computers that don't use byte-length words {% cite intro-computer-networks -l 15 %}.

Network byte order is specified as big endian. POSIX defines helper functions to convert values to and from network byte order, like `htonl` which converts a long integer to network byte order.

### IP - Internet Protocol

IP was developed to allow multiple types of LANs to connect to each other. IP was intended as a way to connect multiple LANs, but now you can imagine IP as a global virtual LAN {% cite intro-computer-networks -l 24 %}.

"IP provides a global mechanism for addressing and routing, so that packets can actually be delivered from any host to any other host". IPv4 (IP version 4) addresses are 4 bytes long and "are part of the IP header that generally follows the Ethernet header. The Ethernet header only stays with a packet for one hop; the IP header stays with the packet for its entire journey across the Internet" {% cite intro-computer-networks -l 24 %}.

IP addresses can be divided into a network part and a host part. Addresses are administratively assigned. Originally the network parts were assigned by IANA (the Internet Assigned Numbers Authority), and a network administrator would administer the host part of the IP range.

The advantage of splitting IP addresses into parts is that routers can forward packets based solely on their network part. The host part is only used when a packet has arrived in the network specified with the network part {% cite intro-computer-networks -l 193 %}.

Destinations at non-backbone sites generally know all hosts on their local network and have a default entry for sending all other requests, usually to their ISP {% cite intro-computer-networks -l 28 %}.

Routers build their IP forwarding tables using route sharing protocols like RIP and [BGP]({{ '/computer-networking/internet/bgp' | relative_url }}).

### Transport

IP gets packets from one host to another, but it can't create a connection or guarantee delivery. The transport layer is the layer above IP that _can_ create connections and guarantee delivery.

The most popular transport protocol is TCP (Transmission Control Protocol). TCP:

- Guarantees delivery by keeping track of each packet and retransmitting any that are dropped.
- Creates a connection between the two hosts.
- Breaks up data into packets before sending them, and reassembles received packets.
- Supports port numbers.
- Manages throughput.

{% cite intro-computer-networks -l 30 %}

An alternative to TCP is UDP (User Datagram Protocol). UDP supports sending messages to a port number, but it doesn't open a connection and it doesn't guarantee delivery {% cite intro-computer-networks -l 32 %}.

## References

{% bibliography --cited_in_order %}
