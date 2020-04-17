---
layout: default
title: Coordination
description: Notes on coordination in distributed systems.
has_children: true
has_toc: false
nav_order: 5
parent: Distributed systems
permalink: /distributed-systems/coordination
---

<!-- prettier-ignore-start -->

# Coordination
{:.no_toc}

Coordination is one of the major challenges of distributed systems.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

<!-- Coordination is how processes coordinate and synchronize with each other over an unreliable network 298. -->

There are several ways that processes must coordinate and synchronize with each other:

1. To ensure one process waits for another to finish its operation (process synchronization).
2. To ensure two sets of data are the same (data synchronization).
3. To manage the interactions and dependencies between activities in a distributed system (coordination).

297

## Clock synchronization

Clock synchronization is not a problem in centralized systems, since all processes share the same clock. In a distributed system, clock synchronization is much more difficult, since each process will have different hardware clocks on their respective machines 298.

One of the problems with hardware clocks is that they can drift over time (300).

With multiple computers, each with their own clock, the difference in time between the computers is known as clock skew 300.

On a single computer, only relative time is important for normal operation. In a distributed system, it becomes more important for there to be a universal time (e.g., for logging).

UTC (Universal Coordinated Time) is the basis for modern timekeeping 302. There are shortwave radio stations around the world tat broadcast a pulse at the start of each UTC second 302. There are also several satellites that offer a UTC service 302.

A UTC receiver is hardware that can receive UTC from satellites. Other computers can then sync to this computer (needs reference).

<!-- Continue p303-->

There are many algorithms to synchronize several computers to one machine with a UTC receiver, or for keeping a set of machines as synchronized as possible (302).

There are two important concepts—precision and accuracy. **Precision** refers to the deviation between two clocks. **Accuracy** refers to the precision in reference to an external reference point, like UTC 303 (CHECK).

Internal clock synchronization is the process of keeping clocks precise. External clock synchronization is the process of making clocks accurate 303.

Hardware clocks suffer from clock drift. The clock drift rate is the DEFINE 303. An average quartz-based hardware clock has a clock drift rate of around 31.5 seconds per year 303.

## NTP

NTP (Network Time Protocol) is a protocol for synchronizing clocks over a network 304.

The main challenge of NTP is that time will be outdated by the time a packet arrives. The solution is to estimate the packet delay 304-5.

When a clock is later than the actual time, the clock must be readjusted. This can't be done by simply setting the time to the earlier correct time, since this would cause problems with programs that rely on there being an ever increasing time. The time change is gradually introduced. One way is to change the number of milliseconds that the computer time is set to on each tick. for example, instead of increasing the computer time by 10ms on each tick, the time could be increased by 9ms until it has caught up to the time it is synchronizing with.

NTP runs between two servers. Both servers probe each other for their current time. The time offset and the packet delay between the two servers is calculated 305.

NTP divides servers into strata. A server with a reference clock is known as a stratum-1 server. When server A contacts server B, it will only adjust its time if its stratum level is higher than server B 306.

Once A has syncronized, it will change its strata level to `stratum(B) + 1` 306.

## The Berkley algorithm

The Berkley algorithm is an internal clock synchronization algorithm. It works by having a time daemon poll other machines periodically to get the time on that machine. It then computes an average time using the answers and tells other machines to advance/slow their clocks to the new time 306.

The Berkley algorithm works well for systems that don't have a UTC receiver 306. It is often sufficient for machines to simply agree on a time with each other, rather than to have the accurate real time 306.

## Logical clocks

Logical clocks are a mechanism for measuring the order of events in distributed systems 310.

<!-- TODO: continue 311 -->
