---
layout: default
title: Interrupts
description: Notes on how Linux implements interrupts.
nav_order: 5
parent: Linux
grand_parent: Operating systems
permalink: /operating-systems/linux/interrupts
---

<!-- prettier-ignore-start -->

# Interrupts
{:.no_toc}

Most hardware communicates with the processor via interrupts, which makes interrupts an important topic to understand.

## Table of contents
{: .no_toc  }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Interrupts

**Interrupts** are a mechanism for hardware, like a mouse or a keyboard, to signal to the kernel that attention is needed. **Interrupt handlers** are the functions that respond to interrupts {% cite lkd -l 113 %}.

An interrupt is produced by the hardware devices sending an electronic signal to a pin on an interrupt controller. An **interrupt controller** is a chip that multiplexes multiple interrupt lines into a single line. When the interrupt controller receives an interrupt, it sends a signal to the processor, which notifies the operating system that an interrupt has occurred. The operating system then handles the interrupt {% cite lkd -l 114 %}.

Devices are differentiated by a unique interrupt value. The interrupt values are often known as interrupt request (**IRQ**) lines. Each IRQ line is assigned a number. For example, on the classic PC, IRQ 0 is the timer interrupt. Some devices have dynamic interrupt numbers, like devices on the PCI {% cite lkd -l 114 %}.

## Interrupt handlers

Each device that generates an interrupt has an associated interrupt handler. The interrupt handler is part of the device driver {% cite lkd -l 114 %}.

In Linux, interrupt handlers are C functions that match a prototype, making it so the kernel can pass the handler information in a standard way. The handlers run in a special interrupt context {% cite lkd -l 115 %}.

Interrupt handlers should execute quickly so that the hardware can continue working. The problem is that some interrupt handlers need to do a lot of work, like a network interrupt handler that needs to copy packets into memory, process them, and send the packets to the correct protocol {% cite lkd -l 115 %}.

To solve this problem, interrupt processing is split into two parts: the top half and the bottom half. The top half runs immediately and performs time-critical work, the bottom half performs processing that can run in the future {% cite lkd -l 115 %}.

## Registering an interrupt handler

Interrupts are the responsibility of the device driver that's managing the hardware. If the device uses interrupts then the driver must register an interrupt handler {% cite lkd -l 115 %}.

Drivers can register an interrupt handler with the `request_irq()` function:

```c
/* request_irq: allocate a given interrupt line */
int request_irq(unsigned int irq,
                irq_handler_t handler,
                unsigned long flags,
                const char *name,
                void *dev)
```

`irq` specifies the interrupt number that should be allocated. This number is normally determined dynamically.

`handler` is a function pointer to the interrupt handler function that is invoked when the OS receives an interrupt.

`flags` is a bit mask of possible flags. Examples are a flag to disable other interrupts while an interrupt handler is running, and a flag that sets interrupts to contribute to the kernel entropy pool {% cite lkd -l 116-7 %}.

`name` is an ASCII representation of the device associated with the interrupt. These are used by /proc/irq and /proc/interrupts to communicate with the user {% cite lkd -l 117 %}.

`dev` is used for shared interrupt lines. `dev` provides a unique cookie to enable the removal of the desired interrupt handler from shared interrupt lines {% cite lkd -l 117 %}.

## Writing an interrupt handler

The following is a declaration of an interrupt handler:

```c
static irqreturn_t intr_handler(int irq, void *dev)
```

`irq` is the numeric value of the interrupt line. `dev` is a pointer to the `dev` passed to `request_irq()`.

The return type `irqreturn_t` can be either `IRQ_HANDLED` or `IRQ_NONE`. `IRQ_HANDLED` means the interrupts was handled successfully, and `IRQ_NONE` should be returned if the interrupt handler was called for the incorrect device {% cite lkd -l 119 %}.

A shared handler is registered and executed similarly to a nonshared handler. The main differences are:

1. The `IRQF_SHARED` flag must be set in the `request_irq()` `flags` argument.
2. The `dev` argument must be unique to each registered handler.
3. The interrupt handler must be able to detect whether its device generated the interrupt. This requires hardware support and logic in the interrupt handler.

