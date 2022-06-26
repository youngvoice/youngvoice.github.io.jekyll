---
title: what story a program has? 
description: the story of a whole life of program
categories:
 - question
tags: [program]
---

# how do we use a computer?
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
系统调用号与 sys_call_table (arch/x86/entry/syscalls/syscall_64.tbl)
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

# 进程如何操作文件？
有三个数据结构将VFS和系统的进程紧密联系在一起，分别是 file_struct, fs_struct, namespace结构体

关于对已打开文件的操作，首先从进程的 fdtable 中根据 fd 找到被打开文件的 struct file *file 然后进行读写操作
```c
fdget_pos(fd);
        __fdget_pos(fd)
                __fget_light(fd, FMODE_PATH);
                        struct files_struct *files = current->files;
                        file = files_lookup_fd_raw(files, fd);
        __to_fd(__fdget_pos(fd));

```
## what will happen when multiple process open the same file to write?
从用户层面文件是在各个进程之间共享的资源，所以，当同时多个进程打开并写入时，操作系统对写入的顺序没有任何保证，这样很可能会使输出的内容交织在一起无法辨认。

note: 
1. 程序运行会涉及到文件系统等别的子系统

# what is the model of the file system?
sys_write 背后的实现逻辑是什么？

首先内核将文件相关的内容分成了
1. 与进程相关的部分
2. 与文件系统相关的部分（与管理磁盘相关的部分）
# In order to run a program, what the kernel need to do(get into through syscall)?
the kernel need to handle the following things.
the main task is load instructions into memory and point the cpu to them.
and also the kernel need to deal with flexibility in several areas:
1. different executable formats
2. shared libraries
3. other information in the execution context(command-line arguments and environment variables)
# How to create processes?
# how to represent the process in kernel?
process descriptor task_struct
task list

task_struct 中包含了能完整描述一个正在执行的程序所包含的数据，即内核管理一个进程所需要的全部信息。

打开的文件，进程的地址空间，挂起的信号，进程的状态...

# 与进程有关的组织结构？
(task_state)进程的状态
```c
#define TASK_RUNNING                    0x0000
#define TASK_INTERRUPTIBLE              0x0001
#define TASK_UNINTERRUPTIBLE            0x0002
#define __TASK_STOPPED                  0x0004
#define __TASK_TRACED                   0x0008

```
进程的家族树
```c
/*
* Pointers to the (original) parent process, youngest child, younger sibling,
* older sibling, respectively.  (p->father can be replaced with
* p->real_parent->pid)
*/

/* Real parent process: */
struct task_struct __rcu        *real_parent;

/* Recipient of SIGCHLD, wait4() reports: */
struct task_struct __rcu        *parent;

/*
        * Children/sibling form the list of natural children:
        */
struct list_head                children;
struct list_head                sibling;
struct task_struct              *group_leader;

```
可以猜想到在创建进程时需要完成以下的动作：

* 分配进程内核栈 及 thread_info
分配进程描述符 task_struct
将内核栈、thread_info 和 task_struct 关联起来

* 设置进程的状态

* 设置进程上下文

* 关联进程到进程家族树

---
three mechanism
1. The Copy on Write
2. lightweight processes: share paging tables, the open file tables, and the signal dispositions
3. The vfork() system call

# fork，vfork，clone之间的区别是什么？

lib function wrapper clone() is used to create Lightweight processes
```c
int clone(int (*fn)(void *), void *stack, int flags, void *arg, ...
                 /* pid_t *parent_tid, void *tls, pid_t *child_tid */ );

```
arguments for this function:
fn
arg
flags
child_stack
tls
ptid
ctid

```c
//sets up the stack of the new lightweight process
// invokes a clone() syscall hidden to the programmer
//save the pointer fn into the child's stack position corresponding to the return address of the wrapper function itself;
// the pointer arg is saved on the child's stack right below fn.

```

when the wrapper function terminates, the cpu fetches the return address from the stack and executes the fn(arg) function.

