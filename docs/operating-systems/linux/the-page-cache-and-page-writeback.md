---
layout: default
title: The page cache and page writeback
description: Notes on the Linux kernel page cache.
nav_order: 11
parent: Linux
grand_parent: Operating systems
permalink: /operating-systems/linux/the-page-cache-and-page-writeback
---

<!-- prettier-ignore-start -->

# The page cache and page writeback
{:.no_toc}

The page cache is a disk cache used to minimize disk I/O. Instead of read requests and write operations running against a disk, they run against an in-memory cache that only reads data from disk when needed and writes changes to disk periodically.

## Table of contents
{: .no_toc  }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Approaches to caching

The page cache contains physical pages in RAM, which correspond to physical blocks on a disk. The page cache is dynamic: "it can grow to consume any free memory and shrink to relieve memory pressure" {% cite lkd -l 323 %}.

### Read strategy

When the kernel begins a read operation, it first checks to see if the page is in the cache. If it is then the operation can be completed without requiring an expensive seek operation. This is called a cache hit. If the page isn't in the cache (a cache miss) the kernel must schedule an I/O operation to read the data off the disk. Once the data has been read from the disk, the kernel adds the data to the cache for future usage {% cite lkd -l 323 %}.

### Write strategy

There are three strategies for implementing write requests with caches:

- No-write—the write request updates the data on disk, and the cache is invalidated.
- Write-through—the write request updates the data on disk and in the cache.
- Write-back—the write request updates the data in the cache and updates the data on disk in the future.

Linux uses the write-back strategy. Write requests update the cached data. The updated pages are then marked as dirty, and added to the dirty list. A process then periodically updates the blocks corresponding to pages in the dirty list {% cite lkd -l 324 %}.

### Cache eviction

Removing items from the cache is known as cache eviction. This is done to either make room for more relevant data, or to shrink it in order to free memory {% cite lkd -l 325 %}.

Linux cache eviction works by removing only clean pages. It uses a variation of the LRU (Least Recently Used) algorithm, the two-list strategy.

In the two-list strategy, Linux maintains two linked lists: the active list and the inactive list. Pages on the active list are considered hot and are not available for eviction. Pages on the inactive list are available for eviction. Pages are placed on the active list if they are already residing in the inactive list {% cite lkd -l 325 %}.

The lists are maintained in a pseudo-LRU manner. Items are added to the tail, and are removed from the head. If the active list grows much larger than the inactive list, items are moved back from the active list to the inactive list {% cite lkd -l 325 %}.

## The Linux page cache

The Linux page cache uses an `address_space` object to manage entries and page I/O operations. Think of `address_space` as the physical analogue to the virtual `vm_area_struct`.

The address_space structure is defined in <linux/fs.h>:

```c
struct address_space {
  struct inode *host; /* owning inode */
  struct radix_tree_root page_tree; /* radix tree of all pages */
  spinlock_t tree_lock; /* page_tree lock */
  unsigned int i_mmap_writable; /* VM_SHARED ma count */
  struct prio_tree_root i_mmap; /* list of all mappings */
  struct list_head i_mmap_nonlinear; /* VM_NONLINEAR ma list */
  spinlock_t i_mmap_lock; /* i_mmap lock */
  atomic_t truncate_count; /* truncate re count */
  unsigned long nrpages; /* total number of pages */
  pgoff_t writeback_index; /* writeback start offset */
  struct address_space_operations *a_ops; /* operations table */
  unsigned long flags; /* gfp_mask and error flags */
  struct backing_dev_info *backing_dev_info; /* read-ahead information */
  spinlock_t private_lock; /* private lock */
  struct list_head private_list; /* private list */
  struct address_space *assoc_mapping; /* associated buffers */
};
```

`i_mmap` "is a priority search tree of all shared and private mappings in this address space". This is used to quickly find mappings associated with this cached file {% cite lkd -l 327 %}.

`address_space` is associated with a kernel object, normally an inode. If so, the `host` field points to an inode. The `host` field is `NULL` if the associated object isn't an inode {% cite lkd -l 327-8 %}.

