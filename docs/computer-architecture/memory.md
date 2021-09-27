---
layout: default
title: Memory
description: Notes on computer memory.
nav_order: 5
parent: Computer architecture
permalink: /computer-architecture/memory
---

<!-- prettier-ignore-start -->

# Memory
{:.no_toc}

"The decisive part of the device, determining more than any other part its feasibility, dimensions and cost, is the memory". {% cite first-draft-edvac %}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

**Memory** stores information for use in a computer.

Three key goals of memory system design are:

1. Offer each program as much memory as possible.
2. Keep memory access as fast as possible.
3. Ensure the technology is cheap as possible.

{% cite P&H:ARM -l 389 %}

A modern memory system is made up of multiple components implemented in hardware and software that aim to achieve these goals.

Performance is improved by exploiting locality of reference. **Locality of reference** (the **principle of locality**) is the tendency of well-written programs to access a small portion of their address space at one time {% cite P&H:ARM -l 388 %}.

**Temporal locality** is the tendency of programs to access memory that has been recently accessed (e.g. a variable that is repeatedly referenced) {% cite P&H:ARM -l 388 %}.

**Spatial locality** is the tendency of programs to access memory that is close to recently accessed memory (e.g. when iterating through an array) {% cite P&H:ARM -l 388 %}.

Loops are an example of a construct that expresses both temporal and spatial locality {% cite P&H:ARM -l 389 %}.

Computers load data from memory to registers and store memory from registers to memory.

The steps for a load instruction include:

1. Place address $$A$$ on the memory bus.
2. Main memory reads address $$A$$ from the memory bus, retrieves word $$w$$ and places it on the bus.
3. CPU reads word $$w$$ from the bus and copies it into the destination register.

{% cite CS:APP -l 568-9 %}

The steps for a store instruction include:

1. Place address $$A$$ on the memory bus. Main memory reads it and waits for the data word.
2. CPU places data word $$V$$ on the bus.
3. Main memory reads data word $$V$$ from the bus and stores it in $$A$$.

{% cite CS:APP -l 570 %}

## Storage technologies

Most computer storage systems are divided into volatile main memory (or primary memory) and nonvolatile secondary storage.

_Note: volatile here means that data is lost when the storage component loses power._

Computers can only operate directly on data that is in main memory. Data stored in secondary storage must first be bought into main memory before it can be read by the processor.

Popular storage technologies include:

- SRAM
- DRAM
- HDD
- SSD

### SRAM

SRAM (Static RAM) is RAM that stores each bit in a memory cell that retains its value indefinitely as long as power is supplied {% cite CS:APP -l 561 %}.

Caches normally use SRAM {% cite P&H:ARM -l 392 %}.

### DRAM

DRAM (Dynamic Random Access Memory) is RAM that stores each bit as a charge on a capacitor.

Since values are stored in a capacitor, DRAMs must periodically refresh the memory by reading and rewriting cells.

Main memory is normally implemented from DRAM, since DRAM consumes less power and is cheaper than SRAM (due to simpler circuitry) {% cite P&H:ARM -l 392 %}.

Conventional DRAM chips are partitioned into $$d$$ supercells, with each supercell consisting of $$w$$ DRAM cells {% cite CS:APP -l 562 %}.

Supercells are organized as a rectangular array with $$r$$ rows and $$c$$ columns where $$rc=d$$. Each supercell has an address $$(i, j)$$ where $$i$$ denotes the row and $$j$$ denotes the column {% cite CS:APP -l 562-3 %}.

Information flows in and out of DRAMs through external connectors called pins.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/memory/dram.svg" alt="">
  <figcaption><h4>Figure: A high-level view of a 128-bit 16X8 DRAM chip {% cite CS:APP -l 563 %}</h4></figcaption>
</figure>

One benefit of organizing DRAM as a 2D array is that it reduces the number of address pins required, with the tradeoff being that this increases access time {% cite CS:APP -l 563 %}.

A DRAM chip is connected to circuitry known as a memory controller that transfers $$w$$ bits at a time to and from each DRAM chip {% cite CS:APP -l 563 %}.

To read the contents of supercell $$(i, j)$$, the memory controller sends $$i$$ to the DRAM followed by $$j$$. The DRAM responds by sending the contents of supercell $$(i, j)$$ to the controller {% cite CS:APP -l 563 %}.