```c
clone(); //wrapper function
// when clone() return then execute fn(arg)
goto: fn(arg);
```


the sys_clone() service routine does not have the fn and arg parameters.
```c
//"kernel/fork.c"

#ifdef __ARCH_WANT_SYS_CLONE
#ifdef CONFIG_CLONE_BACKWARDS
SYSCALL_DEFINE5(clone, unsigned long, clone_flags, unsigned long, newsp,
                 int __user *, parent_tidptr,
                 unsigned long, tls,
                 int __user *, child_tidptr)
#elif defined(CONFIG_CLONE_BACKWARDS2)
SYSCALL_DEFINE5(clone, unsigned long, newsp, unsigned long, clone_flags,
                 int __user *, parent_tidptr,
                 int __user *, child_tidptr,
                 unsigned long, tls)
#elif defined(CONFIG_CLONE_BACKWARDS3)
SYSCALL_DEFINE6(clone, unsigned long, clone_flags, unsigned long, newsp,
                int, stack_size,
                int __user *, parent_tidptr,
                int __user *, child_tidptr,
                unsigned long, tls)
#else
SYSCALL_DEFINE5(clone, unsigned long, clone_flags, unsigned long, newsp,
                 int __user *, parent_tidptr,
                 int __user *, child_tidptr,
                 unsigned long, tls)
#endif
{
        struct kernel_clone_args args = {
                .flags          = (lower_32_bits(clone_flags) & ~CSIGNAL),
                .pidfd          = parent_tidptr,
                .child_tid      = child_tidptr,
                .parent_tid     = parent_tidptr,
                .exit_signal    = (lower_32_bits(clone_flags) & CSIGNAL),
                .stack          = newsp,
                .tls            = tls,
        };

        return kernel_clone(&args);
}
#endif
```

sys_fork and sys_vfork
```c

#ifdef __ARCH_WANT_SYS_FORK
SYSCALL_DEFINE0(fork)
{
#ifdef CONFIG_MMU
        struct kernel_clone_args args = {
                .exit_signal = SIGCHLD,
        };

        return kernel_clone(&args);
#else
        /* can not support in nommu mode */
        return -EINVAL;
#endif
}
#endif

#ifdef __ARCH_WANT_SYS_VFORK
SYSCALL_DEFINE0(vfork)
{
        struct kernel_clone_args args = {
                .flags          = CLONE_VFORK | CLONE_VM,
                .exit_signal    = SIGCHLD,
        };

        return kernel_clone(&args);
}
#endif

```

# fork 系统调用如何实现？
# libc 中的fork是直接调用 fork syscall， 还是调用clone ？
libc 中的 fork wrapper
```c


weak_alias (__libc_fork, fork)
__libc_fork()
        arch_fork();
                INLINE_CLONE_SYSCALL();
```
the traditional fork() system call is implemented by Linux as a clone() system call
```c
//flags = flags | SIGCHLD 
//flags = flags & ~clone_flags

// set child_stack = current parent stack, (so use the Copy On Write mechanism)

```

vfork()
```c
//flags = SIGCHLD | CLONE_VM | CLONE_VFORK
//child_stack = parent stack 
```

kernel_clone() handles the clone(), fork(), and vfork() system calls.
调用 copy_process 返回新分配的子进程（子线程）
```c
/*
 * This creates a new process as a copy of the old one,
 * but does not actually start it yet.
 *
 * It copies the registers, and all the appropriate
 * parts of the process environment (as per the clone
 * flags). The actual kick-off is left to the caller.
 */

kernel_clone();
        copy_process();
//      struct task_struct *p = copy_process();
                //check
                dup_task_struct(); //为新进程创建一个内核栈，thread_info and task_struct
                //设置子进程的状态为 TASK_UNINTERRUPTIBLE 保证不会被调度运行
                // 分配一个 PID


                //other check



```

# what is executable files?
An executable file is a regular file that describes how to initialize a new execution context.
# exec 系统调用如何实现？
The exec functions is the system call that allows a process to start executing a new program 
The sys_execve() service routine finds the corresponding file, checks the executable format, and modifies the execution context of the current process according to the information stored in it, then the process starts executing the code stored in the executable file.

