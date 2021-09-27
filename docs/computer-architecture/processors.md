---
layout: default
title: Processors
description: Notes on computer processors.
nav_order: 3
parent: Computer architecture
permalink: /computer-architecture/processors
---

<!-- prettier-ignore-start -->

# Processors
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

A processor is the electronic circuitry within a computer that executes instructions.

Processors follow a fetch-decode-execute cycle where they fetch instructions from memory, decode them, and then execute them.

The latency of a processor is the delay from when an input enters the system to when its associated output is produced.

The throughput of a processor is how many instructions it can execute in a given period.

Modern processors are split into a data path and a control unit.

## Control unit

The control unit sets control lines that drive the datapath. The bulk of a processor's logic gates are contained in the control unit.

The input to a control unit is normally the opcode field from an instruction {% cite P&H:ARM -l 277 %}.

Examples of control unit outputs are signals used to control multiplexors, signals for controlling reads and writes to the register file, or operation signals for an ALU {% cite P&H:ARM -l 277 %}.

## Datapath

A **datapath** is a collection of functional units, registers, and buses that performs the required operations.

Datapath elements are the functional units used to implement a data path (e.g. an ALU, or a register file) {% cite P&H:ARM -l 263 %}.

### ALU

An ALU (Arithmetic Logic Unit) is circuitry that performs arithmetic operations on integers.

### FPU

An FPU (Floating Point Unit) is circuitry that performs arithmetic operations on floating point numbers.

### Register file

A **register file** is a state element that contains a set of registers that can be read and written to by supplying one or more register numbers {% cite P&H:ARM -l 264 %}.

## Instruction cycle

Processors follow an instruction cycle from boot-up until the computer has shut down.

The instruction cycle at a high-level involves:

1. Fetch an instruction from memory and increment PC by x (e.g. 4 for a byte-addressable system with 32-bit instructions).
2. Decode the instruction.
3. Execute the instruction.

An **instruction unit** is the part of the processor that arranges for instructions to be fetched from memory and forwards them to an execution unit. It contains the PC register, as well as combinational logic for calculating the next PC value {% cite P&H:ARM -l 263 %}.

## Pipelining

**Pipelining** is an implementation technique where instruction execution is split into multiple stages, allowing multiple instructions to be executed in parallel {% cite P&H:ARM -l 283 %}.

The benefits of pipelining include:

1. Clock cycle time can be reduced.
2. Increased instruction throughput.

An example of the pipeline stages:

1. Instruction fetch
2. Instruction decode
3. ALU
4. Memory access
5. Register write

Pipeline registers are added between each stage to store results and additional control information {% cite P&H:ARM -l 314 %}.

<!-- TODO: Add diagram of simple pipelined processor -->

### Hazards

**Pipeline hazards** are situations where the next instruction can't be executed in the following clock cycle {% cite P&H:ARM -l 288 %}.

**Structural hazards** are when a planned instruction can't execute in the proper clock cycle because the hardware does not support the combination of instructions that are set to execute {% cite P&H:ARM -l 288 %}.

**Data hazards** are when a planned instruction can't execute in the proper clock cycle because the data needed to execute the instruction is not yet available (one instruction is dependent on another), e.g. an instruction that uses data that's computed by the preceding instruction {% cite P&H:ARM -l 289 %}.

A **pipeline stall** (also known as a bubble) is a stall in the pipeline that is done to avoid a data hazard. Stalls can be implemented by adding noop instructions to the pipeline {% cite P&H:ARM -l 291, 326 %}.

**Forwarding** (or bypassing) is a method for avoiding stalling due to data hazards where data elements are retrieved from internal buffers rather than waiting for it to be available in registers. Even with forwarding, in some case it can be necessary to stall {% cite P&H:ARM -l 289, 291 %}.

A load-use data hazard is a type of data hazard where the data that is depended on is not yet available (thus requiring a stall) {% cite P&H:ARM -l 291 %}.

**Control hazards** (branch hazard) are where the correct instruction can't execute in the proper clock cycle because the instruction that was fetched was incorrect {% cite P&H:ARM -l 292 %}.

## Branch prediction

**Branch prediction** is a technique where processors attempt to guess the outcome of a conditional branch operation and prepare for the most likely result.

For a conditional branching instruction, if the condition is true then the next instruction should be the branch target address (branch is taken) if false then the next instruction should be $$PC + 4$$ {% cite P&H:ARM -l 266 %}.

One branch prediction strategy is to always predict that conditional branches will not be taken {% cite P&H:ARM -l 294 %}.

