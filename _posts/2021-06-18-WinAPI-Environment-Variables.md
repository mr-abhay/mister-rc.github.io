---
title: Windows API - Let's Play with the Windows Environment Variables! 
author: Mr. Rc
date: 2021-06-18 11:33:00 +0800
categories: [Programming, Windows, Windows Internals, WinAPI]
tags: [Windows, Windows API Series, Virtual Memory Management]
math: false
mermaid: false
---

In this blog, we are going to have a look at two functions that let us manipulate the Environment Variables of our Windows Machine.
There are only two functions that are related to Environment Variables in Windows API, but I found them useful and fascinating.
The functions that we are going to use in this blog are `GetEnvironmentVariable` and `SetEnvironmentVariable`.

Before diving in, let's try to understand what are environment variables and how they are stored in Windows.

# What is An Environment Variable?

In the simplest terms, in a Windows Machine, an Environment Variable is a dynamic object or changeable data that any process on your machine could use and manipulate (if they have necessary privileges).

If you know Python, you can understand them like a dictionary, in which every key has a value assigned to it.
This is not how they actually look, but you can use this to understand Environment Variables:

```python
environment_var = {"var": "value"}
```

There are two types of Environment Variables:

1. User Environment Variables.
2. System Environment Variables.

## User Environment Variables and System Environment Variables

As you might already know, you can have different users on same Windows machine. So, the User Environment Variables are specific to a single user for which they were made, and System Environment Variables are same for every user on they same System / Machine.

The user Environment Variables are stored in the Windows Registry at the path `HKEY_CURRENT_USER\Environment` and the location of System Environment Variables in the Windows registry is `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment`.

You can look them up, user environment variables should look like this:

![https://i.ibb.co/XxPXVXm/image.png](https://i.ibb.co/XxPXVXm/image.png)

## Some more insights on Environment Variables

By default, a child process receives its Environment Variables from its parent process.
The maximum size of a user-defined environment variable is 32,767 characters.    
`SetEnvironmentVariable` function can't actually change an Environment Variable globally, the function only changes the Environment Variable for the process that called the function and the child processes that it will create.
If you actually want to change any Environment Variable for the user/system programmatically, you need to change the registry key from the specific path, rather than just changing the Environment Variables. I will explain this more later in this blog.

# 1. GetEnvironmentVariable

As the name suggests, this function is used to get the contents of an Environment Variable.

The Function Signature looks like this:

```cpp
DWORD GetEnvironmentVariable(
  LPCTSTR lpName,
  LPTSTR  lpBuffer,
  DWORD   nSize
);
```

Don't get scared by the `DWORD` return type, it's defined as `typedef unsigned __LONG32 DWORD;` in the `winmindef.h`, which means it'll just return an integer.

The first parameter of this function is `lpName`. The type of this parameter is `LPCSTR`, this is also defined in the same file as a pointer to a null terminated list of characters.

This parameter is used to specify the name of the Environment Variable whose value we want to use in our Program.

The second parameter is `lpBuffer`, if you read my blogs, you might already know that the parameters with these names are used to specify the pointer to the buffer that receives the output from the function.
In this particular function, the output will be a null terminated string that will contain the value of the Environment Variable that we specified as the first argument.

The last parameter is `nSize`, it is used to specify the size of the `lpBuffer` argument along with the space for null terminator.
If this feels complex to you, let me explain you more easily.
Imagine the following c code in a function: 

```cpp
char data[10];
```

Here, we have made a variable called `data` and we are assuming that this variable will store the value of the Environment Variable as a string.

The size of this variable is 10, which includes the size of the null character that will be stored at the end, so technically, the size of this variable is 9, you can only store 9 characters in this variable, and the 10th character will be a null terminator.

So, if we need to get the value of an Environment Variable, we need to make a char array of its size + 1.

### Return Value

If this function succeeds, it returns the size of characters that it stored in the address of the argument that we supplied in the `lpBuffer` parameter (not including the size of the null terminator).

If `lpBuffer` is not large enough to hold the value of the requested Environment Variable, it will return the size required to store the value including the null terminator, and nothing will be done with the `lpBuffer` parameter.

# 2. SetEnvironmentVariable

`SetEnvironmentVariable` function is used to set an Environment Variable.
Calling this function will not change the value of this Variable for any other process, it will only last in the process that created it until the process is not killed.
As we learned before, a child process inherits its Environment Variables from the parent process.
So, if a process sets an Environment Variable and then creates a child process, then only the child process will have that Environment Variable set.

The Function Signature of this one looks like this:

```cpp
BOOL SetEnvironmentVariable(
  LPCTSTR lpName,
  LPCTSTR lpValue
);
```

The return type of the function is the same as the last one, So it will just return a number.

The first parameter is `lpName`, which, of course, takes the name of the Environment Variable that you want to create/change.

The second parameter is `lpValue`, which will be the value for the Environment Variable in the first parameter.

## What are we supposed to do about this function?

As I told you, we can't change an Environment Variable globally using `SetEnvironmnetVariable` function. Then, how are we supposed to change an environment variable programmatically?

I told you that we can manipulate the registry keys from the specific paths of each Environment Variable and it will change the Environment Variables globally (for the user or the whole system).


In the next part of this blog, we will be writing a wrapper that will do that for us. I could have included it in this part, but it will make the blog post long and boring.

In this blog, we are just going to see the use of the function `GetEnvironmentVariable` and see how can we use it, It will also be fun ;)

I could also have included the usage of the`SetEnvironmentVariable` function by showing you a program that changes its Environment Variable using the `SetEnvironmentVariable` function, but to demonstrate that this function actually works, I would have needed to make a child process of the program itself. We haven't explored the functions that allow us to do so. I promise to write blog posts on those functions too!

