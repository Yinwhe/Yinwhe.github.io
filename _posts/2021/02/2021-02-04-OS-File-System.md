---
layout:	post
title: "OS-File System"
subtitle: ""
date: 2021-02-04 17:00:00
author: "Yinwhe"
header-style: text
tags:
    - Operating system
---


# 1. File

**File** is a contiguous logical space for storing information. The format to store and ways to explain the content is defined by its creator.

- **Attributes**

    **Name** – only information kept in **human-readable form**

    **Identifier** – unique tag (number) identifies file **within file system.** *(Not readable to human)*

    **Type** – needed for systems that support different types

    **Location** – pointer to **file location on device**

    **Size** – current file size

    **Protection** – permissions

    **Time, date, and user identification** – data for protection, security, and usage monitoring

Information about files are kept in the directory structure, which is maintained on the disk.

- **Operations**

    **Create -** Need **space** in file system and **entry** in directory

    **Open** - Most operations need to file to be opened first

    **read/write** - Need to maintain a pointer to files

    **Close**

    **Delete** - Release file space. *(If hard link, file won't be removed until last link is deleted)*

    **truncate** - empty a file but maintains its attributes

Other operations can be implemented using these basic ones.

**File Type Distinguish -** can be implemented in two ways: Extension name  or Magic number of file (elf)

**File Structure**

A file can have different structures, determined by OS or program

- N**o structure** - a stream of bytes or words
- **Simple record structure -** lines of records, fixed length or variable length(like, database)
- **complex structures -** usually defined by program, like, word document, relocatable program file.

**Access Method** - Sequential or Direct

- Sequential means elements are accessed in a predetermined order
- Direct(also called random access) is to access an element at an **arbitrary position** in a sequence in (roughly) **equal time**, independent of sequence size;

Other types of access can be based on `Direct`

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbkz673gkj30n90dymzq.jpg" alt="image-20210204162715091" style="zoom:75%;" />

# 2. Directory

Directory is a collection of nodes containing information about files.

- **Organization Targets**

    **efficiency**: to locate a file quickly

    **naming**: organize the directory structure to be convenient to users

    - two users can have same name for different files
    - the same file can have several different names

    **grouping**: provide a way to logically group files by properties

## 2.1 Structure

### 2.1.1 Single-Level Structure

![image-20210204162801896](https://tva1.sinaimg.cn/large/008eGmZEgy1gnbkzz6ermj31c109oaiv.jpg)

A single directory for all users - naming problems and grouping problems

- Two users want to have same file names
- Hard to group files

### 2.1.2 Two-Level Structure

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbl07bgxmj31c70gg7gd.jpg" alt="image-20210204162812959" style="zoom:50%;" />

Separate directory for each user

- different user can have the same name for different files
- Each user has his own user file directory `UFD`, it is in the master file directory `MFD`
- Efficient to search, but still cannot group files

### 2.1.3 Tree-Structureo

**Advantages**

- efficient in searching, can group files, convenient naming - solving file name conflicts for different users!

**Two types** - Acyclic-Graph or General-Graph

## 2.2 Mounting

A file system must be **mounted** before it can be accessed

- mounting links a file system to the system, usually forms a **single name space**
- the location of the file system being mounted is call the **mount point**
- a mounted file system makes the old directory at the mount point **invisible**

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbl1hk2dfj31jk0lg49l.jpg" alt="image-20210204162929083" style="zoom:33%;" />

**a**: existing file system

**b**: an unmounted partition

**c**: the partition mounted at **/users** *(Old user directory is set invisible)*

## 2.3 Sharing

Sharing must be done through a protection scheme

- **User ID**s identify users, allowing protections to be per-user
- **Group ID**s allow users to be in groups, permitting group access rights

**Remote File Sharing**

**Use networking** to allow file system access between systems

- manually via programs like `FTP`
- automatically, seamlessly using distributed file systems `DFS`

**Client-server** model allows clients to **mount** remote FS from servers

- a server can serve multiple clients
- client and user-on-client identification is complicated, based on the idea that server won't assume clients trustful

**`NFS`** is standard UNIX file sharing protocol, **`CIFS`** is standard for Windows

## 2.4 Protection - ACL

Access Control List **`ACL`**

---

# 3. FS Implementation

> File system is usually implemented and organized into layers

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbl2owzp4j30l212sdmc.jpg" alt="image-20210204163029812" style="zoom:25%;" />

**Layered File System**

- Layering useful for **reducing complexity** and redundancy, but adds overhead and can **decrease performance**
- Translates file name into file number, file handle, location by maintaining file control blocks (**`inodes`** in UNIX)
- Logical layers can be implemented by any coding method according to OS designer

**Logical File System -** Keep all the meta-data necessary for the file system  *(everything except file content)*

- It stores the directory structure
- Also stores a data structure that stores the file description **(File Control Block - `FCB`)**
    - Name, ownership, permissions
    - Reference count, time stamps, pointers to other FCBs
    - Pointers to data blocks on disk
- **Input from above** -  Open/Read/Write file path
- **Output to below** - Read/Write logical blocks (to lower layer)

**File-Organization Module** - It performs **translation from logical file block to corresponding physical block**

- It also manages free space
- **Input from above -** Read logical block 3 ; Write logical block 17
- **Output below -** Read physical block 43 ; Write physical block 421

**Basic File System -** Allocates/maintains various **buffers** that contain file-system, directory, and data blocks. It sends operations to device driver.

- These buffers are caches used for enhancing performance
- Input from above - Read physical block #43 ; Write physical block #421
- Output below - Read physical block #43 ; Write physical block #421

**I/O Control -** Device drivers and interrupt handlers

- Input from above - Read physical block #43 ; Write physical block #124
- Output below - Writes into device controller’s memory to enact disk reads and writes ; React to relevant interrupts

## 3.1 FS Data Structure

> FS info are stored in disk, memory can be used as cache to improve performance.

- **On-disk structures**

    An optional boot control block

    - First block of a volume that stores an OS *(all info needed to start an OS)*
    - It's called boot block in UFS, partition boot sector in NTFS

    A volume control block

    - Contains the number of blocks in the volume, block size, free-block count, free block pointers, free-FCB count, FCB-pointers...
    - Also called superblock in UFS, master file table in NTFS

    A directory - File names associated with an ID, FCB pointers.

    **A per-file File Control Block `FCB`**

    - In NTFS, the FCB is a row in a relational database
- **In-memory structures**

    A mount table with one entry per mounted volume

    A directory cache for fast path translation (performance)

    **A global open-file table**

    **A per-process open-file table**

    Various buffers holding disk blocks “in transit” (performance)

**Some important operations**

**`Create`**

Application process requests the creation of a new file

- logical file system **allocates a new FCB**, i.e., inode structure
- appropriate directory is updated with the new file name and FCB

**`Open`**

search **System-Wide Open-File Table** to see if file is currently in use

- if it is, create a Per-Process Open-File table entry pointing to the existing System-Wide Open-File Table *(performance)*
- if it is not, search the directory according to the given file name; once found, load the **`FCB`** from disk to memory and place it in the System-Wide Open-File Table

Make an entry in the **Per-Process Open-File Table**, with pointers to the entry in the **System-Wide Open-File Table** and other fields, which may include a pointer to the current location in the file *(for read and write)* and the access mode.

Increment the open count in the System-Wide Open-File Table

Returns a pointer to the appropriate entry in the Per-Process Open-File Table

All subsequent operations are performed with **this pointer**

**`Close`**

Process closes the file -> Per-Process Open-File Table entry is removed; **open count decremented**

All processes close the file -> copy in-memory directory information to disk and System-Wide Open-File Table entry is removed from memory

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbl4paquej311j0u0nd1.jpg" alt="image-20210204163233824" style="zoom:40%;" />

## 3.2 VFS

VFS provides an **object-oriented** way of implementing file systems, it separates FS operation from implementation details

- OS defines a common **interface** for FS, all FS shall implement these interfaces.
- System call is implemented based on this common interface, it allows the same syscall API to be used for different types of FS

When performing `write syscall`, OS call `vfs_write`, and in fact `vfs_write` can be a paking of `ext4_file_write_iter`

Linux defines four **VFS object types**

- **`superblock object`**: defines the file system type, size, status, and other metadata
- **`inode object`**: contains metadata about a **file** (location, access mode, owners…)
- **`dentry object`**: associates names to inodes, and the directory layout
- **`file object`**: actual data of the file

## 3.3 Directory Implementation

Directory is a special file, which stores the **mapping from file name to `inode`**

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbl5gr7bqj30x80dvmyh.jpg" alt="image-20210204163318366" style="zoom:50%;" />

**Linear list of file names** with pointer to the file metadata

- simple to program, but **time-consuming to search** (e.g., linear search)
- could keep files ordered alphabetically via linked list or use B+ tree

**Hash table**: linear list with hash data structure to reduce search time

- collisions are possible: two or more file names hash to the same location

## 3.4 Disk Block Allocation

### 3.4.1 Contiguous Allocation

Each file is in a set of **contiguous blocks**, directory keeps track of each file through the address of its **first block and of its length in blocks.**

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbl67dgtvj30nk0lfwm8.jpg" alt="image-20210204163359798" style="zoom:33%;" />

**Advantages** - sequential access causes little disk head movement, and thus **shorten seek times**

**Disadvantages** - Difficult to find free space; External fragmentation; Difficult to have files grow.

### 3.4.2 Linked Allocation

Each file is a linked list of disk blocks, and each block contains pointer to next block, file ends at nil pointer.

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbl6iywtvj30w30u01kx.jpg" alt="image-20210204163416778" style="zoom:25%;" />

**Advantages** - no external fragmentation; file size can be flexible.

**Disadvantages**

- locate a file block can take many I/O and disk seeks
- Pointer takes up space
- Unreliable, one pointer fails, file broken.
- internal fragmentation

File-Allocation-Table `FAT` *(used in DOS)* use linked allocation

### 3.4.3 Indexed Allocation

Each file has its own **index blocks that contains pointers to its data blocks**

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbl7exkjoj30y30u0qr4.jpg" alt="image-20210204163509550" style="zoom:25%;" />

Unix **`FCB`** - **`inode`** *(index node)*

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnbl8jiihsj313w0u0ngm.jpg" alt="image-20210204163614578" style="zoom:35%;" />

First 15 pointers are in inode

- Direct block: first 12 pointers
- Indirect block: next 3 pointers

> If block size is 512B and pointer 4B, then one inode can index 12\*512+128\*512+128\*128\*512+128\*128\*128\*512 Bytes

## 3.5 Free-Space Management

> It's about how to denote and manage a free block.

**Bitmap**

Use one bit for each block, track its allocation status - `1` for block free and `0` for block occupied

- To scan bits is supported by hardware, thus it's fast when in memory.
- But this can consume extra memory space, especially when the disk is huge.

**Linked List**

Keep free blocks in linked list

- no waste of space, just use the memory in the free block for pointers
- cannot get contiguous space easily
- Usually no need to traverse the entire list: return the first one

**Grouping**

Use **indexes** to group free blocks - store address of **n-1** free blocks in the **first free block**, plus a pointer to the next **index block**

- allocating **multiple free blocks does not need to traverse the list**

**Counting**

A link of clusters - each node stores address of starting block + number of contiguous blocks.

- Base on the fact that space is frequently contiguously used and freed
- in link node, keep address of first free block and # of following free blocks

# 4. Recovery

File system needs consistency checking to ensure consistency

- compares data in directory with some metadata on disk for consistency
- fs recovery can be slow and sometimes fails

File system recovery methods

- backup (RAID)
- log-structured file system

**Log Structured File Systems**

In LSFS, metadata for updates sequentially written to a **circular log***(reusable);* once changes written to the log, it is committed, and syscall can return

log can be located on the other disk/partition; meanwhile, log entries are **replayed** on the file system to actually update it

- when a transaction is replayed, it is removed from the log
- a log is circular, but un-committed entries will not be overwritten
- garbage collection can reclaim/compact log entries

upon system crash, only need to replay transactions existing in the log