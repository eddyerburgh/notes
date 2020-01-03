---
layout: default
title: BGP
description: Notes on the BGP protocol.
nav_order: 10
parent: Internet
grand_parent: Computer networking
permalink: /computer-networking/internet/bgp
---

<!-- prettier-ignore-start -->

# BGP
{:.no_toc}

BGP (Border Gateway Protocol) is a protocol for sharing IP route information between administrative zones known as _autonomous systems_.

BGP was created mainly to help large network operators, like ISPs, apply routing policies to selectively allow and disallow traffic from other networks.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Autonomous systems

**Autonomous systems** (ASes) are collections of connected routers managed by a single organization, for example an ISP, or a large company like Amazon. The Internet is made up of ASes that route traffic internally and externally between each other.

IP routers are responsible for the next hop an IP packet takes. In order to know where to forward a request to, IP routers need to share route information with each other. ASes use IGPs (Interior Gateway Protocols), like [RIP](https://en.wikipedia.org/wiki/Routing_Information_Protocol) and [OSPF](https://en.wikipedia.org/wiki/Open_Shortest_Path_First), to share route information within their AS, and EGPs (Exterior Gateway Protocols) to share routing information with other ASes {% cite internet-routing-architecture %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/bgp/autonomous-systems.svg" alt="">
  <figcaption><h4>Figure: Autonomous systems sharing route information {% cite internet-routing-architecture %}</h4></figcaption>
</figure>

BGP is an EGP. BGP1 was specified in 1989, and BGP4 was specified in 1993. BGP4 supports CIDR and subnetting {% cite internet-routing-architecture --locator 101 %}.

_Note: Confusingly, the obsolete precursor to BGP is the Exterior Gateway Protocol (EGP), but both BGP and EGP are examples of exterior gateway protocols. In this document, EGP refers to exterior gateway protocols._

Exterior gateway protocols were developed primarily to allow ASes to set policies on their routing traffic. For example, two ASes might have a commercial agreement which means they can send traffic through each other, whereas another AS might want to disallow traffic from an AS that doesn't pay it for its services. These policies can be configured using BGP {% cite intro-computer-networks -l 301 %}.

Each AS has a number (an ASN) assigned to it by either an internet registry or a provider. Like IP, there are reserved private AS numbers {% cite internet-routing-architecture %}.

ASes can be categorized based on how they interact with other ASes.

### Single-homed ASes

Single-homed ASes connect to another AS via a single exit.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/bgp/single-homed-as.svg" alt="">
  <figcaption><h4>Figure: A single-homed AS {% cite internet-routing-architecture %}</h4></figcaption>
</figure>

Single-homed ASes can be given a number from the private range of AS numbers {% cite internet-routing-architecture -l 97%}.

### Multi-homed ASes

A multi-homed AS has at least two exit points to outside world. A non-transit AS doesn't allow traffic to go through it, whereas a transit AS does {% cite internet-routing-architecture -l 98-9 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/bgp/multi-homed-non-transit-as.svg" alt="">
  <figcaption><h4>Figure: A multi-homed non-transit AS {% cite internet-routing-architecture %}</h4></figcaption>
</figure>

## BGP4 protocol

BGP works by establishing connections to other BGP routers, known as BGP peers. **BGP peers** initially send all their routing information to each other and then send UPDATE messages periodically with any routing changes. A BGP router receives UPDATE messages and uses them to rebuild its IP routing table.

If a route becomes unreachable or a better path becomes available, BGP routers resend information.

Most of the complexity in BGP comes from the BGP decision process: the process used to decide which routes should be used when a BGP router receives multiple possible routes for a single prefix. BGP uses different metrics to determine which routes should be preferred, such as the number of ASes that a route passes through {% cite Caesar:2005:BRP %}.

### iBGP and eBGP

When BGP is used to exchange routing information between ASes, it is known as **eBGP** (external-BGP).

BGP used within an AS to exchange information is known as **iBGP** (internal-BGP).

Routers running iBGP are called transit routers, routers performing eBGP are known as border routers.

### Neighbor negotiation

When a BGP peer is started it must first connect to its neighbors. Neighbors establish a TCP connection on port 179, and then send an OPEN message containing information to create the connection, such as the BGP identifier (the senders ID) and the hold time.

The **hold time** is the maximum time that can elapse between keepalive messages before a neighbor is considered dead. BGP peers use whichever is the lower value of the two peers. A value of 0 means there's no timeout and the connection is always considered up {% cite internet-routing-architecture %}.

During neighbor session establishment, peer routers determine whether they are in the same AS by inspecting the AS Number.

### Errors

NOTIFICATION messages are sent when an error is detected. These are normally errors in the format of a received message, like `Bad Peer AS`, `Unacceptable Hold Time`, or `Malformed Attribute List`.

### UPDATE message

The UPDATE message contains routing information updates for both updated routes and withdrawn routes.

Each UPDATE message contains routes that share common BGP attributes.

An UPDATE message contains the BGP header, as well as the following content:

```
      +-----------------------------------------------------+
      |   Withdrawn Routes Length (2 octets)                |
      +-----------------------------------------------------+
      |   Withdrawn Routes (variable)                       |
      +-----------------------------------------------------+
      |   Total Path Attribute Length (2 octets)            |
      +-----------------------------------------------------+
      |   Path Attributes (variable)                        |
      +-----------------------------------------------------+
      |   Network Layer Reachability Information (variable) |
      +-----------------------------------------------------+
```

{% cite rfc4271 -l 14 %}

_Withdrawn Routes Length_ tells the receiving BGP speaker the total length of the _Withdrawn Routes_ field (in octets).

Each _Withdrawn Route_ entry is a variable length field which includes the length of the prefix in bits, and the prefix that is being withdrawn:

```
                  +---------------------------+
                  |   Length (1 octet)        |
                  +---------------------------+
                  |   Prefix (variable)       |
                  +---------------------------+
```

{% cite rfc4271 -l 14 %}

_Total Path Attribute Length_ tells the receiving BGP speaker the total length of the _Path Attributes_ field (in octets).

_Path Attributes_ contains BGP attributes, like the ORIGIN attribute that defines the origin of the route information.

The final part is the _Network Layer Reachability Information_. The total length of the field isn't included explicitly in the message, but it can be calculated using the UPDATE message length and the other length fields.

### NLRI

**NLRI** (Network layer reachability information) is an indication of networks that are advertised.

NLRI contain a _Length_ field, which contains the length of the _Prefix_ field in bits. The _Prefix_ field contains an IP prefix.

```
                  +---------------------------+
                  |   Length (1 octet)        |
                  +---------------------------+
                  |   Prefix (variable)       |
                  +---------------------------+
```

{% cite rfc4271 -l 19 %}

A BGP speaker uses the NLRI as well as path attributes sent in the UPDATE message to build its BGP table.

### Path attributes

BGP defines a number of attributes that are used to apply policies to routes to determine whether to advertise them or not.

There are four categories of path attributes:

1. Well-known mandatory.
2. Well-known discretionary.
3. Optional transitive.
4. Optional non-transitive.

{% cite rfc4271 -l 22 %}

**NEXT_HOP**

The NEXT_HOP attribute contains the IP address for next hop to follow for the route. NEXT_HOP is not always reachable directly, and it might require a recursive lookup from the BGP speaker {% cite internet-routing-architecture -l 148-9 %}.

**AS_PATH**

The AS_PATH attribute identifies the ASes that a route has gone through to reach its destination. Each time a route passes through a BGP speaker to an external peer, the speaker modifies the AS_PATH attribute to add itself to the path.

The AS_PATH is used in the BGP decision process. Some implementations add extra ASes to the AS_PATH in order to influence where traffic is directed {% cite internet-routing-architecture -l 159 %}.

BGP peers don't accept routes that contain their own AS in the AS_PATH, to avoid routing loops {% cite internet-routing-architecture -l 155 %}.

AN AS_PATH is made up of either AS_SEQUENCEs or AS_SETs.

An AS_SEQUENCE is an ordered set of ASes that have been traversed {% cite internet-routing-architecture -l 154 %}, an AS_SET is an unordered list of ASes that have been traversed {% cite internet-routing-architecture -l 158 %}.

AS_SET is intended for routers that perform route aggregation. A router that performs route aggregation considers itself the origin of that aggregate, which isn't always true. AS_SET is used to provide full information so that routes that fall within the aggregate can be routed correctly {% cite internet-routing-architecture -l 158-9 %}.

A 2011 RFC recommends not using AS_SET due to the complexity, and instead to only advertise aggregates that are less specific than routes existing in a BGP speakers IP table {% cite rfc6472 -l 2 %}.

**LOCAL_PREF**

The LOCAL_PREF attribute is an optional attribute can be used in iBGP to set a common exit point for routes. LOCAL_PREF is a numeric value, and it can be used to ensure traffic uses a high speed link {% cite internet-routing-architecture -l 161-2 %}.

**MULTI_EXIT_DISC**

MED (MULTI_EXIT_DISC) is an optional attribute that can be used to hint at the preferred path to a neighbor for entry to an AS that has multiple entries. It's passed on to neighboring ASes once but is reset to 0 before being sent to others {% cite internet-routing-architecture -l 162-3 %}.

By default BGP peers only compare the MED value for paths from external neighbors in the same AS {% cite internet-routing-architecture -l 163 %}.

**COMMUNITIES**

The COMMUNITIES attribute can be used to group logical networks together. For example, Government websites that exist across multiple ASes. BGP speakers can use this to control which routes they should accept and which routes they should pass on {% cite internet-routing-architecture -l 164 %}.

There are well known COMMUNITIES values like NO_EXPORT and NO_ADVERTISE that are commonly understood by BGP speakers {% cite internet-routing-architecture -l 165 %}.

**ORIGIN**

The ORIGIN attribute is a mandatory attribute generated by the speaker that originates the routing information. There are three possible origins:

- IGP: NLRI is interior to the originating AS.
- EGP: NLRI is learned via EGP.
- INCOMPLETE: NLRI is learned by some other means.

BGP sometimes uses the ORIGIN value in its decision process. When comparing ORIGIN, BGP chooses the path with the lowest ORIGIN value. IGP is the lowest value, EGP second lowest, and INCOMPLETE is highest {% cite internet-routing-architecture -l 167 %}.

### BGP decision process

The BGP decision process is used to determine which route to advertise if there are multiple routes for the same destination.

The rules are:

- Route is ignored if next hop is inaccessible
- Prefer route with largest LOCAL_PREF if different
- Prefer routes originated by this router if exists
- Prefer route with shortest AS_PATH if different
- Prefer route with lowest ORIGIN value if different
- Prefer route with lowest MED if different
- Prefer eBGP to confederation external to iBGP
- Prefer route that can be reached from closest IGP
- Final solution is to arbitrarily choose based on lowest router ID.

{% cite internet-routing-architecture -l 168-9 %}

### Advertising routes

One of the roles of BGP speakers is to advertise routing information from their AS to external ASes. Routing information can be added to BGP either dynamically or statically.

Dynamically injected routes can be added from the ASes IGP routes. If a network stops advertising the dynamically injected routes itself then the routes will be removed from routing tables {% cite internet-routing-architecture -l 133 %}.

Misconfigured dynamic routing could cause inefficiencies where learned routes are advertised from multiple ASes. In the Cisco BGP implementation, external OSPF routes are blocked automatically from being distributed by BGP {% cite internet-routing-architecture -l 134-5 %}.

Fluctuating routes are removed from being advertised by route dampening {% cite internet-routing-architecture -l 136 %}.

Static routing is where routes to destinations are listed manually in the server. Static routing is the most effective way to provide route stability because the route will not be removed by dampening if it fluctuates too much {% cite internet-routing-architecture -l 137 %}.

### Filtering

Filtering can be done on inbound or outbound routes to control prefixes that are received from neighbors and to control which prefixes should be advertised to neighbors. Filtered roots often have attributes manipulated to affect the decision process {% cite internet-routing-architecture -l 169-70 %}.

Routes can be filtered based on IP prefix, AS originator, or BGP attributes. Routes can either be permitted or denied {% cite internet-routing-architecture -l 171 %}.

## Policy routing

Policy routing is a way for an AS to control the traffic that passes through it based on policies added by the administrator.

BGP policies can be divided into four classes:

1. Business relationship policies
2. Traffic engineering policies
3. Scalability policies
4. Security policies

{% cite Caesar:2005:BRP %}

### Business relationship policies

Business relationships can be either **customer-peer** where a customer pays a peer AS administrator to forward its traffic, or **peer-peer** where two ISPs agree to route traffic for each other {% cite Caesar:2005:BRP %}.

Backup relationships are where ISPs have backup links between each other that are only be used in the case of link failure.

These business relationships are generally maintained by either setting LOCAL_PREF or by using the COMMUNITIES attribute to tag the route updates with the appropriate business relationship {% cite Caesar:2005:BRP %}.

### Traffic engineering policies

The goal of traffic engineering is to ensure packets are routed efficiently. For example an ISP might want to route traffic out of the earliest possible exit for an AS in order to reduce traffic inside the AS and to reduce the required hops for routing the packet (early-exit routing) {% cite Caesar:2005:BRP %}.

Another goal is to reduce outbound congestion over links, for example by load balancing. This is difficult in practice. An ISP might also want to reduce inbound traffic. One way it can achieve this is by using the MED attribute. Neighbors must agree to recognise the MED, which is usually incentivized by payment {% cite Caesar:2005:BRP %}.

### Scalability policies

Scalability policies can be put in place to make BGP routers less susceptible to misconfiguration elsewhere in the system. For example, by limiting the routing table size in order to stop a router from spending too much processing time handling updates. One approach is to filter out long prefixes, for example anything longer than /24 {% cite Caesar:2005:BRP %}.

### Security policies

Security policies help avoid issues with bad routes advertised to an AS. An ISP might discard invalid routes with import filtering. Security policies can avoid DDOS attacks like flooding a router with route updates, the BGP router could set a limit on the number of allowed route updates {% cite Caesar:2005:BRP %}.

## Network designs

The following section is on how to use BGP to design robust networks.

### Design considerations

When you design a network, you should consider redundancy and symmetry.

**Redundancy** is important in the Internet. Nodes can go down due to human error, software error, or natural disasters. Networks should be able to cope with nodes becoming unavailable.

Redundancy can be achieved by having multiple paths in case of failure, for example by having multiple entry and exit points to an AS. The key to building in redundancy is to use BGP attributes to control the advertised links {% cite internet-routing-architecture -l 203 %}.

**Symmetry** in a network means that traffic leaves and reenters a network from the same node {% cite internet-routing-architecture -l 193 %}.

Symmetry can be important for firewalls that need to see traffic both leave and return, but symmetry is difficult to achieve with multiple exit points. To achieve symmetry, the AS should be set up with a primary entry and exit router, with backup routers configured as redundancy {% cite internet-routing-architecture -l 201 %}.

### Default gateways

A default gateway is a route in the IP table that is used in case no other routes match. Default routing is an approach to build redundancy into a system without adding additional information to routing tables {% cite internet-routing-architecture -l 195-6 %}.

BGP speakers can set gateway routes dynamically by advertising a route as 0.0.0.0/0.0.0.0. In case the default route becomes unavailable, there should be multiple default routes in a network. BGP can use the LOCAL_PREF attribute to prioritize a default route {% cite internet-routing-architecture -l 196 %}.

The default can also be set statically by an administrator {% cite internet-routing-architecture -l 197 %}.

Defaults should be routers in larger ASes that are closer to large backbone routers with complete IP table {% cite internet-routing-architecture -l 198-99 %}.

### Route reflectors

Using iBGP requires all routers to be in a full mesh. This can be a performance problem for large iBGP networks of 100+ routers. Route reflectors (RRs) solve this problem. RRs are central BGP routers that other routers connect to over iBGP. The route reflector is configured to reflect all routes to the connected BGP routers. {% cite internet-routing-architecture -l 254-55 %}

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/bgp/route-reflector.svg" alt="">
  <figcaption><h4>Figure: A route reflector {% cite internet-routing-architecture %}</h4></figcaption>
</figure>

When talking about route reflectors, clients are routers connected to the reflector. iBGP peers of a route reflector are either clients or non-clients. A route reflector and its clients are known as a cluster {% cite internet-routing-architecture -l 256 %}.

A reflector must observe the following rules:

- Routes from non-client peers are reflected to clients only.
- Routes from client peers are reflected to all clients and non-clients, except the client that sent the routes.
- Routes received from eBGP peers are sent to all clients and non-clients.
  {% cite internet-routing-architecture -l 257 %}

The reflector should be duplicated and clients should be connected physically to multiple reflectors to ensure redundancy {% cite internet-routing-architecture -l 257 %}.

Misconfigured route reflectors can result in routing loops. Route reflectors should not manipulate attributes in iBGP as this can lead to routing loops {% cite internet-routing-architecture -l 259 %}.

In traditional BGP the AS_PATH attribute is used to avoid routing loops that reenter an AS. With RRs there is the possibility that a route will reenter a cluster. You can avoid routing loops in an RR setup by using the CLUSTER_LIST attribute, which contains a list of all cluster IDs that the update has been through. If a route has already passed through the cluster, the route reflector will drop the updates. You can also use an ORIGINATOR_ID attribute which is an non-transitive BGP attribute. If a router receives an update that originated from itself it will drop the update {% cite internet-routing-architecture -l 261 %}.

### Confederations

Confederations are another way of handling a large iBGP mesh {% cite internet-routing-architecture -l 263 %}.

A confederation is where an AS is split into multiple sub-ASes that interact with each other using eBGP, and internally use iBGP. A confederation behaves like a single AS to outside peers {% cite internet-routing-architecture -l 263 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/bgp/bgp-confederation.svg" alt="">
  <figcaption><h4>Figure: AS split into two confederations {% cite internet-routing-architecture %}</h4></figcaption>
</figure>

The problem with confederations is that they can lead to suboptimal routing. Because a federation is treated as a single AS, a path through a confederation may look like it has the shortest path of 2 hops compared to a route that makes 3 hops, but the confederation internally could make 4 hops making 5 hops total {% cite internet-routing-architecture -l 265 %}.

In order to have an AS behave as single entity the sub ASes use iBGP rules even though they use eBGP to communicate between each other {% cite internet-routing-architecture -l 265 %}.

The BGP decision algorithm is the same in confederations except there is a new type of route: a confederation external route. A confederation external route is preferred below an eBGP route but above an iBGP route {% cite internet-routing-architecture -l 265-6 %}.

## References

{% bibliography --cited_in_order %}