It replaces the shell's arguments with new ones passed as parameters in the execve() system call and acquires a new shell environment. All pages inherited from the parent (and shared with the Copy On Write mechanism) are released so that the new computation starts with a fresh User Mode address space; even the privileges of the process could change. However, the new computation inherits from the previous one all open file descriptors that were not closed automatically.



when a process is created, it always inherits the credentials of its parent. However, these credentials can be modified later, either when the process starts executing a new program or when it issues suitable system calls.

# how does layout the new program segments and process memory regions?



# what does linux support executable formats?
In linux, an executable format is described by an object of type linux_binfmt, the type linux_binfmt defined as follow:

```c
struct linux_binfmt {
        struct list_head lh;
        struct module *module;
        int (*load_binary)(struct linux_binprm *);
        int (*load_shlib)(struct file *);
#ifdef CONFIG_COREDUMP
        int (*core_dump)(struct coredump_params *cprm);
        unsigned long min_coredump;     /* minimal dump size */
#endif
} __randomize_layout;

```

binfmt essentially provides three methods:
1. load_binary
2. load_shlib
3. core_dump

All linux_binfmt objects are included in a singly linked list. Elements can be inserted and removed in the list by invoking the register_binfmt() and unregister_binfmt() functions.


# How the linux support bash script?
The last element in the formats list is always an object decribing the executable format for interpreted scripts. This format defines only the load_binary method. 
The corresponding load_script() function checks whether the executable file starts with the !# pair of characters. If so, it interprets the rest of the first line as the pathname of another executable file and tries to execute it by passing the name of the script file as a parameter.

# How the linux support the Java program?
# what does sys_execve() do?
the kernel implement code of sys_execve() is following:
```c
SYSCALL_DEFINE3(execve,
                const char __user *, filename,
                const char __user *const __user *, argv,
                const char __user *const __user *, envp)
{
        return do_execve(getname(filename), argv, envp);
}

```
filename: the address of the executable file pathname
argv: the address of a NULL-terminated array of pointers to strings; each string represents a command-line argument.
envp: the address of a NULL-terminated array of pointers to strings; each string represents an environment variable in the NAME=value format.

```c

do_execve()
        do_execveat_common()
                bprm_execve()
                        sched_exec()
                        exec_binprm()
                                search_binary_handler()
                                        load_binary()
```

search_binary_handler()
the function scans the formats list and tries to apply the load_binary method of each element, passing to it the linux_binprm data structure. The scan of the formats list terminates as soon as a load_binary method succeeds in acknowledging the executable format of the file.



```c
load_elf_binary()
        /* Flush all traces of the currently running executable */
        begin_new_exec();
        setup_new_exec();
        setup_arg_pages();
        /* Now we do a little grungy work by mmapping the ELF image into 
         * the correct location in memory. 
         */


        /* Sets the values of the start_code, end_code, start_data, end_data, 
         * start_brk, brk, and start_stack fields of the process's memory descriptor  
         */


        /* Calling set_brk effectively mmaps the pages that we need
         * for the bss and break sections.  We must do this before
         * mapping in the interpreter, to make sure it doesn't wind
         * up getting placed where the bss needs to go.
         */
        set_brk();
        
        if (interpreter) load_elf_interp();

        
        /* stores in the binfmt field of the process decriptor the address of the 
         * linux_binfmt object of the executable format 
         */

        set_binfmt();
        create_elf_tables();
        finalize_exec();
        START_THREAD();

```
First of all, some simple consistency checks
Some simple consistency checks for the interpreter
Flush all traces of the currently running executable



# what is the system state when multiple process in progress?(多进程同时执行时的样子如何描述)



# what is the user space entry for new process return from kernel?
1. statically linked program
user program entry.
2. dynamically linked program
first run dynamic linker, then transfer control to user program entry.

Update 2022/6/25






