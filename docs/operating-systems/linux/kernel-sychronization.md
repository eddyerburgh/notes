---
layout: default
title: Kernel synchronization
description: Notes on synchronization in the Linux kernel.
nav_order: 6
parent: Linux
grand_parent: Operating systems
permalink: /operating-systems/linux/kernel-synchronization
---

<!-- prettier-ignore-start -->

# Kernel synchronization
{:.no_toc}

Shared resources in the kernel require protection from concurrent access. This is because other threads of execution might modify data at the same time, resulting in problems like the data being overwritten by one thread, or data being accessed in an inconsistent state {% cite lkd -l 160 %}.

Protecting shared resources in Linux can be tough. Linux has symmetrical multiprocessing support: kernel code can simultaneously run on two or more processors. What's more, the kernel is preemptive. The scheduler can preempt kernel code at almost any point {% cite lkd -l 161 %}.

This section is about the problems of concurrency, and the tools for kernel synchronization.

## Table of contents
{: .no_toc  }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Critical regions and race conditions

"Code paths that access and manipulate shared data are called _critical regions_". It's usually unsafe for multiple threads of execution to access the same resources concurrently. To prevent concurrent access in a critical region, the program code must execute _atomically_, where atomic operations are operations that complete without interruption {% cite lkd -l 162 %}.

Two threads of execution executing simultaneously in a critical region is a bug, known as a race condition. Race conditions are very difficult to debug because they will not reproduce deterministically. Ensuring that race conditions don't occur is called synchronization {% cite lkd -l 162 %}.

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

Assume there are two threads of execution that both enter the critical region, and the initial value of `i` is 7. The desired outcome would be:

| Thread 1           | Thread 2           |
| ------------------ | ------------------ |
| movl i(%rip)       | -                  |
| addl \$1, %eax     | -                  |
| movl %eax, i(%rip) | -                  |
| -                  | movl i(%rip)       |
| -                  | addl \$1, %eax     |
| -                  | movl %eax, i(%rip) |

This would result in `i` being set to 9. However, it's possible that the instructions will execute in a different order:

| Thread 1           | Thread 2           |
| ------------------ | ------------------ |
| movl i(%rip)       | -                  |
| addl \$1, %eax     | -                  |
| -                  | movl i(%rip)       |
| movl %eax, i(%rip) | -                  |
| -                  | addl \$1, %eax     |
| -                  | movl %eax, i(%rip) |

This would result in `i` being set to 8, rather than 7.

The solution is for the set of increment instructions to be performed atomically as a single instruction. Most processors provide an instruction to atomically read, increment, and write-back {% cite lkd -l 164 %}. But sometimes the critical region contains many instructions that don't have an atomic equivalent. One solution to this is locks.

## Locking

A **lock** is a way to prevent multiple threads of execution from entering a critical region at the same time. Linux includes several different locking mechanisms {% cite lkd -l 166 %}.

A lock works like a lock on a door. Imagine the critical region as a room. When a process enters a critical region, it locks the door behind it. The process is then free to execute instructions without being interrupted. When it leaves the room, it unlocks the door so that other processes can enter the critical region {% cite lkd -l 165 %}.

Locks are advisory and voluntary. They are a programming construct that must be followed by every other program in order for them to provide atomic access to critical regions {% cite lkd -l 166 %}.

Some lock variants work by **busy waiting**: looping continuously until a lock becomes available. Other locks work by putting the process to sleep until the lock becomes available {% cite lkd -l 166 %}.

Locks can be implemented with atomic instructions that can test the value of an integer and set it to a new value only if it's zero. For example, locks are implemented on x86 with an instruction called copy and exchange {% cite lkd -l 166 %}.

### Causes of concurrency

The kernel has several sources of concurrency:

- Interrupts. An interrupt can occur at almost any time.
- Softirqs and tasklets. The kernel can raise or schedule a softirq or tasklet at almost any time.
- Kernel preemption. One task can preempt another.
- Sleeping and synchronization with user space. When a task sleeps it will invoke the scheduler, resulting in running a new process.
- Symmetrical multiprocessing. Two or more processors can execute code at the same time.

