---
layout: default
title: Time management
description: Notes on time management in the Linux kernel.
nav_order: 7
parent: Linux
grand_parent: Operating systems
permalink: /operating-systems/linux/time-management
---

<!-- prettier-ignore-start -->

# Time management
{:.no_toc}

This section is about how the kernel handles time and how it implements timers.

## Table of contents
{: .no_toc  }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

Time management is an important part of the kernel. Many kernel functions are time driven, rather than event driven. Some functions run at intervals, like refreshing the screen, others occur at a relative time in the future, like delayed disk I/O {% cite lkd -l 206 %}.

Relative time and absolute time are crucial concepts. Relative time, like 5 seconds from now, does not require the concept of absolute time. Absolute time requires the kernel to understand the passing of time and some absolute measure of it {% cite lkd -l 207 %}.

Events that occur at regular intervals are driven by the **system timer**. "The system timer is a programmable piece of hardware that issues an interrupt at a fixed frequency”. The interrupt handler for the system timer (the **timer interrupt**) performs periodic work, like balancing the schedule of runqueues {% cite lkd -l 207 %}.

Dynamic events that happen once at some period in the future are also handled by the kernel.

## The Kernel's notion of time

The kernel must work with system hardware in order to manage time. The system timer provides the kernel the ability to track the passing of time {% cite lkd -l 207 %}.

The system timer uses an electronic time source, like a digital clock or the frequency of the processor. The system timer runs at a preprogrammed frequency, called the **tick rate** {% cite lkd -l 208 %}.

The period between two successive timer interrupts is called the **tick**. A tick is equal to $$1/\text{tick rate}$$ seconds. The kernel can use the tick to calculate the passage of time {% cite lkd -l 208 %}.

The kernel uses ticks to keep track of both wall time and system uptime. Wall time is the actual time of day, the system uptime is "the relative time since the system booted" {% cite lkd -l 208 %}.

The functions that run during the timer interrupt include:

- Updating system time.
- Updating wall time.
- Balancing schedule run queues.
- Running dynamic timers that have expired.
- Updating resource usage and processor time statistics.

{% cite lkd -l 208 %}

## The tick rate: hz

The tick rate is programmed on system boot using the static preprocessor define: `HZ`. The `HZ` value differs for each architecture {% cite lkd -l 208 %}.

`HZ` is defined in <asm/param.h>. "The tick rate has frequency of `HZ` hertz and a period of 1/`HZ` seconds". The most common `HZ` value is 100. Another common `HZ` is 1000, meaning the timer interrupt runs every 1ms {% cite lkd -l 208 %}.

There are a few benefits to larger `HZ` values:

- Kernel timers execute with increased accuracy.
- System calls that use a timeout value are more accurate.
- Measurements like system uptime are more accurate.
- Process preemption is more accurate.

{% cite lkd -l 211 %}

The downside to larger `HZ` values is that there is more overhead related to handling the timer interrupt, however this is generally small {% cite lkd -l 211 %}.

## Jiffies

`jiffies` is a global variable that "holds the number of ticks that have occurred since the system booted" {% cite lkd -l 212 %}. There are `HZ` jiffies in a second.

`jiffies` is initialized to a special value that causes the variable to overflow more often. This is used to catch bugs. The true `jiffies` value can be calculated by removing the offset from `jiffies` {% cite lkd -l 212 %}.

`jiffies` is defined im <linux/jiffies.h>:

```c
extern unsigned long volatile jiffies;
```

Since `jiffies` is an `unsigned long`, on a 32-bit machine with `HZ` of 1000, it would only be able to hold 49.7 days worth of ticks {% cite lkd -l 213 %}.

A second `jiffies_64` variable is defined in <linux/jiffies.h>:

```c
extern u64 jiffies_64;
```

The `ld(1)` script which links the main kernel image (arch/x86/kernel/vmlinux.lds.S on x86) overlays the `jiffies` variable on the start of `jiffies_64`:

```c
jiffies = jiffies_64;
```

