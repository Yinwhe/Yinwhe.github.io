---
layout:	post
title: "OS-Storage and IO"
subtitle: ""
date: 2021-02-04 16:30:00
author: "Yinwhe"
header-style: text
tags:
    - Operating system
---


# 1. Disk

**Moving-head Magnetic Disk**

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbkit7gsqj315c0u0b29.jpg" alt="Storage%20and%20IO%2092b73c3cdae54d3f803a3db0b794d625/Untitled.png" style="zoom: 40%;" />

## 1.1 Operation Time

**`Positioning time`**(also random-access time) ****is time to move disk arm to desired sector, including **`seek time`** and **`rotational latency`.**

- `seek time`: move disk to the target cylinder
- `rotational latency`: for the target sector to rotate under the disk head

**Performance**

- **transfer rate**: **theoretical** 6 Gb/s; **effective** about 1Gb/s
- **seek time** from 3ms to 12ms (9ms common for desktop drives)
- **latency** based on spindle speed: 1/rpm * 60 *(average latency = ½ latency)*

**Average access time** = average seek time + average latency

**Average I/O time**: average access time + (data to transfer / transfer rate) + controller overhead

## 1.2 Disk Scheduling

> Where there is efficiency, there is scheduling.

Disk drives: `access time` and `disk bandwidth`

- **access time**: seek time (roughly linear to seek distance) + rotational latency
- **disk bandwidth** is the speed of data transfer *(data/time)*

When disk is idle, it can response to the request immediately, but if there are plenty requests, **disk itself** will maintain a **waiting queue** for the requests, per disk or device. Thus we need an algorithm to choose between so many requests.

- In the past, OS is responsible for the maintenance. But now it's built into the storage devices, which will finally provide Logical Block Address `LBA` only.

### 1.2.1 FCFS

This algorithm will deal with all requests in coming time order, thus it's uniform and fair, and, low-efficient.

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbkkkfvyyj315a0u0b29.jpg" alt="Storage%20and%20IO%2092b73c3cdae54d3f803a3db0b794d625/Untitled%201.png" style="zoom:33%;" />

**Advantage:**

- Every request gets a fair chance
- No uncertain postponement

**Disadvantages:**

- Low efficiency, no improvement on the order.

### 1.2.2 SSTF

Shortest Seek Time First, which also means **smallest distance** between current magnetic-head and target cylinder.

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbkkwzl4rj314g0radqi.jpg" alt="Storage%20and%20IO%2092b73c3cdae54d3f803a3db0b794d625/Untitled%202.png" style="zoom:33%;" />

**Advantages**

- Average response time decreased *(but may not be the least)*

**Disadvantages**

- Overhead to calculate seek time *(distance)*
- Starvation happens when closer request occurs continuously.
- High variance of response time.*(fluctuation)*

### 1.2.3 SCAN/C-SCAN

Also called elevator algorithm.

- disk arm starts at one **end** of the disk, and moves toward the **other end**
- service requests during the movement until it gets to the other end
- then, the head movement is **reversed** and servicing continues.

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbkl9rxayj30zh0qtaxb.jpg" alt="Storage%20and%20IO%2092b73c3cdae54d3f803a3db0b794d625/Untitled%203.png" style="zoom:33%;" />

**Advantages**

- High throughput
- Low variance of response time. *(Average)*

**Disadvantages**

- Long waiting time for request with cylinders just visited by head.

`C-SCAN` is almost the same, but it will **start from the beginning** when reaching the end, rather than scanning reversely, which provides a more uniform waiting time.

**Note:** `SCAN/C-SCAN` moves head from end to end, even if no request between.

### 1.2.4 LOOK/C-LOOK

In practical implementation of `SCAN` , head only goes as far as the **last request** in each direction, and it's called `LOOK`

- **`LOOK`** is a version of **`SCAN`**, **`C-LOOK`** is a version of **`C-SCAN`**

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbklgsrmrj31450s9qec.jpg" alt="Storage%20and%20IO%2092b73c3cdae54d3f803a3db0b794d625/Untitled%204.png" style="zoom:33%;" />