{% cite lkd -l 167 %}

"Code that is safe from concurrent access from an interrupt handler is said to be _interrupt-safe_. Code that is safe from concurrency on symmetrical multiprocessing machines is _SMP-safe_. Code that is safe from concurrency with kernel preemption is _preempt-safe_" {% cite lkd -l 167 %}.

Using synchronization mechanisms, like locks, to avoid race conditions is not difficult. The difficult part is identifying which parts of your code requires locking {% cite lkd -l 167 %}. Generally, any data that can be accessed concurrently needs protection {% cite lkd -l 169 %}

## Deadlocks

A deadlock is a condition where dependant threads are waiting for each other, putting the threads into a state where none of them able to complete.

There are different types of deadlock. For example, a self-deadlock is where a thread attempts to acquire a lock that it already holds. It will wait for the lock to be released, but it will never be released, because the thread is waiting and unable to release it {% cite lkd -l 169 %}.

Another example is a case with multiple locks. Consider $$n$$ threads and $$n$$ locks. If each thread is waiting for a lock held by another thread, none of the threads will be able to progress. A common case is where $$n$$ is 2, known as the deadly embrace or ABBA deadlock {% cite lkd -l 170 %}.

You can prevent deadlocks by following simple rules:

- Implement lock ordering. If two or more locks are acquired at the same time, they must be acquired in the same order.
- Avoid starvation. "Ask yourself, does this code finish? If foo does not occur, will bar wait forever?".
- Don't double acquire the same lock.

{% cite lkd -l 170 %}

## Contention and scalability

A contended lock is a lock that is in use by one thread with other threads waiting to acquire it. A lock can become highly contended if many threads attempt to acquire the lock, if threads hold the lock for a long time, or both. Highly contended locks can be a bottleneck in a system {% cite lkd -l 170 %}.

Scalability in an operating system is a measurement of how well the system can be expanded. "Ideally, doubling the number of processors should result in a doubling of the system's processor performance". This is never the case, but it should be something system designers strive for {% cite lkd -l 171 %}.

The granularity of locking is a description of how large an amount of data is protected by a lock. Low-granularity locking protects large subsystems of data, high-granularity locking protects a very small amount of data. High-granularity locks helps reduce lock contention {% cite lkd -l 171 %}

## Atomic operations

Atomic operations provide instructions that execute atomically. Atomic operations are the foundation of synchronization methods {% cite lkd -l 175 %}.

The kernel provides two sets of interfaces for atomic operations:

- Atomic integer operations
- Atomic bitwise operations

Atomic integer operations operate on a special data type: `atomic_t`. There are three reasons this type is used rather than having operations work directly on the C `int` type:

1. It ensures that only atomic operations are carried out on the type.
2. It ensures that the compiler doesn't optimize access to the value.
3. It hides architecture-specific implementation differences.

{% cite lkd -l 176 %}

The `atomic_t` type is defined in \<linux/types.h\>:

```c
typedef struct {
  volatile int counter;
} atomic_t;
```

The declarations needed to use the atomic integer operations are in \<asm/atomic.h\>. All architectures provide a minimum set of operations that are used throughout the kernel {% cite lkd -l 177 %}.

You can use the `ATOMIC_INIT()` macro to initialize an `atomic_t` value to 0:

```c
atomic_t u = ATOMIC_INIT(0);
```

The operations are self explanatory:

```c
atomic_set(&v, 4);
atomic_add(2, &v);
atomic_inc(&v);
```

Atomic operations are often used to implement counters. This is easy with `atomic_inc()` and `atomic_dec()` {% cite lkd -l 177 %}.

Another use is atomically performing an operation and testing the result, for example decrement and test:

```c
int atomic_dec_and_test(atomic_t *v)
```

Generally atomic operations are implemented as inline functions with inline assembly. If the operations are inherently atomic, like a read operation, they will just be a plain C code macro. For example, `atomic_read()` is a macro that returns the integer value of the `atomic_t`:

```c
/**
 * atomic_read - read atomic variable
 * @v: pointer of type atomic_t
 *
 * Atomically reads the value of @v.
 */
static inline int atomic_read(const atomic_t *v)
{
    return v->counter;
}
```

