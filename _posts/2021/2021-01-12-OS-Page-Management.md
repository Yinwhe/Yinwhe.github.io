---
layout:	post
title: "OS-Page Management"
subtitle: ""
date: 2021-1-12 18:30:00
author: "Yinwhe"
header-style: text
tags:
    - Operating system
---

> This part can be a bit messy.

# Virtual address in Linux

### some background

**Two Address Space(AS) -** Kernel AS (privileged) and user AS (non-privileged)

- `Kernel code` can access both AS
- `User code` can only access user AS, cannot access kernel AS

In Linux, the kernel uses virtual addresses as user space processes do. **Virtual address space is split:**

- The **upper part** is used for the kernel
- The **lower part** is used for user space
- On 32-bit, the split is at `0xC0000000` , thus we can see that **user space** is `0x0-0xC0000000 3GB` and kernel is `0xC0000000-0xFFFFFFFF 1GB` *(Just default)*
- For 64-bit, the split varies, but it's high enough
    - `0xffff000000000000` – ARM
    - `0xffff000000000000` – x86_64

### What if physical size exceeds virtual one?

> In 32-bits arch, virtual address space is `4GB`, WHAT IF physical address size is more than this?

A frame can be used only after it's been processed by the kernel *(like add some marks or things else),* and kernel only have `1GB` memory, HOW to handle this?

The answer is **reusing**, kernel only maps `896MB` , other `1GB - 896MB = 128MB` virtual address is reserved. 

That is, only bottom part of physical RAM has a kernel logical address **(linear allocation)**, the rest 128MB VA is for non-contiguous allocations.

<img src="https://i.loli.net/2021/01/12/mD4NjKdo8Ei6U9n.png" alt="Page%20Management%203209086d8fa647d0801944ea79040f64/Untitled.png" style="zoom: 33%;" />

- When a new frame is needed, kernel allocate a virtual address to the physical memory and handle it, after it's done, it's given to user and release the virtual address(reusing).

### Three types VA

<img src="https://i.loli.net/2021/01/12/9wbt2m6FMNHPqlO.jpg" alt="Page%20Management%203209086d8fa647d0801944ea79040f64.png" style="zoom: 25%;" />

That is, `user virtual address` , `kernel virtual address` , `kernel logical address`

- **Virtual** - Virtual contiguous, physical non-contiguous
- **Logical** - Virtual and physical contiguous

# Demanding Page

> **Demand paging** brings a page into memory only when it is demanded

Demand means access/read/write

- If page is invalid (error) → abort the operation
- If page is valid but not in memory → bring it to memory.

This is called `page fault`

- If needed page has data in it `(named)` , then swap it in.
- Or it only need more memory space `(anonymous)` , then allocate for it.

**Advantages** : no unnecessary I/O, less memory needed, slower response, more apps.

Then comes the question: How does MMU know the physical frame is not mapped? The answer is `valid bit`

### Page Fault

<img src="https://i.loli.net/2021/01/12/tuoxD1Tn5FPEYUQ.png" alt="Page%20Management%203209086d8fa647d0801944ea79040f64/Untitled%201.png" style="zoom: 33%;" />

When access a **non-present page**, a `page fault` will be generated.

Then OS will check the `mm_struct` of this process to decide whether the reference is valid or not:

- If valid, get an empty frame and swap page into it if data is needed. Then set page table entry to indicate the page is now in memory and restart the instruction that caused the page fault
- If not, abort the operation.

<img src="https://i.loli.net/2021/01/12/3V5Lo2Kusjg87QE.png" alt="Page%20Management%203209086d8fa647d0801944ea79040f64%201.png" style="zoom:33%;" />

### Lazy Allocation

A page fault is a CPU exception, generated when software **attempts to use an invalid virtual** address. There are three cases:

1. The virtual address is **not mapped** for the process requesting it. <—— What we discuss
2. The processes has **insufficient permissions** for the address requested. 
3. The virtual address is valid, but swapped out

##### What "Lazy Allocation" means?

When a process askes for memory, kernel won't allocate pages as requested immediately. The kernel will wait until those pages are actually used.

- This is called lazy allocation and is a performance optimization.
- For memory that doesn't get used, physical frame allocation never has to happen.

##### Procedure

When memory is requested, the kernel simply creates a `record of the request`, and then returns (quickly) to the process, without updating the TLB.

