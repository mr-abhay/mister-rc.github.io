---
title: Windows API - Let's end the Heap Chapter!
author: Mr. Rc
date: 2021-05-29 11:33:00 +0800
categories: [Windows, Windows Internals, WinAPI]
tags: [Windows, Windows API Series, Heap API]
math: false
mermaid: false
---

Welcome to the last blog post on the HeapAPI functions from the Windows API series, this post willn't be the last blog post for the whole Windows API series but it will be the last blog post for the HeapAPI functions.    
    
In this blog, we will play with some more heap functions with some code examples. Let's start!    
    
# 1. GetProcessHeap    
If you remember from the last blog, I told you that **Every process has a heap already when it's created. The function `GetProcessHeap` is used to get the Handle of the default heap of the process.**    
The function's syntax looks like this:    
    
```c    
HANDLE GetProcessHeap();    
```    
    
We don't need any arguments because it will return the Handle of current process.    
And that's pretty much everything you need to know about this function.     
    
# 2. HeapLock and HeapUnlock    
Before I start telling you about this function, you need to understand `critical object` aka `critical section` in operating systems.    
    
## 2.1 Critical section    
The critical section is a code segment where the shared variables can be accessed. An atomic action is required in a critical section i.e. **only one process can execute in its critical section at a time. All the other processes have to wait to execute in their critical sections**.    
    
## 2.2 HeapLock    
This function **attempts to acquire the critical section object, or lock, of a specified heap**.    
In simplest terms, **If this function successfully acquires the critical section of a heap, then only the function will be able to use it until it unlocks it**.    
The syntax of `HeapLock` function looks like this:
```c
BOOL HeapLock(    
  HANDLE hHeap    
); 
```    
    
We just have to pass the Handle of heap which is returned by `GetProcessHeap` or `CreateHeap` function as the first and only argument.    
    
To unlock the heap, we have to use the `HeapUnlock` function. The syntax of `HeapUnlock` looks like this:    
    
```c    
BOOL HeapUnlock(    
  HANDLE hHeap    
);    
```    
    
Same as `HeapLock`, we just have to pass the Handle to the Heap and it will unlock the critical section.     
    
# 3. HeapDestroy    
    
As the name says, `HeapDestroy` function is used to destroy a heap. If you remember from the last blog post, I told that when we free a memory region, we can reallocate it. The Heap doesn't gets destroyed when use `HeapFree` function. To completely destroy the Heap, we use the `HeapDestroy` the function. Remember, we can't delete the heap that we get by default for our process which we get by calling `GetCurrentHeap` function.    
    
The function syntax of `HeapDestroy` looks like this:    
    
```c    
BOOL HeapDestroy(    
  HANDLE hHeap    
);    
```    
    
Same as all the other functions, we just have to supply the Handle to the Heap and the Handle shouldn't be     
    
Nothing complex, right!    
Now, let's make use of these functions and make a program.    
    
# Code Example    
    
In this first example, I have only used the `GetCurrentHeap`. We will use the     
    
```c    
#include <stdio.h>    
#include <Windows.h>    
#include <string.h>    
    
int main() {    
    int heap_size;    
    char data_to_store[2000];    
    char choice[20];    
    int* current_heap = GetProcessHeap(); // Getting the Handle of default Heap.    
    
    printf("How much heap do you want to allocate? (int bytes): ");    
    scanf("%d", &heap_size);    
    
    int* heap = HeapAlloc(current_heap, HEAP_ZERO_MEMORY, heap_size); // Allocating heap of the user supplied size. Filled with zero by default.    
    printf("The handle of heap of this program is %x\n", current_heap);    
    
    printf("Enter the data you want to store in the heap: ");    
    scanf("%s", &data_to_store);    
    memmove(heap, (const void*)data_to_store, strlen(data_to_store)); // Moving the user supplied data into the allocated heap.    
        
    printf("The data stored in heap is: %s\n", heap);    
    
    int freed = HeapFree(current_heap, 0, heap); // Free()'ing the heap.    
    
    if (freed) {    
        printf("The memory has been freed successfully!\n");    
    } else {    
        printf("Was not able to free heap!!");    
        exit(1);    
    }    
        
    return 0;    
}    
```    
    
The code is very simple and straight-forward.        
    
We first get the Handle of the default heap by calling the `GetCurrentHeap` function. The well allocate the amount of heap that user supplied then we ask the user to input the data that they want to move into the allocated heap, then it frees the heap.    
    
Let's run this program and see the output.    
    
The output looks like this:    
    
```    
How much heap do you want to allocate? (int bytes): 8    
The handle of heap of this program is 760000      
Enter the data you want to store in the heap: hello    
The data stored in heap is: hello          
The memory has been freed successfully!    
```    
    
Perfect!...    
    
I guess this is all you need to know about Heap functions in Windows API, I hope you liked this post.    
    
This was all for this one, meet you in the next one!