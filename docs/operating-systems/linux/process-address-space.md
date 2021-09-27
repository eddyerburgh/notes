---
layout: default
title: The process address space
description: Notes on the process address space in the Linux kernel.
nav_order: 10
parent: Linux
grand_parent: Operating systems
permalink: /operating-systems/linux/process-address-space
---

<!-- prettier-ignore-start -->

# The process address space
{:.no_toc}

This section is about the process address space and how it's implemented in Linux.

## Table of contents
{: .no_toc  }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

The process address space is the virtual memory addressable by a process {% cite lkd -l 305 %}.

Each process is given a flat 32 or 64-bit address space. Normally the address space is unique to each process, although it can be shared between processes (e.g. with threads).

A process doesn't have permission to access all memory. The area of legal addresses are called memory areas. A process can dynamically add and remove memory areas via the kernel {% cite lkd -l 306 %}.

Memory areas have permissions associated with them, such as readable, writeable, and executable. If a process accesses memory that it doesn’t have permission to access, the kernel kills the process with a segmentation fault {% cite lkd -l 306 %}.

Memory areas can contain:

- A memory map of the executable file's code (the text section).
- A memory map of the executable file's initial global variables (the data section).
- A memory map of the zero page, containing uninitialized variables (the bss section).
- A memory map of the zero page used for the process's user space stack.
- A text, data, and bss section for each shared library.
- Any memory mapped files.
- Any shared memory segments.
- Any anonymous memory mappings, like this associated with `malloc()`.

{% cite lkd -l 306-7 %}

Memory areas don't overlap.

## The memory descriptor

A process's address space is represented by a memory descriptor {% cite lkd -l 307 %}.

The memory descriptor is represented with the `mm_struct` struct:

```c
struct mm_struct {
  struct vm_area_struct *mmap; /* list of memory areas */
  struct rb_root mm_rb; /* red-black tree of VMAs */
  struct vm_area_struct *mmap_cache; /* last used memory area */
  unsigned long free_area_cache; /* 1st address space hole */
  pgd_t *pgd; /* page global directory */
  atomic_t mm_users; /* address space users */
  atomic_t mm_count; /* primary usage counter */
  int map_count; /* number of memory areas */
  struct rw_semaphore mmap_sem; /* memory area semaphore */
  spinlock_t page_table_lock; /* page table lock */
  struct list_head mmlist; /* list of all mm_structs */
  unsigned long start_code; /* start address of code */
  unsigned long end_code; /* final address of code */
  unsigned long start_data; /* start address of data */
  unsigned long end_data; /* final address of data */
  unsigned long start_brk; /* start address of heap */
  unsigned long brk; /* final address of heap */
  unsigned long start_stack; /* start address of stack */
  unsigned long arg_start; /* start of arguments */
  unsigned long arg_end; /* end of arguments */
  unsigned long env_start; /* start of environment */
  unsigned long env_end; /* end of environment */
  unsigned long rss; /* pages allocated */
  unsigned long total_vm; /* total number of pages */
  unsigned long locked_vm; /* number of locked pages */
  unsigned long saved_auxv[AT_VECTOR_SIZE]; /* saved auxv */
  cpumask_t cpu_vm_mask; /* lazy TLB switch mask */
  mm_context_t context; /* arch-specific data */
  unsigned long flags; /* status flags */
  int core_waiters; /* thread core dump waiters */
  struct core_state *core_state; /* core dump support */
  spinlock_t ioctx_lock; /* AIO I/O list lock */
  struct hlist_head ioctx_list; /* AIO I/O list */
};
```

`mm_users` is the number of users sharing this address space. For example, if two threads share the address space then `mm_users` is 2 {% cite lkd -l 308 %}.

`mm_count` represents whether the address space is used or not. It's kept at 1 until all threads using an address space exit, when it is decremented to 0, and the object is freed {% cite lkd -l 308 %}.

The `mmap` and `mm_rb` fields both contain all memory areas in the address space. `mmap` stores the address spaces in a linked list, whereas `mm_rb` stores them in a red black tree {% cite lkd -l 308 %}.

The linked list makes it easy to traverse the entire address space, whereas the red black tree is useful for searching for a given element {% cite lkd -l 308 %}.

All `mm_struct` objects are linked together in a linked list. The initial list element is the `init_mm` memory descriptor, which describes the `init` process's address space.

## Allocating a Memory Descriptor

