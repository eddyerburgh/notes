---
layout: default
title: ARP
description: Notes on the ARP protocol.
nav_order: 5
parent: Internet
grand_parent: Computer Networking
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

ARP builds and maintains a table of \<protocol type, sender protocol address, sender hardware address\>, which is used to resolve addresses quickly.

## Address resolution

Most commonly, ARP is used to translate an IPv4 address to a MAC address.

When a host wants to translate an IPv4 address to a MAC address, the host calls the Address Resolution module. The Address Resolution module checks its table for the \<protocol type, target protocol address\> pair. If the table contains the mapping, it will return the MAC address for the IPv4 address.

If the Address Resolution module unable to find the pair, it must send an ARP request packet. In this case, the host will broadcast an ARP request packet over Ethernet {% cite rfc826 %}.

Each host on the LAN will check to see if their address matches the target IP address in the ARP request. If a host has the target protocol address, it updates its table with the details of the request sender, and sends an ARP reply to the sender {% cite rfc826 %}.

## ARP format

ARP packets are sent in Ethernet frames, with the protocol version set to ARP {% cite rfc826 %}.

An ARP packet comes in two forms: request and reply. Both share the same format:

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Hardware type         |         Protocol type         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |      HAL      |      PAL      |           Operation           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Sender hardware Address                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Sender protocol Address                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Target hardware Address                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Target protocol Address                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

In ARP, a **protocol address** is the address of the protocol that is being resolved (e.g. IPv4). The **hardware address** is the address of the link layer protocol (e.g. Ethernet).

The _Hardware type_ field specifies the hardware protocol type (Ethernet is 1).

_Protocol type_ specifies the protocol type of the protocol addresses (IPv4 is 0x0800).

_HAL_ (Hardware address length) is the length in octets of the hardware address.

_PAL_ (Protocol address length) is the length in octets of the protocol address.

_Operation_ is the operation that the sender is performing. _request_ is 1, _reply_ is 2.

_Sender hardware address_ is the hardware address of the sender. In a reply, this is the hardware address of the host that the original sender was looking for.

_Sender protocol address_ is the protocol address of the sender.

_Target hardware address_ is the hardware address of the target protocol address. This is blank when the sender sends a request.

_Target protocol address_ is the protocol address that is being resolved.

{% cite rfc826 %}

## References

{% bibliography --cited_in_order %}
