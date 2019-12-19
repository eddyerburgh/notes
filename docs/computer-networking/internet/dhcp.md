---
layout: default
title: DHCP
description: Notes on the DHCP protocol.
nav_order: 6
parent: Internet
grand_parent: Computer networking
permalink: /computer-networking/internet/dhcp
---

<!-- prettier-ignore-start -->

# DHCP
{:.no_toc}

DHCP (Dynamic Host Configuration Protocol) is used for dynamically configuring network information for hosts.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

DHCP has two components: a protocol for sending networking configuration values to hosts, and a mechanism for allocating network addresses to hosts {% cite rfc2131 -l 2 %}.

DHCP uses a client-server model. A DHCP server provides configuration to dynamic hosts (clients) {% cite rfc2131 -l 2 %}.

The DHCP service provides persistent storage of network parameters for clients, stored as a \<key, value\> entry for each client. The protocol messages, sent over UDP, are the interface for accessing configuration parameters from a DHCP server {% cite rfc2131 -l 11-2 %}.

## Dynamic allocation of IP addressses

DHCP has three mechanisms for IP allocation:

1. Automatic allocation
2. Dynamic allocation
3. Manual allocation

With **automatic allocation**, DHCP assigns a permanent IP address to a client. With **dynamic allocation**, DHCP assigns an IP address to a client for a specified period of time. With **manual allocation**, an IP address is assigned by an administrator, and DHCP is simply a way of sending the pre-assigned address to the client {% cite rfc2131 -l 3 %}.

A client requests the use of an address for a period of time. This agreement is known as a **lease**. The allocation system guarantees that the address won't be allocated to another host during that time. A client can extend its lease by making further requests to the server {% cite rfc2131 -l 12 %}.

### Receiving an address

A client sends a DHCPDISCOVER message on its physical subnet to receive an address {% cite rfc2131 -l 13 %}.

This can be picked up by multiple DHCP servers. A server responds with a DHCPOFFER message that includes an available network address in the _Yiaddr_ field {% cite rfc2131 -l 13 %}.

The client receives one or more offers from servers. The client chooses a server and sends a DHCPREQUEST message with the _Server Identifier_ option set to the server it has chosen {% cite rfc2131 -l 16 %}.

The servers receive the DHCPREQUEST broadcast from the client. The selected server stores the binding for the client, and replies with a DHCPACK containing the configuration parameters for the client. If the server can no longer fulfill the request from DHCPREQUEST, it returns a DHCPNAK message {% cite rfc2131 -l 16 %}.

The client receives the DHCPACK and is considered configured {% cite rfc2131 -l 17 %}.

The client can end its lease by sending a DHCPRELEASE message to the server {% cite rfc2131 -l 17 %}.

## DHCP format

DHCP extends the BOOTP protocol, a precursor to DCHP. It shares the same packet format as BOOTP, except that an unused BOOTP field has become a _Flags_ field {% cite rfc951 -l 2-3 %}.

```
   0                   1                   2                   3
   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |     Op (1)    |   Htype (1)   |   Hlen (1)    |   Hops (1)    |
   +---------------+---------------+---------------+---------------+
   |                            Xid (4)                            |
   +-------------------------------+-------------------------------+
   |           Secs (2)            |           Flags (2)           |
   +-------------------------------+-------------------------------+
   |                          Ciaddr  (4)                          |
   +---------------------------------------------------------------+
   |                          Yiaddr  (4)                          |
   +---------------------------------------------------------------+
   |                          Siaddr  (4)                          |
   +---------------------------------------------------------------+
   |                          Giaddr  (4)                          |
   +---------------------------------------------------------------+
   |                                                               |
   |                          Chaddr  (16)                         |
   |                                                               |
   |                                                               |
   +---------------------------------------------------------------+
   |                                                               |
   |                          Sname   (64)                         |
   +---------------------------------------------------------------+
   |                                                               |
   |                          File    (128)                        |
   +---------------------------------------------------------------+
   |                                                               |
   |                          Options (variable)                   |
   +---------------------------------------------------------------+
```

{% cite rfc2131 -l 9 %}

_Op_ is the message operation code, which is either `BOOTREQUEST` or `BOOTREPLY`.

_Htype (Hardware Address Type)_ is the type of hardware address (`1` is Ethernet).

_Hlen_ is the hardware address length.

_Xid_ is the transaction ID, used to identify messages and responses between a client and a server.

_Secs_ is the number of seconds since the client began the address acquisition or renewal process.

_Flags_ contains the _Broadcast_ flag bit. The _Broadcast_ bit is set by a client that hasn't yet received an IP address and can't receive unicast datagrams {% cite rfc2131 -l 25 %}.

_Ciaddr (Client IP Address)_ is only filled in if the "client is in BOUND, RENEW or REBINDING state and can respond to ARP requests" {% cite rfc2131 -l 10 %}.

_Yiaddr (Your IP Address)_ is the client's IP address, filled in by the server.

_siaddr (Server IP Address)_ is the IP address of the server that should be used in the next step of the bootstrap process {% cite rfc2131 -l 10 %}.

_Chaddr_ is the client hardware address.

_Sname_ is the server host name (optional).

_Options_ is a parameter field that includes configuration options. For example, the _Domain Name Server Field_, which contains the IP addresses of domain name servers {% cite rfc1497 -l 4 %}.

## References

{% bibliography --cited_in_order %}
