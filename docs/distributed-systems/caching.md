---
layout: default
title: Caching
description: Notes on caching.
has_toc: false
nav_order: 6
parent: Distributed systems
permalink: /distributed-systems/caching
---

<!-- prettier-ignore-start -->

# Caching
{:.no_toc}

>"There are only two hard things in Computer Science: cache invalidation and naming things."
>
> -- <cite>Phil Karlton</cite>

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

**Caching** is a special form of replication where data is copied by a client and saved for later use {% cite distributed-systems -l 23 %}.

Caching can improve a system's response time significantly and help improve the scalability of a system by reducing load {% cite redis15reasons -l 2 %}.

Some commonly cached items include:

- HTML pages (partial or full)
- API responses
- Database queries
- Session data

{% cite redis15reasons -l 4 %}

## In-memory caching

One form of caching is in-memory caching. With in-memory caching, the result of an expensive computation or network request is stored in memory to be accessed later.

A **local cache** stores entries on the same machine, usually in a hash table. This makes data retrieval fast because accessing the cache doesn't involve the network, however it means that each node in a system will have its own cache. {% cite awscaching2017 -l 2 %}.

A more robust solution is to use a remote (or distributed) cache. A **remote cache** is a separate instance that is dedicated to storing cached values in memory. Commonly, remote caches are distributed key-value stores like Redis or Memcached which can handle millions of requests per second per node. With a remote cache, multiple nodes can share the same cache {% cite awscaching2017 -l 2-3 %}.

In-memory caches generally expose an API to get, set, and delete entries:

- `get(key)`
- `set(key, value, ttl)`
- `delete(key)`

In order to stop the cache from growing indefinitely, caches remove items when the cache reaches a certain size. The items that are removed depend on the cache's eviction policy.

### Eviction policy

A cache's **eviction policy** determines the order in which entries are removed from a full cache.

**LRU** (Least Recently Used) is a popular eviction policy that removes the least recently used entries first. LRU requires extra memory to track the relative usage of entries.

**LFU** (Least Frequently Used) removes the least frequently used entries first. Again, LFU requires extra memory to track the usage of entries.

**FIFO** removes items in the order they were added to the cache.

A TTL eviction policy evicts entries after they expire, based on a TTL (Time To Live) value. To determine a suitable TTL value you should consider both how frequently the origin data is updated, and what the consequences of returning stale data are {% cite awscaching2017 -l 6 %}.

## HTTP caching

The HTTP protocol supports the caching of some responses. HTTP caching can significantly improve the performance of a web site or API.

Browsers, proxies, reverse proxies, and CDNs can all be used to cache HTTP assets.

HTTP distinguishes between shared caches and private caches. Shared caches can be accessed by multiple users (e.g. a CDN) whereas private caches are user-specific (e.g. a browser) {% cite rfc7234 -l 4 %}.

HTTP caches normally store cached assets to disk, with the mapping keys stored in memory.

For more details on the specifics of HTTP/1 caching, see [the HTTP1 section on caching]({{'/computer-networking/internet/http1#caching' | relative_url }}).

## Caching patterns

**Caching patterns** are design patterns for integrating a cache into a system.

### Cache-aside

In the **cache-aside pattern** data is loaded into the cache as it's required.

The workflow is:

1. Before executing an expensive operation, the application checks to see if the result already exists in the cache.
2. If the data is available (a cache hit), the application returns the cached data.
3. If the data is not available (a cache miss), the application executes the expensive operation and stores the result in the cache for future use.

{% cite awscaching2017 -l 4 %}

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/caching/cache-aside-pattern.svg" alt="">
  <figcaption><h4>Figure: The cache-aside pattern {% cite distributed-systems -l 5 %}</h4></figcaption>
</figure>

```python
def get_image(image_id):
  image = cache.get(image_id)
  if image == None:
    image = db.get_image(image_id)
    cache.set(image_id, image, IMAGE_TTL)
  return image
```

