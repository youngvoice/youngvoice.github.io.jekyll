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

# 在本书中，进程在内核中的运行与切换模型（是否抢占）是什么样的？

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
