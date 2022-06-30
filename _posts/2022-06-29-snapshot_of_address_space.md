---
title: argc, argv, envp, auxiliary vector 
description: initial process stack
categories:
 - csu
tags: [csu]
---
Operating System Interface
Process Initialization

This section describes the machine state that exec(BA_OS) creates for ‘‘infant’’ processes, including argument passing, register usage, stack frame layout, and so on. 

Programming language systems use this initial program state to establish a standard environment for their application programs

# how to implement the call to main?（focus on argc, argv, envp, auxiliary vector）
it gives the information necessary to implement the call to main or to the entry point for a program in any other language.

Special Registers
several state registers



##### Process Stack and Registers
# what looks like in initial process stack?
Figure 3-31: Initial Process Stack
When a process receives control, its stack holds the arguments and environment from exec

Whereas the argument and environment vectors transmit information from one application program to another, the auxiliary vector conveys information from the operating system to the program.


Figure 3-35: Example Process Stack



# how to combine many subroutine defined in x86 assembly language into a single program?
How are parameters passed to a subroutine? 
Can subroutines overwrite the values in a register, or does the caller expect the register contents to be preserved? 
Where should local variables in a subroutine be stored? 
How should results be returned from functions?
# how to call a C function through assembly language?
# how C or C++ code call assembly language subroutines?




the calling convension is specified in ABI document 

two sets of rules:
the first set of rules is employed by the caller of the subroutine, and the second set of rules is observed by the writer of the subroutine (the "callee").

the callee’s rules fall cleanly into two halves that are basically mirror images of one another. The first half of the rules apply to the beginning of the function, and are therefor commonly said to define the prologue to the function. The latter half of the rules apply to the end of the function, and are thus commonly said to define the epilogue of the function.

将如下的 C 语言代码在 X86_64 linux 上编译后进行反汇编
```c
long callee(long arg1, long arg2, long arg3, long arg4, long arg5, long arg6, long arg7, long arg8)
{
        long ret = 0xffffffff;
        ret = arg1 + arg2 + arg3 + arg4 + arg5 + arg6 + arg7 + arg8;

        return ret;

}


int main()
{
        long ret = 0;
        long arg1 = 0xf1, arg2 = 0xf2, arg3 = 0xf3, arg4 = 0xf4, arg5 = 0xf5, arg6 = 0xf6, arg7 = 0xf7, arg8 = 0xf8;
        ret = callee(arg1, arg2, arg3, arg4, arg5, arg6, arg7, arg8);
        return  ret;
}

```
objdump 后的反汇编如下
```s
0000000000001191 <main>:
    1191:       f3 0f 1e fa             endbr64
    1195:       55                      push   %rbp
    1196:       48 89 e5                mov    %rsp,%rbp
    1199:       48 83 ec 50             sub    $0x50,%rsp
    119d:       48 c7 45 b8 00 00 00    movq   $0x0,-0x48(%rbp)
    11a4:       00
    11a5:       48 c7 45 c0 f1 00 00    movq   $0xf1,-0x40(%rbp)
    11ac:       00
    11ad:       48 c7 45 c8 f2 00 00    movq   $0xf2,-0x38(%rbp)
    11b4:       00
    11b5:       48 c7 45 d0 f3 00 00    movq   $0xf3,-0x30(%rbp)
    11bc:       00
    11bd:       48 c7 45 d8 f4 00 00    movq   $0xf4,-0x28(%rbp)
    11c4:       00
    11c5:       48 c7 45 e0 f5 00 00    movq   $0xf5,-0x20(%rbp)
    11cc:       00
    11cd:       48 c7 45 e8 f6 00 00    movq   $0xf6,-0x18(%rbp)
    11d4:       00
    11d5:       48 c7 45 f0 f7 00 00    movq   $0xf7,-0x10(%rbp)
    11dc:       00
    11dd:       48 c7 45 f8 f8 00 00    movq   $0xf8,-0x8(%rbp)
    11e4:       00
    11e5:       4c 8b 45 e8             mov    -0x18(%rbp),%r8
    11e9:       48 8b 7d e0             mov    -0x20(%rbp),%rdi
    11ed:       48 8b 4d d8             mov    -0x28(%rbp),%rcx
    11f1:       48 8b 55 d0             mov    -0x30(%rbp),%rdx
    11f5:       48 8b 75 c8             mov    -0x38(%rbp),%rsi
    11f9:       48 8b 45 c0             mov    -0x40(%rbp),%rax
    11fd:       ff 75 f8                pushq  -0x8(%rbp)
    1200:       ff 75 f0                pushq  -0x10(%rbp)
    1203:       4d 89 c1                mov    %r8,%r9
    1206:       49 89 f8                mov    %rdi,%r8
    1209:       48 89 c7                mov    %rax,%rdi
    120c:       e8 18 ff ff ff          callq  1129 <callee>
    1211:       48 83 c4 10             add    $0x10,%rsp
    1215:       48 89 45 b8             mov    %rax,-0x48(%rbp)
    1219:       48 8b 45 b8             mov    -0x48(%rbp),%rax
    121d:       c9                      leaveq
    121e:       c3                      retq
    121f:       90                      nop
    

```

