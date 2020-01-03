---
layout: default
title: Bluetooth
description: Notes on the Bluetooth protocol.
has_children: true
has_toc: false
nav_order: 5
parent: Computer networking
permalink: /computer-networking/bluetooth
---

<!-- prettier-ignore-start -->

# Bluetooth
{:.no_toc}

Bluetooth is a wireless standard for connecting computing devices using short-range, low-power, wireless radios. This section focusses on the architecture of the Bluetooth protocol.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

Siemens began Bluetooth development in 1994. In 1998, Siemens formed the Bluetooth Special Interest Group, which continues to develop the Bluetooth standards {% cite computer-networks -l 320 %}.

Bluetooth lets devices discover and connect to each other (by **pairing**), and then securely transfer data {% cite computer-networks -l 320 %}.

Since Bluetooth 3.0, Bluetooth can be used for pairing in combination with 802.11 for high-throughput data transfer {% cite computer-networks -l 320 %}.

## Bluetooth architecture

The basic unit of a Bluetooth system is called a **piconet**. a piconet consists of a master node, and up to 7 active slave nodes within a distance of 10 meters {% cite computer-networks -l 320 %}.

Multiple piconets can exist in the same room, and they can be connected through a bridge node that takes part in multiple piconets. An interconnected collection of piconets is called a **scatternet** {% cite computer-networks -l 320 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/bluetooth/piconets.svg" alt="">
  <figcaption><h4>Figure: Two piconets connected to form a scatternet {% cite computer-networks -l 321 %}</h4></figcaption>
</figure>

There can be up to 255 **parked nodes** on a piconet. Parked nodes are devices that the master has switched to a low-power state. In parked state a device can't do anything except for respond to an activation or beacon signal from the master {% cite computer-networks -l 320-1 %}.

The reason for the master/slave architecture was the aim to implement complete bluetooth chips for under \$5. Because of this, slaves are relatively dumb {% cite computer-networks -l 321 %}.

A piconet is a centralized TDM (Time-Division Multiplexing) system. The master controls the clock and determines which devices get to communicate in which time slot. All communication is between the master and slave, direct slave to slave communication is not possible {% cite computer-networks -l 321 %}.

## Bluetooth applications

The Bluetooth spec defines specific supported applications, with different stacks for each application. There are over 25 applications, which are called **profiles** {% cite computer-networks -l 321 %}.

Six of the profiles are for audio and video. For example, the headset and handsfree profiles both provide voice communication between a headset and its base station {% cite computer-networks -l 321 %}.

Some profiles are used as building blocks for other profiles. The generic access profile provides a way to establish and maintain secure links between the master and slaves {% cite computer-networks -l 322 %}.

## The Bluetooth protocol stack

Bluetooth has many protocols grouped roughly into layers.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/bluetooth/protocol-architecture.svg" alt="">
  <figcaption><h4>Figure: The Bluetooth protocol architecture {% cite computer-networks -l 323 %}</h4></figcaption>
</figure>

The bottom layer is the **radio layer**, it deals with radio transmission and modulation {% cite computer-networks -l 322 %}.

The **link control layer** deals with how the master controls time slots, and how time slots are grouped into frames. "The link manager handles the establishment of logical channels between devices".

The link manager is below the **host-controller interface line**. Generally, protocols beneath the line are implemented on the Bluetooth chip, and protocols above the line are implemented on the Bluetooth device {% cite computer-networks -l 322-3 %}.

**L2CAP** (Logical Link Control Adaptation Protocol) is above the host-controller interface line. "It frames variable-length messages and provides reliability if needed" {% cite computer-networks -l 323 %}.

"The service discovery protocol is used to locate services within the network. The RFcomm (Radio Frequency communication) protocol emulates the standard serial port found on PCs for connecting the keyboard, mouse, and modem, among other devices" {% cite computer-networks -l 323 %}.

The top layer is the **application layer**. In the figure, profiles are represented as vertical boxes because they define a slice of the protocol stack. They may omit protcols that aren't needed by the profile, for example profiles might skip L2CAP if they only send a steady stream of audio samples {% cite computer-networks -l 323 %}.

## The Bluetooth radio layer

"The radio layer moves the bits from master to slave, or vice versa" {% cite computer-networks -l 324 %}.

"The radio layer is a low-power system with a range of 10 meters operating in the same 2.4-GHz ISM band as 802.11". The band is divided into 79 1MHz channels {% cite computer-networks -l 324 %}.

**Frequency hopping spread spectrum** is used in order to coexist with other networks. "There can be up to 1600 hops/sec over slots with a dwell time of 625 μsec. All the nodes in a piconet hop frequencies simultaneously, following the slot timing and pseudorandom hop sequence dictated by the master" {% cite computer-networks -l 324 %}.

Bluetooth adapts its hop sequence to exclude channels on which there are other RF signals. This is called **adaptive frequency hopping**.

There are "three forms of modulation are used to send bits on a channel. The basic scheme is to use frequency shift keying to send a 1-bit symbol every microsecond, giving a gross data rate of 1 Mb/s. Enhanced rates were introduced with the 2.0 version of Bluetooth. These rates use phase shift keying to send either 2 or 3 bits per symbol, for gross data rates of 2 or 3 Mb/s. The enhanced rates are only used in the data portion of frames" {% cite computer-networks -l 324 %}.

## The Bluetooth link layers

The **link control** (baseband) layer turns bit streams into frames.

In its simplest form, the master in each piconet defines a series of 625-μsec time slots. The master gets half the slots, the slave gets the other half. Master transmission starts in the even slots, slave transmission starts in the odd slots {% cite computer-networks -l 324 %}.

A frame can be 1, 3, or 5 slots long. Each frame has a 126-bit overhead for an access code and header, as well as a 250-60-μsec settling time {% cite computer-networks -l 324 %}.

The frame payload can be encrypted with a key chosen when the master and slave connect {% cite computer-networks -l 324 %}.

Hops can only occur between frames, not during a frame.

The link manager sets up channels (**links**) to carry frames between a master and a slave device that have discovered each other {% cite computer-networks -l 324 %}.

The devices follow a pairing procedure before the link is used. The **simple secure pairing** method allows users to confirm both devices are displaying the same passkey, or to manually enter a passkey from one device into another. Devices with limited input output like headphones can't use this method {% cite computer-networks -l 325 %}.

When pairing is complete, the link manager sets up the links. There are two main kinds of link:

1. SCO (Synchronous Connection Oriented)
2. ACL (Asynchronous ConnectionLess)

**SCO** is used for real-time data, like telephone connection. An SCO link is allocated a fixed slot in each direction. Each SCO link can transmit one 64,000-b/s PCM audio channel, and a slave can have up to three SCO links with its master. Frames sent over SCO are never retransmitted {% cite computer-networks -l 325 %}.

An **ACL** link is used for packet-switched data available at irregular intervals. ACL frames can be lost, and may need to be retransmitted. A slave can only have one SCO link to its master {% cite computer-networks -l 325 %}.

"The data sent over ACL links come from the L2CAP layer". The L2CAP has three main functions:

- Accepts packets (up to 64KB) from higher layers and breaks them into frames for transmission, the frames are reassembled at the receiving end.
- Handles multiplexing and demultiplexing of multiple packet sources.
- Handles error control and retransmission.

{% cite computer-networks -l 325 %}

## Bluetooth frame structure

The Bluetooth standard defines several frame formats. The two frames that are studied in this section are typical Bluetooth frames at basic and enhanced data rates.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/bluetooth/bluetooth-frame.svg" alt="">
  <figcaption><h4>Figure: Typical Bluetooth data frame at basic and enhanced rates {% cite computer-networks -l 326 %}</h4></figcaption>
</figure>

The first field is an access code that usually identifies the master.

Next is a 54-bit header that contains several fields. The Address field identifies which of the active devices the frame is intended for. The Type field identifies the frame type (ACL, SCO, poll, or null), the type of error correction used in the data field, and how many slots long the frame is {% cite computer-networks -l 325-6 %}.

The Flow bit (_F_) is set by a slave when its buffer is full and it can't receive any more data. The Acknowledgement bit (_A_) adds an ACK to the frame, The Sequence bit (_S_) is used to number frames, to enable retransmission. Since the protocol is stop-and-wait, 1 bit is enough to number frames adequately {% cite computer-networks -l 326 %}.

The header is repeated three times. This is to add redundancy. Redundancy is important in Bluetooth, which runs in a noisy environment over low-powered devices {% cite computer-networks -l 326 %}.

There are various formats that are used for the data field. The basic-rate SCO frames are a simple example. The data field is always 240 bits. There are three variants defined that set either 80, 160, or 240 bits of actual payload. The rest are used for error correction. In the 80-bit version, the bits are repeated three times for redundancy {% cite computer-networks -l 326 %}.

## References

{% bibliography --cited_in_order %}
