---
layout: default
title: TCP
description: Notes on the TCP protocol.
nav_order: 3
parent: Internet
grand_parent: Computer networking
permalink: /computer-networking/internet/tcp
---

<!-- prettier-ignore-start -->

# TCP
{:.no_toc}

TCP (Transmission Control Protocol) provides a reliable end-to-end byte stream over an unreliable internetwork. Originally defined in 1981, there have been many amendments since {% cite computer-networks -l 552 %}.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

A machine that supports TCP has a TCP transport entity, which is normally part of the kernel. The entity accepts data streams from local processes. It breaks the streams into chunks no larger than 64KB (usually 1460 bytes to fit in an Ethernet frame), and sends each chunk as an IP datagram. When datagrams containing TCP data arrive at a machine, they are given to a TCP entity that reconstructs the byte stream {% cite computer-networks -l 553 %}.

IP doesn’t guarantee that datagrams will arrive, or how fast they are sent. So TCP is responsible for sending datagrams fast enough to maximize capacity, but not too fast to cause congestion. TCP must also reassemble messages in the correct order, in case they arrive out of order {% cite computer-networks -l 553 %}.

## The TCP service model

TCP service is achieved with endpoints called sockets. A socket has a socket number, which contains the IP address of the host and a 16-bit port number.

For TCP service to be obtained, a connection must be established between a socket on one machine and a socket on another. Sockets can handle multiple connections at a time.

Port numbers below 1024 are reserved for standard services. They are called **well-known ports**. Ports from 1024 to 49151 can be registered for use, but applications can choose there own ports without registering them.

TCP connections are full duplex and point-to-point. TCP doesn’t support multicasting or broadcasting {% cite computer-networks -l 554 %}.

"A TCP connection is a byte stream, not a message stream. Message boundaries are not preserved end to end." If a process writes four times to a TCP stream in 512-byte chunks, the data could be delivered as four 512-byte chunks, or two 1024-byte chunks. There's no way for the receiver to detect the units the data was written in {% cite computer-networks -l 554 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/tcp/tcp-byte-stream.svg" alt="">
  <figcaption><h4>Figure: Data delivered to application {% cite computer-networks -l 555 %} </h4></figcaption>
</figure>

TCP can choose to buffer the data it receives from a user process before sending it. This is to send a larger amount of data at once. Sometimes this behavior is undesirable. To force data out as soon as its received, TCP has a push flag that's carried on packets, this can be set from user space by providing an OS flag check (like `TCP_NODELAY` in Linux) {% cite computer-networks -l 555 %}.

## The TCP Protocol

Every byte on a TCP connection has its own 32-bit sequence number. The sequence numbers are carried for sliding window in one direction and for acknowledgements in the other {% cite computer-networks -l 556 %}.

TCP sends information in TCP segments. A **segment** contains a fixed 20-byte header (with an optional extra part) followed by 0 or more bytes of data. TCP software decides how big segments should be. The maximum segment size is 65,515 bytes (the max IP payload). Each link in a network has an MTU (Maximum Transfer Unit). Generally the MTU is 1,500 bytes (the Ethernet payload size), so 1,500 bytes is often the upper bound on the segment size.

It's still possible for an IP packet to be fragmented when passing over a network where a link has a small MTU. To avoid this, modern TCP implementations implement **path MTU discovery**. This uses ICMP error messages to find the smallest segment sizes over the network {% cite computer-networks -l 556 %}.

"TCP implements a sliding window protocol with a dynamic window size". When a sender transmit a segment, it starts a timer. When the segment arrives at the destination, the receiving TCP entity sends back a segment (with data if any exist, and otherwise without) bearing an acknowledgement number equal to the next sequence number it expects to receive and the remaining window size. If the sender’s timer goes off before the acknowledgement is received, the sender transmits the segment again" {% cite computer-networks -l 556 %}.

### Segment header

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |          Source Port          |       Destination Port        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                        Sequence Number                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Acknowledgment Number                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Data |  Res- |C|E|U|A|P|R|S|F|                               |
   | Offset| erved |W|C|R|C|S|S|Y|I|            Window             |
   |       |       |R|E|G|K|H|T|N|N|                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Checksum            |         Urgent Pointer        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                             Data                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

{% cite rfc793 -l 14 computer-networks -l 557 %}

The _Source Port_ and _Destination Port_ fields specify the TCP ports of the connection endpoints. A TCP port and an IP address identify a connection.

The _Sequence Number_ field is the sequence number of the first data octet in the segment, except when SYN is present. "If SYN is present the sequence number is the initial sequence number (ISN) and the first data octet is ISN+1" {% cite rfc793 -l 16 %}.

If the ACK bit is set, the _Acknowledgement Number_ field contains "the value of the next sequence number the sender of the segment is expecting to receive" {% cite rfc793 -l 16 %}.

The _Data Offset_ field is the number of 32-bit words in the header, used to calculate where the data begins.

The control bits:

- CWR: Congestion Window Reduced field.
- ECE: Used to echo back the congestion indication.
- URG: Urgent Pointer field.
- ACK: Acknowledgment field.
- PSH: Push Function.
- RST: Reset the connection.
- SYN: Synchronize sequence numbers.
- FIN: No more data from sender.

{% cite rfc793 -l 16 %}