There is also a 64-bit variant: `atomic64_t`,

```c
typedef struct {
    volatile long counter;
} atomic64_t;
```

`atomic64_t` has the exact sames helper functions as atomic, except prefixed with 64. For example, `atomic64_dec_and_test()`:

```c
int atomic64_dec_and_test(atomic64_t *v)
```

The kernel also provides atomic bitwise functions that can operate on generic memory addresses {% cite lkd -l 181 %}.

```c
set_bit(0, &word); /* bit zero is now set (atomically) */
set_bit(1, &word); /* bit one is now set (atomically) */
printk("%ul\n", word); /* will print "3" */
clear_bit(1, &word); /* bit one is now unset (atomically) */
change_bit(0, &word); /* bit zero is flipped; now it is unset (atomically) */

/* atomically sets bit zero and returns the previous value (zero) */
if (test_and_set_bit(0, &word)) {
    /* never true ... */
}

/* the following is legal; you can mix atomic bit instructions with normal C */
word = 7;
```

There are also non atomic bitwise functions. These variants may be faster if you don't require atomicity {% cite lkd -l 182 %}.

## Spin locks

A spin lock is a lock that can be held by at most 1 thread. If another thread attempts to acquire a lock that's already held, the thread will loop until the lock becomes available {% cite lkd -l 184 %}.

Spin locks are architecture-specific and are implemented in assembly {% cite lkd -l 184 %}.

A basic use of spin lock is:

```c
DEFINE_SPINLOCK(mr_lock);

spin_lock(&mr_lock);
/* critical region */
spin_unlock(&mr_lock);
```

Spin locks provide protection on multiprocessor machines. On uniprocessor machines the lock compiles away and simply disables kernel preemption {% cite lkd -l 184-5 %}.

Spin locks can be used in interrupt handlers (unlike semaphores which sleep). You must disable local interrupts so that the same interrupt handler is not called again. If the interrupt ran again then the thread would attempt to acquire the lock, leading to deadlock {% cite lkd -l 185 %}.

The kernel provides a function to acquire a lock and disable interrupts:

```c
DEFINE_SPINLOCK(mr_lock);
unsigned long flags;

spin_lock_irqsave(&mr_lock, flags);
/* critical region */
spin_lock_irqrestore(&mr_lock, flags);
```

There are also spin locks for use in bottom halves. `spin_lock_bh` will obtain a lock and then disable all bottom halves, and `spin_unlock_bh` will release the lock and reenable bottom halves {% cite lkd -l 187 %}.

Reader-writer spin locks are useful for when code with shared data can be split into read paths and write paths. Multiple readers can concurrently hold the reader lock, whereas only one writer and no readers can hold the write lock {% cite lkd -l 188 %}.

A reader-writer lock is defined with:

```c
DEFINE_RWLOCK(mr_rwlock);
```

In the reader code path you use the `read_lock()` function:

```c
read_lock(&mr_rwlock);
/* critical region (read only) */
read_unlock(&mr_rwlock);
```

In the writer code path you use `write_lock()`:

```c
write_lock(&mr_rwlock);
/* critical region (read and write  ) */
write_unlock(&mr_rwlock);
```

Reader-writer locks favor readers over writers. Too many readers can starve the write path code {% cite lkd -l 190 %}.

## Semaphores

"Semaphores are sleeping locks". When a task attempts to acquire an unavailable spin lock, the semaphore places the task onto a wait queue and puts the task to sleep {% cite lkd -l 190 %}.

Semaphores are well suited for locks that are held for a long time, but not optimal for locks that are held for a short period of time. They can only be used in process context, because they sleep {% cite lkd -l 190 %}.

Semaphores can permit an arbitrary number of simultaneous lock holders. The number of simultaneous holders allowed is called the _usage count_ (or count for short). A semaphore with a count of 1 is called a mutex.

A semaphore supports two atomic operations: `P()` and `V()`. These stand for the Dutch words proberen (to test) and verhogen (to increase), since Dijkstra, the originator of semaphores, is Dutch. Instead of `P()` and `V()`, linux uses functions named `down()` and `up()` respectively.

