---
layout: default
title: DNS
description: Notes on the DNS protocol.
nav_order: 10
parent: Internet
grand_parent: Computer networking
permalink: /computer-networking/internet/dns
---

<!-- prettier-ignore-start -->

# DNS
{:.no_toc}

DNS (Domain Name System) is a naming scheme and distributed database for converting human readable domain names into network IP addresses.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## DNS name space

DNS is a hierarchical naming system. The Internet is divided into hundreds of **top-level domains**, with each domain covering many hosts. Each domain is partitioned into **subdomains**, which are then further partitioned.

Domains can be represented as a tree.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/dns/tree-domains.svg" alt="">
  <figcaption><h4>Figure: A portion of the Internet domain space {% cite computer-networks -l 613 %}</h4></figcaption>
</figure>

Top-level domains are run by registrars, appointed by ICANN (The Internet Corporation for Assigned Names and Numbers). Getting a domain name normally involves paying the registrar an annual fee {% cite computer-networks -l 612-3 %}.

A domain is named from the path to the root. Each component is separated by a dot (e.g. _eng.cisco.com._). Domain names can be absolute or relative. Absolute domain names end with a period, relative domain names do not. Domain names are case sensitive {% cite computer-networks -l 615 %}.

A domain controls how it allocates domains that are under it.

## Domain resource records

Each domain has a number of **resource records** associated with it. The most common resource for a single host is an IP address, but there are many different kinds of records {% cite computer-networks -l 616 %}.

When a resolver gives a domain to DNS, it gets back the resource records associated with the name {% cite computer-networks -l 616 %}.

Resource records are five-tuple. They are encoded in binary for efficiency, but they are normally presented as ASCII text, with one record per line. These notes will use the following format:

\<Domain Name, Time to Live, Class, Type, Value\>

_Domain Name_ is the domain that the record applies to. This is the primary key used to satisfy DNS queries {% cite computer-networks -l 616 %}.

_Time to Live_ is how long the record is valid for.

The _Class_ field is almost always `IN` for Internet information. It can be other codes, but in practice it rarely is {% cite computer-networks -l 616 %}.

The _Type_ field describes what kind of record it is. There are many possible records. The most important is the A record, which holds an IPv4 address. Others include AAAA for IPv6, and MX for mail servers {% cite computer-networks -l 616 %}.

The _Value_ field is the value of the resource. It can be a number, domain name, or ASCII string {% cite computer-networks -l 616 %}.

## Name servers

In theory, a single name server should contain the entire DNS database and respond to DNS queries with a full response. In practice, there are too many DNS entries for this to be feasible.

Instead, the DNS namespace is divided into non-overlapping **zones**. The zone boundaries are defined by the zone administrator. Normally a zone has one primary name server {% cite computer-networks -l 619-20 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/dns/tree-domains-zones.svg" alt="">
  <figcaption><h4>Figure: A portion of the Internet domain space divided into zones (circled) {% cite computer-networks -l 613 %}</h4></figcaption>
</figure>

Looking up a name and finding an address is called **name resolution**. During name resolution, a resolver passes the query to a local name server. If the domain falls under the jurisdiction of the name server, it returns the authoritative resource records. **Authoritative records** are in contrast to **cached records**, which may be out of date {% cite computer-networks -l 620 %}.

If the local name server doesn't have information about the domain, the server starts a remote query. First, the local name server makes a query to one of the **root name servers**. These name servers have information about each top-level domain. In order to contact one of the root name servers, the local name server must have information about one or more of the name servers. This is normally retrieved from a system configuration file {% cite computer-networks -l 620-1 %}.

There are 13 root name servers, named _a-root-servers.net_ through _m.root-servers.net_. Root name servers will know the name server for the top-level domain (for example, _edu_). It will return the name and IP for the _edu_ name server {% cite computer-networks -l 621 %}.

The local name server continues by sending the query to the _edu_ name server. That name server will return the information for the next name server. In the case that there are no queries in the cache, this process will continue until the authoritative server for the domain name is contacted. The local name server will then forward the full response to the host that initiated the query {% cite computer-networks -l 621 %}.

The process initiated by the local name server is known as a **recursive query**. The name server continues querying until it has a full response. This is in contrast to the queries made to the name servers from the local name server, these are known as **iterative queries**, since the name servers return partial answers {% cite computer-networks -l 621-2 %}.

All of the answers, including the partial answers, are cached. This speeds up the process next time. However, cached answers aren't authoritative. The _Time to Live_ field is used to tell name servers when to expire a cached response {% cite computer-networks -l 622 %}.

Since DNS is normally sent over UDP, it's possible that a packet will be lost in transmission. DNS clients will repeat queries after a short time, in order to account for possible package loss.

## References

{% bibliography --cited_in_order %}
