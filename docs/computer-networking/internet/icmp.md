---
layout: default
title: ICMP
description: Notes on the ICMP protocol.
nav_order: 4
parent: Internet
grand_parent: Computer networking
permalink: /computer-networking/internet/icmp
---

<!-- prettier-ignore-start -->

# ICMP
{:.no_toc}

ICMP (Internet Control Message Protocol) is used for IP routers to send messages to other routers in order to provide feedback about a network, for example to report a processing error.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

An ICMP message is sent in an IP packet, with the _Protocol_ field set to `1` (for ICMP) {% cite computer-networks -l 465 %}.

You can see a list of the main message types sent by ICMP:

| Message type                      | Description                       |
| --------------------------------- | --------------------------------- |
| Destination unreachable           | Packet could not be delivered.    |
| Time exceeded                     | Time to live field hit 0.         |
| Parameter problem                 | Invalid header field.             |
| Source quench                     | Choke packet.                     |
| Redirect                          | Teach a router about geography.   |
| Echo and echo reply               | Check if a machine is alive.      |
| Timestamp request/reply           | Same as Echo, but with timestamp. |
| Router advertisement/solicitation | Find a nearby router.             |
| Information request/reply         | Find number of network.           |

The _Destination unreachable_ message is used when the router is unable to locate the destination. It is also used when a packet that has the DF (Do not Fragment) bit set must be fragmented to pass through a network {% cite computer-networks -l 465 %}.

_Time Exceeded_ is sent when a packet is dropped after its TTL field has reached `0`. _Time Exceeded_ is used by `traceroute`, which finds the routers along a path by sending a sequence of packets with their TTL starting at `1` and incrementing until the message reaches its destination {% cite computer-networks -l 466 %}.

_Parameter problem_ is sent when an illegal value is detected in a header field {% cite computer-networks -l 466 %}.

_Echo_ and _Echo reply_ messages are sent to see if a host is currently reachable. `ping` uses these messages to check if a host is reachable {% cite computer-networks -l 466 %}.

The _Redirect_ message is used when a router detects that a packet has been routed incorrectly {% cite computer-networks -l 466 %}.

The _Router advertisement_ and _Router solicitation_ messages are used to let a host find nearby neighbors {% cite computer-networks -l 467 %}.

## ICMP format

The format of an ICMP message depends on the message type. This is always set in the first octet of the data (the _Type_ field). The _Type_ value determines the rest of the data that's included.

### Destination unreachable message

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |     Type      |     Code      |          Checksum             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                             unused                            |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |      Internet header + 64 bits of original data datagram      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

{% cite rfc792 -l 4 %}

_Type_ is `3` (for the _Destination unreachable_ type).

_Code_ can be one of:

- `0`: net unreachable.
- `1`: host unreachable.
- `2`: protocol unreachable.
- `3`: port unreachable.
- `4`: fragmentation needed and DF bit set.
- `5`: source route failed.

_Checksum_ is the one's complement of the one's complement sum of the ICMP message.

_unused_ is reserved for future extensions.

{% cite rfc792 -l 4 %}

### Timestamp or Timestamp reply message

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |     Type      |      Code     |          Checksum             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Identifier          |        Sequence Number        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |     Originate Timestamp                                       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |     Receive Timestamp                                         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |     Transmit Timestamp                                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

{% cite rfc792 -l 16 %}

_Type_ is either `13` for _Timestamp_ message, or `14` for _Timestamp reply_ message.

_Code_ is `0`.

_Checksum_ is the one's complement of the one's complement sum of the ICMP message.

_Identifier_ is an identifier to help match _Timestamp_ messages with _Timestamp reply_ messages. May be `0`.

_Sequence Number_ is an identifier to help match Timestamp messages with Timestamp replies. May be `0`.

All time values are the time in milliseconds since midnight UTC.

The _Originate Timestamp_ field is the time the sender touched the message before sending it.

The _Receive Timestamp_ field is the time the recipient first touched the message on receipt.

The _Transmit Timestamp_ field is the time the recipient last touched the message before sending it.

{% cite rfc792 -l 16 %}

You can see a full list of the headers in [the ICMP RFC](https://tools.ietf.org/html/rfc792).

## References

{% bibliography --cited_in_order %}