{% cite lkd -l 119 %}

When the kernel receives an interrupt, it invokes each registered interrupt handler for the line.

### RTC interrupt handler

An RTC (Real-Time Clock) is a device that sets the system clock, provides an alarm, or provides a periodic timer. On most architectures, the system clock is set by writing the desired time to a specific register. An alarm or periodic timer is normally implemented with an interrupt {% cite lkd -l 120 %}.

When the RTC driver loads, the `rtc_init()` function is invoked to initialize the driver. `rtc_init()` registers the interrupt handler:

```c
/* register rtc_interrupt on rtc_irq */
if (request_irq(rtc_irq, rtc_interrupt, IRQF_SHARED, "rtc", (void *)&rtc_port)) {
        printk(KERN_ERR "rtc: cannot register IRQ %d\n", rtc_irq);
        return -EIO;
}
```

The interrupt line is stored in `rtc_irq()` which is set to the RTC interrupt for a given architecture. The second parameter is the interrupt handler, and the `IRQF_SHARED` flag is set because it can share an interrupt line with other handlers {% cite lkd -l 120 %}.

You can see the full handler code:

```c
static irqreturn_t rtc_interrupt(int irq, void *dev_id)
{
    /*
     *  Can be an alarm interrupt, update complete interrupt,
     *  or a periodic interrupt. We store the status in the
     *  low byte and the number of interrupts received since
     *  the last read in the remainder of rtc_irq_data.
     */

    spin_lock(&rtc_lock);
    rtc_irq_data += 0x100;
    rtc_irq_data &= ~0xff;
    if (is_hpet_enabled()) {
        /*
         * In this case it is HPET RTC interrupt handler
         * calling us, with the interrupt information
         * passed as arg1, instead of irq.
         */
        rtc_irq_data |= (unsigned long)irq & 0xF0;
    } else {
        rtc_irq_data |= (CMOS_READ(RTC_INTR_FLAGS) & 0xF0);
    }

    if (rtc_status & RTC_TIMER_ON)
        mod_timer(&rtc_irq_timer, jiffies + HZ/rtc_freq + 2*HZ/100);

    spin_unlock(&rtc_lock);

    /* Now do the rest of the actions */
    spin_lock(&rtc_task_lock);
    if (rtc_callback)
        rtc_callback->func(rtc_callback->private_data);
    spin_unlock(&rtc_task_lock);
    wake_up_interruptible(&rtc_wait);

    kill_fasync(&rtc_async_queue, SIGIO, POLL_IN);

    return IRQ_HANDLED;
}
```

`rtc_interrupt()` is invoked whenever the machine receives the RTC interrupt.

Note the spin lock calls. The first set ensures that `rtc_irq_data` is not accessed concurrently by another processor on an SMP machine. The `rtc_irq_data` variable is an `unsigned long` that stores information about the RTC and is updated on each interrupt to reflect the status of the interrupt. The second set protects `rtc_callback`. Locks are discussed in [Kernel synchronization](./kernel-synchronization)

Next, if an RTC periodic timer is set, it is updated via `mod_timer()`. Timers are discussed in [Timer management](./time-management).

The code under the comment "now do the rest of the actions" executes a possible preset callback function. The RTC driver enables a callback function to be registered and executed on each RTC interrupt.

Finally, `rtc_interrupt()` returns `IRQ_HANDLED` to signify that it properly handled this device. Because the interrupt handler does not support sharing, and there is no mechanism for the RTC to detect a spurious interrupt, this handler always returns `IRQ_HANDLED` {% cite lkd -l 121 %}.

## Interrupt context

When the kernel is executing an interrupt handler, it is running in **interrupt context** {% cite lkd -l 122 %}.

In interrupt context there isn't a backing process, which means a handler can't sleep. This limits the functions you can call from an interrupt handler running in interrupt context.

Interrupt handlers receive their own one page sized stack, called the interrupt stack {% cite lkd -l 122 %}.

