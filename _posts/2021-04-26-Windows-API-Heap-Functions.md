---
title: Windows API - Let's learn about heap in deep!
author: Mr. Rc
date: 2021-04-26 11:33:00 +0800
categories: [Programming, Windows, Windows Internals, WinAPI]
tags: [Windows, Windows API Series, Heap API]
math: false
mermaid: false
---

In this blog, We are continuing our series and going to study some Windows API functions related to Heap. We'll see how to use this functions and how they can be useful to us with code examples and explanation of each function.
Let's get started!

# 1. HeapCreate

`HeapCreate` function is used to create the heap at the runtime. After the heap is created, we can use other functions to do things on this allocated heap. We will see that later. For now, let's look at the syntax of this function. 

Here it is:

```c
HANDLE HeapCreate(
  DWORD  flOptions,
  SIZE_T dwInitialSize,
  SIZE_T dwMaximumSize
);
```

As you can see, the type of this function is `HANDLE`, `HANDLE` type is just a pointer to some memory in heap. So this means that this function will return a memory address from the heap and that's what we expect from after creating heap. 
Now, let's have a look at the arguments.

The first arguments is `flOption`. This argument is used to specify the heap allocation options.
This are the valid parameter for this argument:

[![options](https://i.ibb.co/HrdBm7R/options.png)](https://ibb.co/2jNWXqR)

If you don't want to use any of these options, you can specify 0 that will not use any of these options.

The second argument is `dwInitialSize`. This argument is used to specify the size of heap *in bytes*. This is the initial size of the created heap and this can be expanded more by specifying the next argument. If you specify 0 as the third argument, the function will allocate a whole page.

The third and last argument is `dwMaximumSize`. This is used to specify the maximum size of heap. If this value is too big, this values will be rounded to the maximum page size of the machine.

## Return Value

The function returns a handle to the created heap region when it succeeds.
The function returns a `NULL` if the function fails.

Now, we have created heap. The next thing to do is to Allocate the created heap. For creating heap, we have a function from the Windows API called `HeapAlloc`. Now, let's look at this function.

# 2. HeapAlloc

This function is used to allocate heap on the created heap. 
The function syntax looks like this:

```c
DECLSPEC_ALLOCATOR LPVOID HeapAlloc(
  HANDLE hHeap,
  DWORD  dwFlags,
  SIZE_T dwBytes
);
```

Forget about the `DECLSPEC_ALLOCATOR` type of the function. The actual type of the function is `LPVOID`, if you read my blogs you probably already know what this type means. `LPVOID` is just a pointer to a void object, so we can expect this function to return a pointer that will point to the start of the the heap that we are going to allocate. Now, let's look at the function arguments.

The first argument is `hHeap` which is a handle to heap that we allocated. The heap handle is just the  return value from `HeapAlloc` function. 

The second argument is `dwFlags`. This argument is almost same as the `flOptions` argument in the `HeapCreate` function. If you specify any option that is different from the option that you specified in the `HeapCreate` function, it'll be overwritten and the argument that you passed into `dwFlags` will be used. This are the valid parameters for `dwFlags`:

[![options](https://i.ibb.co/HrdBm7R/options.png)](https://ibb.co/2jNWXqR)

The third and last argument is `dwBytes`, This is the amount of bytes to be allocated. 

## Return Value

If the function succeeds, it will return a pointer to the memory block that is allocated on the heap.
If the function fails and you have not specified `HEAP_GENERATE_EXCEPTIONS` in the `dwFlag` argument, the function will return `NULL`. If you have specified `HEAP_GENERATE_EXCEPTIONS` in `dwFlags`, it will return an exception code that we can use to get more information about the error like we do with the function `GetLastError` but we will use the function `GetExceptionCode` instead.

Now let's look at the last important function which we will use to free this memory region that we have allocated.

# 3. HeapFree

As the name suggests, we will use this function to free heap that we allocated using `HeapAllocate`.

One thing to note is that when we free a heap block, it is just freed, it is not destroyed. It can be allocated again until we destroy that heap. We will see how to destroy heap in the next part of this series, for now let's focus on freeing the memory.

This is the function syntax:

```c
BOOL HeapFree(
  HANDLE                 hHeap,
  DWORD                  dwFlags,
  _Frees_ptr_opt_ LPVOID lpMem
);
```

The function type is `BOOL` so we can expect this function to return a Boolean value that will tell us whether the function was successful or not. Let's look at the arguments then we will move to code examples.

The first argument is `hHeap`. I guess you already know what we have to specify here, it's just the handle returned from `HeapCreate`.

The next argument is `dwFlags`. This is used to specify the options while freeing the heap. This is the valid parameters for `dwFlags` argument:

[![options](https://i.ibb.co/bv8sZ1q/options.png)](https://ibb.co/fNPk5MB)

Well yes, this argument has only one parameter, you can specify 0 if you don't want to use this parameter. 

The third and last parameter is `lpMem`. This should be the memory address returned by `HeapAlloc`.

Now as we have learnt about this functions, this is the time to write some code by using this functions. 

# Code Example - 1

Now let's make a program with all this functions, which will first create some heap and then we will allocate the heap then we will free it. You can try making this yourself. Here is the code that I made

```c
#include <stdio.h>
#include <Windows.h>

int main() {
    int* heap_handle = HeapCreate(0, 0, 16); // Created heap of maximum size 16 bytes

    if (heap_handle == NULL) {
        printf("Was not able to create heap!!\nError code: %d\n", GetLastError());
        exit(GetLastError());
    }

    printf("The handle of heap created is %x\n", heap_handle);
    int* heap = HeapAlloc(heap_handle, HEAP_ZERO_MEMORY, 8);

    printf("The base address of heap allocated is %x\n", heap_handle);

    int freed = HeapFree(heap_handle, 0, heap);

    if (freed) {
        printf("The memory has been freed successfully!\n");
    } else {
        printf("Was not able to free heap!!");
        exit(1);
    }

    return 0;
}
```

Let me explain the code. First we created heap using `HeapCreate` function, then we gave the first argument a 0, which means we don't want to use any parameter in place of the first argument then again we supplied 0 as the second argument which means it will allocate a whole page. Then we passed 16 as the third argument which means the maximum size of heap that can be allocated in this heap region is 16 *in bytes*.

After that we printed the memory address of the created heap handle, then we used the `HeapAlloc` function to allocate heap on the heap that we created.
The first argument required was the handle to the created heap, we specified it by specifying the `heap_handle` variable. Then we specified the second argument as `HEAP_ZERO_MEMORY`, which means the memory allocated should be initialised as 0 by default. At last, we specified 8 as the third argument, which means we want to allocate 8 bytes in heap.

Afterwards, we printed the memory address returned by `HeapAlloc`, then we used the `HeapFree` function with the first argument `heap_handle`, which we already know is a handle to the memory address created heap block. We then passed 0, by passing a 0 we meant to say that we don't want to use any special options while freeing the memory and at last, we specified the memory address of the heap that we allocated and at last we checked if it freed or not.

Let's run the program and see what it outputs!
Here is the output of the code:

```c
**The handle of heap created is 1b0000
The base address of heap allocated is 1b0000
The memory has been freed successfully!**
```

and indeed, the code worked as expected.

Now, as we have allocated some Heap, let's make some code to put some data on the heap and use it!

# Code Example - 2

In this example, we are going to use the `memmove` function to copy data to the memory address returned by `HeapAlloc`. The syntax of `memmove` looks like this:

```c
void *memmove(
	void *dest,
	const void *source,
	size_t n
)
```

The first argument for this function is the address of the destination, the second argument is the source from where we want to copy data and the last and third argument is the size of data that we want to move. After implementing this, our code looks like this:

```c
#include <stdio.h>

#include <Windows.h>

int main() {
    int * heap_handle = HeapCreate(0, 0, 16);

    if (heap_handle == NULL) {
        printf("Was not able to create heap!!\nError code: %d\n", GetLastError());
        exit(GetLastError());
    }

    printf("The handle of heap created is %x\n", heap_handle);
    int * heap = HeapAlloc(heap_handle, HEAP_ZERO_MEMORY, 8);
    printf("The base address of heap allocated is %x\n", heap_handle);

    memmove(heap, (const void * )"Hello heap!", 11);
    printf("The data stored in heap is: %s\n", heap);

    int freed = HeapFree(heap_handle, 0, heap);

    if (freed) {
        printf("The memory has been freed successfully!\n");
    } else {
        printf("Was not able to free heap!!");
        exit(1);
    }

    return 0;
}
```

The code is almost same, we just added the `memmove` function and copied the string `"Hello heap!"` to the memory address returned by `HeapAlloc`. and then we printed it. 

Let's run the code and see the output:

```c
The handle of heap created is 6b0000
The base address of heap allocated is 6b0000
The data stored in heap is: Hello heap!
The memory has been freed successfully!
```

Voila!!, We got our expected results. It felt really cool when I did this and I recommend you to make something with this functions!!

This was all for this one, meet you in the next one!