`down()` acquires a semaphore by decrementing the semaphore count by 1. If the new count is 0 or greater the lock is acquired. If the count is negative then the task is placed on a wait queue {% cite lkd -l 191 %}.

The implementation of semaphores is architecture specific. The `semaphore` struct represents a semaphore. You initialize a semaphore with `sema_init()`:

```c
struct semaphore name;
sema_init(&name, count);
```

"The function `down_interruptible()` attempts to acquire the given semaphore. If the semaphore is unavailable, it places the calling process to sleep in the `TASK_INTERRUPTIBLE` state." If the process receives a signal while it is sleeping, the task is awakened and `down_interruptible()` returns `-EINTR` {% cite lkd -l 193 %}.

`down()` places the task on the wait queue with a state of `UNINTERRUPTIBLE`.

To release a semaphore you call `up()`. Consider the following example:

```c
/* define and declare a semaphore, named mr_sem, with a count of one */
static DECLARE_MUTEX(mr_sem);

/* attempt to acquire the semaphore ... */
if(down_interruptible(&mr_sem)) {
    /* signal received, semaphore not acquired */
}

/* critical region */

/* release the given semaphore */
up(&mr_sem);
```

There are also reader-writer semaphores, which can be used to split code that has different read and write code paths. They are represented by the `rw_semaphore` struct {% cite lkd -l 194 %}.

## Mutexes

A mutex is a sleeping lock that enforces mutual exclusion, for example a semaphore with a count of one {% cite lkd -l 196 %}.

The kernel contains a mutex data structure, represented by the `mutex` struct. To statically define a mutex you can use the `DEFINE_MUTEX()` macro:

```c
DEFINE_MUTEX(name);
```

You can define a mutex dynamically with `mutex_init`:

```c
mutex_init(&mutex);
```

You can lock and unlock a mutex with the `mutex_lock()` and `mutex_unlock()` functions:

```c
mutex_lock(&mutex);
/* critical region */
mutex_unlock(&mutex);
```

`mutex` has some constraints:

- Only one task can hold the mutex at a time.
- Whoever locked a mutex must unlock it.
- A mutex can't be acquired by an interrupt handler or a bottom half.
- A mutex can only be managed via its API.

{% cite lkd -l 196 %}

You should always use mutexes instead of semaphores, unless your requirement violates one of the constraints on mutexes {% cite lkd -l 197 %}.

## Sequential locks

Sequential locks (seq locks) work by maintaining a sequence counter. When shared data is written to, a sequential lock is acquired and the counter is incremented. Readers will check the sequence number before and after reading. If the values are equal then a write did not occur, if the value is even then a write has succeeded {% cite lkd -l 199-200 %}.

You can define a seq lock with the `DEFINE_SEQLOCK()` macro:

```c
seqlock_t mr_seq_lock = DEFINE_SEQLOCK(mr_seq_lock);
```

The write path would then look like:

```c
write_seqlock(&mr_seq_lock);
/* write lock is obtained */
write_sequnlock(&mr_seq_lock);
```

And the read path looks like the following:

```c
unsigned long seq;

do {
    seq = read_seqbegin(&mr_seq_lock);
    /* read data here ... */
} while (read_seqretry(&mr_seq_lock, seq));
```

Seq locks are a lightweight lock to be used with many readers and few writers. They favor writers over readers {% cite lkd -l 200 %}.

## Barriers

Barriers are a way to ensure that load and read instructions aren't reordered by either the compiler or the processor {% cite lkd -l 203 %}.

The `rmb` function provides a read barrier. No loads prior to the call will be reordered to lower than the call.

`wmb` provides a write barrier. No stores prior to the call will be reordered to lower than the call {% cite lkd -l 203 %}.

`mb` provides both a read and write barrier {% cite lkd -l 204 %}.

## Conclusion

The kernel is a concurrent environment. It's important to synchronize access to shared data structures.

Linux provides many mechanisms to synchronize access to data, including locks, semaphores, and mutexes.

## References

{% bibliography --cited_in_order %}
