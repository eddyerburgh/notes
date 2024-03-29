---
layout: default
title: Conventions
description: The conventions followed in these notes.
has_toc: false
permalink: /conventions
nav_exclude: true
---

# Conventions

This page details the arbitrary conventions I've chosen to follow in these notes.

## General

1. Use American English.

2. Write bullet points in sentence case and end them with a period:

   > - Handles multiplexing and demultiplexing of multiple packet sources.
   > - Handles error control and retransmission.

   Unless the list contains only nouns:

   > - Coaxial cable
   > - Twisted pair cable

3. Place references at the end of a paragraph, before the terminating punctuation mark:

   > BGP4 supports CIDR and subnetting {% cite internet-routing-architecture --locator 101 %}.

4. No comma after i.e. and e.g.:

   > Each component is separated by a dot (e.g. _eng.cisco.com._).

5. Always use an Oxford comma:

   > Streams go through multiple states: idle, open, half-closed, and closed.

6. End side notes with a period:

   > _Note: this approach can be improved further by storing results in two single variables._

7. Use em dashes without a space on either side:

   > Linux is a monolithic kernel—it runs as a single process in a single address space.

8. Capitalize figures and tables when referencing them in the text:

   > For more information, see Figure 1.

9. Create the plural form of initialisms that end in s by appending _es_:

   > The last mainstream cooperative OSes were Mac OS 9 and Windows 3.1.

   > Single-homed ASes connect to another AS via a single exit.

10. Use ID, IDs, UID, and UIDs:

    > The next step is to allocate a new UID.

11. Define the full form of initialisms in parentheses when used for the first time:

    > UDP (User Datagram Protocol) is a connectionless transport protocol.

12. Use key-value and client-server instead of key/value and client/server.

> DHCP uses a client-server model.

## Math

1. Use MathJax for formulas, variables, and functions:

   > BFS runs in $$O(n + m)$$ time on a graph of $$n$$ vertices and $$m$$ edges.

2. Use angle brackets (\"<\>\") for tuples written in text:

   > ARP maps a \<protocol, address\> pair to a link layer address.

## Code

1. Use parentheses for functions, methods, function-like macros, and system calls:

   > Blink is initialized with `BlinkInitializer::Initialize()`.

   > To statically create a tasklet you can use the macros `DECLARE_TASKLET()` and `DECLARE_TASKLET_DISABLED()`.

   > It's created in response to the `open()` system call, and is destroyed in response to the `close()` system call.

2. Reference C structs by their name:

   > The superblock is represented by the `super_block` struct.

3. Format flags as code:

   > Corresponds to `VM_READ`.

## Protocols

1. Italicize header field names:

   > The _Data_ field contains the payload that is being sent.

2. Write header field names in title case:

   > The _Originate Timestamp_ field is the time the sender touched the message before sending it.

3. Format field values as code:

   > _Type_ is the type of frame (e.g. `DATA`, `HEADERS`).

4. Use _an HTTP request_ instead of _a HTTP request_:

   > An HTTP/2 header field (a header) is a name with an associated value.

5. Write bits per second as b/s and bytes per second as B/s, with the metric prefix in uppercase:
   > 100MB/s is 800Mb/s
