---
layout: default
title: System calls
description: Notes on system calls in the Linux kernel.
nav_order: 3
parent: Linux
grand_parent: Operating systems
permalink: /operating-systems/linux/system-calls
---

<!-- prettier-ignore-start -->

# System calls
{:.no_toc}

System calls are an interface for user processes to call into kernel code.

## Table of contents
{: .no_toc  }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

System calls are a layer between user space and hardware that allow user space programs to access the filesystem, create processes, and perform other privileged tasks. System calls are the only entry point from user space into the kernel other than exceptions and traps.

There are three benefits to the system call layer:

1. It provides an abstracted hardware interface.
2. It ensures security and stability—the kernel can arbitrate access based on permissions.
3. It enables the virtualized system that is provided to processes. Without system calls it would be impossible to provide virtualized memory.

{% cite lkd -l 69 %}

## APIs, POSIX, and the C library

Most user space programs call a user space API rather than invoking system calls directly. This decouples the API and the system call interface provided by the kernel. When different systems implement a common user space API, user programs can use the API while remaining portable {% cite lkd -l 70 %}.

A common Unix interface is POSIX. POSIX is a series of standards from IEEE that defines an API for Unix systems. Linux aims to be POSIX-compliant, as do other non-Unix based systems like Windows {% cite lkd -l 70 %}.

The C library implements the main API on Linux, including the standard C library and the system call interface. The majority of the POSIX API is also implemented in the C API {% cite lkd -l 71 %}.

## Syscalls

System calls (**syscalls**) are usually accessed by function calls defined in the C library.

When a system call returns an error, the C library writes an error code to the global `errno` variable. The variable can be translated into a human-readable error with helper functions, like `perror()` {% cite lkd -l 71 %}.

System calls are defined in the kernel. For example the system call `getpid()` which returns the PID of the current process:

```c
SYSCALL_DEFINE0(getpid)
{
    return task_tgid_vnr(current); /* returns current->tgid */
}
```

`SYSCALL_DEFINE0()` is a macro that defines a system call with 0 parameters (hence the 0). The expanded code looks like this:

```c
asmlinkage long sys_getpid(void);
```

Note the `asmlinkage` modifier. This is a directive that tells the compiler to only look on the stack for this function's arguments. It's a required modifier for system calls.

For compatibility with 32-bit and 64-bit systems, syscalls that return an `int` in user space return a `long` in the kernel.

The kernel implementation of `getpid()` is `sys_getpid()`. Prefixing the systemcall name with `sys_` is the convention used by all system calls in Linux {% cite lkd -l 72 %}.

### System call numbers

Each system call has a unique system call number. Once assigned, a syscall number can't change because it will break compiled code that relies on the system call {% cite lkd -l 72 %}.

