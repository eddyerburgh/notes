---
layout: default
title: Instructions
description: Notes on the computer instructions.
nav_order: 5
parent: Computer architecture
permalink: /computer-architecture/instructions
---

<!-- prettier-ignore-start -->

# Instructions
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

Instructions are the primitive operations that a computer can perform.

An **ISA** (Instruction Set Architecture) is an abstract model of a computer that is used to create physical implementations (e.g. CPUs). x86 and ARMv8 are popular ISAs.

ISAs usually include arithmetic/logic, data transfer, and branching instructions. ISAs also define storage interfaces (i.e. registers and memory). Despite some differences in syntax and operations, ISAs tend to be similar overall.

**CISC** architectures attempt to minimize the number of instructions required for a program by offering a larger number of primitive instructions. x86 is an example of a CISC ISA.

**RISC** architectures reduces the number of instructions with the intention of improving the efficiency of the processor. ARMv8 and RISC-V are examples of a RISC instruction set.

This section uses the RV64 variant of the RISC-V ISA.

An assembler converts a symbolic version of an instruction into the binary version. e.g.: add A,B to $$1000110010100000_2$$. The language is known as assembly language. The binary version is known as machine language.

## Registers

A **register** is a hardware component that can store a value.

A **word** is the name given to a unit of access in a computer (normally 32 bits) and a **doubleword** is a larger unit of access in a computer (normally 64 bits) {% cite P&H:RISC-V -l 173 %}.

The size of registers depends on the ISA, but they usually store either a word or a doubleword.

In RV64 there are 32 64-bit registers. Registers are 0-indexed and are referred to as xN where N is the register number, e.g. x1 or x31.

RISC-V registers are assigned purposes by convention:

| Name    | Usage                              | Preserved on call |
| ------- | ---------------------------------- | ----------------- |
| x0      | The constant value 0               | n/a               |
| x1 (ra) | The return address (link register) | yes               |
| x2 (sp) | Stack pointer                      | yes               |
| x3 (gp) | Global pointer                     | yes               |
| x4 (tp) | Thread pointer                     | yes               |
| x5-x7   | Temporaries                        | no                |
| x8-x9   | Saved                              | yes               |
| x10-x17 | Arguments/ results                 | no                |
| x18-x27 | Saved                              | yes               |
| x28-x31 | Temporaries                        | no                |

{% cite P&H:RISC-V -l 232 %}

There is an additional register which holds the address of the currently-executing instruction. This is called PC (Program Counter) {% cite RISC-V:2.2 -l 9 %}.

Registers have faster access times and higher throughput than memory, and so registers are preferred to memory when possible {% cite P&H:RISC-V -l 180 %}.

Programs are normally register-constrained, i.e. programs often have more variables than there are registers. **Spilling registers** is the process of storing less frequently used variables in memory {% cite P&H:RISC-V -l 180 %}.

A **register-memory architecture** allows operations to be performed on (or from) memory. x86 is a register-memory architecture.

A **register-register architecture**, also known as a load-store architecture, is an architecture where operations are divided into memory operations and operations that only occur between registers. RISC-V follows a load-store architecture—only load and store instructions access memory {% cite P&H:RISC-V -l 347-8 %}.

## RISC-V Instruction format

A RISC-V assembly instruction consists of an operation and a number of operands (the number of operands is instruction-specific):

```
operation operand_1, operand_2
```

**Register operands** hold a value that represents a register. They can either be source registers, where data is read from, or destination registers, where data is written to {% cite P&H:RISC-V -l 181 %}. For example:

```
add x1, x2, x3
```

**Immediate operands** (also known as constant operands) represent values that are encoded directly into an instruction {% cite P&H:RISC-V -l 181 %}. For example:

```
addi x1, x2, 0xff
```

An **instruction format** defines the layout of bits for a machine code instruction (which is usually generated from assembly code by an assembler).

RISC-V instructions are all 32-bit. The instructions are split into several formats which have different binary fields. All instructions have a 7-bit opcode field, which is used to determine the format of the instruction (and therefore how to interpret the rest of the instruction) {% cite P&H:RISC-V -l 198 %}.