## Implementing interrupt handlers

The implementation of interrupt handlers is architecture specific. "A device issues an interrupt by sending an electric signal over its bus to the interrupt controller." If the interrupt line is enabled, the interrupt controller sends the interrupt to the processor. In most architectures this is done by an electric signal sent over a pin to the processor. As long as interrupts are not disabled in the processor, the processor stops what it’s doing, disables the interrupt system, and jumps to a predefined location in memory to execute. This predefined point (the entry point for interrupt handlers) is set up by the kernel {% cite lkd -l 123 %}.

"For each interrupt line, the processor jumps to a unique location in memory and executes the code located there". This is how the kernel knows the IRQ number of the interrupt. The entry point saves the IRQ number value, and stores the current registers on the stack. Then the kernel calls `do_IRQ()`. Most of the code after `do_IRQ()` is C code, but it's still architecture specific {% cite lkd -l 123 %}.

`do_IRQ()` is declared as:

```c
unsigned int do_IRQ(struct pt_regs regs)
```

`regs` contains the initial register values that were previously saved in the assembly entry routine. `do_IRQ()` can extract the IRQ number from the `regs`.

`do_IRQ()` acknowledges the receipt of the interrupt and disables delivery on the line. `do_IRQ()` then ensures that a valid handler is registered, enabled, and not currently executing. If there is an enabled and runnable handler then it calls `handle_irq_event()`:

```c
/**
 * handle_IRQ_event - irq action chain handler
 * @irq:    the interrupt number
 * @action: the interrupt action chain for this irq
 *
 * Handles the action chain of an irq event
 */
irqreturn_t handle_IRQ_event(unsigned int irq, struct irqaction *action)
{
    irqreturn_t ret, retval = IRQ_NONE;
    unsigned int status = 0;

    if (!(action->flags & IRQF_DISABLED))
        local_irq_enable_in_hardirq();

    do {
        trace_irq_handler_entry(irq, action);
        ret = action->handler(irq, action->dev_id);
        trace_irq_handler_exit(irq, action, ret);

        switch (ret) {
        case IRQ_WAKE_THREAD:
            /*
             * Set result to handled so the spurious check
             * does not trigger.
             */
            ret = IRQ_HANDLED;

            /*
             * Catch drivers which return WAKE_THREAD but
             * did not set up a thread function
             */
            if (unlikely(!action->thread_fn)) {
                warn_no_thread(irq, action);
                break;
            }

            /*
             * Wake up the handler thread for this
             * action. In case the thread crashed and was
             * killed we just pretend that we handled the
             * interrupt. The hardirq handler above has
             * disabled the device interrupt, so no irq
             * storm is lurking.
             */
            if (likely(!test_bit(IRQTF_DIED,
                         &action->thread_flags))) {
                set_bit(IRQTF_RUNTHREAD, &action->thread_flags);
                wake_up_process(action->thread);
            }

            /* Fall through to add to randomness */
        case IRQ_HANDLED:
            status |= action->flags;
            break;

        default:
            break;
        }

        retval |= ret;
        action = action->next;
    } while (action);

    if (status & IRQF_SAMPLE_RANDOM)
        add_interrupt_randomness(irq);
    local_irq_disable();

    return retval;
}
```

