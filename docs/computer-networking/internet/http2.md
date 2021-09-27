---
layout: default
title: HTTP/2
description: Notes on the HTTP/2 protocol.
nav_order: 13
parent: Internet
grand_parent: Computer networking
permalink: /computer-networking/internet/http2
---

<!-- prettier-ignore-start -->

# HTTP/2
{:.no_toc}

HTTP (Hyper Text Transfer Protocol) is an application-layer protocol used for communicating between a server and a client.

HTTP/2 enables a more efficient use of network resources over HTTP/1.1 by introducing header compression and allowing multiple concurrent exchanges over the same connection.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

HTTP/2 keeps the core semantics of HTTP/1.1. HTTP methods, status codes, URIs, and header fields are still used. What HTTP/2 changes is the format of the data, and how the data is sent between the client and the server {% cite hpbn -l 207 %}.

In HTTP/2, the HTTP data is encoded in a binary framing layer, rather than ASCII. Because of the new data format, HTTP/2 isn't backward compatible (hence the new version) {% cite hpbn -l 208 %}.

HTTP/2 is based on Google's SPDY protocol {% cite hpbn -l 208 %}. It was standardized by [rfc7540](https://tools.ietf.org/html/rfc7540) in 2015.

The aim of HTTP/2 is to improve performance issues suffered by HTTP/1.x, such as repetitive headers, and inefficient use of TCP {% cite rfc7540 -l 3-4 %}.

HTTP/2 works by creating a long-lived connection between a client and a server. Clients communicate with each other over this connection by sending frames.

A **frame** is the basic protocol unit. There are different types of frames, like HEADERS, DATA, and SETTINGS frames.

Each HTTP request/response is associated with its own **stream** within an HTTP/2 connection. Streams are mostly independent of each other, so blocked streams don't affect other streams (solving the [head-of-line blocking](https://en.wikipedia.org/wiki/Head-of-line_blocking#In_HTTP) issue that HTTP/1.x was prone to) {% cite rfc7540 -l 4, 15 %}.

HTTP/2 includes flow-control and prioritization to ensure streams are used efficiently. It also adds a new push feature, where a server can push responses to a client in response to a client-initiated request {% cite rfc7540 -l 5 %}.

HTTP/2 runs over a TCP connection and uses the same default ports as HTTP/1.x (80 for "http", 443 for "https").

## Starting HTTP2

Clients that request an HTTP URI without knowing whether the server supports HTTP/2, use the HTTP upgrade mechanism. An HTTP/1.1 request is made that includes an _Upgrade_ header field with `h2c` as its value (`h2` for HTTP/2 over TLS) {% cite rfc7540 -l 8 %}.

If the server supports HTTP/2, it can accept the upgrade with a status code of 101 (switching protocols). After the 101 response, the server can send HTTP/2 responses {% cite rfc7540 -l 9 %}.

The first HTTP/2 frame sent by the server is a SETTINGS frame (known as a **server connection preface**) {% cite rfc7540 -l 9 %}.

A client that attempts to upgrade to HTTP/2 must include an _HTTP2-Settings_ header field. The value will be the payload of a SETTINGS frame (Base64 encoded) containing the HTTP2 settings for the requesting client {% cite rfc7540 -l 10 %}.

To start an HTTP/2 connection with a server that is known to support HTTP/2, a client must first send a **client connection preface** to the server. After the connection preface has been sent, the client can send HTTP/2 frames to the server {% cite rfc7540 -l 10-1 %}.

### HTTP/2 Connection Preface

Each endpoint must send a connection preface to confirm that the HTTP/2 protocol is being used and to set initial settings for the connection {% cite rfc7540 -l 11 %}.

The client connection preface in hex is 0x505249202a20485454502f322e300d0a0d0a534d0d0a0d0a. A SETTINGS frame follows the sequence, although it can be empty {% cite rfc7540 -l 11 %}.

Clients can send frames to the server immediately after sending the connection preface {% cite rfc7540 -l 12 %}.

The server connection preface is a SETTINGS frame (that can be empty) {% cite rfc7540 -l 11 %}.

The SETTINGS frame sent from a peer needs to be acknowledged by the receiver. This is done by sending a SETTINGS frame with the ACK flag set {% cite rfc7540 -l 11,40 %}.

## Streams

