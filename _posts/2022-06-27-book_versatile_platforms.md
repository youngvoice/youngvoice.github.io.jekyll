
# 虚拟机需要解决哪些问题？
虚拟机用来解释执行汇编指令，
## 都有哪些汇编指令需要处理？？？




2022.4.10
2.1 Basic interpretation
decode-and-dispatch
complete architected state of a machine implementing the source ISA, including all architected registers and main memory.

2.2 threaded interpretation
2.3 predecoding and direct threaded interpretation
centralized dispatch table

2022.4.11
2.5 Binary translation
Figure 2.18 illustrates state mapping. Here, some of the target ISA registers point to memory image and register block of source ISA. In addition, some of the target registers are mapped directly to some of the source registers; that is, source values are maintained in target register. Other portions of the source state, such as the program counter and the stack pointer, may also be held in target registers. After mapping is performed, some target
registers should be left over for use by the emulator code. Further discussion of register state mapping is in Section 2.8.1.

2.6
2.6.1 The code-discovery problem
Figure 2.21 Causes of the Instruction discovery problem

2.6.2 The code-location problem
translated code is accessed with a target program counter(TPC), which is different from the architected source program counter(SPC).
the target code cannot jump to a source code location, Figure 2.22
2.6.3 incremental predecoding and translation
Figure 2.23 overview of dynamic translation system
emulation manager
code cache
map table: associate the SPC for a block of source code with the TPC for the corresponding block of translated code.
dynamic basic block / static basic block

__Tracking the source program code__
One way of doing this is to map the SPC to a register on the host platform, with the register being updated either at each translated instruction or at the end of each translated block (as in Figure 2.19). Another possibility is shown in Figure 2.27. Here the value of the next SPC is placed in a “stub” at the end of the translated block. When the translated block finishes, control is transferred back to the EM using a jump-and-link (JAL) instruction. The link register can then be used by the EM to access the SPC from the end of the translated code block (Cmelik and Keppel 1994).

2022.4.12
2.7 Control transfer optimizations
eliminating the need to go through the EM between every pair of translation blocks.
2.7.1 Translation chaining
these are situations where the destination of a branch or jump never changes.
2.7.2 software indirect jump prediction
the comparisons are ordered, with the most frequent SPC destination address being given first.
2.7.3 shadow stack
Figure 2.33
The IA-32 return address is loaded from the emulated IA-32 stack. This address is compared with the IA-32 return address saved on the shadow stack. If these match, then the shadow stack PowerPC return value can be used to jump to the return translation block.



2022.4.13

2.8 Instruction set issues
complete instruction set

2.8.1 Register Architectures
If the number of target registers is significantly larger than the number of source registers.
although the condition code values are set frequently, they are seldom used. 
2.8.2 Condition codes
lazy evaluation: the operands and operation that set the condition code, rather than the condition code setting themselves, are saved.
During binary translation, the translator can analyze sequences of instructions to determine whether implicitly set condition codes will actually be used.

2.8.3 Data Formats and Arithmetic
addressing modes
2.8.4 Memory Address Resolution
bytes, halfwords, and full words
2.8.5 Memory data alignment
2.8.6 Byte order
order bytes within a word
How Byte order influence the char string store?
system call
2.8.7 Addressing Architecture
address space

2022.04.15


# Chapter Three Process virtual machines
## some key problem

### 1. Instruction emulation
### 2. Exception emulation
### 3. Operating system emulation







# how can we run programs that have been compiled for the operating system and processor's instruction set that different from the user's on the user's platform?

provide a virtual environment at the program, or process, level.

computer programs are compiled, distributed, and stored as executable binaries that comform to a specific application binary interface(ABI)(includes features of both the hardware instruction set and the operating system)

IA-32 EL(execution layer)
Figure 3.1
two perspective: 
1. guest process 
2. host system

**the guest process run above ABI, so the VM need to provide support the guest ABI.**

major aspects of a process VM
mapping of guest state to host state
emulation of the memory-addressing architecture, 
instructions, 
exceptions, 
operating system calls.

code cache management techniques

3.1 Virtual Machine Implement
Figure3.2

OS call emulator and Exception Emulation (ref a thesis???)

3.2 Compatibility
Figure 3.3
Figure 3.4
control transfer(user --> os , os ---> user)
user managed state
os managed state
3.3 State mapping
resource mapping 
registers and memory
3.3.2 Memory address space mapping
runtime managed translation table


3.5 Instruction Emulation
the integration of instruction emulation into an overall VM.
For optimal performance, the emulaiton engine typically combines multiple emulation methods.
Figure 3.17

signal对user application的意义是什么？
应用程序和操作系统的交互：syscall signal


3.9 System Environment
we consider the integration of guest processes into the host system environment.

为了理解我们需要emulation的接口，我们重新看 ISA Bridging with callback(9513977)