**Advantages**

Prevents extra delay caused by unnecessary traversal to the end of the disk.

## 1.3 Disk Management

### 1.3.1 Formatting

> A new disk is just a container, to write datas into it, disk must be formatted first

**Physical formatting** - Also called Low-level formatting. It will use a special data structure to fill in the sectors; each sector consists of head, data area, and tail

- Head and tail contain useful info for the disk controller, like ECC(Error Correcting Code)

**Logical formatting** - Create an file system on that disk or partition.

### 1.3.2 Boot

**Root partition -** contains the OS; 

- **Mounted at boot time**
- At mount time, file system consistency checked: Is all metadata correct?

**Boot block** can point to boot volume or boot loader set of blocks that contain enough code to load the kernel from the file system

- Or a boot management program for multi-OS booting
- The `bootstrap` is stored in ROM firmware, it's to load a bigger boot loader from boot block.

**Other partitions -** hold other OSes, other file systems, or just be raw

- Mount automatically or manually

**Partition**

There are at most `4` primary partitions on a disk

- If we need to have more than 4 partitions, we need to use the logical partition - number starts from `5`

## 1.4 Swap Space Management

Used for moving entire processes `swapping`, or pages `paging`, from DRAM to secondary storage when DRAM not large enough for all processes.

Operating system provides **swap space management,** secondary storage slower than DRAM, so important to optimize performance

- Multiple swap spaces possible – decreasing I/O load on any given device
- Dedicated devices
- Can be in raw partition or a file within a file system (for convenience of adding but low efficiency)

Nowadays linux only stores anonymous memory *(no file support)* into swap space since load a page from FS is faster than from swap space.

## 1.5 Disk Attachment

> Disks can be attached to the computer as

**Host-Attached Storage `HAS` -** Disks can be attached to the computers directly **via an I/O bus**

- `SCSI`(Small Computer System Interface) - is a bus architecture, up to 16 devices on one cable
- `Fiber Channel` - is high-speed serial bus

**Network-Attached Storage `NAS` -** storage made available **over a network** instead of a local bus

- Client can remotely attach to file systems on the server, typically over TCP or UDP on IP network

**Storage Area Network `SAN` -** A private network connecting servers and storage units.

- SAN uses high speed interconnection and efficient protocols
- Multiple hosts and storage arrays can attach to the same SAN
- Storage can be dynamically allocated to hosts

---

# 2. RAID

> Based on the idea that disks are unreliable, slow, but cheap.

`RAID` Redundant Array of Inexpensive/Independent Disks

Simple idea: let’s use **redundancy**

- **Reliability** - If one fails, you have another one
- **Speed** Aggregate disk bandwidth if data is split across disks

And that is Redundant Array of Independent Disks **`RAID`**

## 2.1 Features

**Data Mirroring**

- Keep the same data on multiple disks, but this takes time.

**Data Striping**

- Keep data split across multiple disks to allow parallel reads. Like, read bits of a byte from 8 disks

**Error-Code Correcting (ECC) - Parity Bits**

- Keep information from which to reconstruct lost bits due to a drive failing

## 2.2 RAID Levels

**RAID 0 - 6**

RAID 0: split, RAID 1: mirror, RAID4/RAID 5/RAID 6: block with parity.



---
# 3. I/O

To encapsulate all details and features of various I/O devices, OS takes device driver to communicate with I/O device.

## 3.1 Hardware

> Common concepts: signals from I/O devices interface with computer

**Bus**: an interconnection between components (including CPU)

**Port**: connection point for device

**Controller**: component that control the device

- can be integrated to device or separate circuit board
- usually contains processor, microcode, private memory, bus controller.

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbkohbd4nj31330u07wh.jpg" alt="image-20210204161636122" style="zoom:33%;" />

### 3.1.1 Communication

Devices usually provide **registers** for data and control I/O of device

