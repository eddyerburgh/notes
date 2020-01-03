---
layout: default
title: OSPF
description: Notes on the OSPF protocol.
nav_order: 9
parent: Internet
grand_parent: Computer networking
permalink: /computer-networking/internet/ospf
---

<!-- prettier-ignore-start -->

# OSPF
{:.no_toc}

OSPF is a protocol for sharing IP route information within an autonomous system (a network managed by a single organization).

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

OSPF converts networks, routers, and links into a directed graph. Each edge is assigned a weight by a network administrator. A point-to-point connection between two routers is represented by a pair of edges, one in each direction {% cite computer-networking-top-down -l 388 %}.

If multiple paths are equally short, OSPF splits traffic across the set. This is called **ECMP** (Equal Cost MultiPath).

## Areas

In OSPF, an AS can be broken into numbered areas to help improve scalability. Areas can't overlap, but they don't need to include every router {% cite computer-networks -l 476 %}.

Routers that exist entirely inside an area are known as **internal routers**. To routers outside an area, an area's destinations are visible, but not its topology {% cite computer-networks -l 476 %}.

Every AS has an a **backbone area** (area 0). Routers in the backbone area are called **backbone routers**. All areas are connected to the backbone (possibly by tunnels), so you can get from any area in an AS to any other via the backbone {% cite computer-networks -l 476 %}.

A router connected to two or more areas is called an area border router. An **area border router** summarizes the destinations in one area, and injects the summary to the other areas. The summary contains cost information, but not details of the topology. This means less computation for other routers {% cite computer-networks -l 476-7 %}.

An AS boundary router injects routes to external ASes into the area. External routes appear as "destinations that can be reached via the AS boundary router with some cost" {% cite computer-networks -l 477 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/ospf/areas.svg" alt="">
  <figcaption><h4>Figure: The relationship between ASes, backbones, and areas {% cite computer-networks -l 477 %}</h4></figcaption>
</figure>

## The protocol

During normal operation, each router runs the same shortest path algorithm to calculate the shortest path from itself to every other router and network in the AS.

For a source and destination in the same area, OSPF chooses the best intra-area route. For a source and destination in different areas, the route goes from the source to the backbone, and then to the destination. This is a star topology {% cite computer-networks -l 477 %}.

"When a router boots, it sends HELLO messages on all of its point-to-point lines and multicasts them on LANs to the group consisting of all the other routers". This is how the router learns its neighbors {% cite computer-networks -l 478 %}.

"OSPF works by exchanging information between adjacent routers, which is not the same as between neighboring routers". It's inefficient to have every node on a LAN communicating with every other route {% cite computer-networks -l 478 %}.

One router is elected as the designated router. The **designated router** is said to be adjacent to all other routers on the LAN, and it exchanges information with each of them. Neighboring routers that aren't adjacent don't share routing information between them. A backup designated router is kept in case of failure {% cite computer-networks -l 478 %}.

During normal operation, each router floods LINK STATE UPDATE messages to its adjacent routers. The messages are acknowledged to ensure reliability. Messages have a sequence number so routers can determine the most up-to-date message {% cite computer-networks -l 478 %}.

DATABASE DESCRIPTION messages include the sequence number of the link state entries held by the sender. The receiver can then determine who has the most recent values by comparing its own values with the values of the sender {% cite computer-networks -l 478 %}.

Partner routers can request updates from each other with the LINK STATE REQUEST messages {% cite computer-networks -l 478 %}.

Using flooding, each router informs the other routers in its area of its links to other routers and networks, and the cost of these links. Each router uses this information to construct a path for its area and computes the shortest path {% cite computer-networks -l 479 %}.

Backbone area routers accept information from border routers in order to compute the best route from each backbone router to every other router. This information is sent to area border routers, which advertise the routes within their areas {% cite computer-networks -l 479 %}.

## References

{% bibliography --cited_in_order %}
