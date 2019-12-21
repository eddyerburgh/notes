---
layout: default
title: Modulation and multiplexing
description: Notes on modulation and multiplexing.
has_children: true
has_toc: false
nav_order: 4
parent: Computer networking
permalink: /computer-networking/modulation-and-multiplexing
---

<!-- prettier-ignore-start -->

# Modulation and multiplexing
{:.no_toc}


## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

**Digital modulation** is the process of converting between digital bits and analog signals (such as a continuously varying voltage) {% cite computer-networks -l 125 %}.

Schemes that directly encode bits into signals result in **baseband transmission**, where the signal occupies frequencies from zero up to a maximum that depends on the signalling rate {% cite computer-networks -l 125 %}.

**Passband transmission** is where signals are shifted so that "the signal occupies a band of frequencies around the frequency of the carrier signal." {% cite computer-networks -l 125 %}.

Channels are often shared by multiple signals. Sharing signals is called **multiplexing**. There are several ways to perform multiplexing, including time, frequency, and code division multiplexing.

## Baseband transmission

Baseband transmission is the transmission of a raw signal that occupies frequencies from zero up to a maximum that depends on the signalling rate {% cite computer-networks -l 125 %}.

**NRZ (Non-Return-to-Zero)** is a simple transmission scheme. In NRZ, a positive voltage could represent a 1, and a negative voltage could represent a 0 {% cite computer-networks -l 125 %}.

An NRZ signal is sent down a line, and a receiving station samples the signal at regular intervals to convert the signal back into bits {% cite computer-networks -l 125 %}.

The signal will not look exactly like the signal that was originally sent. It will have been distorted and attenuated by the channel. The receiver must map the received signal to the closest symbols {% cite computer-networks -l 126 %}.

