---
layout: default
title: Memory managmement
description: Notes on memory management in the Linux kernel.
nav_order: 8
parent: Linux
grand_parent: Operating systems
permalink: /operating-systems/linux/memory-management
---

<!-- prettier-ignore-start -->

# Memory Management
{:.no_toc}

This section is about memory management in the kernel: how the kernel manages memory, and how to manage memory as a kernel developer.

## Table of contents
{: .no_toc  }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Pages

A **page** is a block of virtual memory. Pages are the basic unit of memory management in the kernel {% cite lkd -l 231 %}.

Each architecture determines its page size, and many architectures can support multiple page sizes. Most 32-bit architectures support 4KB pages, and most 64-bit architectures support 8KB pages {% cite lkd -l 231 %}.

The kernel represents a physical page with the `page` struct, defined in \<linux/mm_types.h\>. A simplified copy of the struct is:

```c
struct page {
  unsigned long flags;
  atomic_t _count;
  atomic_t _mapcount;
  unsigned long private;
  struct address_space *mapping;
  pgoff_t index;
  struct list_head lru;
  void *virtual;
};
```

The `flags` field stores the status of the page. For example, whether the page is dirty or not, or whether the page is locked in memory.

The `_count` field stores the usage count of the page, i.e., how many references there are to the page. If `_count` is -1 then the page is free {% cite lkd -l 232 %}.

The `virtual` field is the page's virtual address. Although this is normally the page's virtual memory address, some memory (called **high memory**) isn't permanently mapped in the kernel's address space. For high memory the value will be `NULL` {% cite lkd -l 232 %}.

## Zones

**Zones** are groups of pages with similar hardware-assigned properties (e.g., DMA zones, normal zones) {% cite lkd -l 233 %}.

There are two main hardware limitations that are relevant to zones:

1. Some hardware devices can only perform DMA (Direct Memory Access) to certain address spaces.
2. Some architectures can physically address larger areas than they can virtually address. In these cases some memory is not permanently mapped into the kernel address space.

{% cite lkd -l 233 %}

There are four primary address zones:

1. `ZONE_DMA` contains pages that can undergo DMA.
2. `ZONE_DMA32` contains pages that can undergo DMA and are only accessible by 32-bit devices.
3. `ZONE_NORMAL` contains normal pages.
4. `ZONE_HIGHMEM` contains high memory.

{% cite lkd -l 233 %}

Memory layout and usage is architecture-specific. For architectures that can perform DMA into any memory address `ZONE_DMA` is empty, and `ZONE_NORMAL` is used for allocation, whereas for other architectures `ZONE_DMA` could contain all memory in the range 0-16MB {% cite lkd -l 233 %}.

`ZONE_NORMAL` tends to be what is remaining after `ZONE_DMA` and `ZONE_HIGHMEM`. The following table shows the zones on an x86 architecture:

| Zone           | Description                 | Physical Memory |
| -------------- | --------------------------- | --------------- |
| `ZONE_DMA`     | DMA-able pages.             | < 16MB          |
| `ZONE_NORMAL`  | Normally addressable pages. | 16–896MB        |
| `ZONE_HIGHMEM` | Dynamically mapped pages.   | > 896MB         |

{% cite lkd -l 234 %}

Zones are used during page allocation where some allocations require pages from a particular zone. Allocations must come from a single zone at once {% cite lkd -l 234 %}.

Each zone is represented by a `zone` struct:

```c
struct zone {
  unsigned long watermark[NR_WMARK];
  unsigned long lowmem_reserve[MAX_NR_ZONES];
  struct per_cpu_pageset pageset[NR_CPUS];
  spinlock_t lock;
  struct free_area free_area[MAX_ORDER]
  spinlock_t lru_lock;
  struct zone_lru {
  struct list_head list;
  unsigned long nr_saved_scan;
  } lru[NR_LRU_LISTS];
  struct zone_reclaim_stat reclaim_stat;
  unsigned long pages_scanned;
  unsigned long flags;
  atomic_long_t vm_stat[NR_VM_ZONE_STAT_ITEMS];
  int prev_priority;
  unsigned int inactive_ratio;
  wait_queue_head_t *wait_table;
  unsigned long wait_table_hash_nr_entries;
  unsigned long wait_table_bits;
  struct pglist_data *zone_pgdat;
  unsigned long zone_start_pfn;
  unsigned long spanned_pages;
  unsigned long present_pages;
  const char *name;
};
```

