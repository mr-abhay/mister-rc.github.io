---
title: Windows API - More on Virtual Memory.
author: Mr. Rc
date: 2021-04-18 11:33:00 +0800
categories: [Programming, Windows, Windows Internals, WinAPI]
tags: [Windows, Windows API Series, Virtual Memory Management]
math: false
mermaid: false
---

In this blog, we are going to have a look at some basic memory related things that we can do with the API functions that we learnt about in last blog post along with a new function `VirtualQuery`.

Before we start, I want to tell you about a function from Windows API which is `GetLastError` from Windows API. It is used to get the error code of the last error that occurred and we can get more information about the error code by looking at the error code list which is available here :
[System Error Codes - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/debug/system-error-codes#system-error-codes-1)

# 1. VirtualQuery

This function is used to query the information of Virtual Memory allocated by `VirtualAlloc` and other Virtual Memory allocating functions.

This is the syntax for `VirtualQuery` function:
```c
SIZE_T VirtualQuery(
  LPCVOID                   lpAddress,
  PMEMORY_BASIC_INFORMATION lpBuffer,
  SIZE_T                    dwLength
);
```

Forget about the function's type (`SIZE_T`) for now. Let's look at the arguments that are required...

The first argument is `lpAddress`, which I guess you already know if you are reading this blog. If you don't know, then you can read the first part of Windows API series and learn about those things. 
It's just the base address of Virtual Memory region that we allocated which is returned by `VirtualAlloc`.

The second argument is `lpBuffer`. This is a pointer to a special struct that is already defined in `winint.h` and which looks like this:
```c
typedef struct _MEMORY_BASIC_INFORMATION {
  PVOID  BaseAddress;
  PVOID  AllocationBase;
  DWORD  AllocationProtect;
  WORD   PartitionId;
  SIZE_T RegionSize;
  DWORD  State;
  DWORD  Protect;
  DWORD  Type;
} MEMORY_BASIC_INFORMATION, *PMEMORY_BASIC_INFORMATION;
```

You can see this struct has some pretty useful members. We are going to use some of this to make a program that we are going to make in this blog. 
To define this struct in our program, we can just write `MEMORY_BASIC_INFORMATION info;` and this will define a struct with all this members. After making this struct, we will just pass the memory address of this struct in the `lpBuffer` argument.

The third argument is `dwLength`. If you have guessed this is the size of the memory region whose address we have to specify as the `lpAddress` argument, then sorry you're wrong and I was too when I was reading about it for the first time. It is actually the `sizeof` the struct that we are going to pass into it.

## Return value

Instead of returning something the function just updates the struct that we created.

## Code Example #1

Now as we have done with understanding the things we are going to do, let's code stuff. We are going to make a program that will give us the information about a memory region. Let me show you the code first, then I will explain:
```cpp
#include <Windows.h>
#include <stdio.h>

int main()
{
	MEMORY_BASIC_INFORMATION info;
	int ret;
	int *vm = VirtualAlloc(NULL, 8, MEM_COMMIT, PAGE_READONLY);

	ret = VirtualQuery(vm, &info, sizeof(info));
	if (!ret)
	{
		printf("VirtualQuery failed\n");
		printf("The error code for the last error was %d", GetLastError());
		return 1;
	}

	switch (info.AllocationProtect)
	{
		case PAGE_EXECUTE_READ:
			printf("Protection type : EXECUTE + READ\n");
			break;
		case PAGE_READWRITE:
			printf("Protection type : READ + WRITE\n");
			break;
		case PAGE_READONLY:
			printf("Protection type : READ\n");
			break;
		default:
			printf("Not found");
			break;
	}

	switch (info.State)
	{
		case MEM_COMMIT:
			printf("Region State : Committed");
			break;
		case MEM_FREE:
			printf("Region State : Free");
			break;
		case MEM_RESERVE:
			printf("Region State : Reserve");
			break;
		default:
			break;
	}
	return 0;
}
```

Don't get scared by this code, this is actually so self-explanatory. If you have looked into the code, you have probably seen that I have used `Windows.h` instead of importing any other file which contains all those Windows API functions. That's because we have included `Windows.h`, which contains all functions from the Windows API, all the common macros used by Windows programmers, and all the data types used by the various functions and subsystems.
Let's break down the code...

First, we have declared a variable of type `MEMORY_BASIC_INFORMATION`, which is the struct that we talked about, then we committed eight *bytes* of virtual memory which is read-only. After that, we used `VirtualQuery` function to get information about that memory region.

We gave the address of the memory region as our first parameter, then we gave the address of the info struct that will hold all the returned data from this function, then we gave it the size of our info struct.

Afterwards, we check if the function failed. If you are confused about how it is being checked, then the answer is simple, it checks if the function returned false. According to the documentation, the function returns a zero if it fails and 0 means false in C. So it's just checking if the function is returning false. If the function fails, it will first print that the function is failed and then it will print the error code of the last error by printing the error from the `GetLastError` function and then return.

Then we have a switch-case clause where we are checking the value of `AllocationProtect` in our info struct. If you wonder how they are being checked with those constants which are defined nowhere, they are just constants which are already defined in `Windows.h`. Then we are checking the value of `State`, in our struct. Then we are just printing stuff. One thing to note is that we cannot compare the value with every type of protection type or every type of memory states, I have tried doing that but I was unsuccessful, so I am just adding the types that can be compared.

And that all our program does, Now let's run it and check the output. This is the output:
```cpp
Protection type : READ
Region State : Committed
```

As expected, we had hardcoded the page protection to be read-only and the page is committed. 
Let's make this more interesting by looking at another more cooler example. 

## Code Example #2

So in this example, we are going to implement a functionality to get information from any memory region. This one is more fun. This is the code

```cpp
#include <Windows.h>
#include <stdio.h>

int main()
{
	MEMORY_BASIC_INFORMATION info;
	int ret;
	const void *location;
	int *vm = VirtualAlloc(NULL, 8, MEM_COMMIT, PAGE_READONLY);
	printf("Address of memory returned by VirtualAlloc is %lu\n", vm);
	printf("Enter the memory address that you want to query: ");
	scanf("%lu", &location);
	ret = VirtualQuery(location, &info, sizeof(info));
	if (!ret)
	{
		printf("VirtualQuery failed\n");
		printf("The error code for the last error was %d", GetLastError());
		return 1;
	}
	printf("Protection type : ");
	switch (info.AllocationProtect)
	{
		case PAGE_EXECUTE_READ:
			printf("EXECUTE + READ\n");
			break;
		case PAGE_READWRITE:
			printf("READ + WRITE\n");
			break;
		case PAGE_READONLY:
			printf("READ\n");
			break;
		default:
			printf("Unknown\n");
			break;
	}

	printf("Region State : ");
	switch (info.State)
	{
		case MEM_COMMIT:
			printf("Committed");
			break;
		case MEM_FREE:
			printf("Free");
			break;
		case MEM_RESERVE:
			printf("Reserve");
			break;
		default:
			printf("Unknown");
			break;
	}
	return 0;
}
```

Not so much changed, let me explain. The code is almost same as the first code but here we first printed the memory address that we allocated using `VirtuaAlloc` and then we asked the user for a memory address. Instead of directly querying the memory address returned by `VirtualAlloc`, We first printed it and then asked for that memory address. By doing this, we can verify that the results of `VirtualQuery` by looking at what we hardcoded. Also, you can give this program any other memory address and it'll query it's information. Here is the output of the code:
```cpp
Address of memory returned by VirtualAlloc is 1572864
Enter the memory address that you want to query: 1572864 
Protection type : READ  
Region State : Committed
```

And as we expected, we hardcoded the protection type as READ-ONLY and Region state as Committed and it gave us correct information. Coding this was fun and I also recommend you to make something with all the Memory API related functions that we learnt about. 

I hope you liked this one, meet you in the next one!