So `jiffies` is the lower 32 bits of `jiffies_64`. Code can access jiffies as before. Since most code accessing jiffies only cares about elapses in time, rather than absolute time, the code can work fine with 32 bits. Time management code uses the entire 64 bit `jiffies_64`, which is accessible with `get_jiffies_64()` {% cite lkd -l 214 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/operating-systems/linux/time-management/jiffies-overlay.svg" alt="">
  <figcaption><h4>Figure: Layout of `jiffies` and `jiffies_64`. {% cite lkd -l 214 %}</h4></figcaption>
</figure>

`jiffies` can experience overflow, where the value wraps around to 0. The kernel provides four macros for comparing tick counts that can handle wraparound. Here are simplified versions of the macros:

```c
#define time_after(unknown, known) ((long)(known) - (long)(unknown) < 0)
#define time_before(unknown, known) ((long)(unknown) - (long)(known) < 0)
#define time_after_eq(unknown, known) ((long)(unknown) - (long)(known) >= 0)
#define time_before_eq(unknown, known) ((long)(known) - (long)(unknown) >= 0)
```

The parameter `unknown` is typically jiffies, `known` is the value to compare to {% cite lkd -l 215 %}.

## Hardware clocks and timers

Architectures include two hardware devices for time keeping: the system timer, and the real-time clock. The exact behavior differs between machines, but the design is usually similar.

### Real-time clock

The RTC (Real-Time Clock) provides a non-volatile device for storing system time. The RTC keeps track of time by using a battery, normally included on the system board {% cite lkd -l 216 %}.

On a PC architecture the RTC and CMOS are integrated. A single battery keeps the RTC running and the BIOS settings preserved {% cite lkd -l 219 %}.

On boot, the kernel reads the RTC and uses it to initialize wall time, which is stored in the `xtime` variable {% cite lkd -l 219 %}.

### System timer

The system timer provides a timer interrupt that runs at a regular interval. The system timer can be implemented using an electronic clock that oscillates at a programmable frequency, or with a decrementer: a counter that decrements at a fixed-rate until the counter reaches zero {% cite lkd -l 219 %}.

The primary interrupt timer on x86 is the PIT (Programmable Interrupt Timer). The kernel programs the PIT on boot to run the timer at `HZ` frequency {% cite lkd -l 219 %}.

## The timer interrupt handler

The timer interrupt is made of two pieces: the architecture-dependant routine and the architecture-independent routine {% cite lkd -l 219 %}.

The architecture-dependant routine runs each time the interrupt timer hits. The work it does depends on the architecture, but in general it will:

- Obtain the `xtime_lock` lock.
- Acknowledge or reset the system timer.
- Save the updated wall time to RTC.
- Call the architecture-independent `tick_periodic()`.

`tick_periodic()` then runs and:

- Increments `jiffies_64` value by 1.
- Updates resource usage, like consumed system and user time.
- Runs expired dynamic timers.
- Executes `scheduler_tick()`.
- Updates wall time stored in `xtime`.
- Calculates load average.

You can see the code:

```c
static void tick_periodic(int cpu) {
  if (tick_do_timer_cpu == cpu) {
    write_seqlock(&xtime_lock);

    /* Keep track of the next tick event */
    tick_next_period = ktime_add(tick_next_period, tick_period);

    do_timer(1);
    write_sequnlock(&xtime_lock);
  }

  update_process_times(user_mode(get_irq_reqs()));
  profile_tick(CPU_PROFILING);
}
```

`do_timer()` is responsible for updating `jiffies`:

```c
do_timer(unsigned long ticks) {
  jiffies_64 += ticks;
  update_wall_time();
  calc_global_load();
}
```

After `do_timer()`, `update_process_times()` is called:

```c
void update_process_times(int user_tick) {
  struct task_struct *p = current;
  int cpu = smp_processor_id();

  /* Note: this timer irq context must be accounted for as well. */
  account_process_tick(p, user_tick);
  run_local_timers();
  rcu_check_callbacks(cpu, user_tick);
  printk_tick();
  scheduler_tick();
  run_posix_cpu_timers(p);
}
```

`account_process_tick()` updates the processes time:

```c
void account_process_tick(struct task_struct *p, int user_tick)
{
	cputime_t one_jiffy_scaled = cputime_to_scaled(cputime_one_jiffy);
	struct rq *rq = this_rq();

	if (user_tick)
		account_user_time(p, cputime_one_jiffy, one_jiffy_scaled);
	else if ((p != rq->idle) || (irq_count() != HARDIRQ_OFFSET))
		account_system_time(p, HARDIRQ_OFFSET, cputime_one_jiffy,
				    one_jiffy_scaled);
	else
		account_idle_time(cputime_one_jiffy);
}
```

`run_local_timers()` then marks a softirq to execute any expired timers {% cite lkd -l 219 %}.

`scheduler_tick()` is called to decrement the currently running process's timeslice, and set `need_resched` if needed. On SMP machines it will also balance runqueues {% cite lkd -l 219 %}.

## The time of day

The time of day, known as the wall time, is defined in kernel/time/timekeeping.c:

```c
struct timespec xtime;
```

`timespec` is defined in <linux/time.h> as:

```c
struct timespec {
  __kernel_time_t tv_sec; /* seconds */
  long tv_nsec; /* nanoseconds */
}
```

The `tv_sec` value stores the number of seconds that have elapsed since January 1, 1970. This date is called the epoch. Most Unix systems use the epoch to determine the time. The `v_nsec` value stores the number of nanoseconds that have elapsed since the last second {% cite lkd -l 220 %}.

Reading or writing `xtime` requires the `xtime_lock` lock. `xtime_lock` lock is a seqlock.

To update `xtime` you need a write lock:

```c
write_seqlock(&xtime_lock);

/* update xtime */

write_sequnlock(&xtime_lock);
```

To read `xtime` you need the `read_seqbegin()` and `read_seqretry()`:

```c
unsigned long seq;

do {
  unsigned long lost;
  seq = read_seqbegin(&xtime_lock);

  usec = timer->get_offset();
  lost = jiffies - wall_jiffies;
  if (lost)
    usec += lost * (10000000 / HZ);
  sec = xtime.tv_sec;
  usec += (xtime.tv_nsec / 1000);
} while (read_seqretry(&xtime_lock, seq))
```

The main user interface for getting the wall time is `gettimeofday()`, which is implemented as `sys_gettimeofday()`:

```c
asmlinkage long sys_gettimeofday(struct timeval __user *tv, struct timezone __user *tz)
{
  if(likely(tv)) {
    struct timeval ktv;
    do_gettimeofday();
    if (copy_to_user(tv, &ktv, sizeof(sys_tz)));
      return -EFAULT;
  }

  if (unlikely(tz)) {
    if (copy_to_user(tz, &sys_tz, sizeof(sys_tz)))
      return -EFAULT;
  }
  return 0;
}
```

The architecture-dependant `do_gettimeofday()` is called if the user provides a non-null `tv` value. `do_gettimeofday()` mainly performs the read loop discussed previously {% cite lkd -l 221 %}.

The `settimeofday()` system call sets the wall time to a specified value.

## Timers

Timers, sometimes called dynamic timers or kernel timers, are used to manage the passing of time in the kernel. With kernel timers you can set functions to run at a specified point in the future {% cite lkd -l 222 %}.

Timers are represented by the `timer_list` struct, defined in <linux/timer.h>:

```c
struct timer_list {
	struct list_head entry; /* entry in linked list of timers */
	unsigned long expires; /* expiration value, in jiffies */
	void (*function)(unsigned long); /* the timer handler function */
	unsigned long data; /* lone argument to the handler */
	struct tvec_base *base; /* internal timer field, do not touch */
};
```

To create a timer you must define it:

```c
struct timer_list my_timer;
```

The timers value must then be initialised. You do this with `init_timer()`:

```c
init_timer(&my_timer);
```

Once the timer is initialized, you can add the remaining values:

```c
my_timer.expires = jiffies + delay; /* timer expires in delay ticks */
my_timer.data = 0; /* 0 is passed to the timer handler */
my_timer.function = my_function; /* function to run when timer expires */
```

Then you activate the timer:

```c
add_timer(&my_timer);
```

Although the kernel guarantees to not run a timer prior to its `expires` value, there may be a delay in running the timer {% cite lkd -l 223 %}.

## Conclusion

Time is managed in the kernel using jiffies, which are incremented each time the system timer interrupt runs.

The wall time is saved back to the RTC, which persists when the computer is powered down.

Each _tick_ of the interrupt timer causes the kernel to perform various time-based tasks, such as updating the system time, and running expired timers.

## References

{% bibliography --cited_in_order %}
