---
title: what mechanism exists back in file system? 
description: the story of process read file content
categories:
 - question
tags: [file system]
---
如果说syscall是操作系统的横向大门的话，我就称文件系统为实践入手操作系统的纵向大门。
# 缘从何起之进程如何读取文件内容？
```c
/*
open
write/read
close
*/

char buf[10];
fd = open("/home/xjk/a.c", O_RDWR);
int size = sizeof(buf);
n = read(fd, buf, size);

char *str = "oh it's you";
write(fd, str, strlen(str));

close(fd);

```
# what the open does? 
Once the process call open system call, the open return file descriptor. The file descriptor returned by a successful call will be the lowest-numbered file descriptor not currently open for the process. The file offset is set to the beginning of the file.

A call to open() creates a new open file description, an entry in the system-wide table of open files. The open file description records the file offset and the file status flags. A file descriptor is a reference to an open file description;  

The argument flags must include one of the following access modes: O_RDONLY, O_WRONLY, or O_RDWR.  These request  opening  the  file  read-only, write-only, or read/write, respectively.

The  distinction  between these two groups of flags is that the file creation flags affect the semantics of the open operation itself, while the file status flags affect the semantics of subsequent I/O operations.  The file status flags can be retrieved and (in some cases)  modified;(ref fcntl)


*Open file descriptions*
The term open file description is the one used by POSIX to refer to the entries in the system-wide table of open files.  In other contexts, this object  is  variously  also  called an "open file object", a "file handle", an "open file table entry", or—in kernel-developer parlance—a struct file.

When a file descriptor is duplicated (using dup(2) or similar), the duplicate refers to the same open file description as the original file  descriptor, and the two file descriptors consequently share the file offset and file status flags.  Such sharing can also occur between processes: a child process created via fork(2) inherits duplicates of its parent's file descriptors, and those duplicates refer to the same open  file  descriptions.


Each open() of a file creates a new open file description; thus, there may be multiple open file descriptions corresponding to a file inode.


```c
SYSCALL_DEFINE3(open, const char __user *, filename, int, flags, umode_t, mode)
{
        if (force_o_largefile())
                flags |= O_LARGEFILE;
        return do_sys_open(AT_FDCWD, filename, flags, mode);
}



```

```c
struct task_struct {
        /* Filesystem information: */
        struct fs_struct *fs;

        /* Open file information: */
        struct files_struct *files;

}
```
# what the write does?

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






# how to support multiple file system type?
VFS



Update 2022/06/26

