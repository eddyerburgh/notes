---
layout: default
title: Process Scheduling
description: Notes on process scheduling in the Linux kernel.
nav_order: 2
parent: Linux
grand_parent: Operating systems
permalink: /operating-systems/linux/process-scheduling
---

<!-- prettier-ignore-start -->

# Process scheduling
{:.no_toc}

Scheduling processes is one of the core roles of the kernel. To understand Linux, you must understand process scheduling.

## Table of contents
{: .no_toc  }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

The **process scheduler** is responsible for choosing which processes run and for how long. A scheduler is the basic part of a multitasking operating system like Linux.

A multitasking operating system gives the illusion that multiple tasks are running at once when in fact there is only a limited set of processors. There are two kinds of multitasking operating systems: preemptive and cooperative.

Linux is a preemptive operating system. **Preemptive operating systems** decide when to stop executing a process, and which new process should begin running. The amount of time a process runs is usually determined before it's scheduled, this is called the timeslice and it is effectively a slice of the processors time {% cite lkd -l 41 %}.

In **cooperative operating systems** the scheduler relies on the process to explicitly tell the scheduler that it's ready to stop (this is often called yielding). Cooperative operating systems have a problem where tasks that don't yield can bring down the entire operating system. The last mainstream cooperative OSes were Mac OS 9 and Windows 3.1 {% cite lkd -l 42 %}.

The Linux scheduler has gone through several iterations. The latest scheduler—CFS (the Completely Fair Scheduler)—uses the concept of fair scheduling from queue theory {% cite lkd -l 42 %}.

## Scheduling policies

Scheduling policies are the rules the scheduler follows to determine what should run and when. An effective scheduling policy needs to consider both kinds of processes: I/O-bound processes and CPU-bound processes.

**I/O-bound processes** spend most of their time waiting for I/O operations, like a network request or keyboard operation, to complete. GUI applications are usually I/O-bound because they spend most of their time waiting on user input. I/O-bound processes often run for a short time because they block while waiting for I/O operations to complete {% cite lkd -l 43 %}.

**CPU-bound processes** spend most of their time executing code. CPU-bound processes are often preempted because they don't block on I/O requests very often. An example of a CPU-bound task would be one that performs a lot of Math calculation, like MATLAB {% cite lkd -l 43 %}.

Some processes are I/O-bound and CPU-bound at different times. For example, a word processor is normally waiting for user input, but there might be regular CPU-intensive operations like spellchecking {% cite lkd -l 43 %}.

### Process priority

One type of scheduling algorithm is **priority scheduling**, which gives different tasks a priority based on their need to be processed. Higher priority tasks are run before lower priority tasks, and processes with the same priority are scheduled round-robin style {% cite lkd -l 44 %}.

The kernel uses two separate priority values. A nice value, and a real-time priority value.

The **nice value** is a number from -20 to +19 with a default of 0. The larger the nice value, the lower the priority (processes are being nice by letting other processes run in their place). Processes with a lower nice value receive a larger portion of a systems processor time, processes with a higher nice value receive a smaller portion. Nice values are the standard priority range for Unix systems, although the value is used differently across OSes. In OS X, the nice value controls the absolute timeslice allotted to a process. In Linux, the nice value controls the proportion of timeslice {% cite lkd -l 44 %}.

The **real-time priority value** can range from 0 to 99, although the value is configurable. The real-time value behaves the opposite of the nice value: a higher value means higher priority. "All real-time processes are at a higher priority than normal processes; that is the real-time values and nice values are in disjoint value spaces" {% cite lkd -l 44 %}.

### Timeslice

The timeslice value represents how long a process can run before it is preempted. The scheduler policy must decide on a default timeslice. The default timeslice is important: too long and the system will seem unresponsive, too short and the system becomes less efficient as the processor spends more time performing context switches between processes {% cite lkd -l 44 %}.

A common default timeslice value is 10ms, but Linux works differently. Instead of an absolute time, the CFS algorithm assigns a _proportion_ of the processor, so the amount of processor time depends on the current load. The assigned proportion is affected by the nice value, which acts a weight. A process with a lower nice value gets a higher weighting, and a higher nice value gets a lower weighting {% cite lkd -l 45 %}.

