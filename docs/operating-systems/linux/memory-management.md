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

Memory management in the kernel is more constrained than memory management in user space. The kernel can't deal well with memory allocation errors, and often the kernel must allocate memory without sleeping.

This section is about memory management in the kernel: how the kernel manages memory, and how to manage memory as a kernel developer.

## Table of contents
{: .no_toc  }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Pages

"The kernel treats physical pages as the basic unit of memory management." The hardware memory management unit (MMU) that manages the translation between virtual and physical memory, typically deals in pages. The MMU maintains the system's page tables with page-size granularity, and so in terms of virtual memory, pages are the smallest size that matters {% cite lkd -l 231 %}.

Each architecture determines its page size, and many architectures can support multiple page sizes. "Most 32-bit architectures have 4KB pages, whereas most 64-bit architectures have 8KB pages". So a machine with 8KB pages and 1GB memory is split into 131,072 pages {% cite lkd -l 231 %}.

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

The `flags` field stores the status of the page. For example it can store whether the page is dirty or not, or whether it is locked in memory. The flag values are defined in \<linux/page-flags.h\>.

The `_count` field stores the usage count of the page: how many references there are to the page. If `_count` is -1 then the page is free. Instead of accessing `_count` directly, developers should use the `page_count` function to determine whether a page is free or not. `page_count` returns 0 if the page is free and a non-negative integer if the page is in use. A page can be in use by the page cache, as private data, or as a mapping in a process's page table {% cite lkd -l 232 %}.

The `virtual` field is the page's virtual address. Although this is normally the page's virtual memory address, some memory (called **high memory**) isn't permanently mapped in the kernel's address space. For high memory the value will be `NULL` {% cite lkd -l 232 %}.

## Zones

Because of limits in the hardware, the kernel can't treat all pages as equal. Some pages cannot be used for certain tasks because of their physical address. Because of this, the kernel divides pages with similar properties into **zones** {% cite lkd -l 232 %}.

There are two main limits in hardware:

1. Some hardware devices can only perform direct memory access (DMA) to certain address spaces.
2. Some architectures can physically address larger areas than they can virtually address. As a consequence, some memory is not permanently mapped into the kernel address space.

There are four primary address zones:

1. `ZONE_DMA` contains pages that can undergo DMA.
2. `ZONE_DMA32` contains pages that can undergo DMA and are only accessible by 32-bit devices.
3. `ZONE_NORMAL` contains normal pages.
4. `ZONE_HIGHMEM` contains high memory.

{% cite lkd -l 233 %}

The use and layout of memory is architecure-specific. Some architectures have no problem performing DMA into any memory address. For those architectures, `ZONE_DMA` is empty, and `ZONE_NORMAL` is used for allocation. Conversely, "on the x86 architecture, ISA devices cannot perform DMA into the full 32-bit address space1 because ISA devices can access only the first 16MB of physical memory. Consequently, `ZONE_DMA` on x86 consists of all memory in the range 0MB–16MB" {% cite lkd -l 233 %}.

`ZONE_HIGHMEM` is similar. "On 32-bit x86 systems, `ZONE_HIGHMEM` is all memory above the physical 896MB mark. On other architectures, `ZONE_HIGHMEM` is empty because all memory is directly mapped" {% cite lkd -l 233 %}.

`ZONE_NORMAL` tends to be what is remaining after `ZONE_DMA` and `ZONE_HIGHMEM`. The following table shows the zones on an x86 architecture:

| Zone           | Description                 | Physical Memory |
| -------------- | --------------------------- | --------------- |
| `ZONE_DMA`     | DMA-able pages.             | < 16MB          |
| `ZONE_NORMAL`  | Normally addressable pages. | 16–896MB        |
| `ZONE_HIGHMEM` | Dynamically mapped pages.   | > 896MB         |

{% cite lkd -l 234 %}

