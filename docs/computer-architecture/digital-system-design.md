---
layout: default
title: Digital system design
description: Notes on digital system design.
nav_order: 1
parent: Computer architecture
permalink: /computer-architecture/digital-system-design
---

<!-- prettier-ignore-start -->

# Digital system design
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<style>
/* Reset tables because truth tables look bad at 100% width */
.table-wrapper { 
  box-shadow: initial!important;
} 

table { 
  min-width: initial!important;
}
</style>

<!-- prettier-ignore-end -->

## Introduction

Digital systems represent information using discrete signals, typically bits. This is opposed to analogue systems, which represent information using continuous signals.

The main reason for using binary is that it's simple to represent 0 and 1 as high/low voltage (which also makes binary systems tolerant to noise).

As of 2020, microprocessors are some of the most complicated digital systems and so this section focusses on microprocessors.

A microprocessor is made up of multiple subcomponents (e.g. MMU, an instruction sequencer, caches). These subcomponents are made of the same building blocks: wires and transistors. Transistors are joined together to make larger abstractions (e.g. logic gates and flip-flops) that are used to build increasingly complex circuits {% cite cs61c-sds-notes -l 2-3 %}.

An integrated circuit (chip) is a set of electronic circuits on a single piece of semiconductor material (normally silicone). A chip contains a number of connections to the outside world, many of which are connected to a low-voltage DC power supply (usually between 1-5 volts). The energy provided by the power is what drives the processor as it moves electric charge from one place to another. In the process energy is dissipated as heat {% cite cs61c-sds-notes -l 3 %}.

A clock signal that drives the processor is generated on the motherboard and then sent to the chip {% cite cs61c-sds-notes -l 4 %}.

<!-- TODO: ## Transistors -->

## Circuits

Circuits can be broken into two types:

- Combinational logic (CL) circuits
- Sequential logic (SL) circuits

CL circuits take inputs and combine them to produce a single output after a small propagation delay. The output is only a function of the inputs (they are pure functions) {% cite cs61c-sds-notes -l 8 %}.

SL circuits have state—the output depends on the current input and memory values. Registers are an example of SL circuits.

Most digital systems boil down to registers separating combinational logic. Registers hold inputs stable for the clock cycle, once the CL circuit evaluates a new value, it's captured in a register. After the clock edge, the register sends out the new value on an output wire or bus.

## Logic gates

A **logic gate** is a circuit with one or more input signals and only one output signal {% cite digital-computer-electronics -l 19 %}.

Logic gates are normally restricted to a maximum of four input signals for performance reasons {% cite cs61c-cl-notes -l 4 %}.

Logic gates are the building blocks for more complex digital logic circuits. Normally only a subset of logic gates are used to create a processor's circuits {% cite cs61c-cl-notes -l 5 %}.

NOR and NAND gates are known as universal gates because they can be used to build any other logic function without another gate type {% cite P&H:ARM -l A-8 %}.

### Inverters and buffers