_Note: not all ISAs use fixed-length instructions. x86, for example, has variable-length instructions._

An addressing mode defines how the machine language identifies the operands of each instruction. RISC-V has four addressing modes:

- **Immediate addressing**, where the operand is a constant within the instruction.
- **Register addressing**, where the operand represents a register.
- **Base addressing**, where the operand is at the memory location whose address is the sum of a register and a constant in the instruction (e.g. load and store).
- **PC-relative addressing**, where the branch address is the sum of the PC and a constant in the instruction

{% cite P&H:RISC-V -l 244 %}

There are 6 32-bit instruction formats in RISC-V. These notes cover 3 for demonstration purposes. You can see a full list on the [RISC-V green card](https://www.cl.cam.ac.uk/teaching/1617/ECAD+Arch/files/docs/RISCVGreenCardv8-20151013.pdf).

### R-type instruction

R-type instructions are used for register-register operations.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/instructions/r-type.svg" alt="">
  <figcaption><h4>Figure: R-type RISC-V instruction {% cite RISC-V:2.2 -l 11 %}</h4></figcaption>
</figure>

The _opcode_ field represents part of the instruction opcode.

_rd_ is the destination register operand.

_funct3_ is an additional opcode field.

_rs1_ is the first source register operand.

_rs2_ is the second source register operand.

_funct7_ is another additional opcode field.

### I-type instruction

I-type instructions are used for register-immediate operations.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/instructions/i-type.svg" alt="">
  <figcaption><h4>Figure: I-type RISC-V instruction {% cite RISC-V:2.2 -l 11 %}</h4></figcaption>
</figure>

The _immediate_ field holds an immediate value that's interpreted as either two's complement or an unsigned integer depending on the opcode.

_rs1_ is a source register.

_funct3_ is an additional opcode field.

_rd_ is a destination register.

_opcode_ is the opcode.

### B-type instruction

B-type instructions are used for branch instructions {% cite RISC-V:2.2 -l 17 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/instructions/b-type.svg" alt="">
  <figcaption><h4>Figure: B-type RISC-V instruction</h4></figcaption>
</figure>

_rs1_ is the first source register operand.

_rs2_ is the second source register operand.

funct3 is an additional opcode field.

An immediate is then split between bits x-x and bits x-x.

The B-type is used to encode branch offsets that are in multiples of 2 and so the 0 bit is implicit {% cite RISC-V:2.2 -l 12 %}.

## Computational instructions

Computational instructions use the ALU to perform computations. They include arithmetic operations, comparison operations, logical operations, and shift operations.

There are several RISC-V register-register instructions:

- `add`, `sub`
- `slt` (set if less than), `sltu` (set if less than unsigned)
- `and`, `or`, `xor` (bitwise logical operations)
- `sll` (shift left logical), `srl` (shift right logical), `sra` (shift right arithmetic)

As well as equivalent register-immediate instructions:

- `addi`
- `slti`, `sltui`
- `andi`, `ori`, `xori`
- `slli`, `srli`, `srai`

_Note: there is no `subi` since you can achieve the same effect by using `addi` with a negative constant._

Arithmetic and logical operators are self-explanatory.

Shift operations move bits in doublewords to the left and to the right. Logical shifts fill in the blanks with 0s, arithmetic right shifts fill in the blanks with the sign bit.

Shifting left by $$i$$ bits is equivalent to multiplying the result by $$2^i$$.

## Control flow instructions

By default, instructions execute in sequence one after the other. **Control flow instructions** can alter the order of instruction execution.

Conditional branch instructions test two registers and branch (switch execution to a different instruction sequence) if the test passes, otherwise the processor will continue to execute the next instruction {% cite P&H:RISC-V -l 211 %}. Examples include:

- `beq` `==` branch if equal
- `bne` `!=` branch if not equal
- `blt` `<` branch if less than
- `bge` `>=` branch if greater than or equal to
- `bltu` `<` branch if less than unsigned
- `bgeu` `>` branch if greater than or equal to unsigned

Conditional branches are all B-type instructions with the assembly format `op, rs1, rs2, l1`. `rs1` and `rs2` are the registers whose values are compared. `l1` is a label, which is a symbol for an address.

RISC-V conditional branch instructions use PC-relative addressing. The target address for an instruction is calculated by adding a branch offset (which is stored as an immediate in the instruction) to the PC address ($$target = PC + \text{branch_offset}$$) {% cite P&H:RISC-V -l 242 %}.

For assembly instructions that branch to a label, the label address is first resolved by the assembler during compilation. If the address is close enough to the current address, then an offset is packed into the machine code instruction as a signed immediate to be added to the PC. If the label is too far away, then the assembly instruction is expanded into a conditional branch instruction and an unconditional branch instruction to the label address (which has more reach) {% cite P&H:RISC-V -l 257 %}.

Unconditional branch instructions are called jumps. In RISC-V the `jal` (jump-and-link) and `jalr` (jump-and-link register) instructions are used to perform unconditional jumps {% cite P&H:RISC-V -l 216 %}.

`jal rd l1` jumps to a target specified by `l1` (again this is converted to an offset from the instruction's address by the assembler or linker). The address of the next instruction ($$PC + 4$$) is stored in register `rd`. `jal` uses the UJ-type format (where the J-immediate encodes a signed offset in multiples of 2 bytes) and has ± 1MB range RISC-V {% cite RISC-V:2.2 -l 15 %}.

A plain unconditional jump is written `jal x0 l1`.

`jalr rd imm(rs)` jumps to a target specified by the value of register `rs` + `imm`, thus `jalr` has greater range than `jal`. Like `jal`, $$PC + 4$$ is stored in register `rd`.

A **branch table** is an array of words of addresses to branch to. They can be used to efficiently implement switch statements with multiple cases {% cite P&H:RISC-V -l 216 %}.

## Data transfer instructions

Data transfer instructions move data between memory and registers {% cite P&H:RISC-V -l 173 %}.

A **memory address** is used to locate a data element in memory. For now, you can conceive of memory as a large single-dimension array where the address is the index and the value is the data that's stored {% cite P&H:RISC-V -l 175 %}.

RISC-V uses byte-addressing, meaning that hardware supports access to individual bytes {% cite P&H:RISC-V -l 177 %}. As an example, if there are two doublewords $$d_1$$ and $$d_2$$ stored contiguously in memory with $$d_1$$ preceding $$d_2$$, if $$d_1$$ is stored at address $$A$$ then $$d_2$$ is stored at address $$A + 8$$.

Load instructions copy data from memory to a register. In RISC-V, `ld` is the load doubleword instruction. The format of a load instruction is `op rd, rs1, imm` where rd is the destination register that the data will be stored in. The address is calculated by using the value in `rs1` and adding the constant `imm` {% cite P&H:RISC-V -l 176 %}.

Store instructions copy data from a register to memory. In RISC-V, `sd` is the store doubleword instruction. The format of a store instruction is `op rs1, rs2, imm` where `rs1` is the register to be stored, `rs2` is the base register, and `imm` is an offset used to select the element {% cite P&H:RISC-V -l 179 %}.

_Note: an **alignment restriction** is a requirement that data must be aligned in memory based on some unit, e.g. 32-bit words must start at addresses that are multiples of 4 and 64-bit words must start at addresses that are multiples of 8. RISC-V isn't byte-aligned, but MIPS is {% cite P&H:RISC-V -l 179 %}._

A signed load performs sign extension (filling bits to the left with the left-most bit of the word being load) to keep the value of numbers that are less than 64-bit. `lb` does signed load and sign extends the byte, whereas `lbu` (load byte unsigned) treats the byte unsigned and zero extends {% cite P&H:RISC-V -l 189-90 %}.

## Memory layout

The specifics of a program's memory layout depend on the system running the program as well as the architecture.

On Linux, a program's memory space contains:

- Text segment
- Static data segment
- Stack
- Heap
- A reserved section

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/instructions/memory-layout.svg" alt="">
  <figcaption><h4>Figure: RISC-V convention for memory layout on Linux {% cite P&H:RISC-V -l 230 %}</h4></figcaption>
</figure>

The **text segment** is "the segment of a UNIX object file that contains the machine language code for routines in the source file" {% cite P&H:RISC-V -l 230 %}.

The **static data segment** contains constants and static variables that never change {% cite P&H:RISC-V -l 229 %}.

Additional memory is available in the form of a stack and heap.

### Stack

In computer architecture, a **stack** is a variable-size region of memory. It's used for storing local variables and for preserving the values of registers during procedure calls.

A stack is a LIFO (Last-In-First-Out) data structure that supports the following abstract operations:

- `push(x)` adds `x` to the stack.
- `pop()` removes the top item from the stack.
- `top()` returns the top item from the stack.

In RISC-V, stacks grow from higher addresses to lower addresses, so pushing to the stack decreases the stack pointer value and popping from the stack increases the stack pointer value {% cite P&H:RISC-V -l 222 %}.

The RISC-V stack pointer is register x2, also known as sp.

The segment of a stack that contains a procedure's saved registers is called a procedure frame (or activation record) {% cite P&H:RISC-V -l 227 %}.

Some RISC-V compilers use a frame pointer register (fp) to point to the first doubleword of the current procedure's frame. This is generally to make address calculations simpler since the frame pointer value doesn't change during a procedure's execution {% cite P&H:RISC-V -l 228 %}.

### Heap

A **heap** is a variable-sized region of memory intended for dynamic data structures {% cite P&H:RISC-V -l 229 %}.

As opposed to a stack, a heap grow from lower addresses to higher addresses {% cite P&H:RISC-V -l 230 %}.

In C, `malloc()` allocates space on the heap and returns a pointer to it. `free()` releases the space on the heap to which the pointer points {% cite P&H:RISC-V -l 230 %}.

## Procedures

A **procedure** is a stored (sometimes parameterized) subroutine that performs a task.

A **caller** is the program that calls a procedure, a **callee** is the procedure that is called by the caller {% cite P&H:RISC-V -l 220 %}.

To execute a procedure, the program must follow these steps:

1. Put parameters somewhere accessible to the procedure.
2. Jump to the procedure.
3. Acquire local storage resources needed for the procedure (allocated from the stack).
4. Perform the desired logic.
5. Put result in a place that the caller can access.
6. Restore any registers used.
7. Return control to the caller.

{% cite P&H:ARM -l 100 %}

A **calling convention** is an agreement for how subroutines receive arguments, how they return a result, and what registers need to be saved between procedure calls.

RISC-V designates x10-x17 as parameter registers used to pass parameters and return values. ra (x1) contains the return address (the next address after the instruction that called the procedure) {% cite P&H:RISC-V -l 219 %}.

Registers x5-x7 and x28-x31 are temporary registers that are not preserved by the callee during a procedure call {% cite P&H:RISC-V -l 224 %}.

Registers x8-x9 and x18-x27 are saved registers. If a callee uses the registers then they must save and restore them as part of the procedure call {% cite P&H:RISC-V -l 224 %}.

It can be useful to break a procedure into three parts:

- A prologue that performs setup.
- A body where the main logic runs.
- An epilogue that performs teardown.

To satisfy the guarantees of the RISC-V calling convention, the prologue will

- Decrement the stack pointer to allocate required memory on the stack.
- Store any saved registers that are accessed in the procedure.
- Store ra if a function call is made.

{% cite riscv-calling-convention -l 3 %}

The epilogue will:

- Restore any saved registers that were used.
- Reload ra if required.
- Increment sp to its previous value.
- Jump back to the return address.

{% cite riscv-calling-convention -l 3 %}

A **tail call** is a subroutine call made as the final act of a procedure. **Tail call optimization** is when tail calls are optimized. Tail calls can be optimized by not performing the jump-and-add-stack-frame and then pop-stack-frame-and-return-to-caller sequences. Instead the callee can reuse the existing stack frame, since it's no longer needed by the caller.

<!-- TODO: ## Atomic instructions -->

## References

{% bibliography --cited_in_order %}
