---
title: Windows API - Basics of Virtual Memory Management API.
author: Mr. Rc
date: 2021-04-15 11:33:00 +0800
categories: [Programming, Windows, Windows Internals, WinAPI]
tags: [Windows, Windows API Series, Virtual Memory Management]
math: false
mermaid: false
---

The Virtual Memory Manager in Windows (WMM) is available for user mode programs and can be used by the Win32 APIs to allocate memory and free them in the user space. 
From this blog, I am going to start a series on Windows API where we will explore some windows API functions and play with them. In this particular blog I have covered two functions from the Windows Memory Management API which are `VirtualAlloc` and `VirtualFree` in deep. 

# 1. VirtualAlloc
The `VirtualAlloc` function is used to allocate large private memory blocks and manage them in the user mode. The term Private memory blocks means that the memory region allocated by this function is private to the process that created it. By default, when we locate a memory block by using this function, it is initialized as zero.

Let's take a deeper look into this function and create a C program to use it.

This is the function syntax according to the Microsoft documentation.

```C
LPVOID VirtualAlloc(
  LPVOID lpAddress,
  SIZE_T dwSize,
  DWORD  flAllocationType,
  DWORD  flProtect
);
```

If you look closely, you'll see that the function type is `LPVOID`, `LPVOID` is a pointer to a void object. `LPVOID` is just `typedef void* LPVOID` in the Windef.h. In simple words, `LPVOID` is an alias for `void *`. `LP` in `LPVOID` stands for long pointer.

The first argument required for the function is `lpAddress`. It is used to specify the starting address of the memory region to allocate. If you don't know where to allocate memory, you can simply specify `NULL` and the system will decide where to allocate the memory. If the address that you specified is from a memory region that you don't have access to or if it's an invalid address to allocate the function will fail with ERROR_INVALID_ADDRESS error.

The second argument is `dwSize`. It is used to specify the size of the memory region that you want to allocate in *bytes*. If the `lpAddress` argument was specified as `NULL` then this value will be rounded up to the next page boundary.

