---
layout: default
title: ARP
description: Notes on the ARP protocol.
nav_order: 5
parent: Internet
grand_parent: Computer networking
permalink: /computer-networking/internet/arp
---

<!-- prettier-ignore-start -->

# ARP
{:.no_toc}

ARP (Address Resolution Protocol) is a protocol for translating network addresses into link layer addresses. Most commonly it's used for mapping an IPv4 address to a MAC address.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

ARP maps a \<protocol, address\> pair to a link layer address {% cite rfc826 %}.

To resolve addresses quickly, ARP builds and maintains a **translation table** with entries containing the protocol type, sender protocol address, and sender hardware address.

## Address resolution

When a host wants to translate an IPv4 address to a MAC address, the host calls the Address Resolution module. The Address Resolution module checks its table for the \<protocol type, target protocol address\> pair. If the table contains the mapping, it will return the MAC address for the IPv4 address.

If the Address Resolution module is unable to find the pair, it must send an ARP request packet. In this case, the host will broadcast an ARP request packet over Ethernet {% cite rfc826 %}.

Each host on the LAN will check to see if their address matches the target IP address in the ARP request. If a host is assigned the target protocol address, it updates its table with the details of the request sender, and sends an ARP reply to the sender {% cite rfc826 %}.

## ARP format

ARP packets are sent in Ethernet frames, with the protocol version set to ARP {% cite rfc826 %}.

There are two types of ARP packets: request and reply. Both share the same format.

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Hardware Type         |         Protocol Type         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |      HAL      |      PAL      |           Operation           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Sender Hardware Address                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Sender Protocol Address                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Target Hardware Address                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Target Protocol Address                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

In ARP, a **protocol address** is the address of the protocol that is being resolved (e.g. IPv4). The **hardware address** is the address of the link layer protocol (e.g. Ethernet).

The _Hardware Type_ field specifies the hardware protocol type (Ethernet is `1`).

_Protocol Type_ specifies the protocol type of the protocol addresses (IPv4 is `2048`).

_HAL (Hardware Address Length)_ is the length of the hardware address in octets.

_PAL (Protocol Address Length)_ is the length of the protocol address in octets.

_Operation_ is the operation that the sender is performing. Request is `1`, reply is `2`.

_Sender Hardware Address_ is the hardware address of the sender. In a reply, this is the hardware address of the host that the original sender was looking for.

_Sender Protocol Address_ is the protocol address of the sender.

_Target Hardware Address_ is the hardware address of the target protocol address. This is blank when the sender sends a request.

_Target Protocol Address_ is the protocol address that is being resolved.

{% cite rfc826 %}

## References

{% bibliography --cited_in_order %}
