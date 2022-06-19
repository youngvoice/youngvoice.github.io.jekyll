---
title: the design of unix 
description: the filesystem and process
categories:
 - book
 - review
tags: [book]
---
Preface
This book describes the internal algorithms and structures that form the basis of the operating system and their relationship to the programmer interface.

understanding the code was easier once the concepts of the algorithms had been mastered.
###### 为啥linus让去直接读代码？

the system description is based on UNIX System V Release 2.



Chapter 1, General overview of the system
The system combined of programs and services that readily apparent to users and operating system that supports these programs and services.
the major data structures and algorithms 

1.1 
The phylosophy of the UNIX system is simplicity and consistency.
1.2 SYSTEM structure
the hardware: the hardware provides the operating system with basic services

the operating system interact directly with the hardware, providing common services to programs 

programs interact with the kernel by invoking a well defined set of system calls.


programs are easy to move between UNIX systems running on different hardware if the programs do not make assumptions about the underlying hardware.

the set of system calls and the internal algorithms that implement them form the body of the kernel.(系统调用是通向操作系统的大门)

1.3 user perspective
high-level features of the UNIX system
##### how the kernel support these features???
1.3.1 The file system 
1.3.2 Processing environment
1.3.3 building block primitives
1. redirect 
2. pipe

1.4 Operating system services
1.5 assumptions about hardware
two levels: user mode and kernel mode
address space
privileged instructions

Put simply, the hardware views the world in terms of kernel mode and user mode.
The kernel keeps internal records to distinguish the many processes executing on the system.

The kernel is not a separate set of processes that run in parallel to user processes, but it is part of each user process. What the kernel doing various operations meant is that a process executing in kernel mode does the various operations, e.g allocates the resources.

#### who is the kernel services for?(process and interrupt) 
硬件直接服务的话是不是有点像单片机了？

1.5.1 Interrupts and Exceptions
exception condition refers to unexpected events caused by a process. 
interrupts caused by events that are external to a process.
##### 上面两句话，还是需要考量，比如在执行 interrupt 的时候发生 dividing by zero 的话，如何处理？？？

exceptions happen "in the middle" of the execution of an instruction, the system will attempt restart the instruction after handling the exception;
interrupts are considered to happen between the execution of two instructions, the system will continue the next instruction after servicing the interrupt.

the UNIX uses one mechanism to handle interrupts and exception conditions.

1.5.2 Processor execution levels

Chapter 2, Intro to the Kernel
Unix supports the illusions that the file system has "places" and that processes have "life".

##### 关于运行库与系统调用的关系？
System calls look like ordinary function calls in C programs, and libraries map these function calls to the primitives needed to enter the operating system. Programs frequently use other libraries such as the standard I/O library to provide a more sophisticated use of the system calls. 
2.2 Intro to system concepts
2.2.1 An Overview of the File Subsystem
inode
in-core inode


file table
user file descriptor table
inode table

##### what purposes these table used for?
maintain the state of the file and the user's access to it.
2.2.2 Processes

1. the byte offset in the file where the user's next read or write will start
2. access rights  

a process uses a separate stack for each mode.
##### 对于抢占式内核也是这样的吗？那之前的程序流保存在哪里呢？
The kernel stack of a process is null when the process executes in user mode.

process 0 becomes the swapper process.
process 1 known as init

2.2.2.1 context of a process
The kernel allows a context switch only under specific conditions.  

2.2.2.2 Process states
A processor can execute only one process at a time, at most one process may be in states 1 and 2. The two states correspond to the two modes of execution, user and kernel.
this is the static view of a process
2.2.2.3 State transitions
processes move continuously between the states.
A state transition diagram is a directed graph whose nodes represent the states a process can enter and whose edges represent the events that cause a process to move from one state to another.
State transitions are legal between two states if there exists an edge from the first state to the second.
# 在本书中，进程在内核中的运行与切换模型（是否抢占）是什么样的？

several processes can execute simultaneously in a time-shared manner, as stated earlier, and they may all run simultaneously in kernel mode. If they were allowed to run in kernel mode without constraint, they could corrupt global kernel data structures. By prohibiting arbitrary context switches and controlling the occurrence of interrupts, the kernel protects its consistency.

The kernel allows a context switch only when a process moves from the state "kernel running" to the state "asleep in memory." Processes running in kernel mode cannot be preempted by other processes; therefore the kernel is sometimes said to be non-preemptive, although the system does preempt processes that are in user mode. The kernel maintains consistency of its data structures because it is non-preemptive, thereby solving the mutual exclusion problem.
其实，该系统是支持用户空间并行，而不支持内核并行执行。但是当在内核中需要睡眠时，此时的切换是可以进行的。此时，kernel algorithms are encoded to make sure that system data structures are in a safe, consistent state.

A related problem that can cause inconsistency in kernel data is the handling of interrupts, which can change kernel state information. To solve this problem, the system could prevent all interrupts while executing in kernel mode, but that would delay servicing of the interrupt, possibly hurting system throughput. Instead, the kernel raises the processor execution level to prevent interrupts when entering critical regions of code(A section of code is critical if execution of arbitrary interrupt handlers could result in consistency problems 与中断处理有资源共享). 