A task's memory descriptor is stored in the `task_struct` `mm` field. The `mm_struct` is allocated from the `mm_cachep` slab cache via the `allocate_mm()` macro. "Normally, each process receives a unique `mm_struct` and thus a unique process address space" {% cite lkd -l 308 %}.

Processes can share their address spaces with their children by passing the `CLONE_VM` flag to `clone`. If `CLONE_VM` is set, then `allocate_mm()` isn't called, and the process's `mm` field points to its parents memory descriptor. You can see this in `copy_mm()`:

```c
if (clone_flags & CLONE_VM) {
 /*
  * current is the parent process and
  * tsk is the child process during a fork()
  */
  atomic_inc(&current->mm->mm_users);
  tsk->mm = current->mm;
```

When a process associated with an address space exits, `exit_mm()` is called. `exit_mm()` calls `mmput()`, which decrements the `mm_struct` `mm_users` count. When `mm_users` reaches 0, `mmdrop()` is called to decrement the `mm_count` counter. When `mm_count` is decremented to 0, and the `free_mm()` macro is invoked, to return the `mm_struct` to the `mm_cachep` {% cite lkd -l 309 %}.

Kernel threads don't have a process address space, so their `mm` value is NULL.

Kernel threads never access user space memory, and so they don't have a memory descriptor or page tables. However, kernel threads _do_ need access to some data, such as page tables. To provide kernel threads with page tables to access kernel memory, kernel threads use the memory descriptor of whatever task ran previously (`active_mm`) {% cite lkd -l 309 %}.

## Virtual Memory Areas

VMAs (Virtual Memory Areas) are represented with the `vm_area_struct` struct. The struct describes a single memory area that covers a contiguous interval in a given address space. Each memory area has certain properties, like permissions, and a set of associated operations {% cite lkd -l 310 %}.

You can see the struct:

```c
struct vm_area_struct {
  struct mm_struct *vm_mm; /* associated mm_struct */
  unsigned long vm_start; /* VMA start, inclusive */
  unsigned long vm_end; /* VMA end , exclusive */
  struct vm_area_struct *vm_next; /* list of VMA’s */
  pgprot_t vm_page_prot; /* access permissions */
  unsigned long vm_flags; /* flags */
  struct rb_node vm_rb; /* VMA’s node in the tree */
  union { /* links to address_space->i_mmap or i_mmap_nonlinear */
  struct {
    struct list_head list;
    void *parent;
      struct vm_area_struct *head;
    } vm_set;
    struct prio_tree_node prio_tree_node;
  } shared;
  struct list_head anon_vma_node; /* anon_vma entry */
  struct anon_vma *anon_vma; /* anonymous VMA object */
  struct vm_operations_struct *vm_ops; /* associated ops */
  unsigned long vm_pgoff; /* offset within file */
  struct file *vm_file; /* mapped file, if any */
  void *vm_private_data; /* private data */
};
```

`vm_start` is the lowest memory address of the area, `vm_end` is the first byte after the highest memory address of the area. $$vm_end - vm_start$$ is the total bytes of the memory area {% cite lkd -l 310 %}.

The `vm_flags` field contains bit flags. Unlike the permissions associated with an physical page which the hardware is responsible for, the VMA flags specify behavior that the kernel is responsible for maintaining. You can see a full list of the flags in the following table:

| Flag            | Effect on the VMA and Its Pages             |
| --------------- | ------------------------------------------- |
| `VM_READ`       | Pages can be read from.                     |
| `VM_WRITE`      | Pages can be written to.                    |
| `VM_EXEC`       | Pages can be executed.                      |
| `VM_SHARED`     | Pages are shared.                           |
| `VM_MAYREAD`    | The VM_READ flag can be set.                |
| `VM_MAYWRITE`   | The VM_WRITE flag can be set.               |
| `VM_MAYEXEC`    | The VM_EXEC flag can be set.                |
| `VM_MAYSHARE`   | The VM_SHARE flag can be set.               |
| `VM_GROWSDOWN`  | The area can grow downward.                 |
| `VM_GROWSUP`    | The area can grow upward.                   |
| `VM_SHM`        | The area is used for shared memory.         |
| `VM_DENYWRITE`  | The area maps an unwritable file.           |
| `VM_EXECUTABLE` | The area maps an executable file.           |
| `VM_LOCKED`     | The pages in this area are locked.          |
| `VM_IO`         | The area maps a device’s I/O space.         |
| `VM_SEQ_READ`   | The pages seem to be accessed sequentially. |
| `VM_RAND_READ`  | The pages seem to be accessed randomly.     |
| `VM_DONTCOPY`   | This area must not be copied on fork().     |
| `VM_DONTEXPAND` | This area cannot grow via mremap().         |
| `VM_RESERVED`   | This area must not be swapped out.          |
| `VM_ACCOUNT`    | This area is an accounted VM object.        |
| `VM_HUGETLB`    | This area uses hugetlb pages.               |
| `VM_NONLINEAR`  | This area is a nonlinear mapping.           |

