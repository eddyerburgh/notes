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
GET /glossary/big-o-notation HTTP/1.1
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

| Method  | Description               |
| ------- | ------------------------- |
| GET     | Read a Web page           |
| HEAD    | Read a Web page’s header  |
| POST    | Append to a Web page      |
| PUT     | Store a Web page          |
| DELETE  | Remove the Web page       |
| TRACE   | Echo the incoming request |
| CONNECT | Connect through a proxy   |
| OPTIONS | Query options for a page  |

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

The status code is a 3-digit integer code that describes the status of the HTTP response {% cite rfc2616 -l 39 %}.

The status codes are classed based on the first integer:

| Code | Meaning      | Examples                                           |
| ---- | ------------ | -------------------------------------------------- |
| 1xx  | Information  | 100 = server agrees to handle client’s request     |
| 2xx  | Success      | 200 = request succeeded; 204 = no content present  |
| 3xx  | Redirection  | 301 = page moved; 304 = cached page still valid    |
| 4xx  | Client error | 403 = forbidden page; 404 = page not found         |
| 5xx  | Server error | 500 = internal server error; 503 = try again later |

{% cite computer-networks -l 688 %}

## Caching

Caching is the process of saving HTTP responses to be used later. Caching improves performance by reducing network traffic and latency {% cite computer-networks -l 690 %}.

## References

{% bibliography --cited_in_order %}