- The record is stored in `VMA`, virtual memory area.
- What actually happens is that, OS enlarges the bound of `Heap`, but no actual frames.

When that newly-allocated memory is touched, the CPU will generate a page fault, because the CPU doesn't know about the mapping

<img src="https://i.loli.net/2021/01/12/LAhFgTNZQYMbely.png" alt="Page%20Management%203209086d8fa647d0801944ea79040f64/Untitled%202.png" />

​		(Pic From http://static.duartes.org/img/blogPosts/memoryDescriptorAndMemoryAreas.png)

In the page fault handler, the kernel determines whether the mapping is valid (from the kernel's point of view). Then kernel updates the page table with the new mapping

The kernel returns from the exception handler and the user space program resumes.

![Page%20Management%203209086d8fa647d0801944ea79040f64/Untitled%203.png](https://i.loli.net/2021/01/12/8hLNwuQEd2KX1ox.png)

​		(Pic From http://static.duartes.org/img/blogPosts/heapAllocation.png)

For processes that are time-sensitive, data can be pre-faulted, or simply touched, at the start of execution.

- See `mlock()` and `mlockall()` for pre-faulting.

> I guess I shall spare some time to read this blog.

### Free-Frame List

> Like we said before, when the needed page is in the disk, OS go and fetch it. What if it's an empty page? (Just need a new frame)

Most operating systems maintain a free-frame list -- a pool of free frames for satisfying such requests.

<img src="https://i.loli.net/2021/01/12/HyNzqXlYCfMVTca.png" alt="Page%20Management%203209086d8fa647d0801944ea79040f64/Untitled%204.png" style="zoom: 33%;" />

Operating system typically allocate free frames using a technique known as **zero-fill-on-demand** -- the content of the frames zeroed out before being allocated.

- This can make sure data left by the last process won't be leaked out to the new process, for safety.

When a system starts up, all available memory is placed on the free frame list.

### Support for Demand paging

1. page table entries with **valid / invalid bit**
    - Only when it's invalid, OS do the check and allocation, which will reduce the overhead.
2. **backing storage** (usually disks)
    - OS may need to fetch the frame from disk.
3. **instruction restart**
    - When the page is ready, the instruction shall be re-executed.

# Overhead and Improvement of Demand paging

> Access disk is so sloooooooow! How to improve?

### Swap space

> Swap space I/O faster than file system I/O, even if on the same device

Swap allocated in larger chunks, less management needed than file system; Copy entire process image to swap space at process load time. *(Used in older BSD Unix)*

But this still need to **write** to swap space, which is in fact still very slow.

### Copy-on-Write

**Copy-on-write** `COW` allows parent and child(**limited**) processes to initially share the same pages in memory: the page is shared as long as no process modifies it.

If either process modifies a shared page, then the modified page shall be copied

COW allows more **efficient process creation**

- no need to copy the parent memory during fork
- only changed memory will be copied later

`vfork` syscall optimizes the case that child calls `exec` immediately after fork: 

- parent is suspend until child exits or calls exec
- child shares the parent resource, including the heap and the stack
- child cannot return from the function or call exit

`vfork` could be fragile, **it is invented when COW has not been implemented**

# Page Replacement

> When frame is used up, then we need to find a page not used and swap it. Then how we choose that page?

Policies to select victim page require careful design

- need to reduce overhead and avoid **thrashing**
- use modified (dirty) bit to reduce number of pages to swap out: only modified pages are written to disk

**Procedure to page in a page**

1. find the location of the desired page on disk
2. find a free frame
    - if there is a free frame, just use it
    - if there is none, use a page replacement policy to pick a victim frame, write victim frame to disk if dirty
3. bring the desired page into the free frame and update the page tables
4. restart the instruction that caused the trap

Then we may have `2` page I/O in one page fault*(swap in and out),* which increase the overhead.

<img src="https://i.loli.net/2021/01/12/igWa58zChYeEU1f.png" alt="Untitled" style="zoom: 33%;" />

### Page Replacement Algorithm

##### FIFO

Replace the first page loaded, easy but rather inefficient

- Add more frames may cause more page fault.

|      | 2    | 6    | 9    | 2    | 4    | 2    | 1    | 7    | 3    | 0    | 5    | 2    | 1    | 2    | 9    | 5    | 7    | 3    | 8    | 5    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| f1   | 2    | 2    | 2    | 2    | 4    | 4    | 4    | 7    | 7    | 7    | 5    | 5    | 5    | 5    | 9    | 9    | 9    | 3    | 3    | 3    |
| f2   |      | 6    | 6    | 6    | 6    | 2    | 2    | 2    | 3    | 3    | 3    | 2    | 2    | 2    | 2    | 5    | 5    | 5    | 8    | 8    |
| f3   |      |      | 9    | 9    | 9    | 9    | 1    | 1    | 1    | 0    | 0    | 0    | 1    | 1    | 1    | 1    | 7    | 7    | 7    | 5    |
| Ms   | Y    | Y    | Y    | N    | Y    | Y    | Y    | Y    | Y    | Y    | Y    | Y    | Y    | N    | Y    | Y    | Y    | Y    | Y    | Y    |

##### Optimal

> Pure imagination in fact, for we cannot predict.

Replace page that will not be used for the longest time. 

- This algorithm is not feasible, but can be used to measure other algorithm. *(Belady's Anomaly)*

|      | 2    | 6    | 9    | 2    | 4    | 2    | 1    | 7    | 3    | 0    | 5    | 2    | 1    | 2    | 9    | 5    | 7    | 3    | 8    | 5    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| f1   | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 9    | 9    | 7    | 3    | 8    | 8    |
| f2   |      | 6    | 6    | 6    | 4    | 4    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    |
| f3   |      |      | 9    | 9    | 9    | 9    | 9    | 7    | 3    | 0    | 5    | 5    | 5    | 5    | 5    | 5    | 5    | 5    | 5    | 5    |
| Ms   | Y    | Y    | Y    | N    | Y    | N    | Y    | Y    | Y    | Y    | Y    | N    | N    | N    | Y    | N    | Y    | Y    | Y    | N    |

##### Least Recently Used `LRU`

Replace pages that have not been used for the longest time *(relatively say)*

|      | 2    | 6    | 9    | 2    | 4    | 2    | 1    | 7    | 3    | 0    | 5    | 2    | 1    | 2    | 9    | 5    | 7    | 3    | 8    | 5    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| f1   | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 3    | 3    | 3    | 2    | 2    | 2    | 2    | 2    | 7    | 7    | 7    | 5    |
| f2   |      | 6    | 6    | 6    | 4    | 4    | 4    | 7    | 7    | 7    | 5    | 5    | 5    | 5    | 9    | 9    | 9    | 3    | 3    | 3    |
| f3   |      |      | 9    | 9    | 9    | 9    | 1    | 1    | 1    | 0    | 0    | 0    | 1    | 1    | 1    | 5    | 5    | 5    | 8    | 8    |
| Ms   | Y    | Y    | Y    | N    | Y    | N    | Y    | Y    | Y    | Y    | Y    | Y    | Y    | N    | Y    | Y    | Y    | Y    | Y    | Y    |

##### LRU Implementation

To realize LRU, support from hardwares is needed. Usually two types of implementation: <u>Counter-based</u> and <u>Stack-based</u>

**Counter-based**

Every page table entry has a `counter`, each time page is referenced, copy the **clock** into the counter. And when a page needs to be replaced, search for page with smallest counter.

- min-heap can be used

**Stack-based**

Keep a stack of page numbers (in double linked list), and when a page is referenced, move it to the top of the stack. In this case, the least recently used page shall always  be the bottom one.

- each update is more expensive, but no need to search for replacement



##### LRU Approximation Implementation

In fact there are few computers actually support LRU, but usually provides a reference bit, which can be used to realize some approximately LRU.

**Additional-Reference-Bits Algorithm**

Reordering whether a page is used in regular time intervals.

Like, we have 8-bits byte for each page, during a time interval (100ms), sets the high bit 1 if used and shifts bit rights by 1 bit.

- 00000000 ⇒ has not been used in 8 time intervals
- 11111111 ⇒ has been used in all time intervals
- 11000100 is more recently used than 01110111

**Second-chance**

Generally FIFO + hardware-provided **reference bit.** If page to be replaced has

- Reference bit = 0 → replace it
- reference bit = 1, then set reference bit 0 and leave it in memory, continue with the next page.

**Enhanced Second-Chance Algorithm**

Using **reference bit** and modify bit (if available) together. Take ordered pair `(reference, modify)`:

- `(0, 0)` neither recently used nor modified – best page to replace
- `(0, 1)` not recently used but modified – must write out before replacement
- `(1, 0)` recently used but clean – probably will be used again soon
- `(1, 1)` recently used and modified – probably will be used again soon and need to write out before replacement

**Overhead**: might need to search circular queue several times.

### Counting-based Page Replacement

> Keep a number of times that a page has been referred.

**`LFU`** *(Least Frequently Used)* replaces page with the smallest counter

- A page is heavily used during process initialization and then never used

**`MFU`** *(Most Frequently Used)* replaces page with the largest counter

- based on the argument that page with the smallest count was probably just brought in and has yet to be used

LFU and MFU are not common

### Page-Buffering Algorithms

> If each time we need to search for a free frame, that can be huge overhead, thus here we discuss how to buffer some free pages prepared for later use.

**Keep a pool of free frames**

- Read page into free frames without waiting for victims to write out
- When convenient, evict victim for not used page and buffer it.

**Also keep a list of modified pages**

- When backing store*(disk)* **idle**, write pages into disk and set it to non-dirty, then this page can be replaced **without** writing pages to backing store

Possibly, keep free frame contents intact and note what is in them - a kind of cache

- **If referenced again before reused**, **no need to load contents again from disk - cache hit**



****

# Allocation of Frames

> We will discuss how to allocate frames for process, to reduce page fault.

**Minimum number** of frames needed by a process is decided by how many pages an instruction may need, which differs to instructions semantics. 

Example: IBM 370 – **6 pages to handle SS MOVE** instruction:

- instruction is 6 bytes, might span 2 pages
- 2 pages to handle from
- 2 pages to handle to

**Maximum** is total frames in the system of course.

Two major allocation schemes

- Fixed allocation
- Priority allocation

### Global or Local Allocation

> From where we search and fetch these frames

**Local replacement** – each process selects from only its own set of allocated frames

- More consistent per-process performance
- But possibly underutilized memory.

**Global replacement** – process selects a replacement frame from the set of all frames; one process can take a frame from another.

- But then process execution time can **vary** greatly - depends on others.
- Since a process can have more frames, thus result to a greater throughput; So more commonly used.

### Fixed Allocation

`Equal allocation` – For example, if there are 100 frames (after allocating frames for the OS) and 5 processes, give each process 20 frames. Some free frames can be stored as buffer pool.

`Proportional allocation` – Allocate according to the size of process

### Priority Allocation

This allow a high-priority process to claim pages from low-priority process.



### Reclaiming Pages

> To maintain a page pools

A strategy to implement **global page-replacement** policy, all memory requests are satisfied from the **free-frame list.** This strategy attempts to ensure **there is always sufficient** free memory to satisfy new requests

Rather than waiting for the list to drop to zero, page replacement is triggered when the list falls below a **certain threshold.**

<img src="https://i.loli.net/2021/01/12/IDXEyjLlOT4BaMr.png" alt="Untitled" style="zoom: 25%;" />

When the memory is under the threshold, reclaim pages **aggressively**:

- Kill some processes for free frames.

### Major and minor page faults

`Major` - page is referenced but not in memory

`Minor` - mapping does not exist, but the page is in memory

- Shared library
- Reclaimed and not freed yet

# Thrashing

> **Thrashing is** a process busy swapping pages in and out

If a process doesn’t have “enough” pages, page-fault rate may be high, which may lead to low CPU utilization. Then kernel thinks it needs to increase the degree of multiprogramming to maximize CPU utilization, that is,  another process added to the system, which intensify the situation.

<img src="https://i.loli.net/2021/01/12/73uq59XxeEywcig.png" alt="Untitled" style="zoom:25%;" />

### Demand Paging and Thrashing

> It's demand paging introduce the swap, and thus the thrashing.

Why does demand paging work?

- process memory access has **high locality**
- process migrates from one locality to another, localities may overlap

**Why does thrashing occur?**

- `total size of locality > total memory size`, thus a process need to swap frequently, which causes CPU making no progress.

**How to solve this?**

1. Limit thrashing effects by **using local or priority page** replacement. One process starts thrashing does not affect others, like setting a firewall.
2. Provide a process with as many frames as it needs. *(just not feasible)*

### Working-Set Model

> Here we use working set to measure the size of locality.

A working-set is a collection of all locality pages, and we use `window △` to measure it.

**Working-set window**(Δ): a fixed number of page references

- if Δ too small → will not include entire locality
- if Δ too large → will include several localities
- if Δ = ∞ → will include entire program

<img src="https://i.loli.net/2021/01/12/NImJ6UHYFTpSB4a.png" alt="Untitled" style="zoom: 25%;" />

Explanation: Here the window is 10 locality pages, we can see:

- At `t1` , this process access page {1, 2, 5, 6, 7} in the last 10 pages, thus the working set is {1, 2, 5, 6, 7}
- At `t2` , this process only access page {3, 4} in the last 10 pages, thus the working set is {3, 4}

> Then comes the question: How we get the working set?

**Challenge: Keeping Track of the Working Set**

Approximate with interval timer + a reference bit. 

Take an example:

- Δ = 10,000
- Timer interrupts after every 5000 time units
- Keep in memory `2` bits for each page
- Whenever a timer interrupts copy and sets the values of all reference bits to 0
- If one of the bits in memory = 1 ➠ page in working set

But this not completely accurate, since we can not tell **when (in 5000 time unites)** the access occurs. A improvement is to spare 10 bits for each page and interrupt every **1000** time units. 

<img src="https://i.loli.net/2021/01/12/qVry1snHjf9uiDC.png" alt="Untitled" style="zoom: 33%;" />

Page fault increases due to new locality. *(huge working set changes)*

**Acceptable Page-Fault Frequency**

Adjust number of frames dynamically:

- If actual rate too low, process loses frame
- If actual rate too high, process gains frame

Need to swap out a process if no free fames are available

---

# Kernel Memory Allocation

Kernel memory allocation is treated **differently** from user memory, it is often allocated from a **free-memory pool**

- kernel requests memory for structures of varying sizes, thus we need to minimize waste caused by fragmentation
- Some kernel memory needs to be **physically contiguous** *(like, for I/O device)*

### Buddy System

> Divide and Allocate.

**Memory is allocated in units sized of $2^n$:** split the unit into two **`buddies`** until a proper **(best-fit)** sized chunk is available

Example: If we need 135KB, 14KB, 12KB, 5KB and 3KB free memory, it acts like this:

<img src="https://i.loli.net/2021/01/12/5AUZXoJFYNaKpdz.png" alt="image-20201229195441736" style="zoom: 67%;" />

**Advantage**: it can quickly merge unused chunks into larger chunk

**Disadvantage**: **internal fragmentation**

### Slab Allocator

Slab allocator is a **cache of objects (physical continuous address space)**

- a **cache** in a slab allocator consists of one or more slabs
- a Slab contains **one or more pages**, divided into **equal-sized objects**

Kernel uses one cache for each unique kernel data structure

- when cache created, allocate a slab, divided the slab into free objects
- objects for the data structure is allocated from free objects in the slab
- if a slab is full of used objects, next object comes from an empty/new slab; if cache full, create a new one.

**Benefits:** <u>no fragmentation</u> and <u>fast memory allocation</u>

- some of the object fields may be reusable; no need to initialize again

> **Why no fragmentation?**
> Since slab is physically continuous, data will no longer be separated by pages. For example, a 12K slab can store 4 3K objects, rather than 3.

> **Why fast memory allocation?**
> Take linux task struct for instance. When new task is created, just allocate existing free struct from cache; and when a process dies, its task struct is marked free and reclaimed by cache for next time use.

*A* Slab can be in **three possible states:**

- **Full** – all used
- **Empty** – all free
- **Partial** – mix of free and used

**Upon request**, slab allocator

- Uses free object in **partial** slab
- If none, takes one from **empty** slab
- If no empty slab, create new empty

![linuxslab](https://i.loli.net/2021/01/12/to6JG8zTmlvxKci.gif)

### Slab Allocator in Linux

Slab started in `Solaris`, now wide-spread for both kernel mode and user memory in various OSes

Linux 2.2 had `SLAB`, now has both `SLOB` and `SLUB` allocators. 

- SLOB for systems with limited memory: Simple List of Blocks – maintains 3 list objects for small, medium, large objects
- SLUB is performance-optimized SLAB removes per-CPU queues, metadata stored in page structure