NRZ is normally not used by itself in practice. More complex schemes can convert bits to signals that better meet engineering needs. These schemes are called **line codes** {% cite computer-networks -l 126 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/modulation-and-multiplexing/line-codes.svg" alt="">
  <figcaption><h4>Figure 1: Different line codes {% cite computer-networks -l 126 %}</h4></figcaption>
</figure>

The engineering needs for transmission schemes are:

- Bandwidth Efficiency
- Clock recovery
- Balanced signals

### Bandwidth Efficiency

The NRZ signal can cycle between positive and negative levels every 2 bits at a maximum. This requires a bandwidth of at least B/2, where B is bits/second. This is a fundamental limit that NRZ cannot run faster than (without using more bandwidth) {% cite computer-networks -l 126 %}.

One strategy for using bandwidth more efficiently is to use more than two signalling levels. You could send 2 bits at once as a single **symbol**. This would require four different symbols, and four different levels. This would work as long as the receiver can distinguish the four levels. With this, the rate that the signal changes is half the bit rate, so the required bandwidth is reduced {% cite computer-networks -l 126 %}.

The rate at which the signal changes is called the **signal rate**. This is different from bit rate. The bit rate is $$\text{symbol rate} \times \text{bits per symbol}$$. Another name for symbol rate is the **baud rate** {% cite computer-networks -l 127 %}.

A good scheme maximizes bandwidth efficiency.

### Clock recovery

In order for a receiver to read bits from a signal, it must know when one symbol ends and another begins. In NRZ, a long series of 1s or 0s would leave the signal unchanged, so the only way to tell how many bits were sent would be for the receiver to have a very accurate clock {% cite computer-networks -l 127 %}.

Accurate clocks are too expensive a solution for commodity devices. The alternative is to send a clock signal to the receiver.

Instead of sending a separate signal for the clock, the clock signal can be mixed with the data signal by XORing the two together. The clock makes a transition in every bit time and it runs at twice the bit rate. When the clock is XORed with 0 it makes a low-to-high transition, which is the clock. When it is XORed with 1 it makes a high to low transition, which is the inverse of the clock. This scheme is called **Manchester encoding** (see Figure 1). Manchester encoding requires twice as much bandwidth as NRZ encoding, because of the clock {% cite computer-networks -l 127 %}.

Another approach is to code the data to ensure there are enough transitions in the signal. NRZ will only have a clock problem for long runs of 0s and 1s, so avoiding long runs make it possible for the receiver to stay synchronized {% cite computer-networks -l 127 %}.

One approach is to encode 1 as a transition, and a 0 as no transition. This is called **NRZI (Non-Return-to-Zero Inverted)**. The USB standard uses NRZI. NRZI fixes the problem for long runs of 1s, but long runs of 0s will still cause a problem {% cite computer-networks -l 127-8 %}.

Another approach is to map small groups of bits to be transmitted so that groups with successive 0s are mapped to longer patterns that don't have too many consecutive 0s.

4B/5B is a common code for mappings bits. 4 bits are mapped to a 5 bit pattern. This scheme adds 25% overhead. In 4B/5B there are more patterns than there are 4 bit blocks that are mapped in. This means there a free symbols that can be used by protocols as control symbols, for example an idle symbol {% cite computer-networks -l 128 %}.

| Data (4B) | Codeword (5B) | Data (4B) | Codeword (5B) |
| --------- | ------------- | --------- | ------------- |
| 0000      | 11110         | 1000      | 11110         |
| 0001      | 11110         | 1001      | 11110         |
| 0010      | 11110         | 1010      | 11110         |
| 0011      | 11110         | 1011      | 11110         |
| 0100      | 11110         | 1100      | 11110         |
| 0101      | 11110         | 1101      | 11110         |
| 0110      | 11110         | 1110      | 11110         |
| 0111      | 11110         | 1111      | 11110         |

{% cite computer-networks -l 128 %}

Another approach to avoiding runs of 0s or 1s is to make the data look random by **scrambling**. A scrambler XORs data with a pseudorandom sequence before the data is transmitted. The receiver then XORs the incoming data with the same pseudo random sequence {% cite computer-networks -l 128 %}.

Scrambling doesn't add any bandwidth or time overhead, but scrambling doesn't also doesn't guarantee that there won't be long runs of 0s (although it is unlikely).

### Balanced signals

Balanced signals are signals that have as much positive voltage as negative voltage, even over short periods of time.

Balanced electrical signals have no DC component. This is good for media like coaxial cables, which strongly attenuate a DC component. Thus the aim of a good line code should be a balanced signal {% cite computer-networks -l 129 %}.

One way to balance a signal is to have a logical 1 represented by alternating between two voltage levels (e.g. +1 and -1), and having 0 be represented as 0. This scheme is called **bipolar encoding** {% cite computer-networks -l 129 %}.

Another approach is to use a line code, like 4B/5B. A common code is the 8B/10B line code, which maps bits to balanced symbols. Since there are not enough balanced 10-bit symbols for all 8-bit permutations, not all symbols in in 8B/10B are balanced.

To solve this problem, some input patterns are matched to two symbols: one symbol with an extra 1, and one symbol with an extra 0. In order to keep the signal balanced, the encoder must remember the disparity, next time that it uses an unbalanced symbol, it will choose the symbol that reduces the disparity {% cite computer-networks -l 129-30 %}.

## Passband transmission

You can take a baseband signal that occupies 0 to B Hz and shift it up to occupy a **passband** of S to S+B Hz. At the receiver, the signal can be shifted back down to baseband. This type of transmission is called passband transmission "because an arbitrary band of frequencies is used to pass the signal" {% cite computer-networks -l 130 %}.

With passband transmission, digital modulation is accomplished by modulating a carrier signal that sits in the passband. You can modulate the amplitude, frequency, or phase of the signal {% cite computer-networks -l 130 %}.

In **ASK (Amplitude Shift Keying)**, two different amplitudes are used to represent 0 and 1. Multiple amplitudes can be used to represent more symbols.

In **FSK (Frequency Shift Keying)**, two or more different frequencies are used to represent symbols {% cite computer-networks -l 130 %}.

In **PSK (Phase Shift Keying)** the carrier wave is shifted 0 or 180 degrees each time time a symbol changes. Because there are two phases, it's sometimes called **BPSK (Binary Phase Shift Keying)** {% cite computer-networks -l 130 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/modulation-and-multiplexing/modulation-schemes.svg" alt="">
  <figcaption><h4>Figure: Modulation schemes {% cite computer-networks -l 131 %}</h4></figcaption>
</figure>

These schemes can be combined together, although frequency and phase cannot. Normally it is amplitude and phase that are used in combination {% cite computer-networks -l 131 %}.

## Frequency division multiplexing

**FDM (Frequency Division Multiplexing)** uses passband transmission to share a channel.

FDM divides the spectrum into **frequency bands**. Each user has exclusive possession of a band that they can use to send their signal {% cite computer-networks -l 133 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/modulation-and-multiplexing/frequency-division-multiplexing.svg" alt="">
  <figcaption><h4>Figure: Frequency division multiplexing {% cite computer-networks -l 134 %}</h4></figcaption>
</figure>

A **guard band** keeps the channels separated {% cite computer-networks -l 133 %}.

### OFDM

Digital data can be divided without using guard bands. In **OFDM (Orthogonal Frequency Division Multiplexing)** the channel bandwidth is divided into subcarriers, which independently send data.

Signals from each carrier extend into adjacent channels. However, "the frequency response of each subcarrier is designed so that 0 is at the center of the adjacent subcarriers". So subcarriers can be sampled from their center frequencies, without any interference from their neighbors {% cite computer-networks -l 133-4 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/modulation-and-multiplexing/ofdm.svg" alt="">
  <figcaption><h4>Figure: Orthogonal frequency division multiplexing (OFDM) {% cite computer-networks -l 134 %}</h4></figcaption>
</figure>

## Time division multiplexing

**TDM (Time Division Multiplexing)** works by enabling users full access to the bandwidth for short periods of time {% cite computer-networks -l 135 %}.

In TDM, users take turns round-robin style to get the entire bandwidth. Bits from each input are taken on a fixed time slot, and then output to the aggregate stream. The stream runs at the sum of each individual stream {% cite computer-networks -l 135 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/modulation-and-multiplexing/time-division-multiplexing.svg" alt="">
  <figcaption><h4>Figure: Time division multiplexing (TDM) {% cite computer-networks -l 135 %}</h4></figcaption>
</figure>

Guard time must be added to accommodate timing variations {% cite computer-networks -l 135 %}.

TDM is used in cellular and telephone networks {% cite computer-networks -l 135 %}.

## Code division multiplexing

**CDM (Code Division Multiplexing)** is a form of **spread spectrum** communication, where a narrowband signal is spread over a wider frequency band. This allows multiple users to share the same frequency band {% cite computer-networks -l 135 %}.

CDM is commonly called **CDMA (Code Division Multiple Access)**. CDMA allows stations to transmit across the entire frequency spectrum all the time. The multiple simultaneous transmissions are separated using coding theory. For CDMA to work, the receiving station must be able to extract desired signals {% cite computer-networks -l 136 %}.

In CDMA, each bit time is divided into $$n$$ short intervals (known as chips). Normally there are 64 or 128 chips per bit {% cite computer-networks -l 136 %}.

To transmit a 1, a station sends its chip sequence. To transmit a 0, a station sends the negation of its chip sequence. When multiple stations transmit at the same time, their bipolar sequences add linearly. For example, if during 1 bit period, 3 stations output +1 and 1 station outputs -1, +2 would be received {% cite computer-networks -l 136 %}.

The receiving station recovers the original code by computing the normalized inner product of the received chip sequence and the chip sequence of the station whose bit stream it is trying to recover. "If the received chip sequence is $$S$$ and the receiver is trying to listen to a station whose chip sequence is $$C$$, it just computes the normalized inner product, $$S \times C$$" {% cite computer-networks -l 137 %}.

This works because pairs of chip sequences are orthogonal (meaning that they cancel each other out). To learn more, watch this [video about how CDMA works](https://www.youtube.com/watch?v=XJ81CuujwYE).

## References

{% bibliography --cited_in_order %}
