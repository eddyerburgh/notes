---
layout: default
title: IPv4
description: Notes on the IPv4 protocol.
nav_order: 1
parent: Internet
grand_parent: Computer networking
permalink: /computer-networking/internet/ipv4
---

<!-- prettier-ignore-start -->

# IPv4
{:.no_toc}

IP (the Internet Protocol) is a routing and addressing protocol. When it was first proposed, IP was intended as a way to connect multiple LANs, but now you can imagine IP as a global virtual LAN {% cite intro-computer-networks -l 185 %}.

Although IP version 6 was formalized in 1998, the Internet still mostly uses version 4 of the protocol (IPv4).

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## IP addresses

Public IP addresses are administratively assigned. Originally they were assigned by IANA, but IANA now delegates the task to other organizations {% cite intro-computer-networks -l 24 %}.

IP addresses are 32 bits (4 bytes) long. Commonly, IPv4 addresses are represented in **dotted decimal notation**, where each byte is written in decimal, separated by a period. For example, 172.16.254.1.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/ipv4/ipv4-address.svg" alt="">
  <figcaption><h4>Figure: IPv4 address</h4></figcaption>
</figure>

### Prefixes

IP addresses are hierarchical. They are split into two parts: a network part and a host part. The network part has the same value for all hosts on a LAN {% cite computer-networks -l 443 %}.

This addressing strategy is known as **CIDR** (Classless Interdomain Routing), pronounced _cider_.

The length of the network part determines how many host addresses are available on a network. For example, if the network part is 16 bits, then 16 bits are free for the host part, which means a maximum of 65534 addressed hosts on the network ($$2^{16} - 2 \text{ reserved IP addresses}$$).

Prefixes are written in the form A.B.C.D/P, where P is the number of bits used for the network part. For example, an IP address with a 16-bit network part would be denoted as 172.16.0.0/16.

A network part can't be inferred from the IP address, so routing protocols must provide the length of the network part when sharing route information {% cite computer-networks -l 443 %}.

The length of the network part can be used to create a **subnet mask**, which produces the network part when logically ANDed with an IP address.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/ipv4/ipv4-subnet-mask.svg" alt="">
  <figcaption><h4>Figure: Calculating network part using subnet mask</h4></figcaption>
</figure>

The advantage of splitting IP addresses into parts is that routers can forward packets based solely on their network part. The host part is only used when a packet has arrived in the network specified with the network part {% cite intro-computer-networks -l 193 %}.

### Subnets

Subnets enable a network to be split into multiple smaller networks, rather than requiring networks to be entirely LAN switched {% cite intro-computer-networks -l 195 %}.

Subnets work by assigning a subnet mask to each of the internal networks. In order to route packets internally, a main router must know the subnet mask for each of its subnets. The main router can determine which network to forward a packet to by bitwise ANDing the destination IP address with each of the subnet masks in turn, until a match is found.

## The IPv4 Header

The IPv4 header contains the following information:

- Destination and source address.
- Indication of IPv4 (vs IPv6).
- A Time To Live (TTL) value, to prevent routing loops.
- A field indicating what comes next in the packet (e.g. TCP or UDP).
- "Fields supporting fragmentation and reassembly".

{% cite intro-computer-networks -l 186 %}

The header format is as follows:

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version|  IHL  | DS Field  | * |          Total Length         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Identification        |Flags|      Fragment Offset    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Time to Live |    Protocol   |         Header Checksum       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

