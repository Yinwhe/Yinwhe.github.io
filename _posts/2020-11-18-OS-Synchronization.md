---
layout:	post
title: "OS-Synchronization"
subtitle: ""
date: 2020-11-18 20:00:00
author: "Yinwhe"
header-style: text
tags:
    - Operating system
---

[TOC]



# Data race

**Reason**

1. Processes can execute concurrently, and may be interrupted at any time.
2. Concurrent access to shared data may result in **data inconsistency.**
- Examples

    Two threads do `counter++` concurrently

    ![Synchronization%203e50f335581b4c048a119868f133fb10/Untitled.png](https://i.loli.net/2020/11/18/sAJMClKwfNp3bV5.png)

#### Race Condition

Several processes (or threads) access and **manipulate the same data concurrently** and the outcome of the execution depends on the particular **order in which the access takes place**

# Critical Section

The codes that manipulate **shared data**.

- In order to **solve** data race:
    - when one process in critical section, no other may be in its **critical section**
    - each process must ask permission to enter critical section in **entry section**
    - the permission should be released in **exit section**
    - other codes are called **remainder section**

    ![Synchronization%203e50f335581b4c048a119868f133fb10/Untitled%201.png](https://i.loli.net/2020/11/18/Jxt65dc1eysSOQp.png)

- **Critical-Section Handling in OS**
    - Single-core system: preventing interrupts
    - Multiple-processor: preventing interrupts are not feasible, two approaches depending on if kernel is ***preemptive or non-preemptive***
        - Preemptive – allows preemption of process when running in kernel mode
        - Non-preemptive – runs until exits kernel mode, blocks, or voluntarily yields

#### Solution Requirements

1. **Mutual Exclusion**
   
    - only one process can execute in the critical section
2. **Progress**
    
    - if no process is executing in its critical section and some processes wish to enter their critical section, then only those processes that are not executing in their remainder sections can participate in deciding which will enter its critical section next, and this selection cannot be postponed indefinitely.
    - The **purpose** of this condition is to make sure that either some process is currently in the CS and doing some work, or, if there was at least one process that wants to enter the CS, it will enter and do some work. In both cases, some work is getting done and therefore all processes are **making progress** overall.
3. **Bounded waiting**
    - If there is a process that has requested to enter its CS but has not yet entered it, then there exists a bound, or limit, on the number of times that **other processes** are allowed to enter their critical sections.
    - The purpose of this condition is to make sure that every process gets the chance to actually enter its critical section so that no process starves forever.

    [What is progress and bounded waiting in critical section?](https://stackoverflow.com/questions/33143779/what-is-progress-and-bounded-waiting-in-critical-section?progress-and-bounded-waiting-in-critical-section)

# Synchronization

### Peterson Solution

This works when there only exist two processes.

It assumes that LOAD and STORE are **atomic.** The two processes share two variables

- int **turn**: whose turn it is to enter the critical section
- Boolean **flag[2]**: whether a process is ready to enter the critical section

P0

```cpp
do { 
	flag[0] = TRUE;//mark self ready
	turn = 1;//assert the other process
	while (flag[1] && turn == 1); 
	
	critical section 
	flag[0] = FALSE; 
	remainder section 
} while (TRUE);
```

P1

```c++
do { 
	flag[1] = TRUE; 
	turn = 0; 
	while (flag[0] && turn == 0); 
	
	critical section 
	flag[1] = FALSE; 
	remainder section 
} while (TRUE);
```



![Synchronization%203e50f335581b4c048a119868f133fb10/Untitled%202.png](https://i.loli.net/2020/11/18/vYwCJMKnFSZTO7H.png)

### Hardware Support

- **Uniprocessors: disable interrupts**
    - currently running code would execute without preemption
    - generally too inefficient on multiprocessor systems - need to disable all the interrupts and operating systems using this is not scalable

#### Memory barriers

A **memory barrier** is an instruction that forces any change in memory to be propagated (made visible) to all other processors.

- Example

    Add a memory barrier to the following instructions to ensure Thread 1 outputs 100

    ```cpp
    //Thread 1 now performs
    	while (!flag);
    	memory_barrier();
    	print x

    //Thread 2 now performs
    	x = 100;
    	memory_barrier();
    	flag = true
    ```

#### Hardware Instuction

**Test-and-Set**

```cpp
bool test_set (bool *target)
{
	bool rv = *target;
	*target = TRUE;
	return rv;
}
```

**Compare-and-Swap**

```cpp
int compare_and_swap(int *value, int expected, int new_value){
	int temp=*value;

	if(*value==expected)
			*value=new_value;
	return temp;
}
```

#### Atomic variables

Typically, instructions such as compare-and-swap are used as building blocks for other synchronization tools.

- One tool is an **atomic variable** that provides atomic (uninterruptible) updates on basic data types such as integers and booleans.
- `Increment(&sequence)` for example

```cpp
void increment(atomic_int *v) {
	int temp;

	do {
		temp = *v;
	}while (temp != (compare_and_swap(v,temp,temp+1));
	//if temp was changed by other process, then v!=temp and v won't be increased
}
```

---

## Lock

- With `Test-and-Set`

    shared variable: `bool lock=false` `bool waiting[]=false`

    ```cpp
    do {
    	waiting[i] = true;
    	while(waiting[i] && test_and_set(&lock)); // busy waiting
    	waiting[i] = false;
    	//critical seciton

    	//following code are bounded waiting
    	j = (i + 1) % n;
    	while(j!=i && !waiting[j])
    		j = (j+1)%n;
    	if(j == i)
    		lock = false;
    	else waiting [j] = false;
    	//remainder section
    } while (TRUE);
    ```

- With `Compare-and-Swap`

    ```cpp
    while(true){
    	while(compare_and_swap(&lock, 0, 1)!=0);
    	//critical section
    	lock=0;
    	//remainder seciton
    }
    ```

> Previous solutions are complicated and generally inaccessible to application programmers, so here we pack then into software tools

### Mutex Locks

- Protect a critical section by first **`acquire()`** a lock then **`release()`** the lock, this two functions must be atomic.
- This solution requires **busy waiting,** therefore it's called a **spinlock.**

```cpp
void acquire(){
	while(compare_and_swap(&available, 1, 0)==0);
	//busy waiting
	//-------------------or----------------------
	while(test_and_set(&lock));
	//busy waiting
}

void release(){
	available=true;
	//-------------------or----------------------
	lock=false;
}
```

- A huge problem is **too much spinning**
    - T0 acquires lock -> INTERRUPT->T1 runs, spin, spin spin...
    - T1 busy waits all its CPU time until T0 releases lock
    - A waste of CPU time and power

### Semaphore

- Contain S – integer variable
- Can only be accessed via two atomic operations **wait()** and **signal()**

```cpp
void wait(S){
	while(S<=0); //busy waiting
	s--;
}

void signal(S){
	s++;
}
```

- Two types of semaphore
    - Counting semaphore - integer value can range over an unrestricted domain.
    - Binary semaphore - integer value can range only between 0 and 1 Same as a **mutex lock**
- **Still too much waiting time!**

    ![image-20201118194354153](https://i.loli.net/2020/11/18/pX3j1V2AtcGdxk6.png)

#### Semaphore with waiting queue

With each semaphore there is an associated waiting queue, and each entry in a waiting queue has two data items:

- value (of type integer)
- pointer to next record in the list

Two operations:

- **block** – place the process invoking the operation on the appropriate waiting queue
- **wakeup** – remove one of processes in the waiting queue and place it in the ready queue

```cpp
typedef struct {
	int value;
	struct list_head * waiting_queue;
} semaphore;
```

**Implementation**

```cpp
wait(semaphore *S) { 
	S->value--; 
	if (S->value < 0) {
		//add this process to S->list
		block(); 
	} 
}

signal(semaphore *S) { 
	S->value++; 
	if (S->value <= 0) {
		//remove a process P from S->list
		wakeup(P); 
	} 
}
```

- A practical example

    ```cpp
    typedef struct __lock_t{
    	int flag;//init 0
    	int guard;//init 0
    	queue_t *q;
    }lock_t;

    void lock(lock_t *m){
    	while(TestAndSet(&m->guard));
    		//acquire guard lock by spinning
    	if(m->flag==0){
    		m->flag=1;//acquire flag;
    		m->guard=0;
    	}else{
    		queue_add(m->q, gettid()):
    		m->guard=0;
    		park();
    	}
    }

    void unlock(lock_t *m){
    	while(TestAndSet(&m->guard));
    		//acquire guard lock by spinning
    	if(queue_empty(m->q))
    		m->flag=0;//release lock
    	else
    		unpark(queue_remove(m->q));//hold lock for next thread
    	m->guard=0;
    }
    ```

# Examples

### Bounded-Buffer Problem

> Two processes, the producer and the consumer share **n** buffers: the producer generates data, puts it into the buffer, and the consumer consumes data by removing it from the buffer

**Goals**

- the producer won’t try to add data into the buffer if it is full
- the consumer won’t try to remove data from an empty buffer

**Solution**

- n buffers, each can hold one item
- semaphore **mutex** initialized to the value **1**
- semaphore **full-slots** initialized to the value **0**
- semaphore **empty-slots** initialized to the value **N**

**Producer**

```cpp
do {
	//produce an item
	wait(empty-slots);
	wait(mutex);
	//add the item to the buffer
	signal(mutex);
	signal(full-slots);
} while (TRUE);
```

**Consumer**

```cpp
do {
	wait(full-slots);
	wait(mutex);
	//remove an item from buffer
	signal(mutex);
	signal(empty-slots);
	//consume the item
} while (TRUE);
```

Note: The order CANNOT be changed, or there will be **deadlock**! Producer for example. Now we do `wait(mutex)` first and then  `wait(empty-slots)` , assuming we already got the mutex and what if the slots if full? In this case producer cannot get `empty-slots`, and the consumer cannot get mutex thus cannot consume the full slots, it start waiting too. Deadlock.

### Readers-Writers Problem

> A data set is shared among a number of concurrent processes: readers: only read the data set, they do not perform any updates, and writers can both read and write.

**Goals**

- allow multiple readers to read at the same time (**shared access**)
- only one single writer can access the shared data (**exclusive access**)

**Solution**

- semaphore **mutex** initialized to 1
- semaphore **write** initialized to 1
- integer **readcount** initialized to 0

Writer

```cpp
do {
	wait(write);
	//write the shared data
	signal(write);
} while (TRUE);
```

This variation is `Reader first` 

- no reader kept waiting unless writer is updating data
- If reader holds data, new reader just moves on and reads
- writer may starve

Readers

```cpp
do {
	wait(mutex);
	readcount++ ;
	if (readcount == 1) 
		wait(write);
	signal(mutex)

	//reading data
	//...
	wait(mutex) ;
	readcount--;
	if (readcount == 0) 
		signal(write) ;
	signal(mutex) ;
} while(TRUE);
```

What readcount for?
Note that before reading, we shall block write first. Readcount here is used the to block the write for only once. If we discard this, then only one reader can read at a time, for only one reader can get the write semaphore.

### Dinning Philosophers Problem

> Philosophers spend their lives thinking and eating, they sit in a round table, but don’t interact with each other. There are one chopstick between each adjacent two philosophers. Each time they will pick up one chopstick, and, two chopsticks are needed to eat.

Dining-philosopher problem represents **multi-resource synchronization**

**Solution** (assuming **5 philosophers**):

- semaphore **chopstick[5]** initialized to 1
- only odd philosophers start left-hand first, and even philosophers start right-hand first. There won't be deadlock in such case.

# Linux Synchronization

**Linux provides**

- atomic integers
- spinlocks
- semaphores
    - on single-CPU system, spinlocks replaced by enabling/disabling kernel preemption
- reader-writer locks

### RCU

- **read-copy-update** (**RCU**) is a synchronization mechanism based on mutual exclusion. It is used when performance of reads is crucial and is **an example of space–time tradeoff, enabling fast operations at the cost of more space.**
- Read-copy-update allows multiple threads to efficiently read from shared memory **by deferring updates after pre-existing reads** to a later time while simultaneously marking the data, ensuring new readers will read the updated data.
- This makes all readers proceed as if there were no synchronization involved, hence they will be fast, but also making updates more difficult.

# POSIX Synchronization

> Widely used on UNIX, Linux, and macOS

**POSIX API provides**

- mutex locks
- semaphores
- condition variable

**POSIX semaphore**

two versions – **named** and **unnamed**

- Named semaphores can be used by unrelated processes, unnamed cannot.

**Condition Variables**

**`condition x`** Two operations are allowed on a condition variable:

- **`x.wait()`** – a process that invokes the operation is suspended until **`x.signal()`**
- **`x.signal()`** – resumes one of processes (if any) that invoked **`x.wait()`**
- If no **`x.wait()`** on the variable, then it has no effect on the variable