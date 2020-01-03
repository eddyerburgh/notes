---
layout: default
title: Ethernet
description: Notes on the Ethernet protocol.
nav_order: 1
has_toc: false
parent: Computer networking
permalink: /computer-networking/ethernet
---

<!-- prettier-ignore-start -->

# Ethernet
{:.no_toc}

Ethernet is a standard for sending data from one computer to another. It's used to create LANs that higher-layer protocols, like IP, are built on top of.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

The Ethernet standard describes both the physical medium that data is sent over, and the format of the data. The standard is created by the 802.3 IEEE committee.

Ethernet can run over:

- Coaxial cable
- Twisted pair cable
- Optical fiber

When IEEE need to add a new media system or capability to Ethernet, they develop the new standard as a supplement. When a supplement is voted in by the IEEE members, it becomes part of the full standard {% cite ethernet-definitive-guide -l 13 %}.

Ethernet covers layer 1 (the physical layer) and layer 2 (the data link layer) of the OSI model. Within those two layers, Ethernet is further divided into sublayers defined in the Ethernet protocol {% cite ethernet-definitive-guide -l 17-8 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/ethernet/ethernet-sublayers.svg" alt="">
  <figcaption><h4>Figure: The major IEEE sublayers {% cite ethernet-definitive-guide -l 19 %}</h4></figcaption>
</figure>

At the physical layer, there are IEEE sublayers that are specific to the media and speed of the Ethernet it is standardizing {% cite ethernet-definitive-guide -l 19 %}.

The MAC protocol at the data link layer is independent of the physical media that's being specified {% cite ethernet-definitive-guide -l 19 %}.

The IEEE LLC standard is independent of the IEEE 802.3 Ethernet standard. The LLC fields can be used by any LAN system, not just Ethernet. The layers below LLC are specific to Ethernet {% cite ethernet-definitive-guide -l 19 %}.

Ethernet has four main parts:

- The frame
- The Media Access Control sublayer (MAC)
- The signalling components
- The physical medium

{% cite ethernet-definitive-guide -l 27-8 %}

These all work together to convert bits into signals that are transmitted over a medium from one station to another.

## The Ethernet frame

Frames contain data (like TCP packets) and metadata. Ethernet stations send frames continuously, with a small break between each frame (or by sending a symbol that indicates the start and end of a frame), until all the data is transmitted {% cite ethernet-definitive-guide -l 53 %}.

There are three sizes of frame defined in the Ethernet standard. An ethernet interface must support at least one of them. The standard recommends using the recent **envelope frame**, the other two frames are **basic frames** {% cite ethernet-definitive-guide -l 44 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/ethernet/ethernet-frames.svg" alt="">
  <figcaption><h4>Figure: Ethernet frames {% cite ethernet-definitive-guide -l 28 %}</h4></figcaption>
</figure>

The _Preamble_ field is a predetermined signal which is used to sync the receiving Ethernet station’s clock to the sending device for 10Mb/s Ethernet. Newer standards no longer use the preamble, but it's maintained for backward compatibility {% cite ethernet-definitive-guide -l 28 %}.

In the 802.3 spec, the preamble field is normally divided into two parts. The first 7 bytes are the preamble, and one byte is called the start frame delimiter (_SFD_) {% cite ethernet-definitive-guide -l 46 %}.

The _Length/Type_ field either provides the length of the data field (if the value is less than `1536`), or defines the protocol of the data in the frame (like IPv4) {% cite ethernet-definitive-guide -l 29 %}.

The _Data_ field contains the payload that is being sent.

The _Frame Check Sequence_ field contains a CRC (Cyclic Redundancy Check) that is used by the receiving station to verify that the data in the incoming frame isn't corrupted. The receiving station generates another CRC and compares it to the incoming CRC. If the generated value doesn’t match the received frame value, the frame will be dropped {% cite ethernet-definitive-guide -l 29 %}.

The _Envelope Prefix_ and _Envelope Suffix_ are intended as 482 bytes that can be used by other higher layer protocols to tag Ethernet frames. The contents aren't defined in 802.3 {% cite ethernet-definitive-guide -l 50 %}.

The _Source address_ and _Destination address_ fields are used to send the frame to the correct device, and for the receiving device to send back frames to the original sender. The address is known as a _MAC address_ {% cite ethernet-definitive-guide -l 28 %}.

A MAC address is a 48-bit structure that should uniquely identify an Ethernet station. Whereas an IP address is dynamically allocated, MAC addresses are physically assigned to an Ethernet station by the manufacturer. The MAC address is normally hard-coded into ROM on a NIC (Network Interface Card), but it’s possible to spoof or change the value in the OS {% cite intro-computer-networks -l 23 %}.

## The MAC sublayer

The MAC (Media Access Control) sublayer "defines the protocols used to arbitrate access to the Ethernet systems" {% cite ethernet-definitive-guide -l 18 %}.

Each Ethernet device operates independently of other stations on the network. There's no central controller.

Ethernet is broadcast. Each device on the channel receives each signal transmission. To send in half duplex mode, a station listens to the channel and transmits data as frames if the channel is idle {% cite ethernet-definitive-guide -l 30 %}.

The shared signal channel is called the **medium**. As bits are transmitted, each device connected to the channel reads the bits and looks at the destination address field of the frames. If the destination address matches the device's own unicast MAC address (or an assigned multicast address), the frame is read and delivered to the networking software running on the computer. If the address does not match, the device will stop reading the frame {% cite ethernet-definitive-guide -l 31 %}.

Stations attached to a shared Ethernet cable operating in half duplex mode use CSMA/CD to avoid collisions {% cite ethernet-definitive-guide -l 30 %}.

### The CSMA/CD protocol

The CSMA/CD (Carrier Sense Multiple Access / Collision Detection) protocol is designed for detecting and avoiding collisions. CSMA/CD is used by Ethernet operating at half duplex.

There are two modes of Ethernet: **full duplex** and **half duplex**. Full duplex Ethernet can both send and receive at the same time. Half duplex Ethernet can communicate in both directions, but not at the same time.

The Carrier Sense portion of the protocol means that each interface must wait until the shared channel is idle before it can begin transmitting. If another interface is signalling, the shared channel will not be idle. This condition is called **carrier**. Interfaces must wait until the carrier has ceased before transmitting, this process is called **deferral** {% cite ethernet-definitive-guide -l 32 %}.

The Multiple Access part of the protocol means that each interface connected to the cable has equal priority when transmitting on the cable, and all interfaces can attempt to access the channel when it's idle {% cite ethernet-definitive-guide -l 132 %}.

Collision Detection is required because it's possible that multiple interfaces sense that the channel is idle and attempt to transmit at the same time. When this happens, the transmitting devices detect the collision of signals, and they stop signalling. Each interface then picks a random retransmission time. This process is known as **backoff** {% cite ethernet-definitive-guide -l 32 %}.

## The signalling components

The signalling components are the electronics that implement the ability to transmit and receive data.

The components used depends on the physical medium. For example, the signalling components for a twisted-pair system include:

- An Ethernet interface
- A transceiver
- A twisted-pair cable

The Ethernet interface connects to the media system using a transceiver. A transceiver includes the electronics that "take signals from the station interface and transmit them to the twisted-pair cable segment", and can receive signals from the cable segment and send them to the Ethernet interface {% cite ethernet-definitive-guide -l 34 %}.

The signalling system is always active on Ethernet that doesn't implement energy efficient Ethernet. IDLE symbols are sent continuously between connected stations {% cite ethernet-definitive-guide -l 117 %}.

The media components include the cables and signalling components that create the signal-carrying portion of an Ethernet channel {% cite ethernet-definitive-guide -l 35 %}.

## Physical medium

The physical medium is both the link cable that connects stations, and the hardware used to interface with the cable.

The link could be one of:

- Coaxial cable
- Twisted pair
- Optical fiber

**Coaxial cable** is a copper wire surrounded by insulator and a shield wire.

**Twisted pair** cable is made from pairs of twisted cable. There are lots of cable standards (like CAT 3, CAT 5, CAT 6, etc.). Twisted pair cable can handle multiple wires in a single cable by twisting the wires together to reduce electromagnetic interference. Because twisted pair has multiple wires, it can have multiple channels.

**Optical fiber** works by emitting light down a glass or plastic fiber from either an LED or a laser.

Twisted pair and coaxial cable use electric current to transmit data. Electricity has limitations. First, electricity can suffer from interference from other electrical appliances. Secondly, electricity loses strength quickly over distance, so most Ethernet standards that use coaxial or twisted pair are limited to short distances of around 100 meters or less. Fiber optics can go much further.

Transoceanic cables between continents can be thousands of miles long. In order for the signal to be transmitted across long distances, a transoceanic cable uses repeater components to regenerate the signal.

## Full duplex operation

Full duplex mode was added to the standard in 1997. In full duplex mode both stations can simultaneously transmit and receive, doubling the capacity of the link {% cite ethernet-definitive-guide -l 53 %}.

Recent Ethernet protocols (greater than 100 Mb/s) only support full duplex mode {% cite ethernet-definitive-guide -l 30 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/ethernet/ethernet-full-duplex.svg" alt="">
  <figcaption><h4>Figure: Full duplex operation {% cite ethernet-definitive-guide -l 54 %}</h4></figcaption>
</figure>

Full duplex has some requirements:

- The media system must have independent transmit and receive paths.
- Two stations must be connected point-to-point.
- Both stations must be capable of full duplex operation.

{% cite ethernet-definitive-guide -l 54 %}

Because there is no contention for a shared medium, CMSA/CD is not used {% cite ethernet-definitive-guide -l 54 %}.

In full duplex mode, cables aren't limited in length by the transmission times. The length is set by the signal carrying capacity of the media {% cite ethernet-definitive-guide -l 53 %}.

## Auto-negotiation

The auto-negotiation protocol automatically configures two different Ethernet interfaces. This makes it easier to work with Ethernet stations that support different Ethernet speeds.

In order for auto-negotiation to work, both stations must support the protocol. If auto-negotiation is not supported by one device, the device that does support auto-negotiation will use a process called **parallel detection** to determine the optimal speed that it can send data at.

The auto-negotiation protocol doesn't check the compatibility of the link cable. This can cause problems if a link uses a cable that doesn't support the speed of the connected stations, because the two stations might negotiate a speed which isn't supported. In this case, the user can override the auto-negotiation protocol by manually setting the speed on a station {% cite ethernet-definitive-guide -l 77-8 %}.

Auto-negotiation isn’t supported for most optical fiber links due to the difficulty of transmitting at different speeds. In practice this isn't a problem, because fiber optic links are usually used as backbone links added by experienced network engineers who know how to configure the Ethernet stations that are connected with the link {% cite ethernet-definitive-guide -l 65 %}.

## Switches

A switch device is a computer that retransmits frames to the port that the frame is intended for.

A switch checks the destination address of a frame, and retransmits it to the correct station if it knows the port number of the destination station.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-networking/ethernet/switch-ethernet.svg" alt="">
  <figcaption><h4>Figure: A switched Ethernet network</h4></figcaption>
</figure>

Switches learn the MAC address of devices connected to different ports by looking at the MAC address of frames they receive. They can build up an address table of which port maps to which MAC address {% cite ethernet-definitive-guide -l 303 %}.

The process used to decide where to forward a frame is known as **adaptive filtering** {% cite ethernet-definitive-guide -l 304 %}.

If the switch doesn't know the address, it will send the frame to each connected device, this process is called **flooding** {% cite ethernet-definitive-guide -l 305 %}.

## References

{% bibliography --cited_in_order %}