* ECN
```

{% cite intro-computer-networks -l 186 %}

_Version_ is for the IPv4 version (`0100`).

_IHL_ represents the total length of the header length in 32-bit words. Since IHL is 4 bits, the max header size is 15 32-bit words {% cite intro-computer-networks -l 186 %}.

The _DS (Differentiated services) Field_ is used to specify preferential handling for certain packets, e.g. those involved in VoIP.

_Total Length_ is the length of the datagram in octets.

_Time to Live_ is used to stop routing loops. It's decremented by 1 at each router. If it reaches `0`, the packet is dropped.

_Protocol_ indicates the protocol used in the packet's data. Common values are `6` (TCP), and `17` (UDP).

_Header Checksum_ is used to verify that the header wasn't corrupted during transmission.

The _Source Address_ and _Destination Address_ fields contain the IPv4 addresses of the sender and the recipient.

## Interfaces

IP addresses are given to interfaces, rather than nodes or hosts {% cite intro-computer-networks -l 188 %}.

One example is the loopback interface. For most machines, localhost resolves to the IPv4 loopback address 127.0.0.1. "Delivering packets to the loopback interface is simply a form of interprocess communication" {% cite intro-computer-networks -l 188 %}.

There are also other special interfaces. For example, when VPN connections are created, "each end of the logical connection
typically terminates at a virtual interface" {% cite intro-computer-networks -l 188 %}.

"When a computer hosts a virtual machine, there is almost always a virtual network to connect the host and virtual systems. The host will have a virtual interface to connect to the virtual network. The host may act as a NAT router for the virtual machine, “hiding” that virtual machine behind its own IP address, or it may act as an Ethernet switch, in which case the virtual machine will need an additional public IP address" {% cite intro-computer-networks -l 188 %}.

"Routers always have at least two interfaces on two separate IP networks". Normally the router would have a separate IP address for each interface, although some point-to-point interfaces can be used without IP addresses {% cite intro-computer-networks -l 189 %}.

### Multihomed hosts

A multihomed host is a non-router host with multiple non-loopback network interfaces. For example, many laptops have an Ethernet interface and a Wi-Fi interface. These interfaces can be used simultaneously if they both have a different IP address {% cite intro-computer-networks -l 189 %}.

It's also possible to assign multiple different IP addresses to a single interface. Sometimes this is done to enable two different IP networks to share the same LAN {% cite intro-computer-networks -l 189 %}.

## Special Addresses

IPv4 has a few assigned special addresses.

**Loopback addresses**. The default loopback address is 127.0.0.1, however, any IPv4 address beginning with 127 can serve as a loopback address {% cite intro-computer-networks -l 190 %}.

**Private addresses**. Private addresses are IP addressses that are intended for internal use only. There are three standard private-address blocks:

- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

{% cite intro-computer-networks -l 190 %}

**Broadcast addresses** are IPv4 addresses intended to be used with LAN broadcasting. The common forms are 255.255.255.255 to broadcast to the network the device is on. Historically 0.0.0.0 was also used as a broadcast address. You can also broadcast to a different network by filling the host part of an IP address with all 1-bits. This is why all host ranges have $$2^n - 2$$, where $$n$$ is the number of host bits {% cite intro-computer-networks -l 190 %}.

**Multicast Addresses**: Multicasting means sending packets to a specified set of addresses. Multicast addresses have the first byte beginning 1110 {% cite intro-computer-networks -l 190 %}.

## Fragmentation

IPv4 supports fragmentation to break up large packets into smaller chunks. This means large packets can be sent over networks that cannot support the full size of the packet. The fragments are reassembled once they have been received by the destination host {% cite intro-computer-networks -l 191 %}.

IP follows a path fragmentation and reassembly process where reassembly is done at the far end of the path, rather than by intermediate routers {% cite intro-computer-networks -l 191 %}.

The _Identification_ field in the IP header is used to group fragmented IP packets. Its value should be different for each packet. Fragments of a packet keep the same _Identification_ value as their original packet, so it's possible to identify fragments of a packet by comparing their _Identification_ value {% cite intro-computer-networks -l 191 %}.

The _Fragment Offset_ field marks the start position of the data portion of a fragment within the data portion of the original packet. This is used to reassemble the packet {% cite intro-computer-networks -l 191 %}.

TCP normally uses **Path MTU Discovery** to discover the maximum transmission size that is supported over the network. It will then keep packets under this size in order to avoid IP fragmentation. However, it's not uncommon for fragmentation to occur over UDP, as in the NFS protocol {% cite intro-computer-networks -l 192 %}.

It's worth noting that IPv6 doesn't support fragmentation {% cite intro-computer-networks -l 185 %}.

## NAT

NAT (Network Address Translation) is an approach to use a single IP address for a network of IP-connected devices.

Instead of assigning an IP address to each host in an internal network, a public IP address is assigned only to a gateway router. The gateway router, known as a NAT router, connects the internal network to the Internet.

All hosts in the internal network are assigned private IP addresses. When an internal host makes a request, the NAT router will translate the source private IP address into its own public IP address, and keep the translation in a special NAT forwarding table. When the NAT gateway receives a response from the remote machine, it will check its NAT forwarding table, see that the request is for the internal host, replace the destination IP address with the private source IP address, and forward the packet to the internal host {% cite intro-computer-networks -l 200-1 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/ipv4/nat.svg" alt="">
  <figcaption><h4>Figure: NAT router {% cite intro-computer-networks -l 201 %} </h4></figcaption>
</figure>

The NAT forwarding table includes port numbers, so that it can distinguish between two different internal hosts attempting to connect to the same external host. If two internal hosts attempt to reach the same host from the same port, then the NAT router will need to rewrite one of the source port numbers to be able to distinguish between packets destined for each router.

## References

{% bibliography --cited_in_order %}
