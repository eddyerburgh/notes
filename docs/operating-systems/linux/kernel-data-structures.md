---
layout: default
title: Kernel data structures
description: Notes on kernel data structures, like linked lists and queues.
nav_order: 4
parent: Linux
grand_parent: Operating systems
permalink: /operating-systems/linux/kernel-data-structures
---

<!-- prettier-ignore-start -->

# Kernel data structures
{:.no_toc}

Linux implements several data structures that are used throughout the kernel. If you want to read the Linux source code, you should learn the common data structures first.

## Table of contents
{: .no_toc  }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Linked lists

A **linked list** is a data structure that stores a variable number of nodes.

Linked list nodes are added dynamically. This means the size of a linked list doesn't need to be known at compile time.

Each node in a linked list contains a pointer to the next node. The pointers are updated as nodes are added to and removed from the list.

<figure>
  <img src="{{site.baseurl}}/assets/img/operating-systems/linux/kernel-data-structures/singly-linked-list.svg" alt="">
  <figcaption><h4>Figure: Singly linked list {% cite lkd -l 86 %}</h4></figcaption>
</figure>

The kernel mainly uses **intrusive linked lists**—a linked list variant where the list node contains only pointers to the next and previous nodes:

```c
struct list_head {
    struct list_head *next
    struct list_head *prev;
};
```

A linked list is created by embedding a `list_head` in a larger data structure and then linking the embedded `list_head` structs together. For example, tasks embed a `list_head` as `tasks`:

```c
struct task_struct {
  pid_t	pid;
  struct list_head tasks;
};
```

You can then use the `container_of()` macro to access the containing structure of a `list_head` node:

```c
struct task_struct *next_task = container_of((p)->tasks.next, struct task_struct, tasks)
```

The `container_of()` macro calculates the address of the containing object using the object's type, the name of the list member, and the memory address of the list node:

```c
#define container_of(ptr, type, member) ({ \
        const typeof( ((type *)0)->member ) *__mptr = (ptr); \
        (type *)( (char *)__mptr - offsetof(type,member) );})
```

This is possible because the offset of a given member in a structure is fixed by the ABI at compile time {% cite lkd -l 89 %}.

### Defining a linked list

A linked list needs to be initialized before it can be used.

You can create the list at runtime using `INIT_LIST_HEAD`:

```c
struct node *n;
n = kmalloc(sizeof(*n), GFP_KERNEL);
n->data = 40;
INIT_LIST_HEAD(&n->list);
```

If the list is statically created at compile time you can use `LIST_HEAD_INIT`:

```c
struct node n = {
    .data = 40,
    .list = LIST_HEAD_INIT(n.list),
};
```

### Manipulating linked lists