One advantage of the cache-aside pattern is that the cache only contains data that the application has actually requested {% cite awscaching2017 -l 5 %}.

One downside of the cache-aside pattern is that each cache miss adds extra latency {% cite awscaching2017 -l 5 %}.

### Read-through

In the **read-through pattern**, all requests go through the cache. If the requested item is not in the cache, then the cache fetches the item from the data source {% cite oracle2018caching -l 6 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/caching/read-through-pattern.svg" alt="">
  <figcaption><h4>Figure: The read-through pattern {% cite distributed-systems -l 5 %}</h4></figcaption>
</figure>

The read-through pattern is similar to the cache-aside pattern in that data is only requested as it's required.

### Write-through

In the **write-through pattern**, an application writes to the cache at the same time as it writes to the database {% cite oracle2018caching -l 7 %}.

```python
def edit_comment(comment_id, comment_content):
  db.update_comment(comment_id, comment_content)
  cache.set(comment_id, comment_content)
```

When paired with the cache-aside pattern, write-through ensures that data in the cache is always up-to-date {% cite oracle2018caching -l 7 %}.

### Write-back

In the **write-back pattern** (also known as the write-behind pattern) the client updates the cache, rather than the backing data store. The cache is then responsible for updating the backing data store after some delay {% cite oracle2018caching -l 9 %}.

The write-back pattern improves write performance and works well for write-heavy workloads {% cite oracle2018caching -l 9 %}.

One downside of the write-back pattern is that pending updates can be lost in the case of failure {% cite oracle2018caching -l 9 %}.

### Pre-warming

A cache is said to be **cold** when it has no entries. A cold cache will result in many cache misses.

A **warm** cache contains many entries, resulting in more cache hits.

**Pre-warming** a cache involves loading entries before using the cache in production. Pre-warming can help improve performance by reducing the number of cache misses.

## CDNs

CDNs (Content Delivery Networks) improve latency, reliability, and redundancy by replicating resources geographically and load balancing requests between replicas {% cite cdn-taxonomy -l 1 %}.

CDNs commonly work over HTTP and are used to serve static assets (like images, HTML pages, and JavaScript files), but can also be used for dynamic content {% cite cdn-taxonomy -l 3 %}.

A CDN consists of **edge servers** which serve content and **origin servers** which supply the content {% cite 10.1145/1842733.1842736 -l 5 %}.

Request-routing infrastructure directs user requests to the closest edge server. Akamai, for example, hosts its own authoritative DNS servers to dynamically resolve domain names to a suitable edge server based on the user's location and Akamai's network data {% cite 10.1145/1842733.1842736 -l 16 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/distributed-systems/caching/cdn.svg" alt="">
  <figcaption><h4>Figure: A request to a CDN {% cite distributed-systems -l 5 %}</h4></figcaption>
</figure>

Commonly, CDNs take a pull-based approach to serving content. If an edge server is unable to fulfill the user request (e.g. if it does not have the requested content in its cache), then it must make a request to an origin server to get the content. Once the CDN has the content, it can cache the response for future use {% cite 10.1145/1842733.1842736 -l 16 %}.

Some CDNs take a push-based approach, for example Netflix's Open Connect.

### Netflix Open Connect

Open Connect is Netflix's push-based CDN for serving video and image files {% cite netflixopenconnect2017 -l 3 %}.

Open Connect is made up of OCAs (Open Connect Appliances) and a control plane that manages the OCAs. The control plane is also responsible for resolving client requests to a list of OCA URLs that clients can use to fetch static assets from {% cite netflixopenconnect2017 -l 3 %}.

Each OCA stores a portion of the Netflix catalog. During off-peak hours, the OCAs contact control plane services to update their content {% cite netflixopenconnect2017 -l 5 %}.

OCAs are installed at thousands of IXPs and in ISP data centers around the world {% cite netflixopenconnect2017 -l 2 %}.

## References

{% bibliography --cited_in_order %}
