---
layout: default
title: Linux
description: An introduction to Linux.
has_children: true
has_toc: false
parent: Operating systems
permalink: /operating-systems/linux
---

<!-- prettier-ignore-start -->

# Linux
{:.no_toc}

Linux is a free open-source operating system kernel that's often used by web servers.

## Table of contents
{: .no_toc  }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Linux

Linus Torvalds created linux in 1991 while a student at Helsinki university. He was frustrated at the lack of a free Unix system, so he decided to write his own {% cite lkd -l 3 %}.

Although Linux is inspired by Unix and implements the Unix API (defined in the POSIX standard), it's not a direct descendant of Unix {% cite lkd -l 3 %}.

When people think of Linux, they often imagine a full Linux system including the kernel, the C library, and system utilities. Technically Linux is only the kernel, while the other features are part of the operating system {% cite lkd -l 4 %}.

## Kernel

The kernel is the core functionality of an operating system. It provides an API for user interfaces to be written on top of. For example, interrupt handlers to handle signals (like a key press), a scheduler to manage processes, memory management to manage process address space, and services like networking {% cite lkd -l 5 %}.

Typically a kernel runs with elevated privileges, like access to hardware memory. This execution state and memory space is known as **kernel space**. User applications run in **user space**. When a system is executing kernel code, it's running in kernel mode, and when the kernel is executing a user application, it's running in user mode {% cite lkd -l 5 %}.

Programs running in user mode can communicate with the kernel using **system calls**: functions provided by the kernel that execute code in kernel mode {% cite lkd -l 5 %}.

System calls are generally called as part of a function provided by a library, like the C library. Some C library functions do a lot of work before making a system call, like `printf()`. Others are almost one to one mappings, like `open()`. When an application calls a system call, we say that the application is executing a system call in user space, and the kernel is running in process context {% cite lkd -l 5 %}.

The kernel manages the system's hardware. Nearly all hardware uses interrupts to communicate with the kernel. When hardware sends an interrupt, the processor is interrupted and jumps to execute kernel code that was registered by the kernel on startup, known as an **interrupt handler** {% cite lkd -l 5 %}. For example, a keyboard might issue an interrupt when a key has been pressed to alert the kernel that there's new data in the keyboard buffer. The kernel would use the interrupt number to execute the interrupt handler to process the new keyboard data and tell the keyboard control that it's ready for more data {% cite lkd -l 5 %}.

Interrupt handlers don't run in a process context, they run in a special interrupt context. This exists so that the kernel can handle interrupts as quickly as possible {% cite lkd -l 5 %}.

Linux is a **monolithic kernel**—it runs as a single process in a single address space. This is in contrast to microkernels, where kernel modules are separated into _servers_. Theoretically, most servers run as user processes in their own address space, although in practice the latency introduced by interprocess communication means that servers often run in kernel mode instead, defeating the main purpose of microkernels {% cite lkd -l 7 %}.

## Differences with Unix

Although Linux is similar to Unix, there are some notable differences:

1. Linux can dynamically load and unload kernel modules.
2. Linux can preempt a task as it executes.
3. Linux doesn't differentiate between threads and processes.
4. Linux has an object oriented device model.
5. Linux ignores some Unix features that it believes are badly designed, such as streams.
6. Linux is free and open source.

{% cite lkd -l 8 %}

## References

{% bibliography --cited_in_order %}
