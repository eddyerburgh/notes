---
layout: default
title: STUN
description: Notes on the STUN protocol.
nav_order: 10
parent: Internet
grand_parent: Computer networking
permalink: /computer-networking/internet/stun
---

<!-- prettier-ignore-start -->

# STUN
{:.no_toc}

STUN is a protocol used to discover NATs and firewalls between a host and the Internet. It's used by NAT traversal protocols like ICE.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

STUN (Session Traversal Utilities for NAT) is a protocol implementing features used for NAT traversal. It's designed to be used by higher-level protocols, like ICE {% cite rfc5389 -l 1, 4 %}.

STUN is a client-server protocol and a STUN implementation is known as a STUN Agent {% cite rfc5389 -l 6 %}.

The main STUN method is `Binding`, which is used to determine the particular "binding" a NAT has allocated to a STUN client {% cite rfc5389 -l 7 %}.

In a `Binding` transaction, a request is sent from a client STUN Agent to a server STUN Agent. As the request passes through NATs along its route the NATs will modify the source IP address and port number. When the request reaches the STUN server, the request's source will be the public IP address and port created by the NAT closest to the STUN server agent. This is known as the reflexive transport address {% cite rfc5389 -l 7 %}.

The STUN server then copies the reflexive transport address into an XOR-MAPPED-ADDRESS attribute in the STUN Binding response and sends a response to the client {% cite rfc5389 -l 7 %}.

## STUN format

Each STUN message contains a STUN header and optional message attributes.

### STUN header

```
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|0 0|     STUN Message Type     |         Message Length        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         Magic Cookie                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                     Transaction ID (96 bits)                  |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

{% cite rfc5389 -l 10 %}

The _STUN Message Type_ field defines the message class ($$ \text{message_class} = \{ \text{request}, \text{success}, \text{response}, \text{failure}, \text{response} \} $$), and the message method {% cite rfc5389 -l 10 %}.

The _Message Length_ field is the length (in bytes) of the message not including the 20-byte header {% cite rfc5389 -l 12 %}.

The _Magic Cookie_ field contains the fixed value `0x2112A442`. This was part of the _Transaction ID_ field in a now-obsolete RFC. The _Magic Cookie_ value is set so that agents can determine whether STUN clients are following RFC5389 and therefore support certain attributes {% cite rfc5389 -l 11 %}.

### STUN attributes 

A STUN message also contains zero or more Message Attributes. A Message Attribute is TLV encoded (Type-Length-Value):

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Type                  |            Length             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                             Value                             ....
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The _MAPPED-ADDRESS_ attribute contains the reflexive transport address of the client. It's maintained for legacy STUN servers following RFC3489 {% cite rfc5389 -l 32-3 %}.

The _XOR-MAPPED-ADDRESS_ attribute contains the reflexive transport address of the client obfuscated by XORing the port and address with the _Magic Cookie_ value to get _X-Port_ and _X-Address_ {% cite rfc5389 -l 34 %}.

The format is:

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|x x x x x x x x|    Family     |         X-Port                |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                X-Address (Variable)
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

_Note: See [RFC5829 section 15.2](https://tools.ietf.org/html/rfc5389#section-15.2) for the full specification for generating the XORed values_.

## References

{% bibliography --cited_in_order %}
