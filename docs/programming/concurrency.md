---
layout: default
title: Concurrency
description: Notes on concurrency.
has_children: true
has_toc: false
nav_order: 1
parent: Programming
permalink: /programming/concurrency
---

<!-- prettier-ignore-start -->

# Concurrency
{:.no_toc}

Effective use of concurrency can dramatically speed up a program, so it pays to learn how to use it well.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

There are two common models in concurrent programming:

1. **Shared memory**—concurrent units interact by reading and writing shared objects in memory. e.g. two processors on a machine sharing the same physical memory, two threads in a program sharing the same objects.
2. **Message passing**—concurrent units interact by sending messages to each other via a communication channel. e.g. two processes interacting via stdio streams.

## Processes and threads

A **process** is an executing instance of a program that is isolated from other processes on the same machine, e.g. it has its own address space.

A **thread** (thread of execution) is a context that usually runs within a process and will normally have shared address space with other threads.

A **fiber** is like a thread, except that it uses cooperative multitasking (where the fiber explicitly yields control to other fibers), as opposed to preemptive multitasking—where a scheduler decides when to stop running one thread and start running another.

## Critical regions

A **critical region** (or critical section) is a section of code where shared resources are read and written.

Critical regions need to be protected, e.g. by locking, to stop unexpected behavior.

Consider a shared resource with a single critical region: a global integer with an operation that increments it:

```c
i++;
```

This might translate into the following assembly:

```
movl    i(%rip), %eax # move current value of i to register eax
addl    $1, %eax # add 1 to i in register eax
movl    %eax, i(%rip) # write back the new value of i
```

Assume there are two threads of execution that both enter the critical region, and the initial value of `i` is `7`. The desired outcome would be:

```
| Thread 1           | Thread 2           |
| ------------------ | ------------------ |
| movl i(%rip)       | -                  |
| addl \$1, %eax     | -                  |
| movl %eax, i(%rip) | -                  |
| -                  | movl i(%rip)       |
| -                  | addl \$1, %eax     |
| -                  | movl %eax, i(%rip) |
```

This would result in `i` being set to `9`. However, it's possible that the instructions will execute in a different order:

```
| Thread 1           | Thread 2           |
| ------------------ | ------------------ |
| movl i(%rip)       | -                  |
| addl \$1, %eax     | -                  |
| -                  | movl i(%rip)       |
| movl %eax, i(%rip) | -                  |
| -                  | addl \$1, %eax     |
| -                  | movl %eax, i(%rip) |
```

This would result in `i` being set to `8`, rather than `9`.

This is known as a race condition. A **race condition** is where multiple threads execute simultaneously in a critical region, possibly causing behavior that varies between executions.

The solution is for the set of increment instructions to be performed atomically as a single instruction. Most processors provide an instruction to atomically read, increment, and write-back. But if the critical region contains multiple instructions that don't have an atomic equivalent, then locks can be used instead.

## Atomic operations

**Atomic operations** are instruction sequences that execute as one unit. Atomic operations are the foundation of synchronization methods {% cite lkd -l 175 %}.

## Locks

A **lock** is a way to prevent multiple threads of execution from entering a critical region at the same time.

A lock works like a lock on a door. When a thread enters a critical region, it locks the region. The thread is then free to execute instructions without being interrupted. When the thread leaves the critical region, it unlocks the region so that other processes can enter it {% cite lkd -l 165 %}.

**Cooperative locks** are advisory and voluntary. They are a programming construct that must be followed by all threads in order for them to provide atomic access to critical regions {% cite lkd -l 166 %}. Most locks are cooperative.

**Mandatory locks** will throw exceptions if a thread attempts to access a locked resource.

Locks can be implemented with atomic instructions that can test the value of an integer and set it to a new value only if it's zero. For example, locks are implemented on x86 with an instruction called copy and exchange {% cite lkd -l 166 %}.

### Spin locks

**Spin locks** work by looping continuously until a lock becomes available (busy waiting):

```c
volatile int exclusion = 0;

void lock() {
    while (__sync_lock_test_and_set(&exclusion, 1)) {}
}

void unlock() {
    __sync_lock_release(&exclusion);
}
```

_Note: `__sync_lock_test_and_set` is an atomic exchange operator and an acquire fence.`__sync_lock_release()` writes `0` to exclusion atomically and forces a release fence. See [memory fences](#memory-fences) for more details._

Spin locks are efficient if threads are blocked for a short time, because they avoid OS process rescheduling.

### Futex

A **futex** (fast userspace mutex) is a 32-bit value whose address is supplied to the `futex()` system call, which is used to implement basic locking.

The two basic operations of a `futex()` system call:

- `FUTEX_WAIT(addr, val)`—if the value stored at `addr` is `val`, then the thread sleeps waiting for a `FUTEX_WAIT` opeartion on the futex word.
- `FUTEX_WAKE(addr, num)`—wake at most `num` of the waiters that are waiting on the futex word at `addr`.

## Deadlocks

A **deadlock** is a condition in a group of threads where no thread can proceed because they are each waiting for another member to take an action, such as releasing a lock.

A **self-deadlock** is where a thread attempts to acquire a lock that it already holds—causing the thread to wait forever because the thread is waiting and unable to release it {% cite lkd -l 169 %}.

Another example is a case with multiple locks. Consider $$n$$ threads and $$n$$ locks. If each thread is waiting for a lock held by another thread, none of the threads will be able to progress. A common case is where $$n$$ is 2, known as the deadly embrace or ABBA deadlock {% cite lkd -l 170 %}.

You can prevent deadlocks by following simple rules:

- Implement lock ordering. If two or more locks are acquired at the same time, they must be acquired in the same order.
- Avoid starvation. "Ask yourself, does this code finish? If foo does not occur, will bar wait forever?"
- Don't double acquire the same lock.

{% cite lkd -l 170 %}

## Lock-free programming

TODO

## Memory fences

A **memory fence** (memory barrier) is a type of CPU instruction that causes the CPU (or compiler) to enforce an ordering constraint on memory operations issued before and/or after the barrier.

Memory fences are required because:

1. Most modern CPUs employ performance optimizations that can result in out-of-order execution.
2. Compilers can reorder memory instructions.

A **full memory fence** ensures that no memory operations will move before or after the barrier.

A **release memory fence** ensures that no memory operation which appears before the memory barrier can be reordered to after the barrier.

An **acquire memory fence** ensures that no memory operation which appears after the memory barrier can be reordered to appear before the memory barrier.

## References

{% bibliography --cited_in_order %}