Zones are used to allocate pages when they are needed. Although some allocations require pages from a particular zone, other allocations can use pages from multiple zones. For example, a normal allocation can come from `ZONE_NORMAL` or `ZONE_DMA`, but not both. Allocations must come from a single zone at once {% cite lkd -l 234 %}.

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

`lock` is a spin lock that protects the structure from concurrent access.

The `watermark` array holds the minimum low and high watermarks for the zone. Watermarks are used to set benchmarks for suitable per-zone memory consumption {% cite lkd -l 235 %}.

## Getting pages

The main function to allocate memory from the kernel is `alloc_pages()`:

```c
struct page * alloc_pages(gfp_t gfp_mask, unsigned int order)
```

This allocates $$2^\text{order}$$ contiguous pages, and returns a pointer to the first page's `page` struct. On error, `alloc_pages()` returns `NULL`.

You can convert a given page to its logical address with `page_address()`:

```c
void * page_address(struct page *page)
```

This returns a pointer to the logical address where the given physical page currently resides. If you have no need for the actual `page` struct, you can call `__get_free_pages()`:

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

`kmalloc()` can be used to allocate byte-size chunks. `kmalloc()` works similarly to `malloc()`, except it also takes a `flags` parameter. `kmalloc()` returns a pointer to a byte-size chunk that is at least `bytes` size in length. On error it returns `NULL` {% cite lkd -l 238 %}.

### GFP mask flags

GFP (Get Free Page) mask flags are flags that are passed to kernel allocator functions {% cite lkd -l 238 %}

GFP flags are represented with the `gfp_t` type. There are three types of flags:

- Action modifiers—specify how the kernel should allocate memory.
- Zone modifiers—specify what zone to allocate from.
- Type flags—specify a combination of a zone and an action type.

{% cite lkd -l 238-9 %}

The flags are declared in \<linux/gfp.h\>, which is included in \<linux/slab.h\> {% cite lkd -l 239 %}.

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

#### Zone Modifiers

Zone modifiers specify the zones that the memory allocation should originate from. Normally the allocation will happen from any zone, with the kernel preferring `ZONE_NORMAL`.

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

The majority of allocations in the kernel use the `GFP_KERNEL` flag. "The resulting allocation is a normal priority allocation that might sleep". This flag can only be used from process context, because it might block {% cite lkd -l 242 %}.

Memory allocated with `kmalloc()` should be freed with `kfree()`.

A alternative to `kmalloc()` is `vmalloc()`. While `kmalloc()` returns a pointer to a chunk of contiguous physical and virtual memory, `vmalloc()` only guarantees contiguous virtual memory. `vmalloc()` updates page tables to ensure that the memory block it returns is virtually contiguous {% cite lkd -l 244 %}.

## Slab layer

The slab layer is an abstraction to make free lists more efficient.

A free list is a block of pre-allocated memory that is used to allocate memory from for objects in the future. When the data is no longer needed, it's returned to the free list rather than deallocating {% cite lkd -l 245 %}.

The problem with free lists in the kernel is that there isn't any global control. If memory is low then there is no way for the kernel to communicate with the free lists in order to free some of their cached memory. The slab layer solves this problem {% cite lkd -l 245 %}.

The slab layer follows several tenets:

- Frequently used data structures tend to be allocated and freed often, so cache them.
- Cached free lists avoid fragmentation because they are arranged contiguously.
- Free lists improve performance during frequent allocation and deallocation.
- The allocator can make more intelligent decisions if it's aware of object size, page size, and total cache size.
- Allocations on SMP can be made without a lock if cache is per-processor.
- Allocator can fulfill allocations from same node as requester if allocator is NUMA aware.
- Coloring can be used to prevent multiple objects from mapping to the same cache line.

{% cite lkd -l 246 %}

The slab layer divides objects into groups called caches. Each cache stores a different object. For example, one cache is for `task_struct` structs, and another for `inode` structs. `kmalloc()` is built on top of the slab allocator using general purpose caches {% cite lkd -l 246 %}.

