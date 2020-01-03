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

2. Define the full form of initialisms in parentheses when used for the first time:

   > UDP (User Datagram Protocol) is a connectionless transport protocol.

3. Use sentence case for bullet points and end them with a period:

   > - Handles multiplexing and demultiplexing of multiple packet sources.
   > - Handles error control and retransmission.

   Unless the list contains only nouns:

   > - Coaxial cable
   > - Twisted pair cable

4. Place references at the end of a paragraph, before the terminating punctuation mark:

   > BGP4 supports CIDR and subnetting {% cite internet-routing-architecture --locator 101 %}.

5. Place a comma after Latin abbreviations (like _e.g._ and _i.e._):

   > Each component is separated by a dot (e.g., _eng.cisco.com._).

6. Always use an Oxford comma:

   > Streams go through multiple states: idle, open, half-closed, and closed.

7. End notes with a period:

   > _Note: this approach can be improved further by storing results in two single variables._

8. Use em dashes without a space on either side:

   > Linux is a **monolithic kernel**—it runs as a single process in a single address space.

9. Capitalize figures and tables when referencing them in the text:

   > For more information, see Figure 1.

10. Append _es_ to the plural form of an initialism that end in s:

    > The last mainstream cooperative OSes were Mac OS 9 and Windows 3.1.
    > Single-homed ASes connect to another AS via a single exit.

11. Use ID, IDs, UID, and UIDs:

  > The next step is to allocate a new UID.

## Math

1. Use MathJax for formulas, variables, and functions:

  > BFS runs in $$O(n + m)$$ time on a graph of $$n$$ vertices and $$m$$ edges

2. Use angle brackets (\"<\>\") for tuples written in text:

   > ARP maps a \<protocol, address\> pair to a link layer address.

## Code

1. Use parentheses for functions and methods:

   > Blink is initialized with `BlinkInitializer::Initialize()`.

2. Use parentheses for function-like macros:

   > To statically create a tasklet you can use the macros `DECLARE_TASKLET()` and `DECLARE_TASKLET_DISABLED()`.

3. Reference C structs by their name:

   > The superblock is represented by the `super_block` struct.

4. Write system calls with parentheses:

   > It's created in response to the `open()` system call, and is destroyed in response to the `close()` system call.

5. Format flags as code:

   > Corresponds to `VM_READ`.

## Protocols

1. Italicize header field names:

   > _Type_ is the type of frame.

2. Write header field names in title case:

   > `| Total Length |`

3. Format field values as code:

   > _Type_ is the type of frame (e.g. `DATA`, `HEADERS`).

4. Use _an HTTP_ request instead of _a HTTP_ request:

   > An HTTP/2 header field (a header) is a name with an associated value.

5. Write bits per second as b/s and bytes per second as B/s, with the metric prefix in uppercase:
   > 100MB/s is 800Mb/s