An **inverter** (NOT gate) is a gate with one input signal that outputs the opposite signal {% cite digital-computer-electronics -l 19 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/digital-system-design/not-gate.svg" alt="">
  <figcaption><h4>Figure: Inverter</h4></figcaption>
</figure>

A small circle (bubble) on the output side signals an inverter {% cite digital-computer-electronics -l 19-20 %}.

A **buffer** is a circuit that takes an input signal and outputs the same signal. A buffer can be constructed by cascading two inverters. Buffers can be used for isolating two different circuits {% cite digital-computer-electronics -l 20 %}.

### OR gates

**OR gates** take two or more inputs and return high if any of the inputs is high, or low if none are high {% cite digital-computer-electronics -l 20 %}.

<!-- TODO: Add image from p 21 -->

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/digital-system-design/or-gate.svg" alt="">
  <figcaption><h4>Figure: OR gate</h4></figcaption>
</figure>

| A   | B   | Z   |
| --- | --- | --- |
| 1   | 1   | 1   |
| 1   | 0   | 1   |
| 0   | 1   | 1   |
| 0   | 0   | 0   |

### AND gates

**AND gates** take two or more inputs and return high if all of the inputs is high, or low otherwise {% cite digital-computer-electronics -l 22 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/digital-system-design/and-gate.svg" alt="">
  <figcaption><h4>Figure: AND gate</h4></figcaption>
</figure>

| A   | B   | Z   |
| --- | --- | --- |
| 1   | 1   | 1   |
| 1   | 0   | 0   |
| 0   | 1   | 0   |
| 0   | 0   | 0   |

### NOR gates

**NOR gates** have two or more inputs. The output is high if all inputs are low, otherwise the output is low {% cite digital-computer-electronics -l 32 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/digital-system-design/nor-gate.svg" alt="">
  <figcaption><h4>Figure: OR gate</h4></figcaption>
</figure>

| A   | B   | Z   |
| --- | --- | --- |
| 1   | 1   | 0   |
| 1   | 0   | 0   |
| 0   | 1   | 0   |
| 0   | 0   | 1   |

### NAND gates

**NAND gates** have two or more input signals. The output is low if all input signals are high, otherwise the output is high {% cite digital-computer-electronics -l 34 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/digital-system-design/nand-gate.svg" alt="">
  <figcaption><h4>Figure: NAND gate</h4></figcaption>
</figure>

| A   | B   | Z   |
| --- | --- | --- |
| 1   | 1   | 0   |
| 1   | 0   | 1   |
| 0   | 1   | 1   |
| 0   | 0   | 1   |

### XOR gates

**XOR gates** (exclusive-or) have two or more inputs. For the binary case, the output is high if only one input is high otherwise it is low {% cite digital-computer-electronics -l 37 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/digital-system-design/xor-gate.svg" alt="">
  <figcaption><h4>Figure: NAND gate</h4></figcaption>
</figure>

In the general case, an XOR gate output is high if an odd number of inputs are high and low otherwise {% cite digital-computer-electronics -l 37 %}.

| A   | B   | Z   |
| --- | --- | --- |
| 1   | 1   | 0   |
| 1   | 0   | 1   |
| 0   | 1   | 1   |
| 0   | 0   | 0   |

## Multiplexers

A **multiplexer** (mux) is a circuit with multiple inputs and only one output {% cite digital-computer-electronics -l 58 %}.

When a multiplexor has only two data inputs, the selector is a single bit that selects one of the inputs to be the output if it is true and the other if it is false {% cite P&H:ARM -l A-10 %}.

With $$n$$ data inputs there must be $$\vert log_2 n \vert$$ selector inputs {% cite P&H:ARM -l A-10 %}.

Read ports are often implemented using muxes {% cite P&H:ARM -l A-53 %}.

## Decoders

A **decoder** is a logic block that has an n-bit input and $$2^n$$ outputs. Only one output is asserted for each input {% cite P&H:ARM -l A-9 %}.

A logic component called an encoder performs the opposite action, converting $$2^n$$ input lines into an n-bit output {% cite P&H:ARM -l A-9 %}.

Write ports are often implemented using decoders {% cite P&H:ARM -l A-53 %}.

## Clock

Processors are implemented as **synchronous digital circuits**, where a clock signal is used to regulate the signals throughout the system by controlling when state elements should be updated (as opposed to asynchronous circuits which are clockless or self-timed) {% cite cs61c-sds-notes -l 5 %}.

A clock signal is a square wave signal {% cite digital-computer-electronics -l 93 %}.

A **clock cycle** (tick) is the time from any place on the waveform to the same place on the next instance of the waveform. Normally it's measured from the rising-edge of the signal to the next rising-edge (rising meaning rising from low to high) {% cite cs61c-sds-notes -l 4 %}.

The rate at which the clock runs at is measured in hertz (Hz). Modern processors often have clock speeds measured in gigahertz (GHz).

The clock must have a long-enough period so that all signals in combinational logic blocks stabilize before the clock edge samples those values for storage in the state elements {% cite P&H:ARM -l A-48 %}.

Systems suffer from clock skew where two different state elements receive the clock signal at slightly different times (due to longer paths for one than the other). Designers must minimize clock skew by routing the clock in a way that minimizes difference in arrival time {% cite P&H:ARM -l A-73 %}.

A **clocking methodology** defines "when signals can be read and when they can be written". A clocking methodology makes hardware predictable by avoiding the situation where a value could be written to at the same time as it's read {% cite P&H:ARM -l 261 %}.

A simple example of a clocking methodology is positive edge-triggered clocking. In positive edge-triggered clocking, values stored in sequential logic elements are only updated on a rising clock edge (a transition from low to high) {% cite P&H:ARM -l 261 %}.

<!-- TODO: Add clock wave diagram  -->

The **setup time** is the minimum amount of time that an input must be valid before the clock edge {% cite P&H:ARM -l A-52 %}.

The **hold time** is the minimum time that an input must be valid after the clock edge {% cite P&H:ARM -l A-53 %}.

**Clock-to-Q** is the time it takes for a signal to propagate through a flip-flop so that the flip-flop reaches a stable state {% cite P&H:ARM -l A-72 %}.

Asynchronous devices (like I/O devices which often have their own clock) can communicate with a CPU through a series of handshaking steps {% cite P&H:ARM -l A-75 %}.

## Flip flops

A **flip-flop** is a circuit that has two stable states. It remains in one of these states until triggered into the other {% cite digital-computer-electronics -l 90 %}.

Computers use flip flops to regulate the flow of electricity through the system {% cite digital-computer-electronics -l 93 %}.

Registers can be built from multiple flip-flops.

<!-- TODO: Add flip-flop diagram --->

The clock signal is sent to each flip-flop in a computer to control when a flip-flop changes state {% cite digital-computer-electronics -l 93 %}.

## Field Programmable Devices

Field programmable devices are integrated circuits containing combinational logic (and possibly sequential logic) that can be configured by a user {% cite P&H:ARM -l A-77 %}.

There are two types of FPDs:

- Programmable logic devices (which only contain combinational logic)
- Field programmable gate arrays (which contain combinational logic and flip-flops)

{% cite P&H:ARM -l A-77 %}

In FPDs, gate and register locations are static, but the connections can be configured {% cite P&H:ARM -l A-77 %}.

One way to implement FPDs is by using an SRAM, which is downloaded at power-on, to control the settings of switches (meaning the FPD can be reconfigured multiple times) {% cite P&H:ARM -l A-77 %}.

## References

{% bibliography --cited_in_order %}