## Memory allocation

Memory allocation is the process of assigning sections of memory.

The main function to allocate memory from the kernel is `alloc_pages()`:

```c
struct page * alloc_pages(gfp_t gfp_mask, unsigned int order)
```

This allocates $$2^\text{order}$$ contiguous pages, and returns a pointer to the first page's `page` struct {% cite lkd -l 235 %}.

You can convert a given page to its logical address with `page_address()`:

```c
void * page_address(struct page *page)
```

`page_address()` returns a pointer to the logical address where the given physical page currently resides. If you have no need for the actual `page` struct, you can call `__get_free_pages()`:

```c
unsigned long __get_free_pages(gfp_t gfp_mask, unsigned int order)
```

`__get_free_pages()` works like `alloc_pages()` except it returns the logical address of the first page, instead of the `page` struct.

If you just need a single page, you can use the following helper functions:

```c
struct page * alloc_page(gfp_t gfp_mask)
unsigned long __get_free_page(gfp_t gfp_mask)
```

`get_zeroed_page()` returns a page filled with zeros:

```c
unsigned long get_zeroed_page(unsigned int gfp_mask)
```

`get_zeroed_page()`should be used when pages are given to user space, because the previous memory might have contained sensitive information.

You free pages with the `free()` family of functions:

```c
void __free_pages(struct page *page, unsigned int order)
void free_pages(unsigned long addr, unsigned int order)
void free_page(unsigned long addr)
```

`kmalloc()` can be used to allocate byte-size chunks. `kmalloc()` works similarly to `malloc()`, except it also takes a `flags` parameter. `kmalloc()` returns a pointer to a chunk of memory _at least_ `bytes` size in length that is physically and virtually contiguous {% cite lkd -l 238 %}.

Memory allocated with `kmalloc()` is freed with `kfree()`.

