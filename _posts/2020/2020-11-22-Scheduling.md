---
layout:	post
title: "OS-Scheduling"
subtitle: ""
date: 2020-11-22 23:40:00
author: "Yinwhe"
header-style: text
tags:
    - Operating system
---

[TOC]

# Definition

The decisions made by the OS to figure out which ready processes/threads should run and for how long

- This is essential in multi-programming environments and important for system performance and productivity.

### Burst Cycle

Burst here means the progress that processes alternate between CPU and I/O

Two types - **I/O bound process** and **CPU-bound process**

- I/O-bound process.
    - Mostly waiting for I/O
    - Many short CPU bursts
- CPU-bound process
    - Mostly using the CPU
    - Very short I/O bursts if any

### Type of Scheduler

Four situations needed schedule

1. A process switched from RUNNING to WAITING(l/O request or just `wait()`)
2. A process switched from RUNNING to READY(Interrupt)
3. A process switched from WAITING to READY(I/O finished)
4. A process ended.

**Non-preemptive scheduling:** a process holds the CPU until **it is willing to give it up** (if only case 1 and 4 will happen)

**Preemptive scheduling:** a process can be preempted even though it could have happily continued executing.

### Different Objectives/Goals

There are many conflicting goals that one could attempt to achieve 

- Maximize CPU Utilization - Fraction of the time the CPU isn’t idle
- Maximize Throughput - Amount of “useful work” done per time unit
- Minimize Turnaround Time - Time from process creation to process completion
- Minimize Waiting Time - Amount of time a process spends in the READY state
- Minimize Response Time - Time from process creation until the “first response” is received

### Dispatcher

Dispatcher module gives control of the CPU to the process selected by the short-term scheduler, which involves:

- switching context
- switching to user mode
- jumping to the proper location in the user program to restart that program

**Dispatch latency** – time it takes for the dispatcher to stop one process and start another to run

<img src="https://i.loli.net/2020/11/22/A5hQTXOPzs1SG3n.png" alt="Scheduling%20553926346ca942deab98de4fdbbf10a2/Untitled.png" style="zoom: 33%;" />

# Scheduling Algorithms

- First-Come, First-Served Scheduling
- Shortest-Job-First Scheduling
- Round-Robin Scheduling
- Priority Scheduling
- Multilevel Queue Scheduling
- Multilevel Feedback Queue Scheduling

Relevant knowledge
**Waiting time** - start time -arrival time
**Turnaround time** - finish time - arrival time

### FCFS

This part is rather easy, and of course, obsolete.

### SJF

SJF is optimal – gives minimum average waiting time for a given set of processes.

**Two types**

- Non-preemptive

![Scheduling%20553926346ca942deab98de4fdbbf10a2/Untitled%201.png](https://i.loli.net/2020/11/22/HwiCeTDmWMyvEr2.png)

- Preemptive - shortest remaining time first.

![Scheduling%20553926346ca942deab98de4fdbbf10a2/Untitled%202.png](https://i.loli.net/2020/11/22/Dv4snxl31LotIGU.png)

A big problem is: how to measure the burst duration.

### Round-Robin

> RR Scheduling is preemptive and designed for time-sharing

It defines a **time quantum**: A fixed interval of time (10-100ms)：

- Unless a process is the only READY process, it never runs for longer than a time quantum before giving control to another ready process
- It may run for less than the time quantum if its CPU burst is smaller than the time quantum

Ready Queue is a FIFO - Whenever a process changes its state to READY, it's placed at the end of the FIFO

**Scheduling:**

- Pick the first process from the ready queue
- Set a timer to interrupt the process after 1 quantum
- Dispatch the process

**A balance -** if the quantum is too small, context switch can be rather frequent and that can be a big overhead. Or too long, it becomes a FCFS.

### Priority

This one is easy. Anything new about it can be **Priority scheduling with Round-Robin**

- Run the process with the highest priority. Processes with the same priority run round-robin.

Problem: A low-priority process may never be executed!
Solution: Priority aging - Increase the priority of a process as it ages.

### Multi-Level Queue

**Simple idea:** use one ready queue per class of processes.

**Scheduling within queues**

- Each queue has its own scheduling policy

**Scheduling between the queues**

- Typically preemptive priority scheduling - A process can run only if all higher-priority queues are empty
- Or time-slicing among queues

![Scheduling%20553926346ca942deab98de4fdbbf10a2/Untitled%203.png](https://i.loli.net/2020/11/22/YydWJGeMNLgXsPB.png)

### Multilevel Feedback Queues

Processes can move among the queues

- If queues are defined based on internal process characteristics, it makes sense to move a process whose characteristics have changed `(eg. based on CPU burst length)`
- It’s also a good way to implement priority aging to avoid starvation.
- The Multilevel Feedback Queues scheme is very general because **highly configurable**
    - Number of queues
    - Scheduling algorithm for each queue
    - Scheduling algorithm across queues
    - Method used to promote/demote a process

---

# Multiple-Processor Scheduling

Symmetric multiprocessing (**SMP**) is where each processor is self scheduling.

- All threads may be in a common ready queue (a)
- Each processor may have its own private queue of threads (b)

![Scheduling%20553926346ca942deab98de4fdbbf10a2/Untitled%204.png](https://i.loli.net/2020/11/22/6JLSNsqGCekyAon.png)

### Memory stall on multicore processor

- When a core access the memory, it will spend up to 50% time waiting for the data to be accessible.
- Takes advantage of memory stall to make progress on another thread while memory retrieve happens.

![Scheduling%20553926346ca942deab98de4fdbbf10a2/Untitled%205.png](https://i.loli.net/2020/11/22/cs2XUvpmIPg98Tl.png)

- In such a case, one core can be seen as two logical processors.

### Two level of scheduling

1. **The operating system** deciding which software thread to run on a logical CPU.**(OS decides)**
    - In such case, all scheduling methods mentioned before can be used here.
2. How each **core** decides which hardware thread to run on the physical core.**(Core decides)**
    - Can using simple RR or priority.

<img src="https://i.loli.net/2020/11/22/YedbupsZIEL6N9i.png" alt="Scheduling%20553926346ca942deab98de4fdbbf10a2/Untitled%206.png" style="zoom:25%;" />

### Processor Affinity

As we know, when a process has been running in one processor, the cache will store its relevant data, and access to them can be rather quick. Thus a processor will try to keep that process in it since cache flush can be really a huge overhead.

- **Soft affinity** – the operating system attempts to keep a thread running on the same processor, but no guarantees.
- **Hard affinity** – allows a process to specify a set of processors it may run on.

### Loading balance

In SMP system, the most important task is to balance loading to maximum the usage of all processors.

**Load balancing** attempts to keep workload evenly distributed, two ways to achieve that:

- **Push migration** – periodic task checks load on each processor, and pushes task from overloaded CPU to other CPUs
- **Pull migration** – idle processors pulls waiting task from busy processor.

These two methods DONOT actually exclude each others, in fact, they are usually implemented parallelly.

**Note:** Load balancing may affect processor affinity, as a thread may be moved from one processor to another to balance loads but lost its content in cache.