The syscall number identifies which system call was invoked. All registered system calls are kept in the `sys_call_table` which is defined for each architecture. The x86 table is in [arch/x86/kernel/syscall_64.c](https://elixir.bootlin.com/linux/v2.6.39.4/source/arch/x86/kernel/syscall_64.c).

## System call handler

User space code can't execute kernel code directly, because the kernel exists in a protected memory space. Instead, user space programs must cause an exception or a trap to enter kernel mode, which will then execute the exception handler {% cite lkd -l 73 %}.

The system call handler is named `system_call()`, and it’s architecture-dependant. On x86 it's implemented in [entry_64.s](https://elixir.bootlin.com/linux/v2.6.39.4/source/arch/x86/kernel/entry_64.S#L455) {% cite lkd -l 73 %}.

### Denoting the correct system call

The system call number must be passed to the kernel. On x86 this is done by setting the `eax` register to the sys call number. The system call handler will then read the syscall number from the `eax` register {% cite lkd -l 73-4 %}.

`system_call()` validates the system call number by comparing it to `NR_syscalls`. If it is larger than or equal to `NR_syscalls`, the function returns `ENOSYS`. Otherwise, the specified system call is invoked:

```text
call *sys_call_table(,%rax,8)
```

On x86-64, each element in the system call table is 64 bits (8 bytes). The kernel multiplies the syscall number by 8 to arrive at its location in the system call table.

### Parameter passing

Parameters can be passed in a similar way to the syscall number. This is also architecture specific.

On x86 parameters are stored in registers `ebx`, `ecx`, `edx`, `esi`, and `edi`. If there are more than 6 parameters then a single register holds a pointer to user space where the parameters are stored {% cite lkd -l 74 %}.

## Adding a system call

Adding a system call is easy. The complexity of passing arguments to the syscall are taken care of for you, all you need to do is:

1. Add an entry to the system call table for each architecture that supports the syscall.
2. Define the syscall number in `<asm/unistd.h>`.
3. Compile the system call into the kernel image (as opposed to a module) by placing the system call code in a file in the /kernel directory.

As an example example, imagine you wanted to add a `foo()` system call. First you add it to a system call table by appending it at the bottom of the table code, which looks something like this:

```c
ENTRY(sys_call_table)
.long sys_restart_syscall /* 0 */
.long sys_exit
.long sys_fork
.long sys_read
.long sys_write
.long sys_open /* 5 */
 // ..
.long sys_eventfd2
.long sys_epoll_create1
.long sys_dup3 /* 330 */
.long sys_pipe2
.long sys_inotify_init1
.long sys_preadv
.long sys_pwritev
.long sys_rt_tgsigqueueinfo /* 335 */
.long sys_perf_event_open
.long sys_recvmmsg
.long sys_foo
```

The system call will then have a number based on its position in the table. In the above example that would be 338.

You would then need to repeat the process for each architecture. The system call number doesn't need to be the same for each architecture, because the syscall number is part of the architecture's unique ABI {% cite lkd -l 80 %}.

Next, the system call is added to `<asm/unistd.h>`, which looks like this:

```c
/*
* This file contains the system call numbers.
*/

#define __NR_restart_syscall 0
#define __NR_exit 1
#define __NR_fork 2
#define __NR_read 3
#define __NR_write 4
#define __NR_open 5

// ..

#define __NR_signalfd4 327
#define __NR_eventfd2 328
#define __NR_epoll_create1 329
#define __NR_dup3 330
#define __NR_pipe2 331
#define __NR_inotify_init1 332
#define __NR_preadv 333
#define __NR_pwritev 334
#define __NR_rt_tgsigqueueinfo 335
#define __NR_perf_event_open 336
#define __NR_recvmmsg 337
```

You would add a new macro for the `foo()` syscall number:

```c
#define __NR_foo 338
```

Finally you would implement the syscall. This can be in any file that's compiled into the core kernel image:

```c
#include <asm/page.h>

/*
* sys_foo – everyone’s favorite system call.
*
* Returns the size of the per-process kernel stack.
*/
asmlinkage long sys_foo(void)
{
    return THREAD_SIZE;
}
```

## Accessing the System Call from User Space

Linux provides a set of macros for wrapping access to system calls. The macros set up the register contents and issue the trap instructions {% cite lkd -l 81 %}.

The macros are named `_syscalln`, where `n` is between 0 and 6. The number refers to the number of parameters that can be passed in. This is because the macro needs to know how many parameters to expect so that it can push them into registers {% cite lkd -l 81 %}.

For example, the `open()` system call is defined as:

```c
long open(const char *filename, int flags, int mode)
```

The syscall macro to use this system call without library support would be:

```c
#define __NR_open 5
_syscall3(long, open, const char *, filename, int, flags, int, mode)
```

Then, the application can simply call `open()`.

Each macro has $$2 + 2n$$ parameters. The first parameter is the return type, and the second parameter is the name of the system call. The remaining parameters are pairs of the type and name for each parameter {% cite lkd -l 81 %}.

The `_syscall3()` macro expands into a C function with inline assembly. The assembly pushes the system call number and parameters to the correct registers and issues the software interrupt to trap into the kernel. Placing the macro into an application is all that's required to use the `open()` system call {% cite lkd -l 81 %}.

## Conclusion

System calls are an important API for user space code to execute privileged kernel actions.

Calling a system call from user space involves adding the correct arguments to registers, and trapping into the kernel. Normally this is taken care of by functions in the C library.

## References

{% bibliography --cited_in_order %}