When a process becomes eligible to run, the decision of whether to run the process or not depends on how much of a proportion of the processor the newly runnable process has consumed. If it has run a smaller proportion than the currently executing process then it will be run, otherwise it will be scheduled to run later {% cite lkd -l 45 %}.

### Scheduling policy in action

Imagine a machine that is running only two processes: a video encoder and a text editor. The video encoder is CPU-bound, whereas the text editor is I/O-bound, because it spends much of its time waiting for user input.

The text editor should respond instantly when it receives a key press, but the video encoding can afford some latency. It doesn’t matter to the user if there’s a half second delay encoding the video, whereas a half second delay on the text editor would be noticeably laggy.

If both processes have the same nice value they will be allocated 50% of the processor. The text editor will not use much of its allocated processor time because it will spend so much time blocked, waiting for I/O. The video encoder will be able to use more than its 50% of processor time. However, when the text editor wakes up in response to user input, CFS will see that the text editor has used less than its allotted 50% and therefore less time than the video encoder. It will then preempt the video encoder and run the text editor, enabling the text editor to respond quickly to user input {% cite lkd -l 45-6 %}.

## The Linux scheduling algorithm

The Linux scheduler supports different scheduling algorithms to schedule different types of processes. These are known as **scheduler classes**. Each scheduler class has a different priority, the scheduler iterates over each scheduler class in order of priority. The highest priority scheduler class with a runnable process wins, and the winning scheduler class selects which process to run next {% cite lkd -l 46 %}.

The CFS scheduler class is the registered class for normal processes (`SCHED_NORMAL`). CFS is defined in kernel/sched_fair.c {% cite lkd -l 46 %}.

### Process scheduling in Unix systems

There are a few problems with traditional Unix scheduling where each process is given a timeslice in absolute time, and the nice value affects the absolute timeslice:

1. Mapping nice values to absolute timeslices requires a decision on default timeslice value which can cause suboptimal behavior.
2. _Nicing_ a process up or down has different effects depending on the starting nice value.
3. The absolute timeslice must be multiples of the timer tick, which can cause problems.
4. Prioritizing newly woken tasks can lead to situations where one process gets unfair amount of time scheduled.

{% cite lkd -l 47-8 %}

These problems have been solved by the CFS approach, which is to do away with timeslices and assign each process a proportion of the processor. "CFS thus yields a constant fairness but a variable switching rate" {% cite lkd -l 48 %}.

### Fair scheduling

"CFS is based on a simple concept: Model process scheduling as if the system had an ideal, perfectly multitasking processor. In such a system, each process would receive $$1/n$$ of the processor's time, where $$n$$ is the number of runnable processes, and we'd schedule them for infinitely small durations, so that in any measurable period we'd have run all $$n$$ processes for the same amount of time" {% cite lkd -l 48 %}.

CFS runs each process for a period of time, then selects the next process that has run the least. CFS calculates how long a process should run for as a function of the total number of runnable processes {% cite lkd -l 49 %}.

The nice value is used to weight the proportion of processor time a process receives. This makes the nice value yield geometric differences rather than additive differences, solving the problem where the effect of nicing a value up or down depends on the starting nice value {% cite lkd -l 49 %}.

Each process runs for a timeslice proportional to its weight divided by the total weight of all runnable processes {% cite lkd -l 49 %}.

CFS sets a targeted latency, which is the total time that all processes should run in. For example, if the targeted latency is 10ms, then 2 equally weighted task run for 5ms each, and 5 tasks would run for 2ms each. There is a minimum time that a process can run for (known as the **minimum granularity**) to avoid too much cost from context switching, set to 1ms by default {% cite lkd -l 49 %}.

## Implementation