`a_ops` is an operation table. The operations table is represented by the `address_space_operations` struct:

```c
struct address_space_operations {
  int (*writepage)(struct page *, struct writeback_control *);
  int (*readpage) (struct file *, struct page *);
  int (*sync_page) (struct page *);
  int (*writepages) (struct address_space *,
    struct writeback_control *);
  int (*set_page_dirty) (struct page *);
  int (*readpages) (struct file *, struct address_space *,
    struct list_head *, unsigned);
  int (*write_begin)(struct file *, struct address_space *mapping,
  loff_t pos, unsigned len, unsigned flags,
    struct page **pagep, void **fsdata);
  int (*write_end)(struct file *, struct address_space *mapping,
    loff_t pos, unsigned len, unsigned copied,
    struct page *page, void *fsdata);
  sector_t (*bmap) (struct address_space *, sector_t);
  int (*invalidatepage) (struct page *, unsigned long);
  int (*releasepage) (struct page *, int);
  int (*direct_IO) (int, struct kiocb *, const struct iovec *,
    loff_t, unsigned long);
  int (*get_xip_mem) (struct address_space *, pgoff_t, int,
    void **, unsigned long *);
  int (*migratepage) (struct address_space *,
    struct page *, struct page *);
  int (*launder_page) (struct page *);
  int (*is_partially_uptodate) (struct page *,
    read_descriptor_t *,
    unsigned long);
  int (*error_remove_page) (struct address_space *,
    struct page *);
};
```

{% cite lkd -l 328 %}

These functions implement I/O for the cached object. The `readpage()` and `writepage()` methods are most important {% cite lkd -l 328 %}.

In the `readpage()` method, Linux first attempts to get the page:

```c
page = find_get_page(mapping, index);
```

`mapping` is the given `address_space` and `index` is the desired offset into the file, in pages. "If the page does not exist in the cache, `find_get_page()` returns `NULL` and a new page is allocated and added to the page cache":

```c
struct page *page;
int error;

/* allocate the page ... */
page = page_cache_alloc_cold(mapping);
if (!page)
  /* error allocating memory */
  /* ... and then add it to the page cache */
  error = add_to_page_cache_lru(page, mapping, index, GFP_KERNEL);

if (error)
  /* error adding page to page cache */
```

{% cite lkd -l 329 %}

The requested data can then be read from disk, added to the page cache, and returned to the user:

```c
error = mapping->a_ops->readpage(file, page);
```

{% cite lkd -l 329 %}

"Write operations are a bit different. For file mappings, whenever a page is modified, the VM simply calls" `SetPageDirty(page)` {% cite lkd -l 329 %}.

"The kernel later writes the page out via the `writepage()` method. Write operations on specific files are more complicated. The generic write path in mm/filemap.c performs the following steps":

```c
page = __grab_cache_page(mapping, index, &cached_page, &lru_pvec);
status = a_ops->prepare_write(file, page, offset, offset+bytes);
page_fault = filemap_copy_from_user(page, offset, buf, bytes);
status = a_ops->commit_write(file, page, offset, offset+bytes);
```

## The Flusher Threads

The flusher threads periodically write dirty pages to disk {% cite lkd -l 331 %}.

There are three situations that cause writes:

- When free memory shrinks below a predefined threshold.
- When dirty data grows older than a specific threshold.
- When a user process calls the `sync()` or `fsync()` system calls.

{% cite lkd -l 330 %}

When free memory shrinks below the threshold (defined by the `dirty_background_ratio` sysctl), the kernel invokes `wakeup_flusher_threads()` to wake up one or more flusher threads to run `bdi_writeback_all()`. `bdi_writeback_all()` takes the number of pages to write-back as a parameter. It will continue writing back pages until either the free memory is above the `dirty_background_ratio` threshold, or until the minimum number of pages has been written out {% cite lkd -l 331-2 %}.

To ensure dirty data doesn't grow older than a specific threshold, a kernel thread periodically wakes up and writes out old dirty pages {% cite lkd -l 332 %}.

## References

{% bibliography --cited_in_order %}