A conventional DRAM contains an internal row buffer. When a memory controller sends row address $$i$$ the DRAM copies the contents of row $$i$$ into the row address buffer. When the memory controller sends row address $$j$$, the DRAM would then put the $$d$$ bits from supercell $$(i, j)$$ onto the data pins by reading the row buffer{% cite CS:APP -l 563 %}.

A **memory module** (RAM stick) is a circuit board that consists of multiple memory ICs.

DRAM chips are packed onto memory modules, like the 168-pin DIMM (Dual Inline Memory Module), which transfers data to and from the controller in 64-bit chunks, or the 72-pin SIMM (Single Inline Memory Module) which transfers data in 32-bit chunks {% cite CS:APP -l 564 %}.

For an x8 (8-DRAM) DIMM to retrieve a 64-bit doubleword at address $$A$$, the memory controller converts $$A$$ to supercell address $$(i, j)$$ and sends it to the memory module which then broadcasts $$(i, j)$$ to each DRAM. Each DRAM outputs the 8-bit content of its supercell $$(i, j)$$ which is then collected and formed into a 64-bit doubleword {% cite CS:APP -l 564 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/memory/dimm.svg" alt="">
  <figcaption><h4>Figure: DIMM used to build double word {% cite CS:APP -l 565 %}</h4></figcaption>
</figure>

Main memory can be increased by connecting multiple memory modules to a memory controller. In this case, when the controller receives an address $$A$$, the controller selects the module $$k$$ that contains $$A$$, converts $$A$$ to $$(i, j)$$ and sends it to $$k$$.

There are many extended forms of DRAM that have optimizations to improve memory access speeds:

- **FPM DRAM (Fast Page Mode DRAM)** allows for consecutive accesses to the same row to be served from the same row buffer.
- **SDRAM (Synchronous DRAM)** uses the rising edge of the external clock that drives the memory controller which allows it to output the contents of its supercells at a faster rate.
- **VRAM (Video RAM)** is used in the frame buffer of graphics systems. VRAM is similar to FPM DRAM except that VRAM output is produced by shifting the contents of the internal row buffer in sequence. VRAM also allows concurrent reads and writes to memory.

{% cite CS:APP -l 566 %}

### HDD

HDDs (Hard Disk Drives) store data on a magnetic disk which is read using a mechanical disk arm.

A disk surface is divided into concentric rings called tracks. Each track is further divided into sectors which contain the information {% cite P&H:ARM -l 396 %}.

The sequence of data stored on disk includes a sector number, followed by a gap, and then sector information (data including an error correction code), followed by another gap {% cite P&H:ARM -l 396 %}.

**Seeking** is the process of moving a read/write head to the correct track on a disk (disk seek). The **seek time** is the time taken to move the head {% cite P&H:ARM -l 396 %}.

Once the head is in the correct position, the device waits for the desired sector to pass under the head. This time is called the rotational latency (or rotational delay) of the disk {% cite P&H:ARM -l 397 %}.

Most disks also have a built-in cache which stores sectors as they pass under the head {% cite P&H:ARM -l 397 %}.

The layout of the block numbers used to be intuitive, but it isn't anymore (sequential blocks may be on different tracks) meaning traditional models for calculating disk access times are less accurate and less relevant than they once were {% cite P&H:ARM -l 397 %}.

### SSD

**SSDs (Solid State Disks)** store data in flash memory (a type of nonvolatile electrically erasable programmable memory).

An SSD is normally made up of one or more flash memory chips and a **flash translation layer**—a hardware/firmware device that translates requests for logical blocks into accesses of the physical disk {% cite CS:APP -l 582 %}.

Flash memory is made up of a series of blocks which each consist of a number of pages that are typically 512B–4KB in size. Data is read and written in units of pages. A page can only be written when an entire block has been erased, leading to relatively slow random write throughput {% cite CS:APP -l 582 %}.

Blocks wear out after around 100,000 repeated writes, and so the flash translation layer includes wear levelling logic that attempts to spread erasures evenly across all blocks {% cite CS:APP -l 582 %}.

SSDs have many advantages over HDDs (mostly because there's no mechanical part to them):

- Faster random access time.
- More robust (especially important for wearables and phones).
- Use less power.

{% cite CS:APP -l 583 %}

The main downside is that SSDs are more expensive than HDDs.

## Memory hierarchy

The **memory hierarchy** is a memory organization that uses multiple levels of storage components to minimize memory access times while maintaining cost efficiency {% cite P&H:ARM -l 389 %}.

As the distance from the processor grows, the size of the memory increases, the time taken to access the memory increases, and the cost/byte of memory usually goes down {% cite P&H:ARM -l 389 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/memory/memory-hierarchy.svg" alt="">
  <figcaption><h4>Figure: Example memory hierarchy</h4></figcaption>
</figure>

Data is only copied between two levels at a time. The data closer to a processor is a subset of the data further from the processor {% cite P&H:ARM -l 389 %}.

At instruction-level the memory hierarchy appears as a single unit, but perf-sensitive programmers must understand the memory hierarchy to get good performance (they can't just conceptualize memory as one large contiguous array) {% cite P&H:ARM -l 390 %}.

Data is transferred between hierarchy levels in fixed-sized transfer units which differ between levels. For example, transfers between registers and L1 might be in 64-bit doublewords, whereas transfers between L1 and L2 caches might be in 512-bit blocks {% cite CS:APP -l 593 %}.

## Caches

In the memory system, a **cache** is a hardware component that stores and serves data. The memory hierarchy levels between the processor and main memory are caches.

Caches have fast access times (L1 cache can typically be accessed in 2-4 clock cycles and an L2 cache can typically be accessed in 10 clock cycles) {% cite CS:APP -l 596 %}.

A **cache line** (also known as a cache block) is the minimum unit of information stored in a cache. A typical cache line in 2020 processors is 64 bytes.

A cache line has associated bookkeeping bits, like the **valid bit** which indicates whether the associated line contains valid data. Initially the valid bit is set to 0 meaning there is not match for the line {% cite P&H:ARM -l 400 %}.

A **cache hit** is when a cache line requested by the processor is found in the cache. The **cache hit rate** is the fraction of memory accesses which result in a hit. The **cache hit time** is the time taken to access data from the cache {% cite P&H:ARM -l 390 %}.

A **cache miss** is when the requested cache line is not stored in the cache. In this case the lower levels of the hierarchy must be accessed to retrieve the missing line. The **miss rate** is the fraction of memory accesses which result in a miss {% cite P&H:ARM -l 390 %}.

The **cache miss penalty** is the time it takes to replace a block from the lower level, plus the time it takes to make this available to the processor {% cite P&H:ARM -l 390 %}.

As an example, consider a load instruction for a word $$w$$. The CPU would request $$w$$ from the cache. The cache would check to see if it contains $$w$$. In the case of a cache hit, the cache would extract $$w$$ and return it to the CPU. In the case of a miss, the cache would request the cache line from the next level of the hierarchy. When the requested cache line containing $$w$$ becomes available, the cache would store the cache line, and return $$w$$ to the CPU {% cite CS:APP -l 599 %}.

### Cache organization

A cache is organized into $$S$$ sets that each store $$E$$ cache lines.

A cache line contains a data block of $$B$$ bytes.

A memory address is split into three parts:

1. A **set index field** ($$s$$) used to select the set that should contain the cache line.
2. A **tag field** ($$t$$) used to find the cache line in the set. A tag contains the upper bits of the word (the bits that aren't used as an index into the cache).
3. A **block offset field** ($$b$$) which gives the offset of the address in the cache line.

{% cite CS:APP -l 597 %}

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/memory/memory-address.svg" alt="">
  <figcaption><h4>Figure: Example m-bit memory address {% cite CS:APP -l 597 %}</h4></figcaption>
</figure>

_Note: The benefit of using the middle bits as the set index field is that it avoids contiguous lines mapping to the same set {% cite CS:APP -l 605 %}._

Together the tag field and set index field uniquely identify a cache line {% cite P&H:ARM -l 402 %}.

For a system where each memory address is $$m$$ bits: $$S = 2^s$$, $$B = 2^b$$, $$t = m - (b + s)$$ {% cite CS:APP -l 597 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/memory/cache-organization.svg" alt="">
  <figcaption><h4>Figure: Cache organization {% cite CS:APP -l 597 %}</h4></figcaption>
</figure>

A cache's organization can be described with the tuple $$(S, E, B, m)$$ {% cite CS:APP -l 597 %}.

A cache determines whether it contains an address $$A$$ by checking the set (indexed using the set index field) for a line matching $$tag\_bits(A)$$ with the valid bit set to 1 {% cite CS:APP -l 598 %}.

The $$b$$ block offset bits are used to access the word from the cache line data block {% cite CS:APP -l 598 %}.

The process for determining whether a request is a hit or a miss is:

1. Set selection
2. Line matching
3. Word extraction

Caches are grouped into different classes based on their parameters.

A **direct-mapped cache** is a cache where each set stores one cache line ($$E = 1$$) {% cite CS:APP -l 599 %}. Early caches were built as direct-mapped caches.

A **set-associative cache** is a cache that holds more than one cache line per set {% cite P&H:ARM -l 417 %}.

A set-associative cache with $$E$$ line locations in each set is called an E-way set-associative cache (where $$1 \lt E \lt C/B$$) {% cite CS:APP -l 606 %}.

**Associative memory** is an array of key-value pairs which takes a key as input and returns a value from one of the key-value pairs as output. You can conceptualize a cache set as a small associative memory {% cite CS:APP -l 607 %}.

A **fully associative cache** is a cache that consists of a single set ($$E = C/B$$) {% cite CS:APP -l 608 %}.

A cache set is often implemented using a comparator circuit associated with each cache line entry. Comparators are expensive and so fully associative caches are only practical for small cache sizes {% cite P&H:ARM -l 417 %}.

Increasing associativity normally decreases the miss rate. The main disadvantage is potentially increasing the hit time {% cite P&H:ARM -l 418 %}.

### Writes

If a store instruction writes to a cache then the main memory and cache are now inconsistent.

In the case of a cache hit, there are two main policies:

- Write-through
- Write-back

**Write-through** is a scheme where writes update both the cache and the next lower level of the hierarchy. The downside of write-through is that it takes a long time to write back to main memory, but write-through can be sped up by using a write buffer (buffered write-through) {% cite P&H:ARM -l 407 %}.

**Write-back** is a scheme where a value is only written to the cache initially. The modified line is then written to the lower level of the memory hierarchy when the line is replaced {% cite P&H:ARM -l 408 %}.

In the case of a cache miss, there are two main policies:

- No-write-allocate (only write to main memory)
- Write-allocate (fetch into cache and then write)

Common combinations are: write-through and no-write-allocate, or write-back and write-allocate.

### Multilevel caches

Most microprocessors include multiple levels of caching.

The naming scheme is L1, L2, ..., Ln, where L1 is closest to the processor and Ln is furthest. In 2020 processors typically have 2-3 cache levels.

An L2 cache is normally situated on the same chip as the processor whereas an L3 cache is normally shared between cores.

The next-level cache is accessed when there is a miss in the preceding cache.

Each level in a multilevel cache can be optimized for different use cases. For example, the L1 cache can optimize for hit time and L2 can optimize for low miss rate {% cite P&H:ARM -l 425 %}.

An L1 cache of a multilevel cache is often smaller than a single-level L1 cache. The primary cache in a multilevel cache can use smaller block size to reduce the miss penalty. The secondary cache is often much larger than a primary cache since it's optimizing to reduce cache misses {% cite P&H:ARM -l 425 %}.

**Split cache** is a scheme where a cache level is split into two separate caches—an instruction cache (i-cache) and a data cache (d-cache) that operate in parallel. Most processors today use split L1 caches to increase the cache bandwidth {% cite P&H:ARM -l 411 %}.

### Replacement policies

A **cache replacement policy** is an algorithm for deciding which cache line should be replaced by a new entry when a cache is full.

The simplest policy is a **random policy** that randomly chooses a block to be replaced {% cite CS:APP -l 608 %}.

An **LRU policy** (Least Recently Used) replaces the block that has been unused for the longest time. To use LRU, the cache must keep track of when each element in a set was used relative to other elements (adding extra hardware complexity) {% cite P&H:ARM -l 423 %}.

An **LFU policy** (Least Frequently Used) replaces the block that has been least frequently used within a window {% cite CS:APP -l 602 %}.

A **FIFO policy** (First-In-First-Out) removes the block that has been in the cache the longest.

An **NMRU policy** (Not Most Recently Used) is FIFO except the most recently used block is never replaced.

### Cache coherence

The **cache coherence problem** is the challenge of keeping data synchronized between caches and main memory (particularly in CPUs with multiple processors) {% cite P&H:ARM -l 477 %}.

Informally, a system is coherent if any read of a data item returns the most recently written value of the data item {% cite P&H:ARM -l 477 %}.

In a multiprocessor system where more than one processor has a cached copy of memory location $$X$$ the following conditions are required to achieve cache coherence:

1. A read by processor $$P$$ to location $$X$$ which follows a write by $$P$$ to $$X$$ with no writes to $$X$$ from another processor between the write and the read should see the value written by $$P$$.
2. A read by processor $$P$$ to location $$X$$ that follows a write by another processor to $$X$$ returns the written value if the write and the read are separated by a sufficient amount of time and there has been no other write to $$X$$ in between.
3. Writes to the same location are _serialized_ (two writes to the same location by any processors are seen in the same order by all processors).

{% cite P&H:ARM -l 477-8 %}

In a cache-coherent multiprocessor, the caches provide migration (a data item can be moved to a local cache and used there) and replication (when shared data is simultaneously being read, the caches both make a copy of the data item in the local cache) of shared data {% cite P&H:ARM -l 479 %}.

In order to provide migration and replication while maintaining coherent caches, a multiprocessor must implement a cache coherence protocol {% cite P&H:ARM -l 479 %}.

**Snooping** is a popular cache coherence protocol where caches store the sharing status of shared cache lines. Caches are accessible via a broadcast mechanism (like a bus) and all cache controllers monitor broadcasts to determine whether they have a copy of a line that is requested on the bus {% cite P&H:ARM -l 479 %}.

A write-invalidate snooping protocol invalidates other copies of a data item on a write {% cite P&H:ARM -l 480 %}.

## Virtual memory

**Virtual memory** is an abstraction of main memory where programs are given a virtual address space. Virtual addresses are then mapped to physical addresses to access data.

An **address space** is an ordered set of nonnegative integer addresses. Modern systems typically support 32-bit or 64-bit virtual address spaces. A system has a physical address space the corresponds to the $$M$$ bytes of physical memory in the system {% cite CS:APP -l 778 %}.

A **physical address** is an address in main memory and a **virtual address** is an address in the virtual memory space. A virtual address is translated to a physical address when the memory is accessed {% cite P&H:ARM -l 442 %}.

Virtual memory is implemented with a combination of software, MMU hardware, and a data structure (the page table) which holds the mappings between virtual pages and physical pages {% cite CS:APP -l 780 %}.

**Address translation** is the process of mapping a virtual address to a physical address {% cite P&H:ARM -l 442 %}.

In virtual memory, each byte of main memory has a virtual address and a physical address {% cite CS:APP -l 779 %}.

The processor always uses virtual addresses, which are then translated to physical addresses by an MMU (Memory Management Unit) {% cite CS:APP -l 778 %}.

The main motivations for virtual memory are:

- Sharing memory efficiently between multiple programs.
- Protecting one program's address space from another.
- Allowing a program's address space to exceed the size of the available physical memory.

{% cite P&H:ARM -l 442-3 %}

A virtual memory address is not always mapped to a physical address. In this case, the virtual can reside on secondary storage (like disk) {% cite P&H:ARM -l 443 %}.

Virtual memory must use write-back because write-through would require potentially millions of cycles (writes to disk) {% cite P&H:ARM -l 452 %}.

### Paging

**Paging** is a form of virtual memory that divides virtual memory into pages {% cite P&H:ARM -l 445 %}.

A **page** is a fixed-size block of virtual memory. Most 32-bit architectures have 4KB pages and most 64-bit architectures have 8KB pages {% cite lkd -l 231 %}

In paging systems, a virtual address is split into two parts:

1. A **page number** which identifies a page.
2. A **page offset** which is the number of bytes from the page's base address that the virtual memory address is located at. The page offset is unchanged from the physical address.

{% cite P&H:ARM -l 443 %}

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/memory/virtual-page-address.svg" alt="">
  <figcaption><h4>Figure: Address split into page number an offset</h4></figcaption>
</figure>

_Note: **segmentation** is an alternative address mapping scheme that uses variable-size blocks. In segmentation an address consists of two numbers: a segment number and a segment offset {% cite P&H:ARM -l 445 %}._

Physical pages can be shared between processes by having two virtual memory pages map to the same physical page {% cite P&H:ARM -l 443 %}.

A **page fault** occurs when a program attempts to access a virtual page that is not stored in main memory. In this case, address translation hardware triggers a page fault exception {% cite CS:APP -l 782 %}.

Page fault exceptions are handled by the OS which:

1. Selects a victim page to overwrite.
2. Saves the victim page to disk (if it's been modified since being read).
3. Modifies the page table entry to reflect that the victim page is no longer in memory.
4. Copies the incoming page from disk to memory.
5. Updates the page table entry.
6. Returns and restarts the faulting instruction.

{% cite CS:APP -l 782 %}

A page fault to disk takes millions of clock cycles to process and so the number of page faults need to be minimized. The high cost of a page fault influences the following design decisions:

- Pages are large to amortize the cost of fetching them.
- Fully associative placement of pages in memory can reduce page fault rate by allowing the OS to smartly manage page replacement (frequently an LRU strategy).
- Page faults can be handled in software since the overhead of handling them in software is small compared to the cost of fetching from disk (this allows for more complex page replacement policies).
- Virtual memory systems use write-back because write-through takes too long.

{% cite P&H:ARM -l 444, 448 %}

Some computers have an access bit which is set whenever a page is accessed. The OS periodically cleans access bits so that it can later check them and see what's been accessed within a given window {% cite P&H:ARM -l 450 %}.

An OS normally creates an address space on flash memory or disk for all pages of a process when it creates the process. This is known as the swap space {% cite P&H:ARM -l 448 %}.

### Page tables

A **page table** is a data structure (usually stored in memory) that contains virtual-to-physical address mappings {% cite P&H:ARM -l 446 %}.

The page table is normally indexed by the virtual page number {% cite P&H:ARM -l 446 %}.

Processors often have a page table register that stores the memory address of the start of the page table for the current process (each process has its own page table) {% cite P&H:ARM -l 446 %}.

Page tables are normally implemented as a hierarchy to reduce the memory footprint of the page table {% cite CS:APP -l 792 %}.

In multi-level page tables, the page number is split into multiple parts where each part is an index for different page table levels.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/memory/multi-level-page-table.svg" alt="">
  <figcaption><h4>Figure: Multi-level page table {% cite CS:APP -l 794 %}</h4></figcaption>
</figure>

Page table lookups are sped up by the TLB.

### TLB

A **TLB** (Translation Lookaside Buffer) is a hardware cache that stores recently used address mappings to avoid the expensive process of address remapping.

A TLB has a tag field as well as bookkeeping flags (like dirty, valid, and reference bits) {% cite P&H:ARM -l 453 %}.

TLBs can work with multi-level page tables as well as single-level {% cite P&H:ARM -l 453 %}.

Every memory reference initiates a page number lookup in the TLB. In the case of a TLB hit, the physical page number is used to form an address. If the processor is performing a write then the TLB entry's dirty bit is set to 1 {% cite P&H:ARM -l 453 %}.

In the case of a TLB miss, the page table must be walked (possibly resulting in a page fault if the page is not in memory). TLB misses can either be handled in hardware or in software but are now often handled in hardware.

The OS has to know the layout of the hardware page table structure for a given architecture. The OS adds entries to the page table but the TLB usually performs walks in hardware through cached page tables.

As of 2017, typical TLB sizes can range from 16-512 entries, with block sizes of 1-2 page table entries, hit times of 0.5-1 clock cycles, miss penalties of 10-100 cycles, and miss rates of 0.01%-1% {% cite P&H:ARM -l 454 %}.

There is usually a separate instruction TLB and data TLB.

### Memory protection

Most OSes provide memory protection to control memory access.

To support memory protection in virtual memory, the processor must:

1. Offer at least two modes to indicate whether the running process is a user process or a supervisor process
2. Provide a portion of processor state that a user process can read but not write (e.g. the TLB, the kernel mode bit, page table pointer). To write to this state the OS must use special instructions that can only run in supervisor mode.
3. Provide a mode for the processor to switch between user mode and supervisor mode. User-to-supervisor switching is normally implemented with a system call exception, and supervisor-to-user switching is normally done using a special return instruction.

{% cite P&H:ARM -l 459 %}

Page tables are often kept in protected OS address space to stop processes from modifying their page table (which would enable them to access other processes' memory) {% cite P&H:ARM -l 460 %}

## References

{% bibliography --cited_in_order %}