The implementation of the scheduler is in [kernel/sched_fair](https://elixir.bootlin.com/linux/v2.6.34/source/kernel/sched_fair.c).

CFS uses the [`sched_entity`](https://elixir.bootlin.com/linux/v2.6.34/source/include/linux/sched.h#L1090) struct to keep track of time left for a process:

```c
struct sched_entity {
	struct load_weight	load;		/* for load-balancing */
	struct rb_node		run_node;
	struct list_head	group_node;
	unsigned int		on_rq;

	u64			exec_start;
	u64			sum_exec_runtime;
	u64			vruntime;
	u64			prev_sum_exec_runtime;

	u64			last_wakeup;
	u64			avg_overlap;

	u64			nr_migrations;

	u64			start_runtime;
	u64			avg_wakeup;

  /* many stat variables omitted, enabled only if CONFIG_SCHEDSTATS is set */
 };
```

The `sched_entity` is embedded in the process's `task_struct`.

The `vruntime` field on `sched_entity` stores the virtual runtime of a process, which is the actual runtime (the amount of time spent running) weighted by the number of runnable processes. The `vruntime`'s units are nanoseconds {% cite lkd -l 51 %}.

`update_curr()` manages the updating of `vruntime`. `update_curr()` is run by the system timer and also whenever a process runs or blocks:

```c
static void update_curr(struct cfs_rq *cfs_rq)
{
	struct sched_entity *curr = cfs_rq->curr;
	u64 now = rq_of(cfs_rq)->clock;
	unsigned long delta_exec;

	if (unlikely(!curr))
		return;

	/*
	 * Get the amount of time the current task was running
	 * since the last time we changed load (this cannot
	 * overflow on 32 bits):
	 */
	delta_exec = (unsigned long)(now - curr->exec_start);
	if (!delta_exec)
		return;

	__update_curr(cfs_rq, curr, delta_exec);
	curr->exec_start = now;

	if (entity_is_task(curr)) {
		struct task_struct *curtask = task_of(curr);

		trace_sched_stat_runtime(curtask, delta_exec, curr->vruntime);
		cpuacct_charge(curtask, delta_exec);
		account_group_exec_runtime(curtask, delta_exec);
	}
}
```

After `update_curr()` calculates the execution time of the current process, it passes the value to `__update_curr()`. `__update_curr()` weights the time by the number of runnable processes and updates the current process's `vruntime` with the value:

```c
/*
 * Update the current task's runtime statistics. Skip current tasks that
 * are not in our scheduling class.
 */
static inline void
__update_curr(struct cfs_rq *cfs_rq, struct sched_entity *curr,
	      unsigned long delta_exec)
{
	unsigned long delta_exec_weighted;

	schedstat_set(curr->exec_max, max((u64)delta_exec, curr->exec_max));

	curr->sum_exec_runtime += delta_exec;
	schedstat_add(cfs_rq, exec_clock, delta_exec);
	delta_exec_weighted = calc_delta_fair(delta_exec, curr);

	curr->vruntime += delta_exec_weighted;
	update_min_vruntime(cfs_rq);
}
```

### Process selection

CFS decides which task to run next by running the task with the smallest `vruntime` {% cite lkd -l 52 %}.

CFS uses a red-black tree to manage the list of runnable processes, where the tree key is the `vruntime` value. Because a red-black tree is self-balancing, the shortest `vruntime` will be the leftmost node. The function to find this node is `__pick_next_entity()` {% cite lkd -l 52-3 %}:

```c
static struct sched_entity *__pick_next_entity(struct cfs_rq *cfs_rq)
{
	struct rb_node *left = cfs_rq->rb_leftmost;

	if (!left)
		return NULL;

	return rb_entry(left, struct sched_entity, run_node);
}
```

If there are no runnable processes then `__pick_next_entity()` returns `NULL` and the idle task is scheduled {% cite lkd -l 53 %}.

`vruntime` objects are added to the red-black tree in `enqueue_entity()`:

```c
static void
enqueue_entity(struct cfs_rq *cfs_rq, struct sched_entity *se, int flags)
{
	/*
	 * Update the normalized vruntime before updating min_vruntime
	 * through callig update_curr().
	 */
	if (!(flags & ENQUEUE_WAKEUP) || (flags & ENQUEUE_MIGRATE))
		se->vruntime += cfs_rq->min_vruntime;

	/*
	 * Update run-time statistics of the 'current'.
	 */
	update_curr(cfs_rq);
	account_entity_enqueue(cfs_rq, se);

	if (flags & ENQUEUE_WAKEUP) {
		place_entity(cfs_rq, se, 0);
		enqueue_sleeper(cfs_rq, se);
	}

	update_stats_enqueue(cfs_rq, se);
	check_spread(cfs_rq, se);
	if (se != cfs_rq->curr)
		__enqueue_entity(cfs_rq, se);
}
```

Which calls `__enqueue_entity()`:

```c
/*
 * Enqueue an entity into the rb-tree:
 */
static void __enqueue_entity(struct cfs_rq *cfs_rq, struct sched_entity *se)
{
	struct rb_node **link = &cfs_rq->tasks_timeline.rb_node;
	struct rb_node *parent = NULL;
	struct sched_entity *entry;
	s64 key = entity_key(cfs_rq, se);
	int leftmost = 1;

	/*
	 * Find the right place in the rbtree:
	 */
	while (*link) {
		parent = *link;
		entry = rb_entry(parent, struct sched_entity, run_node);
		/*
		 * We dont care about collisions. Nodes with
		 * the same key stay together.
		 */
		if (key < entity_key(cfs_rq, entry)) {
			link = &parent->rb_left;
		} else {
			link = &parent->rb_right;
			leftmost = 0;
		}
	}

	/*
	 * Maintain a cache of leftmost tree entries (it is frequently
	 * used):
	 */
	if (leftmost)
		cfs_rq->rb_leftmost = &se->run_node;

	rb_link_node(&se->run_node, parent, link);
	rb_insert_color(&se->run_node, &cfs_rq->tasks_timeline);
}
```

The `rb_link_node()` function adds the new node to the tree. The `rb_insert_color()` function updates the tree to ensure it's balanced {% cite lkd -l 55 %}.

A process is removed from the tree whenever the process blocks or terminates {% cite lkd -l 55 %}. This is done with`dequeue_entity()`:

```c
static void
dequeue_entity(struct cfs_rq *cfs_rq, struct sched_entity *se, int sleep)
{
	/*
	 * Update run-time statistics of the 'current'.
	 */
	update_curr(cfs_rq);

	update_stats_dequeue(cfs_rq, se);
	if (sleep) {
#ifdef CONFIG_SCHEDSTATS
		if (entity_is_task(se)) {
			struct task_struct *tsk = task_of(se);

			if (tsk->state & TASK_INTERRUPTIBLE)
				se->sleep_start = rq_of(cfs_rq)->clock;
			if (tsk->state & TASK_UNINTERRUPTIBLE)
				se->block_start = rq_of(cfs_rq)->clock;
		}
#endif
	}

	clear_buddies(cfs_rq, se);

	if (se != cfs_rq->curr)
		__dequeue_entity(cfs_rq, se);
	account_entity_dequeue(cfs_rq, se);
	update_min_vruntime(cfs_rq);

	/*
	 * Normalize the entity after updating the min_vruntime because the
	 * update can refer to the ->curr item and we need to reflect this
	 * movement in our normalized position.
	 */
	if (!sleep)
		se->vruntime -= cfs_rq->min_vruntime;
}
```

Again the main work is done with a helper `__dequeue_entity()` function:

```c
static void __dequeue_entity(struct cfs_rq *cfs_rq, struct sched_entity *se)
{
	if (cfs_rq->rb_leftmost == &se->run_node) {
		struct rb_node *next_node;

		next_node = rb_next(&se->run_node);
		cfs_rq->rb_leftmost = next_node;
	}

	rb_erase(&se->run_node, &cfs_rq->tasks_timeline);
}
```

The main entry to the scheduler is the `schedule()` function. `schedule()` is used by the rest of the kernel to invoke the process scheduler {% cite lkd -l 56 %}.

`schedule()` calls the `pick_next_task()` function to select a task:

```c
/*
 * Pick up the highest-prio task:
 */
static inline struct task_struct *
pick_next_task(struct rq *rq)
{
	const struct sched_class *class;
	struct task_struct *p;

	/*
	 * Optimization: we know that if all tasks are in
	 * the fair class we can call that function directly:
	 */
	if (likely(rq->nr_running == rq->cfs.nr_running)) {
		p = fair_sched_class.pick_next_task(rq);
		if (likely(p))
			return p;
	}

	class = sched_class_highest;
	for ( ; ; ) {
		p = class->pick_next_task(rq);
		if (p)
			return p;
		/*
		 * Will never be NULL as the idle class always
		 * returns a non-NULL p:
		 */
		class = class->next;
	}
}
```

### Sleeping and waking up

Sleeping tasks are in a non-runnable state. There are many reasons that a task might sleep, like waiting for file I/O to complete, but a sleeping task is always waiting for an event to wake it up.

When a task needs to sleep it marks itself as sleeping, puts itself on a wait queue, removes itself from the tree of processes to run, and calls `schedule()` to pick a new process to run {% cite lkd -l 60 %}.

There are two sleeping states: `TASK_INTERRUPTIBLE` and `TASK_UNINTERRUPTABLE`. The difference between them is that `TASK_UNINTERRUPTABLE` ignores signals {% cite lkd -l 60 %}.

#### Wait queues

A wait queue is a linked list of processes waiting for an event to occur. When a wait queue's event occurs, the queued processes are awakened {% cite lkd -l 60 %}.

The recommended way to add an event to a wait queue is to:

1. Create a wait queue with the `DEFINE_WAIT()` macro.
2. Add a process to wait queue using `add_wait_queue()`.
3. Call `prepare_to_wait()` to change process state to `TASK_INTERRUPTIBLE` or `TASK_UNINTERRUPTABLE`.
4. Handle signals if the task is interruptible.
5. Check that the wait condition is true in a loop when the task awakens. Exit the loop if true.
6. When condition is true set process to `TASK_RUNNING`.

{% cite lkd -l 60 %}

You can see an example of this in `inotify_read()`:

```c
static ssize_t inotify_read(struct file *file, char __user *buf,
                size_t count, loff_t *pos)
{
    struct fsnotify_group *group;
    struct fsnotify_event *kevent;
    char __user *start;
    int ret;
    DEFINE_WAIT(wait);

    start = buf;
    group = file->private_data;

    while (1) {
        prepare_to_wait(&group->notification_waitq, &wait, TASK_INTERRUPTIBLE);

        mutex_lock(&group->notification_mutex);
        kevent = get_one_event(group, count);
        mutex_unlock(&group->notification_mutex);

        if (kevent) {
            ret = PTR_ERR(kevent);
            if (IS_ERR(kevent))
                break;
            ret = copy_event_to_user(group, kevent, buf);
            fsnotify_put_event(kevent);
            if (ret < 0)
                break;
            buf += ret;
            count -= ret;
            continue;
        }

        ret = -EAGAIN;
        if (file->f_flags & O_NONBLOCK)
            break;
        ret = -EINTR;
        if (signal_pending(current))
            break;

        if (start != buf)
            break;

        schedule();
    }

    finish_wait(&group->notification_waitq, &wait);
    if (start != buf && ret != -EFAULT)
        ret = buf - start;
    return ret;
}
```

#### Waking up

Waking up is handled by the `wake_up()` function which wakes all tasks on the given wait queue. It calls `try_to_wake_up()` on each task which sets the task `state` to `TASK_RUNNING`, adds the task to the red-black tree with `enqueue_task()`, and sets `need_resched` if the awakened task priority is higher than the current task. Generally the code that causes the event calls `wake_up()` {% cite lkd -l 61 %}.

There can be spurious wake ups where a task is woken up without the event occurring. To avoid this, sleeping should be handled in a loop that checks the condition has occurred before exiting {% cite lkd -l 61 %}.

## Preemption and context switching

**Context switching** is the switching from one runnable task to another. This is handled by the `context_switch()` function, which is called by `schedule()` when a process has been selected to run {% cite lkd -l 62 %}.

Context switching:

1. Switches the virtual memory mapping using `switch_mm()`.
2. Switches processor state (including saving and restoring stack and register values) using `switch_to()`.

"The kernel provides a `need_resched` flag to signify whether a reschedule should be performed". The flag is set by `scheduler_tick()` when a process should be preempted, and by `try_to_wake_up()` when a process with higher priority than the currently running process is woken up {% cite lkd -l 62 %}.

Each time the kernel returns to user space from a system call or returns to user space from an interrupt handler, the `need_resched` flag is checked. If `need_resched` is set, the kernel calls `schedule()` to invoke the scheduler. The flag is per-process instead of global {% cite lkd -l 62 %}.

### Kernel preemption

Since Linux 2.6, the kernel code has been preemptive. Any task can be rescheduled as long as the kernel is in a state where it’s safe to reschedule (as long as the kernel process doesn’t hold a lock).

This is implemented with a `preempt_count` value added to each process's `thread_info`. The value begins at 0. Each time a lock is acquired the value is incremented by 1, each time a lock is released the value is decremented by 1. If the value is 0 then it’s safe to preempt {% cite lkd -l 63 %}.

Kernel preemption can occur:

- When an interrupt handler exits, before returning to kernel space.
- When kernel code becomes preemptible again.
- When a task in the kernel calls `schedule()`.
- When a task in the kernel blocks.

{% cite lkd -l 63 %}

## Real-time scheduling policies

Linux includes two real-time scheduling policies: `SCHED_FIFO`, and `SCHED_RR`. These policies are managed by a real-time scheduler (defined in [kernel/sched_rt.c](https://elixir.bootlin.com/linux/v2.6.39.4/source/kernel/sched_rt.c)), rather than by the CFS {% cite lkd -l 64 %}.

`SCHED_FIFO` is a first-in-first-out scheduler that doesn’t use timeslices. A `SCHED_FIFO` task is scheduled before any `SCHED_NORMAL` tasks. "Only a higher priority `SCHED_FIFO` or `SCHED_RR` can preempt a `SCHED_FIFO` task". A `SCHED_FIFO` task runs until it blocks or yields the processor. `SCHED_FIFO` with the same priority run round-robin style {% cite lkd -l 64 %}.

`SCHED_RR` is identical to `SCHED_FIFO` except each task runs for a specific timeslice.

The real-time policies use static priorities. "This ensures that a real-time process at a given priority always preempts a process at a lower priority" {% cite lkd -l 64 %}.

Linux's real-time scheduling provides soft real-time behavior. That means Linux attempts to schedule real-time processes within a deadline, but it's not always able to.

Real-time priorities range from 0 to `MAX_RT_PRIO` - 1. The default value of `MAX_RT_PRIO` is 100. The priority space is shared by nice values, which range from `MAX_RT_PRIO` to `MAX_RT_PRIO` + 40. By default the -20 to 19 of nice values map to priority space 100 to 139 {% cite lkd -l 64 %}.

## Scheduler related system calls

Linux has system calls for managing scheduler parameters. You can use them to set process priority, and scheduling policy, as well as yielding the processor {% cite lkd -l 64 %}.

| System Call                | Description                         |
| -------------------------- | ----------------------------------- |
| `nice()`                   | Sets a process's nice value         |
| `sched_setscheduler()`     | Sets a process's scheduling policy  |
| `sched_getscheduler()`     | Gets a process's scheduling policy  |
| `sched_setparam()`         | Sets a process's real-time priority |
| `sched_getparam()`         | Gets a process's real-time priority |
| `sched_get_priority_max()` | Gets the maximum real-time priority |
| `sched_get_priority_min()` | Gets the minimum real-time priority |
| `sched_rr_get_interval()`  | Gets a process's timeslice value    |
| `sched_setaffinity()`      | Sets a process's processor affinity |
| `sched_getaffinity()`      | Gets a process's processor affinity |
| `sched_yield()`            | Temporarily yields the processor    |

{% cite lkd -l 65 %}

### Processor Affinity System Calls

A hard process affinity tells the scheduler that the process must be kept running on this subset of processors no matter what.

The hard affinity is stored in a bitmask in the task's `task_struct` as `cpus_allowed`. The bitmask contains one bit for each possible processor. Initially they are all set, so a process can run on any machine. The user can set a different bitmask with `sched_setaffinity()` to change which processors the process can run on {% cite lkd -l 65 %}.

### Yielding processor time

The `sched_yield()` system call explicitly yields the processor to other waiting tasks.

`sched_yield()` works by removing the process from the active array and inserting it to the expired array {% cite lkd -l 66 %}.

## Conclusion

The process scheduler is an important part of Linux. It gives the impression that multiple processes are running simultaneously, when in fact the number of running processes is limited by the number of processors.

Different tasks have different requirements (I/O-bound vs CPU-bound). The scheduler must consider each of these cases in order to provide a responsive and effective experience.

## References

{% bibliography --cited_in_order %}