The _Window_ field is "the number of octets beginning with the one indicated in the acknowledgment field which the sender of this segment is willing to accept". The windowing mechanism is explained in the [TCP Sliding Window section](#tcp-sliding-window).

_Checksum_ contains a checksum that's used to improve reliability.

The _Options_ field can contain extra options that aren't covered by the header, for example the MSS (Maximum Segment Size) option. Another option is SACK (Selective ACKnowledgement), which is covered later in the notes {% cite computer-networks -l 559-60 %}.

## TCP Connection Establishment

TCP connections are established in a three-way-handshake.

One server must be listening for connections. The server listens for incoming connections by executing the LISTEN and ACCEPT primitives {% cite computer-networks -l 560 %}.

To establish a connection, the client executes a CONNECT primitive with the IP address and port number of the server, and the maximum segment size it will accept. The CONNECT primitive sends a TCP segment with the SYN bit on and the ACK bit off and waits for a response {% cite computer-networks -l 560 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/internet/tcp/tcp-connection-established.svg" alt="">
  <figcaption><h4>Figure: TCP connection established {% cite computer-networks -l 561 %} </h4></figcaption>
</figure>

When the segment arrives at the server, the TCP entity checks to see if there is a process that is listening on the port. If there isn’t, it sends a response with the RST bit on to reject the connection {% cite computer-networks -l 560 %}.

If a process is listening on the port, the process is given the incoming TCP segment. "It can either accept or reject the connection. If it accepts, an acknowledgement segment is sent back" {% cite computer-networks -l 560 %}.

## TCP connection release

Each TCP connection is released independently. To release a connection, either side of the connection can send a TCP segment with the FIN bit set. When the FIN bit is acknowledged, the direction is closed down for new data. "When both directions Have been shut down, the connection is released" {% cite computer-networks -l 562 %}.

If there is no response to a FIN within two maximum packet lifetimes, the FIN sender releases the connection. The other side will eventually notice that nobody is listening, and will time out {% cite computer-networks -l 562 %}.

## TCP connection management modeling

The steps of a establishing and releasing a TCP connection can be represented as a finite state machine with 11 states.

| State       | Description                                       |
| ----------- | ------------------------------------------------- |
| CLOSED      | No connection is active or pending.               |
| LISTEN      | The server is waiting for an incoming call.       |
| SYN RCVD    | A connection request has arrived; wait for ACK.   |
| SYN SENT    | The application has started to open a connection. |
| ESTABLISHED | The normal data transfer state.                   |
| FIN WAIT 1  | The application has said it is finished.          |
| FIN WAIT 2  | The other side has agreed to release.             |
| TIME WAIT   | Wait for all packets to die off.                  |
| CLOSING     | Both sides have tried to close simultaneously.    |
| CLOSE WAIT  | The other side has initiated a release.           |
| LAST ACK    | Wait for all packets to die off.                  |

{% cite computer-networks -l 563 %}

## TCP sliding window

TCP sliding window is a method to control how much data is sent by a sender to a receiver. It's used to avoid situations where data is sent too quickly to be processed by the receiver {% cite rfc793 -l 4 %}.

The _Window_ value is the number of octets that can be sent since the last acknowledged segment.

When the _Window_ value is `0`, the sender can't send any data. There are two exceptions:

1. Data can be sent if it's urgent.
2. A 1-byte segment can be sent to "force the receiver to reannounce the next byte expected and the window size" (a technique known as **TCP window probe**) {% cite computer-networks -l 566 %}.

## TCP timer management

TCP uses timers to do part of its work. These include:

- RTO timer
- Persistence timer

The RTO (Retransmission TimeOut) timer is used for retransmission. Each time a segment is sent, a retransmission timer is started. If the timer completes before the segment has been acknowledged, the segment is retransmitted. The timer value is calculated dynamically, and adjusted according to current network conditions {% cite computer-networks -l 568-9 %}.

The persistence timer is used to avoid deadlock when a receiver sends an acknowledgement with a window of `0`, but when the receiver updates the window, the message is lost. When the persistance timer runs, the sender transmits a window probe to ensure it has up-to-date window information {% cite computer-networks -l 571 %}.

## TCP congestion control

TCP is the main network protocol in the Internet stack that's responsible for avoiding network congestion (IP simply drops packets).

TCP uses the window mechanism to control the data flow, and packet loss as the signal that there's congestion. It maintains a congestion window, separate from the current window value. The congestion window size is the number of bytes that the sender can have in flight at a time. TCP stops sending traffic if either the congestion window or the window are full {% cite computer-networks -l 571-2 %}.

The initial congestion window is set to a small value of at most 4 segments. Each time an acknowledgement is received, the sender sends 1 extra segments worth of bytes. This algorithm is called **slow start**, and it exponentially increases the amount of data sent {% cite computer-networks -l 574 %}.

The sender keeps a slow start threshold for the connection. This is set high initially, so that it doesn't limit the connection. When packet loss is detected, the slow start threshold is set to half the congestion window. When the slow start threshold is reached, TCP switches to additive increase. The congestion window is increased by 1 segment each round trip time {% cite computer-networks -l 576 %}.

Instead of waiting for the retransmission timeout to detect packet loss, TCP also uses out-of-order packets as a signal that packets have been lost. When a receiver receives out of order packets, it sends duplicate acknowledgements to the sender so that the sender can retransmit the packet, which is presumed to be lost. This heuristic is known as **fast retransmission** {% cite computer-networks -l 577 %}.

## References

{% bibliography --cited_in_order %}
