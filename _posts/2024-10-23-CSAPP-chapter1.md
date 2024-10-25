---
title: ICS CSAPP第一章笔记
date: 2024-10-23 12:52:49 +0800
tags: [ICS, CSAPP]
categories: [ICS]
math: true
---

#### From Source File to Executable Object File

* *Preprocessing phase* in which the preprocessor(预处理器,`cpp`) modifies the original C program according to directives(命令) that begin with the '#' character(`#include <stdio.h>`,etc.).The result is typically with the `.i` suffix.
* *Compilation phase* in which the compiler(编译器,`cc1`)translates the text file into another text file with the `.s` suffix,which contains an *assembly-language program*(汇编语言程序).Assembly language is a common output language for different compilers for different high-level languages.
* *Assembly phase* in which the assembler(汇编器,`as`)translates the `.s` file into machine-language instructions,packages them in *relocatable object program*(可重定位目标程序) form,and stores it in the object file with the `.o` suffix.This file is a binary file.
* *Linking phase* in which the linker(链接器,`ld`) merges other object file if needed,`printf.o` for example.The resule is an executable object program(or simply executable). 


#### Hardware organization of a system

* Buses(总线) is a collection of exectrical conduits(管道) that transfers fixed-size chunks of bytes known as *words*(字) between the components.
  * The number of bytes in a word(字中的字节数) is called the *word size*(字长),which is a fundamental system parameter that varies across systems.4 bytes(32 bits) or 8 bytes(64 bits) is common nowadays.
* I/O devices
  * Disks are also I/O devices.
  * I/O devices are connected to the I/O bus by either a *controller*(控制器) or an *adapter*(适配器).
* Main memory(主存)
  * temporarily holds both a program and the data it manipulates *while* the processor is executing the program.
  * Logically,memory is organized as a linear array of bytes.
* The central processing unit(CPU)
  * Core:The *program counter*(PC), which is a word-size(一个字大小) storage device/*register*(寄存器).It points at some machine-language instruction in main memory.The processor repeatedly executes the instruction pointed at by it,then updates it to point to the next instruction.
  * We can regard the operation of a processor as a simplified model,defined by its *instruction set architecture*(指令集架构,如x86,ARM).Different processors will implement the same instruction set architecture differently,or have different *microarchitecture*s(微架构).
  * Component:The *register file* is a small storage device that consists of a collection of word-size registers,each with its own unique name.

#### Running the hello Program

* When we type `./hello`,the shell program reads each one into a register and stores it in main memory.
* When we hit the `enter` key,the shell loads(including copying the code and data) the executable `hello` file from the disk to main memory using a technique known as *direct memory access*(直接存储器存取,DMA),which makes this process without passing through the processor.
* Once the code and data in the `hello` object file are loaded into memory,the processor begins executing the machine-languages instructions in the `hello` program's `main` routine.
* To display the `hello, world\n` string,these instructions copy the bytes from memory to the register file,and from there to the display device.

#### The Necessity of Cache Memories(高速缓存)

Cache memories serves as temporary staging areas for information that the processor is likely to need in the near future.Since loading files from main memory takes much more time than from register,and the copy operations are which actually slows down the "real work" of the program,cache memories can accumulate infomations and exploit *locality* to hasten operations.

#### Storage Devices Form a Hierarchy

The smaller,faster storage devices are on higher levels and the larger,slower devices are on lower levels.The main idea of a memory hierarchy is that storage at one level serves as a cache for storage at the next lower level.

#### The Operating System

The operating system manages the hardware via some fundamental abstractions of the hardware devices.

##### Processes(进程)

A process is the operating system's abstraction for a running program.

Multiple processes appear to be running concurrently on the same system.Yet in fact,for a single CPU,it appears to execute multiple processes by having the processor switch among them,known as *context switching*(上下文切换).

The operating system holds *context*(上下文),including informations that the process needs in order to run,such as the current values of the PC,the register file and the contents of the main memory.When the operating system performs a *context switch*,it saves the context of the current process,restores the context of the new process,and passes control to the new process.This series of operations is activated by invoking a special function known as a *system call*(系统调用).

> The **Kernal**(内核) is the portion of the operating system code that is always resident in memory.One of its functions is managing the transition from one process to another.The kernal is a collection of code and data structures that the system uses to manage all the processes.

##### Threads(线程)

Threads are the units of execution that make up a process.They run in the context of the process,sharing the same code and global data.

* It's easier to share data between multiple threads then between multiple processes.
* Threads are typically more efficient than processes.

##### Virtual Memory(虚拟内存)

Virtual memory provides each process with the illusion that it has exclusive use of the main memory.The same uniform view of memory each process holds is known as its *virtual address space*(虚拟地址空间).

The virtual address space consists of a number of well defined areas,each with a specific purpose.

* *Program code and data*,initialized directly from the contents of an executable object file.We have mentioned this operation before.
* *Heap*.The memory area of a process.The heap expands and contracts dynamically at run time as a result of calls to C standard library routines such as `malloc` and `free`.
* *Shared libraries*,such as the C standard library and the math library.
* *Stack* that the compiler uses to implement function calls.It grows when we call a function and contracts when we return from a function.
* *Kernal virtual memory*(内核虚拟内存) whose contents are not allowed to be read or written by application programs.Instead,they must invoke the kernal.

Physically,the basic idea to implement virtual memory is to store the contents of a process's virtual memory on disk,then use the main memory as a cache for the disk.

##### Files(文件)

A file is just a sequence of bytes.Every I/O device is modeled as a file.

#### Communicate with Other Systems Using Networks

The network can be viewed as just another I/O device.

![image-20241023201401715](https://github.com/yangxu286/picx-images-hosting/raw/master/myblog/微信图片_20241023185002.4xujfp7kfw.webp)

#### Amdahl's Law

A formula that describes the acceleration effect of a component on the entire system.


$$
S = \frac{1}{(1 - \alpha) + \alpha/k}
$$



$$ \alpha $$ is the fraction of time the component of the system requires.

$$ k $$ is the factor we improved its performance by.

$$ S $$ is the speedup $$ T_{old}/T_{new} $$.

Insight:to significantly speed up the entire system,we must improve the speed of a very large fraction of the overall system.

#### Concurrency and Parallelism

Concurrency(并行):The word makes a general reference to a system with multiple,simultaneous activities.

Parallelism(并发):It refers to the use of concurrency to make a system run faster.

##### Thread-Level Concurrency

* *Multi-core* processors have several CPUs("cores") integrated onto a single integrated-circuit chip(集成电路芯片).
* *Hyperthreading* involves having multiple copies of some of the CPU hardware,such as PC and register files,while having only single copies of the other parts,arithmetic parts for example.
  * Decides which of its threads to execute on a cycle-by-cycle basis

##### Instruction-Level Parallelism

Modern processors can execute multiple instructions at one time.

*Superscalar*(超标量) processors are processors that can sustain execution rates faster than 1 instruction per cycle.

##### Single-Instruction,Multiple-Data(SIMD) Parallelism

Many modern processors have special hardware that allows a single instruction to cause multiple operations to be performed in parallel,a mode known as SIMD.SIMD instructions are provided mostly to speed up image,sound and video data processing.

#### Abstraction

![image-20241023201416820](https://github.com/yangxu286/picx-images-hosting/raw/master/myblog/微信图片_20241023200459.9kg6ge643h.webp)

#### 其他概念/生词

* compiler driver(编译器驱动程序)
* uniprocessor system(单处理器系统)
* multiprocessor system(多处理器系统)
* gibberish(乱码)
* prompt(提示符)