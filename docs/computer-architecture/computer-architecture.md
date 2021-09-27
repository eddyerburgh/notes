---
layout: default
title: Computer architecture
description: An introduction to computer architecture.
has_children: true
has_toc: false
nav_order: 1
permalink: /computer-architecture
---

<!-- prettier-ignore-start -->

# Computer architecture
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

The study of computer architecture focusses on both the hardware and the hardware-software interface that together provide an abstraction for software to build upon.

These notes cover digital design (briefly) and computer architecture, as well as data representation such as binary, floating point, and unicode.

## Computer architecture

Modern computers are programmable digital machines that represent information (data and instructions) as binary numbers.

Most computers follow the Von Neumann architecture, that is they are stored-program computers (computers that store program instructions as numbers in memory) with the following components:

- A CPU (Central Processing Unit) containing an ALU and registers
- A Control Unit containing a program counter
- Memory
- External storage
- I/O devices (like a keyboard, monitor, etc.)

## Translation hierarchy

The **translation hierarchy** is the hierarchy of components responsible for converting a program into a process running on a computer {% cite P&H:ARM -l 128 %}.

The process for converting a program to machine code depends on the language the program is written in. For C, there are four steps:

1. Compiler
2. Assembler
3. Linker
4. Loader

{% cite P&H:ARM -l 128 %}

_Note: some systems combine phases to save time._

### Compiler

A **compiler** converts a C program into an _assembly language program_.

**Assembly language** is a human-readable language that's close to machine language code. For example:

```nasm
add x1, x2, x3
```

### Assembler

An **assembler** is a program that converts an assembly language program (input.s) into a machine language object file (output.o).

**Machine code** is the binary instructions that are fetched and executed by a processor.

An object file contains:

- An object file header describing the size and position of other parts of the file.
- A text segment containing the machine language code.
- A data segment containing static data.
- A relocation table which lists position-dependent instructions and data words whose addresses must be updated when the loader relocates the program.
- A symbol table which contains labels that haven't been identified, like external references.

{% cite P&H:ARM -l 125 %}

Assemblers also expand **pseudoinstructions**—assembly language instructions that don't correspond to machine language instructions—into machine instructions, as well as performing other utility transformations, like splitting large constants into multiple instructions.

There are often two passes in an assembler:

1. Build a symbol table and expand pseudoinstructions.
2. Generate machine code (using symbol table to resolve local labels).

Assembly programs can contains assembler directives that give directions to the assembler but do not produce code. e.g. `.text` meaning put subsequent items in the text segment, `.data` meaning put subsequent items in the data segment, and `.global symbol` declares a symbol that can be referenced from other files.

### Linker

A **linker** is a program that combines individual object files into an executable. It also resolves undefined labels and updates position-dependent instructions {% cite P&H:ARM -l 131 %}.

One benefit of linkers is that you can create a program from multiple object files, meaning a code change to one file will only require a single object file to recompiled {% cite P&H:ARM -l 131 %}.

The steps for linking are:

1. Place code and data modules symbolically in memory.
2. Determine the addresses of data and instruction labels.
3. Patch both the internal and external references.

{% cite P&H:ARM -l 131 %}

The linker uses the relocation information and symbol tables in each object file to resolve undefined labels {% cite P&H:ARM -l 131 %}.

The final executable file is in an OS-specific format. ELF is the standard for UNIX systems.

The process described above is known as **static linking**, where every library routine is copied into the executable. Static linking has some downsides:

- Library routines become part of the executable code, so any bugs in the original routine are baked into the compiled executable forever.
- It loads all routines in the library even if they are never called.

{% cite P&H:ARM -l 134 %}

The solution is to perform dynamic linking at runtime.

### Loader

A **loader** is a program that places an object program into main memory so that the program can be executed {% cite P&H:ARM -l 134 %}.

In UNIX the loader follows these steps:

1. Read the header to determine the size and position of the text and data segments.
2. Create an address space large enough for the text and data segments.
3. Copy the instructions and data from the executable into main memory.
4. Copy the parameters for the main program to the stack.
5. Initialize the processor registers and set the stack pointer to the first free address.
6. Branch to a start-up routine that copies parameters into argument registers and calls the main routine of the program. When the main routine returns, the startup routine terminates the program with an exit syscall.

{% cite P&H:ARM -l 134 %}

### Dynamic linker

A dynamic linker performs dynamic linking—linking libraries at runtime.

DLLs (Dynamically Linked Libraries) are library routines that are linked to a program during the program's execution. The program and library routines contain metadata on the location of nonlocal procedures and their names {% cite P&H:ARM -l 134 %}.

Lazy procedure linkage DLLs is where each routine is linked after it's called, as opposed being linked by the loader before the program begins executing {% cite P&H:ARM -l 135 %}.

Lazy procedure linkage DLLs are implemented like so:

- The first time a routine is called, the program calls a dummy entry and follows an indirection branch. This points to code that puts a value in a register to identify the library routine and then branches to the linker/loader.
- The linker/loader finds the desired routine, remaps it, and changes the address in the indirect branch location to point to that routine before branching to it.
- Subsequent calls to the library routine will branch indirectly to the routine {% cite P&H:ARM -l 136 %}.

<!-- TODO: ## Moore's law -->

<!-- TODO: ## Amdahl's law -->

## References

{% bibliography --cited_in_order %}
