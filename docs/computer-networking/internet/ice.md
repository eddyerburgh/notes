---
layout: default
title: ICE
description: Notes on the ICE protocol.
nav_order: 10
parent: Internet
grand_parent: Computer networking
permalink: /computer-networking/internet/ice
---

<!-- prettier-ignore-start -->

# ICE
{:.no_toc}

ICE is a NAT traversal protocol used by higher-level protocols like WebRTC to provide peer-to-peer communication.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

ICE enables devices (that are possibly behind NATs) to establish a peer-to-peer connection by determining a pair of transport addresses that the devices can communicate over.

_Note: A **transport address** is a combination of IP address, port number, and transport protocol_ {% cite rfc5245 -l 8 %}.

At a high-level, ICE works by determining candidate transport addresses, exchanging candidate addresses between peers, testing that the peers can connect over the candidate addresses, and then selecting a pair of addresses for sending and receiving data {% cite rfc5245 -l 6,12 %}.

Each data stream might consist of multiple components which each require a single transport address {% cite rfc5245 -l 6,17 %}.

An **ICE Agent** is an implementation of the protocol. Normally there are two ICE agents involved in exchanging candidates. In this case, one agent is designated as the Controlling Agent and the other is designated as the Controlled Agent {% cite rfc5245 -l 6,14 %}.

_Note: ICE does not solve NAT traversal for the signaling protocol used for candidate exchange. ICE assumes that the agents are able to establish a signaling connection between each other somehow, normally via a signaling server_ {% cite rfc5245 -l 7 %}. 

The stages are of an ICE session are:

1. Candidate gathering
2. Candidate exchange
3. Connectivity checks
4. Candidate selection

## Candidate gathering

An ICE Agent must first gather possible transport addresses, known as candidate addresses. 

There are several types of candidate addresses:

* **Host addresses**—addresses obtained directly from a local interface.
* **Server-Reflexive addresses**—translated addresses on the public side of a NAT obtained from a STUN or TURN server.
* **Relayed addresses**—addresses obtained from a TURN server.

{% cite rfc5245 -l 8 %}

_Note: A **symmetric NAT** is a NAT where each request from an internal IP address and port to a destination IP address and port is mapped to a unique external source IP address and port._

If TURN servers are used then both Server-Reflexive and Relayed candidate addresses are retrieved from the  server. Otherwise STUN servers are used to obtain only Server-Reflexive candidates {% cite rfc5245 -l 9 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/ice/ice-addresses.svg" alt="">
  <figcaption><h4>Figure: Different types of transport address 9</h4></figcaption>
</figure>


Once an ICE Agent has collected its candidate addresses, it sorts them in order of highest to lowest priority before sending them to the peer agent.

## Candidate exchange

**Candidate exchange** is where ICE agents exchange information like candidate transport addresses and passwords. 

The information is sent through a signaling chanel which isn't defined by ICE. SDP is a commonly used protocol for candidate exchange {% cite rfc5245 -l 10, 14 %}.

Some of the information that is exchanged includes:

* Candidate addresses
* Username fragments and passwords (used for performing message integrity checks)

{% cite rfc5389 -l 22 %} {% cite rfc5245 -l 25 %}

Once an agent has gathered its candidates and exchanged candidates with its peer, it will create an ordered Check List of candidate pairs to check for connectivity {% cite rfc5245 -l 10, 14 %}.

## Connectivity checks

Connectivity checks are run on candidate pair from the ordered Check List.

A **connectivity check** on a candidate pair involves sending a STUN request from the local candidate to the remote candidate. If the STUN transaction succeeds,the candidate pair becomes a valid pair and is added to the Valid List {% cite rfc5245 -l 11 %}.

## Candidate selection

The Controlling Agent is responsible for selecting a valid candidate pair from the Valid List {% cite rfc5245 -l 12 %}.

When a Controlling Agent has nominated a valid pair it sends a STUN request with an attribute indicating to the Controlled Agent that the pair has been nominated. When a Controlled Agent receives a STUN request with the attribute, it checks the same pair (unless the check has already been done) {% cite rfc5245 -l 12 %}. 

If both transactions succeeds, the agents set the nominated flag for the pair and will cancel any future checks for that component of the data stream. When an agent has set the nominated flag for each component of the data stream, the pairs become the selected pairs and they will become the only pairs that are associated with that stream {% cite rfc5245 -l 12 %}.

## Restart

An ICE Agent can restart ICE for an existing data stream {% cite rfc5245 -l 53 %}. 

A restart causes all previous states of the streams to be flushed and a new ICE session to begin {% cite rfc5245 -l 53-4 %}. 

## Keepalives

An ICE Agent must send keepalives for each session to keep the NAT bindings active {% cite rfc5245 -l 54 %}.

By default, agents use STUN keepalives {% cite rfc5245 -l 55 %}.

## Lite ICE

An ICE Agent can either run full ICE or Lite ICE. 

Agents running **Lite ICE** only use host candidates and don't generate connectivity checks {% cite rfc5245 -l 13 %}.

Lite ICE is intended for agents that are always connected to the public Internet and have a public IP address. 

## References

{% bibliography --cited_in_order %}
