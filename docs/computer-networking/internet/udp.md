---
layout: default
title: UDP
description: Notes on the UDP protocol.
nav_order: 2
parent: Internet
grand_parent: Computer networking
permalink: /computer-networking/internet/udp
---

<!-- prettier-ignore-start -->

# UDP
{:.no_toc}

UDP (User Datagram Protocol) is a connectionless transport protocol. "UDP provides a way for applications to send
encapsulated IP datagrams without having to establish a connection" {% cite computer-networks -l 541 %}.

UDP does not provide protection against lost packets. It's the responsibility of application-layer protocols, like QUIC and DNS, to manage packet retransmission.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## The UDP protocol

UDP sends segments containing an 8-byte header, followed by the payload data.

```

                  0      7 8     15 16    23 24    31
                 +--------+--------+--------+--------+
                 |     Source      |   Destination   |
                 |      Port       |      Port       |
                 +--------+--------+--------+--------+
                 |                 |                 |
                 |     Length      |    Checksum     |
                 +--------+--------+--------+--------+
                 |
                 |          data octets ...
                 +---------------- ...
```

{% cite rfc768 -l 1 %}

The _Source Port_ and _Destination Port_ fields specify the port of the sending and receiving processes respectively.

The _Length_ field is the total length of the datagram (in octets), including both the 8-byte header and the data.

The _Checksum_ field "is the 16-bit one's complement of the one's complement sum of a pseudo header of information from the IP header, the UDP header, and the data, padded with zero octets at the end (if necessary) to make a multiple of two octets" {% cite rfc768 -l 2 %}.

## Uses of UDP

The main value of UDP over raw IP is the addition of source and destination ports.

UDP is a simple protocol that can be used as a building block for other protocols, like QUIC (soon to become the HTTP3 protocol).

UDP is also used for most DNS queries.

## References

{% bibliography --cited_in_order %}
