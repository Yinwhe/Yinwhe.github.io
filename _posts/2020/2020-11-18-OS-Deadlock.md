---
layout:	post
title: "OS-Deadlock"
subtitle: ""
date: 2020-11-18 21:00:00
author: "Yinwhe"
header-style: text
tags:
    - Operating system
---



[toc]

# Definition

A set of blocked processes each holding a resource and waiting to acquire a resource held by another process in the set.

- An simple example

    Two processes, two resources

    ```cpp
    //P1
    	wait(A);
    	wait(B);
    //P2
    	wait(B);
    	wait(A);
    ```

### Condition of Deadlock

- **Mutual exclusion**: only one process at a time can use a resource
- **Hold and wait**: a process holding at least one resource is waiting to acquire additional resources held by other processes
- **No preemption**: a resource can be released only **voluntarily** by the process holding it, after it has completed its task
- **Circular wait**: there exists a set of waiting processes {P0, P1, …, Pn}

    P0 is waiting for a resource that is held by P1

    P1 is waiting for a resource that is held by P2 …

    Pn–1 is waiting for a resource that is held by Pn

    Pn is waiting for a resource that is held by P0

### Resource-Allocation Graph

Two types of nodes:

- P = {P1, P2, …, Pn}, the set of all the **processes** in the system
- R = {R1, R2, …, Rm}, the set of all **resource** types in the system

Two types of edges:

- **request edge**: directed edge Pi ➞ Rj
- **assignment edge**: directed edge Rj ➞ Pi

**An example**

<img src="https://i.loli.net/2020/11/18/a3nbwJS7CgBXydf.png" alt="Deadlock%200a3be237fbe148ab8e725459c4afb6e6/Untitled.png" style="zoom: 33%;" />

- We can see, there exist a deadlock: p1->r1->p2->r3->p3->r2->p1 p2->r3->p3->r2->p2



**And we can conclude that:** No circles means no deadlock, or if any circles, then exist a **possibility** of deadlock

- if only **one instance per resource type → deadlock**
- if **several instances** per resource type → **possibility** of deadlock



---

# How to handle

Tree types

1. Ensure that the system will never enter a deadlock state - **Prevention** and **Avoidance**
2. Allow the system to enter a deadlock state and then recover - **Deadlock detection** and **recovery**:
3. **Ignore the problem** and pretend deadlocks never occur in the system



### Deadlock prevention

This means we need to **break the condition of deadlock**, considering the 4 conditions that may cause deadlocks.

1. How to prevent **mutual exclusion**
    - not required for sharable resources
    - must hold for non-sharable resources
2. How to prevent **hold and wait**
    - All or nothing type:
    - require process to request ***all*** its resources before it begins execution
    - allow process to request resources only when the process has none
    - Problems: low resource utilization; starvation possible
3. How to handle **no preemption**
    - if a process requests a resource not available, release all resources currently being held
    - preempted resources are added to the list of resources it waits for, and process will be restarted only when it can get all waiting resources
4. How to handle **circular wait**
    - impose a total ordering of all resource types, that is, require that each process requests resources in an increasing order.
    - **Many operating systems adopt this strategy for some locks**
    
    

### Deadlock Avoidance

This means we need to avoid get into deadlock, but not solving it after it happens.

More needed info: **Resource-allocation state**

- the number of **available** and **allocated** resources
- the **maximum demands** of the processes

#### Safe State

When a process requests an available resource, system must decide if immediate allocation leaves the system in a **safe state**:

- a **sequence <P1, P2, …, Pn>** of all processes in the system**(in amount of needed-resource order)**
- for each Pi, resources that Pi can still request can be satisfied by **currently available resources + resources held by all the Pj**, with j < i



**Safe state can guarantee no deadlock,** if Pi’s resource needs are not immediately available:

- wait until all Pj have finished (j < i)
- when Pj (j < i) has finished, Pi can obtain needed resources,
- when Pi terminates, Pi +1 can obtain its needed resources, and so on

#### Banker’s Algorithm

Banker’s algorithm is for **multiple-instance resource deadlock avoidance**

- each process must claim **maximum** use of each resource type **in advance**
- when a process requests a resource it may have to wait
- when a process gets all its resources it must release them in a finite amount of time



**Algorithm - Safe State**

`work` is used to track the allocable resources, and `finish` track whether a process has finished(allocation = 0)

1. find an i such that **finish[i] = false && need[i] ≤ work,** if no such i exists, go to step 3
2. **work = work + allocation[i]**, **finish[i] = true**, go to step 1
3. if finish[i] == true for all i, then the system is in a safe state



**Algorithm - Allocate Resource**

1. if **request[i]≤ need[i]** go to step 2; otherwise, raise error condition (the process has exceeded its maximum claim)

2. if **request[i] ≤ available**, go to step 3; otherwise Pi must wait.

3. **pretend** to allocate requested resources to Pi by modifying the state
    - available = available – request[i]
    - allocation[i] = allocation[i] + request[i]
    - need[i] = need[i] – request[i]
    
4. use previous algorithm to test if it is a safe state, if so, allocate the resources to Pi

5. if unsafe, Pi must wait, and the old resource-allocation state is restored.

    

## Deadlock Detection

Allow system to **enter** deadlock state, but detect and recover from it.



**Detection Algorithm**

1. Find an process i such that **finish[i] == false && request[i] ≤ work,** if no such i exists, go to step 3

2. **work = work + allocation[i]**; **finish[i] = true**, go to step 1

3. If finish[i] == false for some i, the system is in deadlock state, and if finish[i] == false, then Pi is deadlocked

   

**Recovery**

**Option 1** - terminate deadlocked processes:

- abort all deadlocked processes
- or abort one process at a time until the deadlock cycle is eliminated

**Option 2 -** Resource preemption

- Select a victim and roll it back
- May cause starvation