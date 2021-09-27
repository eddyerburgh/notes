---
layout: default
title: IPv6
description: Notes on the IPv6 protocol.
nav_order: 2
parent: Internet
grand_parent: Computer networking
permalink: /computer-networking/internet/IPv6
---

<!-- prettier-ignore-start -->

# IPv6
{:.no_toc}

IPv6 is the next version of IP after IPv4. It solves the problem of address scarcity that IPv4 suffers from.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

There were several design goals for IPv6:

- Support billions of hosts.
- Reduce the size of routing tables.
- Speed up packet processing.
- Improve security.
- Support different types of service, e.g. VoIP.
- Enable host roaming without changing IP address.
- Enable protocol to evolve in future.
- Permit both protocols to coexist for long period of time.

{% cite computer-networks -l 456 %}

Despite IPv6 being standardized in 1998, the Internet still mostly uses version IPv4.

## The IPv6 header

The IPv6 main header is simple. IPv6 uses an extensions header system to add extra options, which will be discussed later in this section.

The main header:

```
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version| Traffic Class |           Flow Label                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Payload Length        |  Next Header  |   Hop Limit   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                         Source Address                        +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                      Destination Address                      +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

{% cite rfc2460 -l 4 %}

The _Version_ field is always `6`. Routers use this field to determine whether to process packets as IPv4 packets or IPv6 packets.

The _Traffic Class_ field describes the class of service, e.g. VoIP.

The _Flow Label_ field is used to mark groups of packets as having the same requirements.

The _Payload Length_ field specifies the number of bytes that follow the 40 byte header.

The _Next Header_ field can specify whether another header follows the current one. If this header is the last IP header, _Next Header_ defines which transport protocol is contained in the IP payload.

The _Hop Limit_ field is the TTL field renamed. It limits the number of hops a packet can take before it's dropped.

The _Source Address_ and _Destination Address_ fields specify the IPv6 address of the source and destination respectively.

{% cite computer-networks -l 458-9 %}

Some fields included in the IPv4 header were removed from the IPv6 header:

- _Protocol_ (_Next Header_ is used instead).
- Fragmentation fields (IPv6 takes different approach to fragmentation).
- The _Checksum_ field (calculating it reduces performance, and other protocols perform their own checksums anyway).

{% cite computer-networks -l 460-1 %}

### Extension headers

Extension headers are an approach for adding extra options to an IPv6 header.

There are six kinds of extension headers:

| Extension header           | Description                                 |
| -------------------------- | ------------------------------------------- |
| Hop-by-hop options         | Miscellaneous information for routers.      |
| Destination options        | Additional information for the destination. |
| Routing                    | Loose list of routers to visit.             |
| Fragmentation              | Management of datagram fragments.           |
| Authentication             | Verification of the sender’s identity.      |
| Encrypted security payload | Information about the encrypted contents.   |

{% cite computer-networks -l 461 %}

Each header is optional, but if one is present it must appear directly after the fixed header {% cite computer-networks -l 461 %}.

Some headers are a fixed length, others contain a variable number of variable-length options.

For fields that contain variable number of items, each item is encoded as a \<Type, Length, Value\> tuple. _Type_ is a 1-byte field that specifies the type of option, _Length_ is a 1-byte field that tells how long the value is (up to 255 bytes). _Value_ can be any information required {% cite computer-networks -l 461 %}.

Each extension header includes a _Next Header_ field which works the same way as the Next header field in the IP main header, and is used to specify which extension header (if any) comes next.

## IPv6 addresses

IPv6 addresses are 128-bits, which gives $$3 \times 10^{38}$$ possible addresses.

The notation for IPv6 address is eight groups of four hexadecimal digits separated by colons:

```
8000:0000:0000:0000:0123:4567:89AB:CDEF
```

Leading zeroes in a group can be omitted, and one or more groups of zeros can be replaced by two colons:

```
8000::123:4567:89AB:CDEF
```

{% cite computer-networks -l 460 %}

## References

{% bibliography --cited_in_order %}