# Code Example - 1

Now, let's make a function that uses the function `GetEnvironmentVariable` and does something.

Here is the code of the function that I made:

```c

#include <stdio.h>
#include <windows.h>

// this program will get the value of the environment variable "COMPUTERNAME" (which contains the name of your Windows computer) and print it

int main(){
    char com_name[200]; // this variable will store the value returned from the Environment Variable
    
    GetEnvironmentVariable("COMPUTERNAME", com_name, 200); // calling the GetEnvironmentvVariable to get the value of COMPUTERNAME environment variable
    puts(com_name); // printing the environment variable

    return 0;
}
```

Quite simple, right?

The comments of the code pretty much explain what's going on, but I'm still going to explain what this code is doing.

First, We declare the Variable `com_name`, which we will use to store the value of the Environment Variable, the size of this variable is 200 and it can store 199 characters and a null terminator.

Next, we call the function `GetEnvironmentVariable` to get the value of the Environment Variable which is `COMPUTERNAME` and store it in `com_name` variable. And, as you have learnt, we have to specify the size of the variable in the third argument, which is `nSize`, We also do that by specifying 200.

Then we simply call the `puts` function to print the Environment Variable.

The output of the code is pretty simple and, as expected, it just prints the value of the Environment Variable which we stored in the `com_name` variable and nothing else, which looks like this:

![https://i.ibb.co/5MVZZcV/image.png](https://i.ibb.co/5MVZZcV/image.png)

Awesome!!

But, this one was too small, lets look at a bigger one and make something more cool.

# Code Example - 2

Now, we are going to add some more functionality to our little program. This time, we will ask the user to enter the name of the Environment Variable and we will also add error throw error if the Environment Variable that the user is requesting doesn't exits.

The code that I made to achieve this is here:

```c
#include <stdio.h>
#include <windows.h>

int main(){
    char env_name[200]; // to store the name of the Environment Variable.
    char env_value[200]; // to store the value of the Environment Variable.

    puts("Enter the name of the variable whose value you want to know: ");
    scanf("%s", &env_name);

    GetEnvironmentVariable(env_name, env_value, 200); // calling the GetEnvironmentVariable function to get the value of the Environment variable.

    if (GetLastError()==ERROR_ENVVAR_NOT_FOUND) // Error checking.
    {
        puts("ERROR: Environment Variable %s does not exits.");
        printf("ERROR CODE: %d", GetLastError());
        exit(GetLastError());
    }

    printf("The value of the Environment variable %s is %s.\n", env_name, env_value);

    return 0;
}
```

This Program is also simple as it looks, let me explain...

First we declare two variables, `env_name` and `env_value` of both 200 size.

Then we ask the user to input the name of the Variable whose value they want to know, and we store the input in the `env_name` variable.

Then we call our almighty function, `GetEnvironmentVariable` with `env_name` variable as the first parameter, which stores the name of the Environment Variable whose value the user wants to know.

We pass the `env_value` variable as the second parameter, which is the variable that will store the value of the Environment Variable (if the function succeeds). And, we specify the last and third parameter as the size of the second parameter.

After all this, comes the interesting part. If you read my blogs frequently (which you should), you probably already know the usage of the `GetLastError`.

If you don't know read my blogs frequently, let me explain you the function `GetLastError`, it's used to get the last error, but what "last"?

"Last" is referring to the last function that you called, So, In this program, I have called the `GetEnvironmentVariable` function. So, this function will return the error code of the error that occurred in the `GetEnvironmentVariable` function and we can lookup this error codes here: [System Error Codes (0-499) (WinError.h) - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/debug/system-error-codes--0-499-)

But, if you look at the code, I have compared the return value of `GetLastError` with the constant `ERROR_ENVVAR_NOT_FOUND` which is defined in `winerror.h` which is actually `203` in decimal.

The reason why I am comparing this with `ERROR_ENVVAR_NOT_FOUND` constant or the value `203` is because it's the return value that `GetLastError` function will return when we try to access an Environment Variable that doesn't exits.

So, How do I know that I need to use this constant and not any other?

Well, the answer to this question is simply because I have experimented this and if you ask me how I did that, This is the code that I used:

```c
#include <stdio.h>
#include <windows.h>

int main(){
    char env_value[200];
    int error_code;

    GetEnvironmentVariable("aenvvarthatnoappwillmakeibet", env_value, 200); // calling the GetEnvironmentVariable function and requesting the value of a non-existing Environment Variable.
    error_code = GetLastError();

    printf("The Error code is: %d", error_code);
    return 0;
}
```

If you have understood the blog until this line, you probably have understood that this code is trying to request a value of a non-existing environment variable then it's calling the `GetLastError` function to get the error code of the last error that the last function call had and then printing it.

The output of this code looks like this:

![https://i.ibb.co/Sx7Rznr/image.png](https://i.ibb.co/Sx7Rznr/image.png)

Okay, now you know what error code the `GetEnvironmentVariable` returns when the Environment Variable does not exits, Now, How can I know which constant to compare it to?

Well, to know that, I'll refer you to this page again: [System Error Codes (0-499) (WinError.h) - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/debug/system-error-codes--0-499-)

You can search the error code there and find which constant you should use, in this case, It was this:

![https://i.ibb.co/kxpQ5K1/image.png](https://i.ibb.co/kxpQ5K1/image.png)

You can use pretty much the same approach to find error codes for any other error.

Let's get back to the program that I made, After I compared the error code with the constant, I displayed the error message exited with the return code of the error, which will be 203 in this case.
This was the whole explaination of this program, I hope you found it interesting!


And that was all for this one, I hope you learnt something new, I will be back with the second part of this blog very soon :)
