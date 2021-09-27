---
layout: default
title: HTTP/1
description: Notes on the HTTP/1 protocol.
nav_order: 12
parent: Internet
grand_parent: Computer networking
permalink: /computer-networking/internet/http1
---

<!-- prettier-ignore-start -->

# HTTP/1
{:.no_toc}

HTTP (Hyper Text Transfer Protocol) is an application-layer protocol used for communicating between a client and a server.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

HTTP is a request/response protocol. It specifies what clients can send to a server, and what they can expect to receive back {% cite computer-networks -l 683 %}.

Originally HTTP was intended to transfer HTML documents from servers to browsers, but it's now used for many different kinds of media.

HTTP/1.1 requests are made up of multiple lines. The first line is the most important, it contains the request method and the HTTP version number:

```
GET /glossary/internet HTTP/1.1
```

## URIs

URIs (Universal Resource Identifiers) are strings that identify a resource {% cite rfc2616 -l 18 %}.

URIs can be represented either in absolute from, or relative form (relative to some base URI). An absolute URI begins with a _scheme name_ (e.g. https) {% cite rfc2616 -l 19 %}.

HTTP URLs (Universal Resource Locators) include the information required to reach the resource. They have the following structure:

```
"http:" "//" host [ ":" port ] [ abs_path [ "?" query ]]
```

{% cite rfc2616 -l 19 %}

## Connections

HTTP usually runs over TCP. HTTP/1.0 would close the TCP connection after receiving an HTTP response from a server. This meant each connection had to perform a TCP handshake, even if multiple HTTP requests were made to the same domain while loading a webpage (a common scenario) {% cite computer-networks -l 684 %}.

To solve this, HTTP/1.1 supports **persistent connections**. The TCP connection can be kept alive, and additional requests can be sent over the same connection (also known as **connection reuse**). This improves performance {% cite computer-networks -l 684 %}.

It's also possible to pipeline requests (send 2 requests at the same time) {% cite computer-networks -l 684 %}.

Connections are typically closed after a short time (e.g. 60 seconds) to avoid servers holding too many connections open {% cite computer-networks -l 685 %}.

## HTTP Methods

An HTTP request has an associated method.

The first word on the first line of an HTTP request (the **Request-Line**) is its method name:

```
Request-Line = Method Request-URI HTTP-Version CRLF
```

{% cite rfc2616 -l 35 %}

Method names indicate the intent of the request:

| Method  | Description                |
| ------- | -------------------------- |
| GET     | Read a Web page.           |
| HEAD    | Read a Web page’s header.  |
| POST    | Append to a Web page.      |
| PUT     | Store a Web page.          |
| DELETE  | Remove the Web page.       |
| TRACE   | Echo the incoming request. |
| CONNECT | Connect through a proxy.   |
| OPTIONS | Query options for a page.  |

{% cite computer-networks -l 686 %}

## HTTP Headers

The first line of an HTTP request/response can be followed by additional lines, called **request headers**. Responses can have **response headers** {% cite computer-networks -l 688 %}.

Each header field is made of a name and a field value, separated by a colon (:):

```
message-header = field-name ":" [ field-value ]
```

{% cite rfc2616 -l 31-2 %}

Headers can be used to set caching policies, provide authorization, and provide metadata about the user agent making the request (as well as many other uses) {% cite computer-networks -l 688 %}.

## HTTP status codes

HTTP status codes are included in HTTP responses, as part of the **Status-Line**:

```
Status-Line = HTTP-Version Status-Code Reason-Phrase CRLF
```

{% cite rfc2616 -l 39 %}

The status code is a 3-digit integer that describes the status of the HTTP response {% cite rfc2616 -l 39 %}.

The status codes are classed based on the first integer:

| Code | Meaning      | Examples                                            |
| ---- | ------------ | --------------------------------------------------- |
| 1xx  | Information  | 100 = server agrees to handle client’s request.     |
| 2xx  | Success      | 200 = request succeeded. 204 = no content present.  |
| 3xx  | Redirection  | 301 = page moved. 304 = cached page still valid.    |
| 4xx  | Client error | 403 = forbidden page. 404 = page not found.         |
| 5xx  | Server error | 500 = internal server error. 503 = try again later. |

{% cite computer-networks -l 688 %}

## Caching

**Caching** is the process of storing HTTP responses to be used later. Caching improves performance by reducing network traffic and latency {% cite computer-networks -l 690 %}.

In HTTP, a **cache** is defined as the cache entries and the program that manages the cache entries. The primary cache key consists of the request method and URI, and the value is the HTTP response {% cite rfc7234 -l 5-6 %}.

_Vary_ headers can be used to create secondary cache keys {% cite rfc7234 -l 9-10 %}.

HTTP has shared caches and private caches. **Shared caches** can be accessed by multiple users whereas **private caches** are user-specific (e.g. a browser). Responses can be set to be cacheable for private caches only with the _Cache-Control_ _private_ directive {% cite rfc7234 -l 4 %}.

A cache entry can be either fresh or stale. "A fresh response is one whose age has not yet exceeded its freshness lifetime. Conversely, a stale response is one where it has" {% cite rfc7234 -l 11 %}.

The **freshness lifetime** of a response is the time between the generation of the response by an origin server and the expiration time of the response. Explicit expiration times can be set with HTTP headers (e.g. the _Cache-Control_ _max-age_ directive), but an expiration time can be implicitly calculated if the response is missing the required headers and the response is considered cacheable {% cite rfc7234 -l 11 %}.

_Note: a cacheable response is defined as a response with a cacheable request method (e.g. GET), and either a cacheable status code (e.g. 200, 301, 404) or a header marking the request as cacheable (e.g. with the public response directive) {% cite rfc7234 -l 13 %}._

A fresh response can be returned to a client by a cache without the cache contacting the origin server. A stale response can be served if the origin server returns an error, or while the cache is revalidating a request (using the _stale-if-error_ and _stale-while-revalidate_ directives) {% cite rfc7234 -l 11 15 %}{% cite rfc5861 -l 2 %}.

When a stored response is stale, the cache must revalidate the response. The cache can reduce the amount of data transferred by making a conditional request to the origin server. If the origin server validates the resource (usually meaning the stored response hasn't changed), the origin server responds with a 304 and the cache can respond to the client with its stored response. If the response has changed, the origin server responds with the full response {% cite rfc7232 -l 15 %}.

The origin server can validate a resource by using a variety of methods, such as the _Last-Modified_ header {% cite rfc7234 -l 15 %}.

Another way to validate a resource is by using a response's ETag. An **ETag** (entity tag) is an identifier that can be added to a response (commonly an ETag is generated by taking a hash of the response body). To validate a response using an ETag, a cache can send a GET request with an _If-None-Match_ header containing the ETag of the response that is being validated. A recipient server can then respond with a _304_ _Not Modified_ if the response has the same ETag as provided in the _If-None-Match_ header {% cite rfc7234 -l 15 %}{% cite rfc7232 -l 15 %}.

## References

{% bibliography --cited_in_order %}