Dynamic branch prediction involves changing the prediction for branches over time. For example, you could store the history of each branch decision and then use the past results to predict the future. Modern branch predictors can correctly predict with over 90% accuracy {% cite P&H:ARM -l 295 %}.

One implementation of dynamic branch prediction involves using a branch prediction buffer (or branch history table)—a small memory that is indexed by the lower portion of the address of the branch instruction. The memory contains a bit that indicates whether a branch was taken last or not {% cite P&H:ARM -l 331 %}.

A **branch target buffer** can be used to store the destination instruction for a branch, this means the branch target address doesn't need to be calculated before fetching {% cite P&H:ARM -l 333 %}.

A **correlating predictor** is a predictor that combines the local behavior of a branch with the global behavior of some recent number of executed branches. This has been found to improve prediction rate {% cite P&H:ARM -l 333 %}.

A **tournament branch predictor** uses multiple predictors for each branch, tracking which is the most accurate {% cite P&H:ARM -l 334 %}.

Flushing instructions is the process of discarding a pipeline (usually done by setting control signals) {% cite P&H:ARM -l 329 %}.

When the prediction is incorrect, the pipeline must be flushed and the processor must restart the pipeline from the proper branch address {% cite P&H:ARM -l 295 %}.

## Speculation

**Speculation** is an approach where a compiler guesses at the outcome of an instruction to remove it as a dependency in executing other instructions, e.g. guess outcome of branch or speculate that a store which precedes a load does not refer to the same address (so the load can be executed before the store) {% cite P&H:ARM -l 344 %}.

Speculation can be wrong, so there must be a mechanism to verify a guess and to roll back the effects of speculatively executed instructions.

Compilers can perform speculative execution by reordering instructions. Some processors also perform speculative execution {% cite P&H:ARM -l 344 %}.

To recover from incorrect speculative execution, a compiler will often include instructions to verify and resolve an incorrect guess {% cite P&H:ARM -l 344 %}.

In hardware, speculative results are normally buffered until they are no longer speculative. If speculation was correct, thes instructions can be completed by allowing the buffered contents to be written to registers/memory. If speculation is incorrect, then the buffer is flushed and the correct instructions are executed instead {% cite P&H:ARM -l 344 %}.

Exceptions thrown by speculatively executed instructions must also be buffered until the speculation can be confirmed {% cite P&H:ARM -l 344 %}.

## Out-of-order execution

Out-of-order execution is where a pipelined processor executes instructions in a different order to the order in which they are written in, so that the processor can make use of otherwise-wasted instruction cycles (e.g. where an instruction is blocked waiting for a memory load) {% cite P&H:ARM -l 350 %}.

## SIMD

SIMD (Single-Instruction Multiple Data) processors allow a single instruction to perform multiple operations in parallel {% cite CS:APP -l 24 %}.

## Superscalar

A **superscalar processor** (also known as dynamic multi-issue) uses instruction-level parallelism to execute multiple instructions during a clock cycle {% cite P&H:ARM -l 349 %}.

A simple superscalar processor issues instructions in order and the processor decides whether zero, one, or more instructions can issue in a given clock cycle {% cite P&H:ARM -l 349 %}.

Superscalars can also implement dynamic pipeline scheduling which is hardware support for reordering instruction execution to minimize stalls.

In dynamic pipeline scheduling processors, the pipeline is divided into an instruction fetch and issue unit, multiple functional units, and a commit unit. The instruction unit fetches instructions, decodes them, and sends each instruction to a functional unit for execution. Each functional unit has buffers (reservation stations) that hold operands and the operation. Once the buffer contains all its operands, the result is calculated. The completed result is sent to reservation stations waiting for the result as well as to the commit unit, which buffers the results until it's safe to write the results to register/memory {% cite P&H:ARM -l 350 %}.

The commit unit buffer (the reorder buffer) is used to supply operands in the same way as forwarding {% cite P&H:ARM -l 350 %}.

## Interrupts

Interrupts (exceptions) are unscheduled events that disrupt a processor {% cite P&H:ARM -l 336 %}.

Interrupts can be triggered by errors inside a program, like divide by 0, or external events like a system timer triggering {% cite CS:APP -l 704 %}.

Interrupts are partly handled in hardware and partly handled in software. When a processor detects that an interrupt has occurred, it makes an indirect procedure call via an interrupt table to an OS interrupt handler that handles the interrupt appropriately {% cite CS:APP -l 704 %}.

In a pipeline, multiple interrupts can occur at once. Usually the interrupt in the later stage is handled.

<!-- TODO: precise interrupts -->

<!-- TODO: restartable instructions -->

## References

{% bibliography --cited_in_order %}