The third argument is `fAllocationType`. It is used to specify which type of memory allocation we need. Here are valid types as defined in the Microsoft documentation for `fAllocationType`: [Read here](https://www.notion.so/a8e846fdb9a346d38a6fdc116bbe61b4?v=4dd087bea5cd4bd0bda480e1df654438)

If you are confused about the hex digits which are written after every value, it is the actual value that will be translated to if you will use the value. If you use `MEM_COMMIT` it will be replaced with `0x00001000` and same with all other values.

There are some other arguments also that can be used with this which are not so useful to us and not commonly used, if you want to read about them here:

[VirtualAlloc function (memoryapi.h) - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc#arguments)

Our last argument is `flProtect`, this is used to specify the memory protection for the memory region that we are allocating.
This are the values that we can specify in `flProtect`:

- PAGE_NOACCESS
- PAGE_GUARD
- PAGE_NOCACHE
- PAGE_WRITECOMBINE

This is how we can use this values:
Note: Pages and memory regions words are used both referring to the memory region that is being allocated. As we're allocating Virtual memory, the memory regions will be created with their pages so I think they can be used interchangeably.

[Some memory protection constants that can be used](https://www.notion.so/6e58799af3f941d681bd31a299aa8957)

This are just a small number of examples of constants, you can read about many more here : 

[Memory Protection Constants (WinNT.h) - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/memory/memory-protection-constants)

## Return value

If the function succeeds, it will return the starting address of the memory region that we specified or left `NULL`. If the function fails, it will return `NULL`.

# 2. VirtualFree

As we have learnt to Allocate memory, we will now see how to free it. It's pretty simple but necessary to learn.

This is the syntax for `VirtualMemory` according to the Microsoft documentation:

```jsx
BOOL VirtualFree(
  LPVOID lpAddress,
  SIZE_T dwSize,
  DWORD  dwFreeType
);
```

As you can see, the type of this function is `BOOL`, it means that it will either return true or false. It depends on the result of the operation that we are doing.

The first argument to this function is `lpAddress`. We already know that this argument is used to pass the base address of the memory region with which we want to play. If you're confused, don't worry we will see the code examples and it will eventually clear your doubts. 

The second argument is `dwSize`. We also know about this argument, it is used to pass the size in *bytes* of the memory region which we want to play with. Here, we will use it specify the size of the memory region that we want to free.

The last argument is `dwFreeType`. It is used to specify the type which we want to use to free the memory. You may be little bit confused but you will not be once you see the valid free types.

[Valid dwFreeTypes](https://www.notion.so/771971f5e05341babefcb2750aee6207)

## Return value

If the function does its job successfully, it returns nonzero.

If the function fails, it will return a zero (0).

# Code examples

As we have looked into all the things, now it's time to write some code and clear all the doubts.

## Example 1 - VirtualAlloc

Let's start with taking example of `VirtualAlloc`. We will write some code which will commit 8 bytes of virtual memory.
Let's first include the `memoryapi.h` file, this file contains almost all the utilities that we need in the user mode to use Virtual Memory related functions:

```c
#include <stdio.h>
#include <memoryapi.h>
```

Now let's define a main function that will use the `VirtualAlloc` function and commit 8 bytes of Virtual Memory. We will specify the `lpAddress` argument as `NULL` so that the system will determine from where to allocate the memory. Here is how the code looks like:

```c
#include <stdio.h>
#include <memoryapi.h>

int main(){
	int *pointer_to_memory = VirtualAlloc(NULL, 8, MEM_COMMIT, PAGE_NOCACHE);
	printf("%x", pointer_to_memory);
}
```

If you wonder why I used `PAGE_NOCACHE`, if you remember the valid types for the argument `flProtect`, `PAGE_NOCACHE` is one of them. The other reason why I used it is that it is the only one argument that we can use normally, by normally I mean that if you look at all other valid types they are intended to be used in complex and real world coding scenarios but now we are just understanding how it works so it's good to use a normal type as the first example.

The explanation of code is simple, first we included the required files that had all the functions of our interest then we used the `VirtualAlloc` and assigned the memory address given by it to the `pointer_to_memory` variable then we printed it.

Do you know something is missing in the code?
Think about it....
I hope you found it, it's the `VirtualFree` function. Whenever we allocate memory which is not from the stack, we have to free it.

Now it's time to implement the `VirtualFree` function, so here it is:

```c
#include <stdio.h>
#include <memoryapi.h>

int main(){
    int *pointer_to_memory = VirtualAlloc(NULL, 8, MEM_COMMIT, PAGE_READWRITE);
    printf("The base address of allocated memory is: %x", pointer_to_memory);
    VirtualFree(pointer_to_memory, 8, MEM_DECOMMIT);
}
```

You shouldn't have doubts but if you have, let me clear all of them. Let's understand the `main` function again.
First I created a integer type pointer variable which is pointing to the memory address returned by `VirtualAlloc`. We have passed four parameters to the `VirtualAlloc` function. 

The first parameter is `NULL`, by passing `NULL` as a argument we are telling the function that the starting point of the memory region should be decided by the system.

The second parameter is the size of the memory region that we want to allocate in bytes.

The third parameter is the allocation type, we are specifying that we want to commit the memory. After we commit a memory region, it is available to us for our use.

The last parameter is `PAGE_READWRITE`, it means that we want the permission to read and write on this memory region. 

The we are printing memory address return by `VirtualAlloc` function as a hex value.

At last, we are decommitting the memory region that we allocated. 

The first parameter is the base address of the memory region that we allocated.

The second parameter is the size of memory region in bytes, we specified `8` while allocating it so the we'll specify `8` while deallocating it.

Then we have specified the type of deallocation that we want. As we are using `MEM_DECOMMIT`, the memory region will be reserved after it gets decommitted, which means that any other function will not be able to use it after you decommit it until you use `VirtualFree` function again to release the memory region.

## Running the code example #1

As we almost done everything, let's compile and run the code. I suggest you to write the code by yourself and see the result. This is the result when I ran it:

```markdown
The base address of allocated memory is: `61fe18`
```

Cool, right?
We have just used the `VirtualAlloc` function to allocate 8 bytes of memory and we freed it by ourselves. Now let's add a string in the allocated memory and print it. 

## Example 2 - VirtualAlloc

Now let's assign some data to the memory that we allocated:

```c
#include <stdio.h>
#include <memoryapi.h>

int main(){
    int *pointer_to_memory = VirtualAlloc(NULL, 8, MEM_COMMIT, PAGE_READWRITE);
    printf("The base address of allocated memory is: 0x%d\n", pointer_to_memory);
    memmove(pointer_to_memory, (const void*)"1337", 4);
    printf("The data which is stored in the memory is %s", pointer_to_memory);
    VirtualFree(pointer_to_memory, 8, MEM_DECOMMIT);
}
```

The `memmove` function is used to copy data from one destination to other. The first argument to this function is the destination memory address where you want to copy the data and the second argument is the data that will be copied and the last and third argument is the size of data. Here, we have copied 1337 to the memory location which was returned by `VirtualAlloc`. If you're confused about the type conversion, it's used because `memmove` takes second argument as a `const void` pointer and we can't directly pass it a `char` array.

## Running the code example #2

Let's compile and run the code. This is the output that we'll get:

```c
The base address of allocated memory is: 61fe18
The data which is stored in the memory is 1337
```

More cool, Right?

This was all for this one, meet you in the next one!