First, interrupts are turned back on (unless they were disabled during the handler's registration). Next, each handler in the `action` list is executed in a loop. Then, if `IRQF_SAMPLE_RANDOM` is set, `add_interrupt_randomness()` is called with the `irq` to generate entropy for the random number generator. Finally interrupts are disabled (`local_irq_disable()`), and the function returns {% cite lkd -l 125 %}.

"Back in `do_IRQ()`, the function cleans up and returns to the initial entry point, which then jumps to `ret_from_intr()`" {% cite lkd -l 125 %}.

`ret_from_intr()` is written in assembly. `ret_from_intr()` checks whether there is a reschedule pending. If a reschedule is pending and the kernel is returning to user space, `schedule()` is called. If the kernel is returning to kernel space, `schedule()` is only called if the `preempt_count` is 0. After `schedule()` returns, the initial registers are restored and the kernel resumes where it was interrupted {% cite lkd -l 125 %}.

## Top halves and Bottom halves

Interrupt handlers are constrained:

- They must run quickly to avoid stalling interrupted code.
- They must run quickly to stop disabling other interrupts while they run.
- They are unable to make blocking calls.
- They are often time critical.

{% cite lkd -l 133 %}

To ensure interrupt handlers run quickly, they should be split into a **top half** and a **bottom half**. The top half should run immediately and return quickly, the bottom half can perform non-critical processing at a later time.

## Bottom halves

As much work as possible should be done in the bottom half. There are multiple ways to write the bottom half:

1. Softirqs are a set of statically defined bottom halves that can run simultaneously on any processor.
2. Tasklets are dynamically created bottom halves built on top of softirqs
3. Work queues are functions that are run in a kernel worker thread

{% cite lkd -l 136 %}

Tasklets are suitable for most bottom halves. Softirqs are more performant but more difficult to use because they can run on multiple processors and are statically defined. {% cite lkd -l 136 %}

## Softirqs

Softirqs are rarely used directly: there are only 9 softirqs in the 2.6 kernel. However, tasklets are built on softirqs, so it's important to understand softirqs before discussing tasklets {% cite lkd -l 137 %}.

Softirqs are statically allocated at compile time. You can't dynamically allocate softirqs {% cite lkd -l 137 %}.

Softirqs are represented with the `softirq_action` struct:

```c
struct softirq_action
{
    void    (*action)(struct softirq_action *);
};
```

A 32-entry array of `softirq_action` is declared in [kernel/softirq.c](https://elixir.bootlin.com/linux/v2.6.39.4/source/kernel/softirq.c). This limits the number of softirqs to `NR_SOFTIRQS` (32) {% cite lkd -l 138 %}:

```c
static struct softirq_action softirq_vec[NR_SOFTIRQS];
```

The softirq `action` handler prototype looks like:

```c
void softirq_handler(struct softirq_action *);
```

When the kernel executes a softirq handler, it executes the `action` function with a pointer to the `softirq_action` action as its argument. The benefit of passing the entire structure is that it makes it easy to extend the data passed to the action by updating the `softirq_action` structure {% cite lkd -l 138 %}.

A softirq never preempts another softirq: only an interrupt handler can preempt a softirq {% cite lkd -l 138 %}.

A softirq needs to be marked before it will execute, known as _raising the softirq_. "Usually, an interrupt handler marks its softirq for execution before returning". Pending softirqs are checked for, and executed in the following places:

- In the return from the hardware interrupt.
- In the `ksoftirq` kernel thread.
- In code that explicitly checks for pending softirqs, like the networking subsystem.

{% cite lkd -l 138 %}

Softirq execution occurs in `__do_softirq()`. `__do_softirq()` loops over each pending softirq and calls its handler. The following is a simplified variant of `__do_softirq()`:

```c
u32 pending;

pending = local_softirq_pending();
if (pending) {
    struct softirq_action *h;

    /* reset the pending bitmask */
    set_softirq_pending(0);

    h = softirq_vec;
    do {
        if (pending & 1)
            h->action(h);
        h++;
        pending >>= 1;
    } while (pending);
}
```

The above code performs the following steps:

1. Sets the `pending` local variable to a 32-bit mask of pending softirqs. If bit $$n$$ is set then the $$nth$$ softirq is pending.
2. Clears the actual bitmask. This occurs with interrupts disabled, although it is not included in the simplified example.
3. `h` is set to the first entry in `softirq_vec`.
4. If first bit is set (first softirq is pending) then `h->action(h)` is called.
5. `h` is incremented by 1 so that it points to the the next entry in the `softirq_vec` array.
6. The `pending` bitmask is right-shifted by one, so the second bit is now the first.
7. Repeat previous steps until pending is 0 (meaning there are no more pending softirqs).

{% cite lkd -l 139 %}

Softirqs are reserved for the most time critical bottom-half processing. In Linux 2.6 only 2 subsystems directly use it: networking and block devices, but timers and tasklets are built on top of softirqs {% cite lkd -l 140 %}.

Softirqs are declared at compile time using an enum in \<linux/interrupt.h>\. The enum value (which starts at 0) is used as a relative priority. Softirqs with lower values are executed before softirqs with higher values {% cite lkd -l 140 %}.

As part of creating a new softirq, you must add a new entry to the enum. When you add a new softirq you should decide where in the list it should go in terms of priority. It doesn't always make sense to put it on the end of the list. By convention, `HI_SOFTIRQ` is always the first and `RCU_SOFTIRQ` is always the last entry. A new entry likely belongs in between `BLOCK_SOFTIRQ` and `TASKLET_SOFTIRQ` {% cite lkd -l 140 %}.

You can see a table of the existing tasklet types:

| Tasklet           | Priority | Softirq Description      |
| ----------------- | -------- | ------------------------ |
| `HI_SOFTIRQ`      | 0        | High-priority tasklets   |
| `TIMER_SOFTIRQ`   | 1        | Timers                   |
| `NET_TX_SOFTIRQ`  | 2        | Send network packets     |
| `NET_RX_SOFTIRQ`  | 3        | Receive network packets  |
| `BLOCK_SOFTIRQ`   | 4        | Block devices            |
| `TASKLET_SOFTIRQ` | 5        | Normal priority tasklets |
| `SCHED_SOFTIRQ`   | 6        | Scheduler                |
| `HRTIMER_SOFTIRQ` | 7        | High-resolution timers   |
| `RCU_SOFTIRQ`     | 8        | RCU locking              |

The softirq handler is then registered at run-time with `open_softirq()`. `open_softirq()` takes two parameters: the index of the softirq and its handler function.

For example, the networking subsystem registers its softirqs like this:

```c
open_softirq(NET_TX_SOFTIRQ, net_tx_action);
open_softirq(NET_RX_SOFTIRQ, net_rx_action);
```

The softirq handlers run with interrupts enabled and cannot sleep. While a handler runs, softirqs are disabled on the current processor. However, another processor can execute other softirqs. If the same softirq is raised again while it's executing, another process can run it simultaneously. This means shared data needs locking. Rather than using locking, most softirqs use per-process data {% cite lkd -l 141 %}.

Once a softirq is added to the enum list and registered, it's ready to run. To mark it as pending you use the `open_softirq()` function. The softirq will then run the next time `do_softirq()` is called {% cite lkd -l 141 %}.

## Tasklets

Tasklets are softirqs. They are represented by two softirqs: `HI_SOFTIRQ` and `TASKLET_SOFTIRQ`, where `HI_SOFTIRQ`-based tasklets run before `TASKLET_SOFTIRQ`-based tasklets.

Tasklets are represented with the `tasklet_struct` struct. The struct is declared in \<linux/interrupt.h>\:

```c
struct tasklet_struct {
    struct tasklet_struct *next;  /* next tasklet in the list */
    unsigned long state;          /* state of the tasklet */
    atomic_t count;               /* reference counter */
    void (*func)(unsigned long);  /* tasklet handler function */
    unsigned long data;           /* argument to the tasklet function */
};
```

`func()` is the tasklet handler, and it receives `data` as its argument. The `state` member is either 0, `TASKLET_STATE_SCHED`, or `TASKLET_STATE_RUN`. `TASKLET_STATE_SCHED` marks a tasklet that's scheduled to run, and `TASKLET_STATE_RUN` marks a tasklet that's running. `count` is used as a reference count for the tasklet.

Scheduled tasklets are stored in two per-processor structures: `tasklet_vec` and `tasklet_hi_vec`. They are both linked lists of `tasklet_struct` structures {% cite lkd -l 143 %}.

Tasklets are scheduled with the `tasklet_schedule()` and `tasklet_hi_schedule()` functions, which receive a pointer to a tasklets `tasklet_struct` as an argument. Each function checks that the provided tasklet is not yet scheduled and then runs `__tasklet_schedule()` and `__tasklet_hi_schedule()` respectively. Both functions work similarly {% cite lkd -l 143 %}.

The steps that `tasklet_schedule()` takes are:

1. Check that tasklets state is `TASKLET_STATE_SCHED`, if it is then the task is already scheduled to run and the function can exit early.
2. Call `__tasklet_schedule()`.
3. Save the state of the interrupt system and disable local interrupts so that nothing on this processor will change the tasklet code while `tasklet_schedule()` is manipulating the tasklet.
4. Add the tasklet to be scheduled to the head of `tasklet_vec`.
5. Raise the `TASKLET_SOFTIRQ` so that `do_softirq()` executes this tasklet.
6. Restore interrupts to their previous state and return.

{% cite lkd -l 143 %}

`do_softirq()` is then run at the next possible moment, which will execute the associated handlers of the `TASKLET_SOFTIRQ` or `HI_SOFTIRQ` that were raised. The handlers (`tasklet_action()` and `tasklet_hi_action()`) process the tasklets. They run the following steps:

1. Disable local interrupt delivery and retrieve the `tasklet_vec` or `tasklet_hi_vec` list for the process.
2. Clear the list for the processor by setting it to `NULL`.
3. Enable local interrupt delivery.
4. Loop over each pending tasklet in the retrieved list.
5. Check whether tasklet is running on another process, and skip if so.
6. Set the `TASKLET_STATE_RUN` flag.
7. Check that count is 0 to ensure tasklet isn't disabled.
8. Run the tasklet handler.
9. Clear the `TASKLET_STATE_RUN` flag.
10. Repeat for next pending task until there are no more tasks to be scheduled.

{% cite lkd -l 143-4 %}

### Using tasklets

You can create tasklets either statically or dynamically.

To statically create a tasklet you can use the macros `DECLARE_TASKLET()` and `DECLARE_TASKLET_DISABLED()`:

```c
DECLARE_TASKLET(name, func, data)
DECLARE_TASKLET_DISABLED(name, func, data);
```

Both these macros statically create a `tasklet_struct` with a given name. When a tasklet is scheduled, `func()` is executed with `data` as an argument {% cite lkd -l 144 %}.

You can create a tasklet with a pointer to a dynamically created `tasklet_struct` with `tasklet_init()`:

```c
tasklet_init(t, tasklet_handler, dev); /* dynamically as opposed to statically */
```

The tasklet handler must match the prototype:

```c
void tasklet_handler(unsigned long data)
```

Like softirqs, tasklets can't sleep. You can't use semaphores or other blocking functions in a tasklet. Tasklets also run with interrupts enabled, but two of the same tasklets never run concurrently {% cite lkd -l 145 %}.

You schedule a tasklet using `tasklet_schedule()` function, which takes a pointer to your `task_struct`:

```c
tasklet_schedule(&my_tasklet); /* mark my_tasklet as pending */
```

When a tasklet has been scheduled it will run once at some time in the future. A tasklet always runs on the processor that scheduled it {% cite lkd -l 145 %}.

### ksoftirqd

Softirq processing is helped by a set of per-processor kernel threads. Since tasklets are softirqs, the kernel threads also impact tasklets {% cite lkd -l 146 %}.

The reason for the processes is to solve a problem that occurs with softirq scheduling. The problem is that softirq functions can reschedule themselves, which means situations can occur where a high frequency of softirqs rescheduling themselves would starve user processes of processor time if they were processed immediately. However, not processing reactivated softirqs is also unacceptable {% cite lkd -l 146 %}.

The solution is to not immediately process reactivated softirqs. "Instead, if the number of softirqs grows excessive, the kernel wakes up a family of kernel threads to handle the load". The kernel threads run with the lowest priority to ensure they don't run in place of any important process. This solution means that heavy softirq activity will not starve user space processes, but at the same time it ensures that excess softirqs will run eventually. An added benefit is that softirqs will be handled quickly on an idle system {% cite lkd -l 147 %}.

Each thread is named ksoftirqd/n, where n is the processor number. The benefit of a thread on each processor means that an idle processor can always execute softirqs {% cite lkd -l 147 %}.

The threads run in a tight loop, similar to this:

```c
for (;;) {
    if (!softirq_pending(cpu))
        schedule();

    set_current_state(TASK_RUNNING);

    while (softirq_pending(cpu)) {
        do_softirq();
        if (need_resched())
            schedule();
    }

    set_current_state(TASK_INTERRUPTIBLE);
}
```

"Softirq threads are awakened whenever `do_softirq()` detects an executed kernel thread reactivating itself" {% cite lkd -l 147 %}.

## Work queues

Work queues defer work into a kernel thread. They run in process context, so they can sleep {% cite lkd -l 148 %}.

If your process doesn’t need to sleep you should use a tasklet/softirq. If it does, use a work queue {% cite lkd -l 151 %}.

"The work queue subsystem is an interface for creating kernel threads to handle work queued from elsewhere". The kernel threads created by work queues are known as worker threads {% cite lkd -l 151 %}.

The default worker threads are named events/n, where n is the processor number. There is one per processor.

Worker threads are represented by the `workqueue_struct` struct:

```c
/*
 * The externally visible workqueue abstraction is an array of
 * per-CPU workqueues:
 */
struct workqueue_struct {
    struct cpu_workqueue_struct cpu_wq[NR_CPUS];
    struct list_head list;
    const char *name;
    int singlethread;
    int freezeable;
    int rt;
};
```

The `cpu_wq` member is a list of `cpu_workqueue_struct` with one per processor on the system {% cite lkd -l 151 %}.

The `cpu_workqueue_struct` struct is defined in [kernel/workqueue.c](https://elixir.bootlin.com/linux/v2.6.39.4/source/kernel/workqueue.c):

```c
struct cpu_workqueue_struct {
    spinlock_t lock;             /* lock protecting this structure */
    struct list_head worklist;   /* list of work */
    wait_queue_head_t more_work;
    struct work_struct *current_struct;
    struct workqueue_struct *wq; /* associated workqueue_struct */
    task_t *thread;              /* associated thread */
};
```

The actual work that needs to be done is represented as a linked list of `work_struct` structs:

```c
struct work_struct {
    atomic_long_t data;
    struct list_head entry;
    work_func_t func;
};
```

When a worker thread wakes up, it runs each of the work in its list. When it completes work it removes the `work_struct` from the list, and it goes back to sleep once the list is empty {% cite lkd -l 151 %}.

You can see a simplified example of `worker_thread`:

```c
for (;;) {
    prepare_to_wait(&cwq->more_work, &wait, TASK_INTERRUPTIBLE);
    if (list_empty(&cwq->worklist))
        schedule();
    finish_wait(&cwq->more_work, &wait);
    run_workqueue(cwq);
}
```

`run_workqueue()` performs the deferred work:

```c
while (!list_empty(&cwq->worklist)) {
    struct work_struct *work;
    work_func_t f;
    void *data;
    work = list_entry(cwq->worklist.next, struct work_struct, entry);
    f = work->func;
    list_del_init(cwq->worklist.next);
    work_clear_pending(work);
    f(work);
}
```

To use a work queue you can either create it statically at runtime with the `DECLARE_WORK()` macro:

```c
DECLARE_WORK(name, void (*func)(void *), void *data);
```

Or create work dynamically at runtime with the `INIT_WORK()` macro:

```c
INIT_WORK(struct work_struct *work, void (*func)(void *), void *data);
```

The work queue handler prototype is:

```c
void work_handler(void *data)
```

A worker thread executes the handler function. Although the handler runs in process context, it cannot access user space memory because there is no associated user space memory for kernel threads {% cite lkd -l 153 %}.

Work can be scheduled with `schedule_work()`:

```c
schedule_work(&work);
```

You can also delay the work with the `schedule_delayed_work()` function:

```c
schedule_delayed_work(&work, delay);
```

You can flush a worker queue using the `flush_scheduled_work()` function:

```c
void flush_scheduled_work(void);
```

## References

{% bibliography --cited_in_order %}
