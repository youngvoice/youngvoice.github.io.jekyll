---
title: what story a program has? 
description: the story of a whole life of program
categories:
 - question
tags: [program]
---

# how we use a computer?
In common, we use a computer through command line or GUI, which all through program.

# what is the composition of a user program?
data and text
# what is a user process
program + runtime context

# what happend, when we run a program in linux system ?
1. the shell call exec system call enter kernel space
2. the kernel create the process image and prepare runtime environment 
3. then switch to the new process and execute it
4. return from kernel space and run user code

a simple shell design:
```c
main()
{
    char command[];
    while (1)
    {
        scanf("user#%s", command);
        if (!fork()) {exec(command, NULL);}
        wait();
    }
}
```

the child process run command.

# what is the execute follow of user program in user space? 
we can gdb debug the execute process
we can try to compiler the program


When we do something in program, we need to use some operating system functions to access hardware or some other resources that we can't directly to operate. The gates or interfaces are syscalls.

For example:
```c
.text
.global _start
_start:
    xorl %eax, %eax
    movb $4, %al
    xorl %ebx, %ebx
    incl %ebx
    movl $.LC0, %ecx
    xorl %edx, %edx
    movb $13, %dl
    int $0x80
    xorl %eax, %eax
    movl %eax, %ebx
    incl %eax
    int $0x80

.section .rodata
.LC0:
.string "Hello World\xa\x0"

```
compile the program
```bash
as --32 -o hello.o hello.s
ld -melf_i386 -o hello hello.o

```

通过strace跟踪系统调用，如下
```bash
strace hello

execve("./hello", ["./hello"], 0x7fff53978d40 /* 41 vars */) = 0
strace: [ Process PID=2500301 runs in 32 bit mode. ]
write(1, "Hello World\n\0", 13Hello World
)         = 13
exit(0)                                 = ?
+++ exited with 0 +++

```

可以发现打印字符串到屏幕的任务中，调用了execve, write, exit系统调用
## what is system calls?
Operating system offer processes running in User mode a set of interfaces (__the most interfaces are system calls__) to interact with hardware devices. Putting an extra layer between the application and the hardware has several advantages. 

First, it makes programming easier by freeing users from studying low-level programming characteristics of hardware devices. 
Second, it greatly increases system security, because the kernel can check the accuracy request at the interface level before attempting to satisfy it.
Last but not least, these interface make program more portable, because they can be compiled and executed correctly on every kernel that offers the same set of interfaces.
Reference ULK.
# why we not call syscall directly from normal program?
lib wrapper 和 syscall 之间的对应关系
POSIX API
API(application programmer interface) is a function definition that specifies how to obtain a given service.

The POSIX standard refers to API. A system can be certified as POSIX-compliant if it offers the proper set of APIs to the application program, no matter how the corresponding functions are implemented. 

Several non-Unix systems have been certified as POSIX-compliant, because they offer all traditional Unix services in User mode libraries.

Some of the APIs defined by the libc standard C library refer to wrapper routines (routines whose only purpose is to issue a system call). Usually, each system call has a corresponding wrapper routine, which defines the API that application programs should employ. The converse is not true, by the way, an API does not necessarily correspond to a specific system call. First of all, the API could offer its services directly in User Mode. Second, a single API function could make serveral system calls. Moreover, several API functions could make the same system call, but wrap extra functionaliy around it.
Reference ULK.

# how to implement syscall in kernel space?
就像调用了一个函数，通过传递的参数来判断执行，（系统调用号作为一个参数，在内核）
系统调用号与 sys_call_table 
syscall in kernel are a c function, so how to pass parameter and return value? (calling conventions, ABI, the specification specify store parameter in register or stack)


## how to implement write syscall?

let's directly look the kernel implement
```c
SYSCALL_DEFINE3(write, unsigned int, fd, const char __user *, buf,
                size_t, count)
{
        return ksys_write(fd, buf, count);
}


ssize_t ksys_write(unsigned int fd, const char __user *buf, size_t count)
{
        struct fd f = fdget_pos(fd);
        ssize_t ret = -EBADF;

        if (f.file) {
                loff_t pos, *ppos = file_ppos(f.file);
                if (ppos) {
                        pos = *ppos;
                        ppos = &pos;
                }
                ret = vfs_write(f.file, buf, count, ppos);
                if (ret >= 0 && ppos)
                        f.file->f_pos = pos;
                fdput_pos(f);
        }

        return ret;
}
```
writes data from a buffer declared by the user to a given device or a file.

找到内核中管理文件的结构，然后将内容写入文件，再更新文件的位置指示器。

fdget_pos function gets the file descriptor table of the current process, current->files , and then get the fd structure for the given file descriptor number. If there are multiple thread, then mutex_lock(&file->f_pos_lock);

we get the current postion in the file with the call of the file_ppos that just return f_pos field of our file.

and then call the vfs_write function.

we change the position in the file. That just updates f_pos with the given position in the given file

finally unlocks the f_pos_lock mutex that protects file position during concurrent writes from threads that share file decriptor.

## what will happen when multiple process open the same file to write?
从用户层面文件是在各个进程之间共享的资源，所以，当同时多个进程打开并写入时，操作系统对写入的顺序没有任何保证，这样很可能会使输出的内容交织在一起无法辨认。

# what is the system state when multiple process in progress?(多进程同时执行时的样子如何描述)



# what is the user space entry for new process return from kernel?
1. statically linked program
user program entry.
2. dynamically linked program
first run dynamic linker, then transfer control to user program entry.



# how to represent the process in kernel?