The kernel provides a family of functions to manipulate linked lists that can be found in [include/linux/list.h](https://elixir.bootlin.com/linux/v2.6.39.4/source/include/linux/list.h).

You can add a node to a linked list with `list_add()`:

```c
list_add(struct list_head *new, struct list_head *head)
```

Because the list is circular and has no concept of first or last, you can pass any element for `head`.

You can delete a node from a linked list with `list_del()`:

```c
list_del(struct list_head *entry)
```

`list_del()` removes the `entry` node from the list. It doesn’t free any memory belonging to entry, it just removes `entry` from the list it belongs to by modifying pointers. You can see this in the implementation:

```c
static inline void __list_del(struct list_head *prev, struct list_head *next)
{
    next->prev = prev;
    prev->next = next;
}

static inline void list_del(struct list_head *entry)
{
    __list_del(entry->prev, entry->next);
}
```

You can iterate over each item in a linked list using the `list_for_each_entry()` macro. `list_for_each_entry()` takes three parameters: `pos`, `head`, and `member`. On each iteration `pos` is a pointer to the next list entry.

You can see a real example in `inotify_watch()`, the kernel's file notification system:

```c
static struct inotify_watch *inode_find_handle(struct inode *inode,
					       struct inotify_handle *ih)
{
	struct inotify_watch *watch;

	list_for_each_entry(watch, &inode->inotify_watches, i_list) {
		if (watch->ih == ih)
			return watch;
	}

	return NULL;
}
```

## Queues

Queues are a first-in-first-out data structure. Data is removed from a queue in the order that it's added, with the oldest data removed first.

<figure>
  <img src="{{site.baseurl}}/assets/img/operating-systems/linux/kernel-data-structures/queue.svg" alt="">
  <figcaption><h4>Figure: A queue {% cite lkd -l 96 %}</h4></figcaption>
</figure>

The Linux queue implementation is called kfifo. It's implemented in [kernel/kfifo.c](https://elixir.bootlin.com/linux/v2.6.39.4/source/kernel/kfifo.c).

Kfifo has two operations: enqueue (named `in`) and dequeue (named `out`).

The kfifo object maintains an `in` offset and an `out` offset into the queue. The `in` offset is the position that the next enqueue will add data to, and the `out` offset is the position where the next dequeue will remove data from {% cite lkd -l 97 %}.

The enqueue operation copies data to the queue starting at the `in` offset. After the data is added, the `in` offset is increased by the amount that was enqueued.

The dequeue operation copies data from the queue into a buffer. After the dequeue operation, the `out` offset is incremented by the number of items removed {% cite lkd -l 97 %}.

You can define and initialize a queue either statically or dynamically. The most common way is dynamically with `kfifo_alloc()`:

```c
int kfifo_alloc(struct kfifo *fifo, unsigned int size, gfp_t gfp_mask);
```

_Note: `gfp_mask` is discussed in [the memory management section](/operating-systems/linux/memory-management)_

`kfifo_alloc()` creates and initializes a queue of `size` bytes, where `size` is a power of 2. On success `kfifo_alloc()` returns 0, on error it returns a negative code {% cite lkd -l 97 %}.

For example:

```c
struct kfifo fifo;
int ret;

ret = kfifo_alloc(&kfifo, PAGE_SIZE, GFP_KERNEL);

if(ret) {
    return ret;
}

/* fifo now represents a PAGE_SIZE sized queue */
```

You can also allocate your own buffer and create a queue using `kfifo_init()`:

```c
void kfifo_init(struct kfifo *fifo, void *buffer, unsigned int size);
```

You can statically declare a kfifo with the `DECLARE_KFIFO()` macro:

```c
DECLARE_KFIFO(name, size);
INIT_KFIFO(name);
```

Enqueuing data is done with the `kfifo_in()` function:

```c
unsigned int kfifo_in(struct kfifo *fifo, const void *from, unsigned int len);
```

This copies `len` bytes starting at `from` into the queue. The function only copies as many bytes as are free in the queue. The return value is the number of bytes that were copied successfully {% cite lkd -l 98 %}.

You can remove data from a queue with `kfifo_out()`:

```c
unsigned int kfifo_out(struct kfifo *fifo, void *to, unsigned int len);
```

`kfifo_out()` copies up to `len` bytes from the queue to the `to` buffer and returns the number of bytes copied. If the buffer has less than `len` bytes then only that amount is copied.

Once dequeued, data is no longer accessible from the queue. If you want to access data from the queue without removing it, you can the `kfifo_out_peek()` function:

```c
unsigned int kfifo_out_peek(struct kfifo *fifo, void *to, unsigned int len, unsigned offset);
```

`kfifo_is_empty()` and `kfifo_is_full()` return nonzero if the queue is empty or full respectively.

## Maps

A map is a collection of unique keys, where each key is associated with a value {% cite lkd -l 100 %}.

Maps support at least three operations:

1. Add
2. Remove
3. Value

The kernel provides a map data structure called idr. idr is used for mapping user space UIDs, like POSIX timer IDs, to their associated data structures {% cite lkd -l 100 %}.

You can define an idr by either statically or dynamically allocating an `idr` struct, then calling `idr_init()`:

```c
struct idr example_idr;
idr_init(&example_idr);
```

The next step is to allocate a new UID. First you do this by telling the idr that you want a new UID (which lets it resize the underlying tree if needed). Then you make the request for the UID {% cite lkd -l 101 %}.

The reason for this two-step process is so that the initial call, which can result in additional memory allocation, can run without requiring a lock.

The function to resize the backing tree is `idr_pre_get()`:

```c
int idr_pre_get(struct idr *idp, gfp_t gfp_mask);
```

_Note: Unlike almost every other kernel function, `idr_pre_get()` returns 1 on success and 0 on error {% cite lkd -l 101 %}._

You then call `idr_get_new()` to get the UID:

```c
int idr_get_new(struct idr *idp, void *ptr, int *id);
```

`idr_get_new()` associates the pointer `ptr` with the new UID.

You can see a full example of how these functions are used:

```c
int id;

do {
    if(!idr_pre_get(&example_idr, GFP_KERNEL))
        return -ENOSPC;
    ret = idr_get_new(&example_idr, ptr, &id);
} while (ret == -EAGAIN);
```

You can look up a value by passing the UID to `idr_find`:

```c
void *idr_find(struct idr *idp, int id);
```

## Red-black trees

Red-black trees are a type of self-balancing binary search tree. The red-black tree is Linux's primary binary tree data structure {% cite lkd -l 105 %}.

Red-black trees have a color attribute which is either red or black. They remain semi-balanced by enforcing the following rules:

1. All nodes are either red or black.
2. Leaf nodes are black.
3. Leaf nodes don't contain data.
4. All non-leaf nodes have two children.
5. If a node is red, both of its children are black.
6. The path from a node to one of its leaves contains the same number of black nodes as the shortest path to any of its other leaves.

{% cite lkd -l 105 %}

These properties ensure that the deepest leaf has a depth no more than double the depth of the shallowest leaf. Maintaining the properties during insertion and deletion will keep the tree semi-balanced {% cite lkd -l 105 %}.

The Linux implementation of red-black trees is rbtree, defined in \<lib/rbtree.h\>.

The root of an rbtree is represented by the `rb_root` struct. To create an rbtree you allocate a new `rb_root` and initialize it to `RB_ROOT`:

```c
struct rb_root root = RB_ROOT;
```

rbtree doesn't provide search or insert routines: users of rbtree must define their own. The reason for not including these is that generic programming is very difficult in C, so the kernel developers decided the most efficient approach was for each user to implement them manually using provided rbtree helper functions {% cite lkd -l 105-6 %}.

You can see this by looking at an example. The following `rb_search_page_cache()` function implements a search of Linux's page cache for a chunk of a file (represented as an inode and offset pair). The function searches the `inode` rbtree for a matching offset:

```c
static inline struct page * rb_search_page_cache(struct inode * inode,
						 unsigned long offset)
{
	struct rb_node * n = inode->i_rb_page_cache.rb_node;
	struct page * page;

	while (n)
	{
		page = rb_entry(n, struct page, rb_page_cache);

		if (offset < page->offset)
			n = n->rb_left;
		else if (offset > page->offset)
			n = n->rb_right;
		else
			return page;
	}
	return NULL;
}
```

Insert is more complicated because it needs to both search and insert:

```c
static inline struct page * __rb_insert_page_cache(struct inode * inode,
						   unsigned long offset,
						   struct rb_node * node)
{
	struct rb_node ** p = &inode->i_rb_page_cache.rb_node;
	struct rb_node * parent = NULL;
	struct page * page;

	while (*p)
	{
		parent = *p;
		page = rb_entry(parent, struct page, rb_page_cache);

		if (offset < page->offset)
			p = &(*p)->rb_left;
		else if (offset > page->offset)
			p = &(*p)->rb_right;
		else
			return page;
	}

	rb_link_node(node, parent, p);

	return NULL;
}
```

## References

{% bibliography --cited_in_order %}
