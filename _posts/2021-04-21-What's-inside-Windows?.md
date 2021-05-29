---
title: What's inside Windows? 
author: Mr. Rc
date: 2021-04-12 00:00:00 +0800
categories: [Windows Internals, Windows]
tags: [Windows]
math: false
mermaid: false
---


# Windows Fundamentals

This page contains the basic definitions of some windows internal things. You can google the name of any concept if you don't understand what is it.

# 1. Virtual memory

Not only in windows but in every operating system, Virtual memory / Virtual address space is a core and important concept of operating system fundamentals. In this concept, instead of giving applications directly access to the physical memory addresses, the processor and operating system creates a layer between application and physical memory. Each application(process) has their own virtual memory, ranging from 0 to 4,294,967,295 (2*32-1 = 4 GBs), regardless of how much RAM is installed on the computer(in x86 systems). In 64-bit Windows, the theoretical amount of virtual address space is 2^64 bytes (16 exabytes), but only a small portion of the 16-exabyte range is actually used. With default windows(x86), you get 2 GBs of virtual space that are designed for the private use of a process. The other 2GB is a shared memory between all other processes and the operating system.
There are many concepts in this single concept like paging and some others, if you want to know about them, I'll refer you to this page:

---

[Virtual Memory in Operating System - GeeksforGeeks](https://www.geeksforgeeks.org/virtual-memory-in-operating-system/)

# 2. Paging

In paging, when a memory region is not in use, it is moved to the hard drive. It's because the RAM is much faster and more expensive than the hard drive space, it's useful to use a file as a backup of memory areas. When you have many applications open in your system and some of them are not in use, rather than keeping the whole application in the RAM, the virtual memory architecture allows the system to save all this memory region to a file in the hard drive and load the memory back to RAM when needed.
To know and understand more about paging, I refer you to this YouTube video:

[Paging in Operating Systems with Example & Working - Memory Management](https://www.youtube.com/watch?v=pJ6qrCB8pDw)

# 3. Kernel mode and user mode

This is also one of the most important concepts in operating systems. All the applications that we run on our systems are run in the user mode or user memory, each application / process has their own private virtual address table. As this virtual address table is private, any other application or process cannot change the data that belongs to this process. It is said that if you want to create a powerful and secure operating system, applications in your operating system shouldn't have access to the operating system's internal data structures. Kernel mode has more privileges than the user mode. This is implemented to protect the operating system's internals from being overwritten by any program or by any malware. The operating system's internals include the drivers and other core operating system components. These all OS's internals are run on the kernel mode.

All the code that runs on kernel mode uses a single virtual address space. This means that there is no private/isolated address space for every process. A driver running on kernel mode can alter the data of the other kernel-mode drivers. This also means that if a driver accidently writes something to the wrong virtual address, data belongs to that operating system's component or kernel driver can be changed. This can lead to a crash in a kernel-mode driver; when a kernel-mode driver crashes, the whole operating system crashes.

---

References:

[Syscalls, Kernel vs. User Mode and Linux Kernel Source Code - bin 0x09](https://www.youtube.com/watch?v=fLS99zJDHOc)

[User mode and kernel mode - Windows drivers](https://docs.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode)

# 4. Kernel stack

Like we have a stack on the user mode, the kernel mode also has the stack of its own, and obviously it is not directly accessible by the user mode processes. When a user process calls a syscall, the kernel stack of the current running process is used.

---

References:

[Kernel Stack and User Space Stack | Baeldung on Linux](https://www.baeldung.com/linux/kernel-stack-and-user-space-stack)

# 5. System Page-Table Entries (PTE)

System Page-Table space is like a heap for the operating system, but rather than just being a virtual memory space, this can be used by the kernel-mode components whenever they need some large chunk of memory for any task. The kernel uses PTE to manage and locate the kernel mode components such as driver executables, kernel stack etc. To allocate memory region on the System PTE, drivers can use `MmAllocateMappingAddress` from the kernel API.

# 6. Section objects

Section objects are the sections of memory that can be shared. A process can use Section objects if it wants to share its memory with any other process. Before anything from a Section object can be accessed, it must be mapped. After we map a Section object, it becomes accessible in that address range. This can also be used to share memory sections between kernel and user-mode processes. This is a kernel concept and is originally called `memory mapping`.

### There are two types of Section objects.

1. Pagefile-backed.
2. File-backed.

### 1. Pagefile-backed

A pagefile-backed Section object can be used for temporary storage and is commonly used to share data between processes and the kernel. A pagefile-backed section can be saved in a pagefile if needed.

### 2. File-backed

A file-backed Section object is connected to a physical file on the hard drive. When first mapped, it contains the contents of the file that it is connected to. If the memory is writeable, then any changes that will be made with the memory address will reflect in the actual file.

# 7. VAD trees

A Virtual Address Descriptor (VAD) tree is a data structure which is used by windows to manage every process's address allocation. The VAD tree is a binary tree that contains the information about all the address ranges that are currently being used. Every running process has its own tree.
Normally, there are two different types of allocations in kernel space:

1. Mapped allocation.
2. Private allocation.

### 1. Mapped allocation

As we are reading about the kernel space, we already know that every process running on the kernel memory shares the same virtual address space, and there is no default protection of memory being read by any other process. In the simplest terms, it is a type of memory allocation in the kernel space where the memory is shared with other processes. 

### 2. Private allocation

This method of allocation is the exact opposite of the mapped allocation. Any memory allocation that is done with the allocation method remains private and can only be accessed by the process which created it. Stacks, heaps are an example of private allocated memory.

All the things that we read were not specific but more related to the kernel space. Now let's read about how application use memory in the user space.

### 1. Private allocation

The most basic type of allocation in the user space is private memory allocation. Applications can use `VirtualAlloc` function from the Win32 API to request a memory region. These allocations are commonly used for the allocation of stacks and heaps.

### 2. Heaps

If you have ever made some programs in C and have some experience with it, you would probably have used the `malloc` function, which is used to allocate heap at run time. Programmers do not commonly use the `VirtualAlloc` function to allocate memory on the heap, rather they usually use `malloc` or `HeapAlloc`. If you don't know about `malloc`, it's a library function from the glibc library which is used to allocate heap and `HeapAlloc` is a function from Win32 API. `VirtualAlloc` is used for allocating private blocks of heap, `HeapAlloc` can be used to allocate heaps that are shared between the OS internals.

### 3. Stacks

Stacks on user-mode are generally normal Private allocations of memory. As a thread is created, the system automatically creates a stack for that thread.

### 4. Executables

How do you think executables run?
They are first loaded into memory along with any modules/libraries if required. Then it's run from the memory where it was loaded.