- device driver places (pointers to) commands and data to register
- registers include **data-in**/**data-out**, **status**, **control** ( or command) register

**Two ways of communication**

- **direct I/O instructions** - use special I/O instruction to control devices
- **memory-mapped I/O -** data and command registers mapped to processor address space; write to corresponding address to control devices

### 3.1.2 Access

> Two ways of access a I/O device - `polling` and `interrupt`

**Polling**

For each I/O operation: **busy-wait** if device is busy *(checking status register again and again until it's available)*

- Cannot accept any command if busy
- read status register until it indicates command has been executed

Polling requires busy wait, reasonable if device is fast; inefficient if device slow.

**Interrupts**

Interrupts can avoid busy-wait

- Device driver send a command to the controller, and return; OS can schedule other activities during the time, no need to wait until operation has been done.
- Device will interrupt the processor when command has been executed.
- OS retrieves the result by handling the interrupt

Interrupt-based I/O requires **context switch** at start and end

- if interrupt frequency is extremely high, context switch wastes CPU time; In such case, polling can be used.

### 3.1.3 DMA

> Direct Memory Access `DMA` transfers data directly between I/O device and memory.

If every data transfer operation needs CPU to take care, that can be a huge overhead especially when the data is huge, too. Thus DMA is put forward.

**To transfer data:**

- OS issues commands to the DMA controller
- Command includes: operation, memory address for data, count of bytes…
- Then DMA will do the transfer bypass CPU; CPU can do other things
- When done, device interrupts CPU to signal completion

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbkqw5igkj311s0twdq6.jpg" alt="image-20210204161914901" style="zoom: 50%;" />

## 3.2 Application I/O Interface

> Highly capsulated interface that can be used directly by applications

Driver - hide the differences between various devices for the subsystem

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbkskvhm7j313r0u0ajd.jpg" style="zoom:33%;" />

### 3.2.1 Block and Character Devices

`Block devices` access data in blocks, such as disk drives…

- Commands include **read, write, seek**
- Can be accessed as raw I/O, direct I/O, or file-system access

`Character devices` include keyboards, mice, serial ports…

- very diverse types of devices

### 3.2.2 Network Devices

Popular interface for network access is the **`socket`** interface

- It separates network protocol from detailed network operation
- Some non-network operations are implemented as sockets (e.g., Unix socket)

### 3.2.3 Clocks and Timers

Clocks and timers can be considered as character devices

Basic functions include:

- Get current time; Get elapsed time; Timer

### 3.2.4 Synchronous/Asynchronous I/O

**Synchronous I/O(a)** includes blocking and non-blocking I/O

**`blocking I/O` -** process **suspended** until I/O completed

- easy to use and understand, but may be less efficient
- insufficient for some needs

**`non-blocking I/O`** - I/O calls return **as much data as available**

- process does not block, returns whatever existing data (read or write)

**Asynchronous I/O(b)** process runs while I/O executes,

- I/O subsystem signals process when I/O completed via signal or callback
- difficult to use but very efficient

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbksqww7uj31ip0u0qpp.jpg" alt="image-20210204162104390" style="zoom:25%;" />

### 3.2.5 Kernel I/O Subsystem

> This system provides I/O related services

**I/O scheduling** - decides a **order** to service various request

- to queue I/O requests via per-device queue
- to schedule I/O for fairness and quality of service

**Buffering** - store data in memory while transferring between devices

- to cope with **device speed mismatch**: receive from network and write to ssd, double buffering
- to cope with **device transfer size mismatch**: network buffer divide and reassembly of message
- to maintain “copy semantics”: write()

**Caching** - hold **a copy of data for fast access**

- key to improve performance - When receive an I/O request, kernel shall access cache first to check whether target is in the memory or not.
- sometimes combined with buffering
- Buffer in memory is also used as caching for file operations

**Spooling** - A spool is a **buffer** that holds the output (device’s input) if device can serve only one request at a time, like the printer.

**Device reservation** - provides exclusive access to a device

- system calls for allocation and de-allocation
- watch out for deadlock

### 3.2.6 Life Circle of An I/O Request

![image-20210204162135487](https://tva1.sinaimg.cn/large/008eGmZEgy1gnbktbenofj312f0u04qp.jpg)

- "Already satisfy" means in the cache, no need for physical I/O