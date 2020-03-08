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

Linux is an open-source operating system kernel. Most large Internet companies run their software on Linux distributions, so it's useful to learn Linux in detail.

## Table of contents
{: .no_toc  }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

When people think of Linux, they often imagine a full Linux system including the kernel, the C library, and system utilities. Technically, Linux is only the kernel, while the other features are part of the operating system {% cite lkd -l 4 %}.

## The kernel

The kernel implements the core functionality of an operating system. It provides a platform for applications to run on.

Some of the functionality implemented by the kernel includes:

1. Interrupt handlers to process hardware signals.
2. A scheduler to manage processes.
3. Memory management.
4. Networking services.

The memory space where the kernel runs is known as **kernel space**. The memory allocated to user programs is known as **user space**.

When a system is executing kernel code, it's said to be running in **kernel mode**. When the kernel is executing a user application, it's said to be running in **user mode** {% cite lkd -l 5 %}.

Programs running in user mode can communicate with the kernel using **system calls**—functions provided by the kernel that execute code in kernel mode {% cite lkd -l 5 %}.

System calls are generally triggered as part of a function provided by a library, such as the C library. When an application triggers a system call, we say that the application is executing a system call in user space and the kernel is running in **process context** {% cite lkd -l 5 %}.

The kernel manages the system's hardware, which uses interrupts to communicate with the kernel. When hardware sends an interrupt, the processor is interrupted and jumps to execute kernel code that was registered by the kernel on startup {% cite lkd -l 5 %}.

Linux is a **monolithic kernel**—it runs as a single process in a single address space. This is in contrast to microkernels, where kernel modules are separated into _servers_ {% cite lkd -l 7 %}.

## Linux distributions

Linux distributions are operating systems built on top of the Linux kernel. Some common Linux distributions include Ubuntu, CentOS, and Arch Linux.

Most Linux distributions include a package management system to install and manage applications.

## References

{% bibliography --cited_in_order %}