仔细阅读汇编代码可以发现，main 函数在调用callee时，将前面6个参数分别通过 rdi, rsi, rdx, rcx, r8, r9 寄存器进行传递，剩余的两个参数通过堆栈传递，最终在调用callee 时的堆栈分布如下：


| address offset (rbp)| register / variable name| local variable |
|---------------------|---------|----|
| 0| rbp |
| -8  | arg8 |yes|
| -10 | arg7 |yes|
| -18 | r9/arg6 |yes|
| -20 | r8/arg5 |yes|
| -28 | rcx/arg4 |yes|
| -30 | rdx/arg3 |yes|
| -38 | rsi/arg2 |yes|
| -40 | rdi/arg1 |yes|
| -48 | ret  |yes|
| -50 | | |
| | arg8 | |
| | arg7 | |

##### 为什么把堆栈分配到了 50？是不是因为 rsp 对准？该堆栈是满堆栈吗？满堆栈除了pop/push 内在语义不一样之外，还能怎么判断？


glibc start code
```s
//  "./sysdeps/x86_64/start.S"
// Startup code compliant to the ELF x86-64 ABI


            /* Clear the frame pointer.  The ABI suggests this be done, to mark the outermost frame obviously.  */
                /* Extract the arguments as encoded on the stack and set up
           the arguments for __libc_start_main (int (*main) (int, char **, char **),
                   int argc, char *argv,
                   void (*init) (void), void (*fini) (void),
                   void (*rtld_fini) (void), void *stack_end).
           The arguments are passed via registers and on the stack:
        main:           %rdi
        argc:           %rsi
        argv:           %rdx
        init:           %rcx
        fini:           %r8
        rtld_fini:      %r9
        stack_end:      stack.  */
0000000000401bc0 <_start>:
  401bc0:       f3 0f 1e fa             endbr64
  401bc4:       31 ed                   xor    %ebp,%ebp
  401bc6:       49 89 d1                mov    %rdx,%r9
  401bc9:       5e                      pop    %rsi
  401bca:       48 89 e2                mov    %rsp,%rdx
  401bcd:       48 83 e4 f0             and    $0xfffffffffffffff0,%rsp
  401bd1:       50                      push   %rax
  401bd2:       54                      push   %rsp
  401bd3:       49 c7 c0 80 2d 40 00    mov    $0x402d80,%r8
  401bda:       48 c7 c1 e0 2c 40 00    mov    $0x402ce0,%rcx
  401be1:       48 c7 c7 e5 1c 40 00    mov    $0x401ce5,%rdi
  401be8:       67 e8 92 04 00 00       addr32 callq 402080 <__libc_start_main>
  401bee:       f4                      hlt
  401bef:       90                      nop




```

# 从汇编的 application 调用 glibc 的 printf？
```s
# "assembly_call_printf.S"
.text
#extern printf
.global printf
.global _start
_start:
    mov $msg, %rdi
    callq printf
    mov $60, %rax
    mov $0, %rdi
    syscall

msg:
    .ascii "hello\n"
len = . - msg

```

```bash
# 编译方式1
as -o assembly_call_printf.o assembly_call_printf.S
# 编译方式2
gcc -c -g  -fno-builtin assembly_call_printf.S

# 下面的命令链接失败，
ld -static assembly_call_printf.o -lc -o assembly_call_printf.elf

# 下面的命令能链接通过，但是无法运行
ld  assembly_call_printf.o -lc -o assembly_call_printf.elf



# 下面的链接能通过，运行时segment fault
# segment 发生在 fs 寄存器访问时，如何修正？？？
/usr/lib/gcc/x86_64-linux-gnu/9/collect2 -plugin /usr/lib/gcc/x86_64-linux-gnu/9/liblto_plugin.so -plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/9/lto-wrapper -plugin-opt=-fresolution=/tmp/cckUJeVO.res -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_eh -plugin-opt=-pass-through=-lc --build-id -m elf_x86_64 --hash-style=gnu --as-needed -static -z relro /usr/lib/gcc/x86_64-linux-gnu/9/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/9/crtbeginT.o -L/usr/lib/gcc/x86_64-linux-gnu/9 -L/usr/lib/gcc/x86_64-linux-gnu/9/../../../x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/9/../../../../lib -L/lib/x86_64-linux-gnu -L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib -L/usr/lib/gcc/x86_64-linux-gnu/9/../../.. assembly_call_printf.o --start-group -lgcc -lgcc_eh -lc --end-group /usr/lib/gcc/x86_64-linux-gnu/9/crtend.o /usr/lib/gcc/x86_64-linux-gnu/9/../../../x86_64-linux-gnu/crtn.o
```

# how to resolve (printf) the (argc, argv, envp, auxiliary vector) on initial process stack?




Reference
http://www.mindfruit.co.uk/2012/01/initial-stack-reading-process-arguments.html
https://refspecs.linuxbase.org/elf/abi386-4.pdf
https://refspecs.linuxbase.org/elf/x86_64-abi-0.99.pdf
https://aaronbloomfield.github.io/pdr/book/x86-64bit-ccc-chapter.pdf