e.g a disk interrupt handler manipulates the buffer queues, the section of code where the kernel manipulates the buffer queues is a critical region of code with respect to the disk interrupt handler.

Critical regions are small and infrequent so that system throughput is largely unaffected by their existence.

This problem can also be solved by preventing all interrupts when executing in system states or by using elaborate locking schemes to ensure consistency. In multiprocessor systems, the solution outlined here is insufficient.


* To conclude, the kernel protects its consistency by allowing a context switch only when a process puts itself to sleep and by preventing one process from changing the state of another process.
* It also raises the processor execution level around critical regions of code to prevent interrupts that could otherwise cause inconsistencies. 
* The process scheduler periodically preempts processes executing in user mode so that processes cannot monopolize use of the CPU.


2.2.2.4 Sleep and wakeup
Using "while-sleep" loop insures that at most one process can gain access to a resource.

Chapter 3, The buffer Cache

In figure 2.1, the buffer cache module is between the file subsystem and block devices drivers in the kernel architecture.

High level kernel algorithms instruct the buffer cache module to pre-cache data or to delay-write data to maximize the caching effect.

buffer 的数量比磁盘block数量上小得多，一个block一次只能map到一个buffer，否则会出现歧义。

3.3 scenarios for retrieval of a buffer
high-level kernel algorithms in the file subsystem invoke the algorithms for managing the buffer cache.  
high-level algorithms determine the logical device number and block number that they wish to access when they attempt to retrieve a block.

for example, if a process whats to read data from a file, the kernel determines which file system contains the file and which block in the file system contains the data.

when about to read data from a particular disk block, the kernel checks whether the block is in the buffer pool and, if it is not there, assigns it a free buffer. When about to write data to a particular disk block, the kernel checks whether the block is in the buffer pool, and if not, assigns a free buffer for that block. 
The algorithms for reading and writing disk blocks use the algorithm getblk to allocate buffers from the pool.


conclude in one word: 
when high-algorithm attempt to access a disk block, the kernel checks whether the block is in the buffer pool, and if not, assigns a free buffer for that block.
the getblk is algorithm used to allocate buffers from the pool.

# what are five typical scenarios allocating a buffer for a disk block?

## algorithm brelse

The kernel leaves the buffer marked busy; no other process can access it and change its contents while it is busy, thus preserving the integrity of the data in the buffer.


When the kernel finishes using the buffer, it releases the buffer according to algorithm brelse.
It wakes up 
1. processes that had fallen asleep because the buffer was busy
2. processes that had fallen asleep because no buffers remained on the free list.


## scenario 4
the kernel, acting for process A, cannot find the disk block on its hash queue, so it attempts to allocate a new buffer from the free list, however, no buffers are available on the free list, so process A goes to sleep,
until another process executes algorithm brelse, freeing a buffer.


when the kernel schedules the process A, it must search the hash queue again for the block.
### why it can not allocate a buffer immediately from the free list?
because it is possible that several processes were waiting for a free buffer and that one of them allocated a newly freed buffer for __the target block sought by process A__(one block can only correspond to one buffer, thus , searching for the block again insures that only one buffer contains the disk block).
这个地方其实有两种原因，
其一，free list 可能任然为空，因为可能有别的进程已经将free list上新出现的buffer分配完了
其二，有可能，已经有别的一个进程给Process A请求的block分配了buffer


## scenario 5
kernel, acting for process A, searches for a disk block and allocate a buffer but goes to sleep before freeing the buffer.
While process A sleeps, suppose the kernel schedules a second process, B, which tries to access the disk block whose buffer was just locked by process A.
So, the scenario is that process B will find the locked block on the hash queue.

process B marks the buffer "in demand" and then sleeps and waits for process A to release the buffer.
睡眠与唤醒的不匹配
Process A will eventually release the buffer and notice that the buffer is in demand. __It awakens all processes sleeping on the event "the buffer becomes free" including process B__.
when the kernel again schedules process B, process B must verify that the buffer is free.

思考一下：下面这句话是什么意思？
all processes sleeping on the event "the buffer becomes free" including process B
当释放的 buffer 有 in demand 标记的时候，究竟有没有唤醒等待 free buffer list 为空的进程？因为，等待出现新 buffer 的进程有两种情况，1. 等待 free buffer list 非空，2. 等待 buffer 被 free，这两种情况也对应了引起睡眠的两种原因。

### why process B can not directly to access the block?
another process, C, may have been waiting for the same buffer, and the kernel may have scheduled C to run before process B; process C may have gone to sleep leaving the buffer locked. Hence, process B must check that the block is indeed free. 



# when call algorithm brelse?
1. a process has no more need a buffer
2. handling a disk interrupt to release buffers used for asynchronous I/O to and from the disk.


## how to handle shared?
prevent disk interrupts
1. kernel raises the processor execution level to prevent disk interrupts while manipulating the free list, result from a nested call to brelse.

2. if an interrupt handler invoked brelse while a process was executing getblk, so the kernel raises the processor execution level at strategic places in getblk, too.


# when a process wakes up, where should it to execute?
it must search the hash queue again for the block.
当一个进程由于buffer被唤醒的时候，只是通知这个进程可能有buffer空闲，


3.4 READING AND WRITING DISK BLOCKS
?????????