A alternative to `kmalloc()` is `vmalloc()`. `vmalloc()` returns a chunk of virtually contiguous memory (with no guarantees that it's physically contiguous) {% cite lkd -l 244 %}.

### GFP mask flags

**GFP (Get Free Page) mask flags** are flags that are passed to kernel allocator functions {% cite lkd -l 238 %}

GFP flags are represented with the `gfp_t` type. There are three types of flags:

- Action modifiers—how the kernel should allocate memory.
- Zone modifiers—what zone to allocate from.
- Type flags—a combination of a zone and an action type.

{% cite lkd -l 238-9 %}

You can see the action modifiers in the following table:

| Flag                | Description                                                                                |
| ------------------- | ------------------------------------------------------------------------------------------ |
| `__GFP_WAIT`        | The allocator can sleep.                                                                   |
| `__GFP_HIGH`        | The allocator can access emergency pools.                                                  |
| `__GFP_IO`          | The allocator can start disk I/O.                                                          |
| `__GFP_FS`          | The allocator can start filesystem I/O.                                                    |
| `__GFP_COLD`        | The allocator should use cache cold pages.                                                 |
| `__GFP_NOWARN`      | The allocator does not print failure warnings.                                             |
| `__GFP_REPEAT`      | The allocator repeats the allocation if it fails, but the allocation can potentially fail. |
| `__GFP_NOFAIL`      | The allocator indefinitely repeats the allocation. The allocation cannot fail.             |
| `__GFP_NORETRY`     | The allocator never retries if the allocation fails.                                       |
| `__GFP_NOMEMALLOC`  | The allocator does not fall back on reserves.                                              |
| `__GFP_HARDWALL`    | The allocator enforces "hardwall" cpuset boundaries.                                       |
| `__GFP_RECLAIMABLE` | The allocator marks the pages reclaimable.                                                 |
| `__GFP_COMP`        | The allocator adds compound page metadata (used internally by the hugetlb code).           |

{% cite lkd -l 239 %}

These flags can be specified together. For example:

```c
ptr = kmalloc(size, __GFP_WAIT | __GFP_IO | __GFP_FS);
```

Zone modifiers specify the zones that a memory allocation should originate from. Normally the allocation will happen from any zone, with the kernel preferring `ZONE_NORMAL`.

You can see the zone modifiers in the following table:

| Flag            | Description                                    |
| --------------- | ---------------------------------------------- |
| `__GFP_DMA`     | Allocates only from `ZONE_DMA`                 |
| `__GFP_DMA32`   | Allocates only from `ZONE_DMA32`               |
| `__GFP_HIGHMEM` | Allocates from `ZONE_HIGHMEM` or `ZONE_NORMAL` |

{% cite lkd -l 240 %}

#### Type Flags

The `type` flags are a combination of action modifiers and zone modifiers, which makes allocations easier.

You can see the type flags in the following table:

| Flag           | Description                                                                                                                                                                                                                                            |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `GFP_ATOMIC`   | The allocation is high priority and must not sleep. This is the flag to use in interrupt handlers, in bottom halves, while holding a spinlock, and in other situations where you cannot sleep.                                                         |
| `GFP_NOWAIT`   | Like `GFP_ATOMIC`, except that the call will not fallback on emergency memory pools. This increases the liklihood of the memory allocation failing.                                                                                                    |
| `GFP_NOIO`     | This allocation can block, but must not initiate disk I/O. This is the flag to use in block I/O code when you cannot cause more disk I/O, which might lead to some unpleasant recursion.                                                               |
| `GFP_NOFS`     | This allocation can block and can initiate disk I/O, if it must, but it will not initiate a filesystem operation. This is the flag to use in filesystem code when you cannot start another filesystem operation.                                       |
| `GFP_KERNEL`   | This is a normal allocation and might block. This is the flag to use in process context code when it is safe to sleep. The kernel will do whatever it has to do to obtain the memory requested by the caller. This flag should be your default choice. |
| `GFP_USER`     | This is a normal allocation and might block. This flag is used to allocate memory for user space processes.                                                                                                                                            |
| `GFP_HIGHUSER` | This is an allocation from `ZONE_HIGHMEM` and might block. This flag is used to allocate memory for user space processes.                                                                                                                              |
| `GFP_DMA`      | This is an allocation from `ZONE_DMA`. Device drivers that need DMA-able memory use this flag, usually in combination with one of the preceding flags.                                                                                                 |

{% cite lkd -l 241-2 %}

The majority of allocations in the kernel use the `GFP_KERNEL` flag which creates a normal priority memory allocation that might sleep (thus it must only be used when it's safe to sleep) {% cite lkd -l 242 %}.

## Slab allocation

**Slab allocation** is a mechanism for efficient memory allocation of objects. Slab allocation reduces memory fragmentation compared to earlier approaches {% cite lkd -l 246 %}.

In the kernel, a slab allocator divides objects into groups called caches. Each cache stores a different object (e.g. one cache for `task_struct` structs, another for `inode` structs) {% cite lkd -l 246 %}.

Caches are divided into slabs. A **Slab** consists of one or more contiguous pages that contains a number of objects of the data structure that's being cached {% cite lkd -l 246 %}.

A slab is in one of three states: full, partial, or empty. When the kernel requests a new object, the request is satisfied by a partial slab if one exists, otherwise an empty slab is used {% cite lkd -l 246 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/operating-systems/linux/memory-management/slab-cache.svg" alt="">
  <figcaption><h4>Figure: The relationship between caches, slabs, and objects {% cite lkd -l 247 %}</h4></figcaption>
</figure>

A cache is represented with the `kmem_cache` struct. `kmem_cache` contains three slab lists stored inside a `kmem_list3` struct: `slabs_full`, `slabs_partial`, and `slabs_empty`.

A `slab` struct represents a slab:

```c
struct slab {
  struct list_head list; /* full, partial, or empty list */
  unsigned long colouroff; /* offset for the slab coloring */
  void *s_mem; /* first object in the slab */
  unsigned int inuse; /* allocated objects in the slab */
  kmem_bufctl_t free; /* first free object, if any */
};
```

The slab allocator creates new slabs with the `__get_free_pages()` function:

```c
static void *kmem_getpages(struct kmem_cache *cachep, gfp_t flags, int nodeid)
{
  struct page *page;
  void *addr;
  int i;

  flags |= cachep->gfpflags;
  if (likely(nodeid == -1)) {
    addr = (void*)__get_free_pages(flags, cachep->gfporder);
    if (!addr)
      return NULL;
    page = virt_to_page(addr);
  } else {
    page = alloc_pages_node(nodeid, flags, cachep->gfporder);
    if (!page)
      return NULL;
    addr = page_address(page);
  }

  i = (1 << cachep->gfporder);
  if (cachep->flags & SLAB_RECLAIM_ACCOUNT)
    atomic_add(i, &slab_reclaim_pages);
  add_page_state(nr_slab, i);
  while (i––) {
    SetPageSlab(page);
    page++;
  }
  return addr;
}
```

Memory is freed with `kmem_freepages()`, which calls `free_pages()` on the given cache’s pages. The freeing function is only called when available memory grows low, or when a cache is explicitly destroyed {% cite lkd -l 249 %}.

A new cache is created with `kmem_cache_create()`:

```c
struct kmem_cache * kmem_cache_create(const char *name,
  size_t size,
  size_t align,
  unsigned long flags,
  void (*ctor)(void *));
```

`name` is the cache name. `size` is the size of each element in the cache. `align` is the offset of the first element in a cache, this is done to ensure a particular alignment in the first page. `ctor` is a constructor for the cache. It's called whenever a new item is added to the cache. You can pass `NULL` for `ctor` {% cite lkd -l 249-50 %}.

`flags` is a flags parameter used to control the behavior of the cache. It takes the following options:

- `SLAB_HWCACHE_ALIGN`—instructs the slab layer to align each object within a slab to a cache line.
- `SLAB_POISON`—causes the slab layer to fill the slab with a known value. This is called poisoning, and can be useful to catch access to uninitialized memory.
- `SLAB_RED_ZONE`—causes the slab layer to insert red zones around the cache to detect buffer overflows.
- `SLAB_PANIC`—causes the kernel to panic if slab allocation fails.
- `SLAB_CACHE_DMA`—instructs slab layer to allocate each slab in DMA-able memory.

{% cite lkd -l 249-50 %}

On success, `kmem_cache_create()` returns a pointer to the created cache. To destroy a cache, you call `kmem_cache_destroy()`:

```c
int kmem_cache_destroy(struct kmem_cache *cachep)
```

Once a cache is created, you can allocate objects from it using `kmem_cache_alloc()`:

```c
void * kmem_cache_alloc(struct kmem_cache *cachep, gfp_t flags)
```

To later free an object and return it to its originating slab, use `kmem_cache_free()`:

```c
void kmem_cache_free(struct kmem_cache *cachep, void *objp)
```

This marks the object `objp` in `cachep` as free {% cite lkd -l 251 %}.

## High Memory Mappings

**High memory** is memory that isn't permanently mapped into the kernel address space {% cite lkd -l 253 %}.

Pages obtained via `alloc_pages()` with the `__GFP_HIGHMEM` flag might not have a logical address {% cite lkd -l 253 %}.

To map a `page` structure into the kernel’s address space, use `kmap`, declared in \<linux/highmem.h\>:

```c
void *kmap(struct page *page)
```

`kmap()` works on both high memory and low memory. If the `page` structure represents a page in low memory, then the virtual address is returned. If the page is in low memory, a permanent mapping is created and the address is returned. `kmap()` might sleep, so it works only in process context {% cite lkd -l 254 %}.

Since the number of permanent mappings of high memory are limited, you should unmap high memory when it's no longer needed. You do this with the `kunmap()` function.

When you need to create a mapping but the current context can't sleep, you can create a temporary mapping which uses a reserved mapping {% cite lkd -l 254 %}.

Setting up a temporary mapping is done with `kmap_atomic()`:

```c
void *kmap_atomic(struct page *page, enum km_type type)
```

The mapping is undone wit `kunmap_atomic()`.

## References

{% bibliography --cited_in_order %}