Caches are divided into slabs. Slabs can consist of multiple contiguous pages, although normally they are only a single page {% cite lkd -l 246 %}.

Each slab contains a number of objects (the data structure that's being cached). A slab is in one of three states: full, partial, or empty. When the kernel requests a new object, the request is satisfied by a partial slab if one exists, otherwise an empty slab is used {% cite lkd -l 246 %}.

Take the `inode` structure. An `inode` is a filesystem structure that represents a disk inode, and is allocated by the slab allocated {% cite lkd -l 247 %}.

An `inode` is allocated from the `inode_cachep` cache (this is the standard naming convention). The cache is made up from many slabs, and each slab contains as many inode objects as can fit. When the kernel requests a new `inode` object, the slab allocator returns a pointer to an already allocated, but currently unused structure from a partial slab if one exists, or an empty slab if one does not. When the kernel is finished with the inode, the slab allocator will mark the object as free.

<figure>
  <img src="{{site.baseurl}}/assets/img/operating-systems/linux/memory-management/slab-cache.svg" alt="">
  <figcaption><h4>Figure: The relationship between caches, slabs, and objects {% cite lkd -l 247 %}</h4></figcaption>
</figure>

A cache is represented with a `kmem_cache` structure. The structure contains three lists stored inside a `kmem_list3` structure: `slabs_full`, `slabs_partial`, and `slabs_empty`. The list contains all slabs associated with the cache.

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

"Slab descriptors are allocated either outside the slab in a general cache or inside the slab itself, at the beginning. The descriptor is stored inside the slab if the total size of the slab is sufficiently small, or if internal slack space is sufficient to hold the descriptor." {% cite lkd -l 248 %}

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

Memory is freed by `kmem_freepages()`, which calls `free_pages()` on the given cache’s pages. The freeing function is only called when available memory grows low, or when a cache is explicitly destroyed {% cite lkd -l 249 %}.

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
- `SLAB_PANIC`—causes the kernel to panic if the slab allocation fails.
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

Pages in high memory might not be permanently mapped into the kernel address space. Thus, pages obtained via `alloc_pages()` with the `__GFP_HIGHMEM` flag might not have a logical address {% cite lkd -l 253 %}.

To map a `page` structure into the kernel’s address space, use `kmap`, declared in \<linux/highmem.h\>:

```c
void *kmap(struct page *page)
```

`kmap()` works on both high memory and low memory. If the `page` structure represents a page in low memory, then the virtual address is simply returned. If the page is in low memory, a permanent mapping is created and the address is returned. `kmap()` may sleep, so it works only in process context {% cite lkd -l 254 %}.

Since the number of permanent mappings of high memory are limited, you should unmap high memory when it's no longer needed. You do this with the `kunmap()` function.

When you need to create a mapping but the current context can't sleep, you can create a temporary mapping. "These are a set of reserved mappings that can hold a temporary mapping. The kernel can atomically map a high memory page into one of these reserved mappings" {% cite lkd -l 254 %}.

Setting up a temporary mapping is done with `kmap_atomic()`:

```c
void *kmap_atomic(struct page *page, enum km_type type)
```

`type` is one of the following enums:

```c
enum km_type {
  KM_BOUNCE_READ,
  KM_SKB_SUNRPC_DATA,
  KM_SKB_DATA_SOFTIRQ,
  KM_USER0,
  KM_USER1,
  KM_BIO_SRC_IRQ,
  KM_BIO_DST_IRQ,
  KM_PTE0,
  KM_PTE1,
  KM_PTE2,
  KM_IRQ0,
  KM_IRQ1,
  KM_SOFTIRQ0,
  KM_SOFTIRQ1,
  KM_SYNC_ICACHE,
  KM_SYNC_DCACHE,
  KM_UML_USERCOPY,
  KM_IRQ_PTE,
  KM_NMI,
  KM_NMI_PTE,
  KM_TYPE_NR
};
```

The mapping is undone wit `kunmap_atomic()`.

## References

{% bibliography --cited_in_order %}
