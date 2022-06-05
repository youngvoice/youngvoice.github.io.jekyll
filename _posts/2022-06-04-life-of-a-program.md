---
title: what story a program has? 
description: the story of a whole life of program
categories:
 - question
tags: [program]
---
# what is a user process
1. place user program (text + data) into address space to form an image
2. prepare runtime environment (context)
3. then to execute it.

# what happend, when we run a program in linux system ?
1. the shell call exec system call enter kernel space
2. the kernel create the process image and prepare runtime environment 
3. then switch to the new process and execute it
4. return from kernel space and run user code


# what is the user space entry for new process return from kernel?
1. statically linked program
user program entry.
2. dynamically linked program
first run dynamic linker, then transfer control to user program entry.
# what is the execute follow of user program in user space? 
we can gdb debug the execute process
we can try to compiler the program


# what is the composition of a user program?