{% cite lkd -l 311 %}

The `vm_ops` field points to the object of operations associated with a given memory area. The operations object is different for different types of memory area. {% cite lkd -l 312 %}

The methods object is represented by `operations_struct`:

```c
struct vm_operations_struct {
  void (*open) (struct vm_area_struct *);
  void (*close) (struct vm_area_struct *);
  int (*fault) (struct vm_area_struct *, struct vm_fault *);
  int (*page_mkwrite) (struct vm_area_struct *vma, struct vm_fault *vmf);
  int (*access) (struct vm_area_struct *, unsigned long , void *, int, int);
};
```

- `open` is invoked when the memory area is added to an address space.
- `close` is invoked when the memory area is removed from an address space.
- `fault` is invoked by the page fault handler when a page not present in physical memory is accessed.
- `page_mkwrite` is "invoked by the page fault handler when a page that was read-only is made writable".
- `access` "is invoked by access_process_vm when get_user_pages fails".

{% cite lkd -l 313 %}

`mmap` links together all memory area objects is a singly linked list. each `vm_area_struct` is linked in via its `vm_next_field`. The areas are sorted by ascending address {% cite lkd -l 313 %}.

The `find_vma()` function finds a VMA in which an address resides:

```c
struct vm_area_struct * find_vma(struct mm_struct *mm, unsigned long addr);
```

It searches for the first memory area whose `vm_end` value is greater than the `addr`. It can return a VMA that starts at an address greater than the `addr` {% cite lkd -l 316 %}.

You can see the implementation of `find_vma()`:

```c
struct vm_area_struct * find_vma(struct mm_struct *mm, unsigned long addr)
{
struct vm_area_struct *vma = NULL;

  if (mm) {
    vma = mm->mmap_cache;
    if (!(vma && vma->vm_end > addr && vma->vm_start <= addr)) {
      struct rb_node *rb_node;

      rb_node = mm->mm_rb.rb_node;
      vma = NULL;
      while (rb_node) {
        struct vm_area_struct * vma_tmp;

        vma_tmp = rb_entry(rb_node, struct vm_area_struct, vm_rb);
        if (vma_tmp->vm_end > addr) {
          vma = vma_tmp;
          if (vma_tmp->vm_start <= addr)
            break;
          rb_node = rb_node->rb_left;
        } else
          rb_node = rb_node->rb_right;
      }
      if (vma)
        mm->mmap_cache = vma;
    }
  }

  return vma;
}
```

`find_vma_prev()` works the same as `find_vma()` but it returns the last vma before addr:

```c
struct vm_area_struct * find_vma_prev(struct mm_struct *mm, unsigned long addr,
  struct vm_area_struct **pprev)
```

The `pprev` argument stores a pointer to the VMA preceding `addr` {% cite lkd -l 317 %}.

## mmap and do_mmap

`do_mmap()` creates a new address interval. If the new VMA created is adjacent an existing VMA that has the same privileges as the new address range, the existing VMA is extended. Otherwise a new VMA is created {% cite lkd -l 318 %}.

`do_mmap()` is declared in \<linux/mm.h\>:

```c
unsigned long do_mmap(struct file *file, unsigned long addr,
  unsigned long len, unsigned long prot,
  unsigned long flag, unsigned long offset)
```

`do_mmap()` "maps the file specified by file at offset `offset` for length `len`". `file` can be `NULL` and `offset` can be 0, which will mean the mapping does not have an associated file {% cite lkd -l 318 %}.

"The `prot` parameter specifies the access permissions for pages in the memory area. The possible permission flags are defined in <asm/mman.h> and are unique to each supported architecture" {% cite lkd -l 318 %}. In practice each architecture defines the flags listed in the following table:

| Flag         | Effect on the Pages in the New Interval |
| ------------ | --------------------------------------- |
| `PROT_READ`  | Corresponds to `VM_READ`.               |
| `PROT_WRITE` | Corresponds to `VM_WRITE`.              |
| `PROT_EXEC`  | Corresponds to `VM_EXEC`.               |
| `PROT_NONE`  | Cannot access page.                     |

`flags` specifies flags corresponding to VMA flags. These include the following flags:

| Flag             | Effect on the New Interval                             |
| ---------------- | ------------------------------------------------------ |
| `MAP_SHARED`     | The mapping can be shared.                             |
| `MAP_PRIVATE`    | The mapping cannot be shared.                          |
| `MAP_FIXED`      | The new interval must start at the given address addr. |
| `MAP_ANONYMOUS`  | The mapping is not file-backed, but is anonymous.      |
| `MAP_GROWSDOWN`  | Corresponds to `VM_GROWSDOWN`.                         |
| `MAP_DENYWRITE`  | Corresponds to `VM_DENYWRITE`.                         |
| `MAP_EXECUTABLE` | Corresponds to `VM_EXECUTABLE`.                        |
| `MAP_LOCKED`     | Corresponds to `VM_LOCKED`.                            |
| `MAP_NORESERVE`  | No need to reserve space for the mapping.              |
| `MAP_POPULATE`   | Populate (prefault) page tables.                       |
| `MAP_NONBLOCK`   | Do not block on I/O.                                   |

`do_mmap()` returns the initial address of the newly created address interval. If possible, the interval is merged with an
adjacent memory area, otherwise "a new `vm_area_struct` structure is allocated from the `vm_area_cachep` slab cache, and the new memory area is added to the address space’s linked list and red-black tree of memory areas via the `vma_link` function". The, "the `total_vm` field in the memory descriptor is updated" {% cite lkd -l 319 %}.

`do_mmap()` is made available to user space via the `mmap()`, and `mmap2()` system calls. The system call is implemented as:

```c
void * mmap2(void *start,
  size_t length,
  int prot,
  int flags,
  int fd,
  off_t pgoff)
```

`do_munmap()` removes an address interval from a specified process address space. The function is declared in <linux/mm.h>:

```c
int do_munmap(struct mm_struct *mm, unsigned long start, size_t len)
```

Users can call the `munmap()` system call to use `do_munmap()`:

```c
int munmap(void *start, size_t length)
```

## Page Tables

"Although applications operate on virtual memory mapped to physical addresses, processors operate directly on those physical addresses." When an application accesses a virtual memory address, it must be translated into a physical page address so that the processor can resolve the request. Translating a virtual memory address to a physical memory address is done using page tables {% cite lkd -l 320 %}.

"Page tables work by splitting the virtual address into chunks. Each chunk is used as an index into a table. The table points to either another table or the associated physical page" {% cite lkd -l 320 %}.

In Linux, page tables consist of three levels. The multiple levels enable a sparsely populated address space. If page tables were implemented as a static array, their size would be enormous.

The top-level page table is PGD (the Page Global Directory), which consists of an array of `pgd_t` types. `pgd_t` is normally an `unsigned long`. The PGD entries point to entries in the second-level, the page middle directory (PMD), which is an array of `pmd_t` types. These entries point to entries in the page table entries structure {% cite lkd -l 321 %}.

"The final level is called simply the page table and consists of page table entries of type `pte_t`. Page table entries point to physical pages" {% cite lkd -l 321 %}.

"In most architectures, page table lookups are handled (at least to some degree) by hardware. In normal operation, hardware can handle much of the responsibility of using the page tables. The kernel must set things up, however, in such a way that the hardware is happy and can do its thing" {% cite lkd -l 321 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/operating-systems/linux/process-address-space/virtual-to-physical.svg" alt="">
  <figcaption><h4>Figure: Virtual-to-physical address lookup {% cite lkd -l 321 %}</h4></figcaption>
</figure>

Each process has its own page table. "Because nearly every access of a page in virtual memory must be resolved to its corresponding address in physical memory, the performance of the page tables is very critical. Unfortunately, looking up all these addresses in memory can be done only so quickly. To facilitate this, most processors implement a translation lookaside buffer, or simply TLB, which acts as a hardware cache of virtual-to-physical mappings. When accessing a virtual address, the processor first checks whether the mapping is cached in the TLB. If there is a hit, the physical address is immediately returned. Otherwise, if there is a miss, the page tables are consulted for the corresponding physical address" {% cite lkd -l 321-2 %}.

## References

{% bibliography --cited_in_order %}