A **stream** is a sequence of frames sent between a client and a server over an HTTP/2 connection {% cite rfc7540 -l 15 %}.

An HTTP/2 connection can contain multiple streams, which can be interleaved. A stream is identified by an integer, and can be closed by either endpoint {% cite rfc7540 -l 15 %}.

Streams go through multiple states: idle, open, half-closed, and closed {% cite rfc7540 -l 16 %}.

All streams start idle. Sending or receiving a HEADERS frame causes stream to become open {% cite rfc7540 -l 17 %}.

<!-- prettier-ignore-start -->
<!-- Prettier can't handle underscores ("_") -->
Sending a PUSH_PROMISE reserves an idle (local) stream. Receiving PUSH_PROMISE reserves an idle (remote) stream. A PUSH_PROMISE frame references the stream to be reserved in a _Promised Stream Id_ field {% cite rfc7540 -l 17 %}.
<!-- prettier-ignore-end -->

An open stream can be used by both peers to send any kind of frame {% cite rfc7540 -l 18 %}.

Either peer can send a frame with the END_STREAM flag set, which puts the stream in a half-closed state {% cite rfc7540 -l 18 %}.

The terminal state is the closed state. Only PRIORITY frames can be sent on a closed stream {% cite rfc7540 -l 19-20 %}.

### Stream Identifiers

Streams are identified with a 31-bit unsigned int. Streams initiated by a client use odd numbered values, streams initiated by a server use even values {% cite rfc7540 -l 20-1 %}.

The stream identifier must be greater than the previous stream identifiers used on the connection. Once a stream is opened, all streams initiated by the peer in the idle state with a lower stream value are closed {% cite rfc7540 -l 21 %}.

Stream identifiers can't be reused. If a connection exhausts all stream identifiers, the client and server must reestablish a connection {% cite rfc7540 -l 21 %}.

## Headers

An HTTP/2 header field (a header) is a name with an associated value. Headers are used in request, response, and server push operations {% cite rfc7540 -l 14 %}.

**Header lists** are collections of headers. A header list is serialized into a **header block** when transmitted, and compressed following the HPACK compression scheme {% cite rfc7540 -l 14 %}.

The header block is divided into one or more **header block fragments**, and then transmitted within the payload of a HEADERS, PUSH_PROMISE, or CONTINUATION frame {% cite rfc7540 -l 14 %}.

The receiving endpoint reassembles a header block by concatenating its fragments. It gets the header list by decompressing the block {% cite rfc7540 -l 14 %}.

Header compression is stateful. A compression context and decompression context are used for the entire HTTP connection {% cite rfc7540 -l 14 %}.

Header frames must be transmitted as a contiguous sequence, they can't be interleaved with other frames. The last frame in a headers sequence has the END_HEADERS flag set {% cite rfc7540 -l 14-5 %}.

(See a [list of HTTP headers](https://www.iana.org/assignments/message-headers/message-headers.xhtml) for more details)

## Response/ Request exchange

An HTTP request or response consists of:

- For a response, zero or more HEADERS frames (and CONTINUATION frames) containing messages of 1xx responses.
- One HEADERS frame followed by zero or more CONTINUATION frames.
- Zero or more DATA frames containing the request/response body.
- Optional HEADERS frame (and possible CONTINUATION frames) containing the trailer-part.

{% cite rfc7540 -l 52 %}

A request is sent on a new stream, using an unused stream identifier. A corresponding response and request are sent on the same stream {% cite rfc7540 -l 52 %}.

The last frame in a sequence has its END_STREAM flag set. END_STREAM puts the stream into a half-closed (local) state for the client, and half-closed (remote) state for the server {% cite rfc7540 -l 52-3 %}.

HTTP2 uses pseudoheaders that begins with a colon (:) to convey metadata like the target URI (:path), the HTTP method (:method), and the status code of the response (:status) {% cite rfc7540 -l 54 %}.

HTTP/2 connections are persistent, and it's expected that a client won't close a connection until it determines that no further communication is necessary (e.g. if a user navigates away from a webpage). Servers should maintain connections for as long as possible, but are allowed to close connections if necessary (e.g. if a server is running low on memory) {% cite rfc7540 -l 65 %}.

## Frames

Endpoints can send frames once a connection is established {% cite rfc7540 -l 12 %}.

Frames have the following format:

```
    +-----------------------------------------------+
    |                 Length (24)                   |
    +---------------+---------------+---------------+
    |   Type (8)    |   Flags (8)   |
    +-+-------------+---------------+-------------------------------+
    |R|                 Stream Identifier (31)                      |
    +=+=============================================================+
    |                   Frame Payload (0...)                      ...
    +---------------------------------------------------------------+
```

{% cite rfc7540 -l 12 %}

_Length_ is a 24-bit unsigned integer representing the length of the frame payload.

_Type_ is the type of frame (e.g. DATA, HEADERS).

_Flags_ contains boolean flags that are specific to the type of frame.

_R_ is a reserved bit.

_Stream Identifier_ is an unsigned 31-bit integer that identifies a stream.

{% cite rfc7540 -l 12-3 %}

The _Frame Payload_ depends on the type of frame. This section will cover the frame payloads of DATA, HEADER, and PUSH_PROMISE frames in detail.

The different types of frames are:

- **DATA**: contains the application data.
- **HEADERS**: used to a open a stream and (optionally) carry a headers block fragment {% cite rfc7540 -l 32 %}.
- **SETTINGS**: used for configuring how the endpoints communicate, and to acknowledge receipt of the initial parameters {% cite rfc7540 -l 36-7 %}.
- **PUSH_PROMISE**: notifies a receiving endpoint of a stream that a sending endpoint intends to initiate {% cite rfc7540 -l 40 %}.
- **PRIORITY**: used to set priority for the stream that the frame is part of. It can be sent in any stream state {% cite rfc7540 -l 34 %}.
- **RST_STREAM**: terminates a stream {% cite rfc7540 -l 36 %}.
- **PING**: used for measuring the round-trip time from the sender {% cite rfc7540 -l 42 %}.
- **WINDOW_UPDATE**: used to implement flow control {% cite rfc7540 -l 46 %}.

### The DATA frame

DATA frames contain application data. They are associated with a stream and are used to carry the payload of HTTP requests and responses.

```
    +---------------+
    |Pad Length? (8)|
    +---------------+-----------------------------------------------+
    |                            Data (*)                         ...
    +---------------------------------------------------------------+
    |                           Padding (*)                       ...
    +---------------------------------------------------------------+
```

The _Pad Length_ field is conditional: it only appears if the `PADDED` flag is set. _Pad Length_ contains the length of the _Padding_.

The _Data_ field is the application data that's being sent.

_Padding_ contains octets set to `0`.

The DATA frame has two flags that can be set in the _Flags_ field: `END_STREAM`, and `PADDED`.

<!-- prettier-ignore-start -->
<!-- Prettier strips out escape character for _ -->
- _END\_STREAM_: indicates that the frame is the last frame for the identified stream.
- _PADDED_: indicates that the _Pad Length_ field exists and _Padding_ exists.
<!-- prettier-ignore-end -->

The length of the _Data_ field is the length of the frame payload after subtracting the other fields.

{% cite rfc7540 -l 31 %}

### The HEADERS frame

HEADERS frames open streams and (optionally) carry a header block fragment {% cite rfc7540 -l 32 %}.

```
    +---------------+
    |Pad Length? (8)|
    +-+-------------+-----------------------------------------------+
    |E|                 Stream Dependency? (31)                     |
    +-+-------------+-----------------------------------------------+
    |  Weight? (8)  |
    +-+-------------+-----------------------------------------------+
    |                   Header Block Fragment (*)                 ...
    +---------------------------------------------------------------+
    |                           Padding (*)                       ...
    +---------------------------------------------------------------+
```

The _Pad Length_ field is conditional: it only appears if the `PADDED` flag is set. _Pad Length_ contains the length of the _Padding_.

_E_ indicates that the stream dependency is exclusive (meaning this stream is the only stream dependant on a parent stream). It's only present if the `PRIORITY` flag is set {% cite rfc7540 -l 33 %}.

_Stream Dependency_ is a stream identifier for the stream that this stream depends on {% cite rfc7540 -l 33 %}.

_Weight_ is an unsigned 8-bit integer representing the priority of the stream {% cite rfc7540 -l 33 %}.

_Header Block Fragment_ contains a header block fragment {% cite rfc7540 -l 33 %}.

HEADERS frames have the following flags:

<!-- prettier-ignore-start -->
<!-- Prettier strips out escape character for _ -->
- _END\_STREAM_: puts the stream into a half-closed state.
- _END\_HEADERS_: indicates the frame has a header block and isn't followed by a CONTINUATION frame {% cite rfc7540 -l 34 %}.
- _PADDED_: indicates whether the frame contains padding.
- _PRIORITY_: indicates that Exclusive Flag (_E_), _Stream Dependency_, and _Weight_ fields are present {% cite rfc7540 -l 34 %}.
<!-- prettier-ignore-end -->

If a header block fragment doesn't fit in a header frame, it's included in a CONTINUATION frame {% cite rfc7540 -l 34 %}.

### The PUSH_PROMISE frame

The PUSH_PROMISE frame is used to implement server push.

```
    +---------------+
    |Pad Length? (8)|
    +-+-------------+-----------------------------------------------+
    |R|                  Promised Stream ID (31)                    |
    +-+-----------------------------+-------------------------------+
    |                   Header Block Fragment (*)                 ...
    +---------------------------------------------------------------+
    |                           Padding (*)                       ...
    +---------------------------------------------------------------+
```

The _Pad Length_ field is conditional: it only appears if the _PADDED_ flag is set. _Pad Length_ contains the length of the _Padding_.

_R_ is a reserved bit.

_Promised Stream ID_ is an unsigned 31-bit integer that represents the stream that is reserved by the PUSH_PROMISE frame.

_Header Block Fragment_ is a header block fragment for the pushed response.

_Padding_ octets are set to 0.

{% cite rfc7540 -l 40 %}

## Server push

Server push enables a server to preemptively send responses to a client that has made a request {% cite rfc7540 -l 60 %}.

Pushed responses are associated with an explicit request from a client. PUSH_PROMISE frames are sent on the stream of the associated request. A PUSH_PROMISE frame includes a promised stream identifier, created from the valid list of stream identifiers {% cite rfc7540 -l 62 %}.

<!-- prettier-ignore-start -->
<!-- Prettier can't handle underscores ("_") -->

A PUSH_PROMISE frame creates a stream in a reserved (local) state for the server and reserved (remote) state for the client using the identifier sent in the _Promised Stream ID_ field {% cite rfc7540 -l 62 %}.
<!-- prettier-ignore-end -->

Once a server has sent a PUSH_PROMISE frame, it can send a response on a stream with the stream identifier sent in the PUSH_PROMISE frame.

## Flow-control

The flow-control scheme ensures that streams do not interfere with each others use of the TCP connection. Flow-control is applied to both individual streams and the overall connection {% cite rfc7540 -l 22 %}.

Flow-control is implemented using a window that is kept by each sender on every stream. The window is a value that represents how many octets of data a sender is able to send {% cite rfc7540 -l 47 %}.

Currently only DATA frames are affected by flow control {% cite rfc7540 -l 23 %}.

There are two flow-control windows:

1. The stream flow-control window
2. The connection flow-control window

After a sender sends a flow-controlled frame, it will reduce both flow-control windows by the length of the sent frame {% cite rfc7540 -l 47 %}.

After a receiver consumes data, it sends a WINDOW_UPDATE frame with the number of octets it has freed. There are separate WINDOW_UPDATE frames for the stream-level and connection-level windows {% cite rfc7540 -l 47 %}.

When a sender receives a WINDOW_UPDATE frame, it increases the relevant window size by the specified length {% cite rfc7540 -l 47 %}.

Flow control is determined by the receiver, and is directional {% cite rfc7540 -l 23 %}.

The initial flow control value is 65,535 octets for both new streams and for the overall connection {% cite rfc7540 -l 23 %}. These values can be configured by the SETTINGS_INITIAL_WINDOW_SIZE setting {% cite rfc7540 -l 48 %}.

## Stream priority

Clients can assign the priority for streams by including prioritization information in the HEADERS frame {% cite rfc7540 -l 24 %}.

At any other point during a stream's lifetime, the PRIORITY frame can be used to change the priority of a stream {% cite rfc7540 -l 24 %}.

Streams can also be marked as dependent on the completion of other streams. A stream dependant on another stream is a **dependant stream**. The stream that it's dependant on is its **parent stream** {% cite rfc7540 -l 24-5 %}.

## References

{% bibliography --cited_in_order %}
