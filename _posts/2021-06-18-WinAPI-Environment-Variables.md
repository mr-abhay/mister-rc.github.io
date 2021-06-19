---
title: Windows API - Let's Play with the Windows Environment Variables! 
author: Mr. Rc
date: 2021-06-18 11:33:00 +0800
categories: [Programming, Windows, Windows Internals, WinAPI]
tags: [Windows, Windows API Series, Virtual Memory Management]
math: false
mermaid: false
---

In this blog, we are going to have a look at two functions that lets us manipulate the Environment Variables of our Windows Machine.
There are only two functions that are related to Environment Variables in Windows API but I found them useful and fascinating.
The functions that we are going to use in this blog are `GetEnvironmentVariable` and `SetEnvironmentVariable`.

Before diving in, let's try to understand what are environment variables and how are they stored in Windows.

# What is An Environment Variable?

In the simplest terms, In a Windows Machine, An Environment Variable is a dynamic object or changeable data that any process on your machine could use and manipulate(if they have necessary privileges).

If you know Python, you can understand them like a dictionary, in which every key has a value assigned to it.
This is not how they actually look but you can use this to understand Environment Variables:

```python
environment_var = {"var": "value"}
```

There are two types of Environment Variables:

1. User Environment Variables.
2. System Environment Variables.

## User Environment Variables and System Environment Variables

As you might already know, you can have different users on same Windows Machine.
So, the User Environment Variables are specific to a single user for which they were made and System Environment Variables are same for every user on they same System / Machine.

The user Environment Variables are stored in the Windows Registry at the path `HKEY_CURRENT_USER\Environment` and the location of System Environment Variables in the Windows registry is `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment`.

You can look them up, user environment variables should look like this:

![https://i.ibb.co/XxPXVXm/image.png](https://i.ibb.co/XxPXVXm/image.png)

## Some more insights on Environment Variables

By default, a child process receives it's Environment Variables from it's parent process.
The maximum size of a user-defined environment variable is 32,767 characters.
And an interesting thing to note is that **we can't use the `SetEnvironmentVariable` function to actually change any of the Environment Variable**.

`SetEnvironmentVariable` only changes the environment variable for it's own process and the child processes that it will create.

If you actually want to change any Environment Variable, you need to change the registry key from the specific path, rather than just changing the Environment Variables.
Don't worry if this all things feels new to you, I'll explain and give you examples soon in the blog.

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

Don't get scared by the `DWORD` return type, it's defined as `typedef unsigned __LONG32 DWORD;` in the `winmindef.h`, which means, it'll just return an integer.

The first parameter of this function is `lpName`. The type of this parameter is `LPCSTR`, this is also defined in the same file as a pointer to a null terminated list of characters.

This parameter is used to specify the name of the Environment Variable whose value we want to use in our Program.

The second parameter is `lpBuffer`, if you read my blogs, you might already know that the parameters with this names are used to specify the pointer to the buffer that receives the output from the function.
In this particular function, the output will be a null terminated string that will contain the value of the Environment Variable that we specified as the first argument.

The last parameter is `nSize`, it is used to specify the size of the `lpBuffer` argument along with the space for null terminated string.
If this feels complex to you, let me explain you more easily.
Imagine the following c code: 

```cpp
char data[10];
```

Here, we have made a variable called data, that will store the value of the Environment Variable as a string.

The size of this variable is 10, which includes the size of the null character that will be stored at the end, so technically, the size of this variable is 9, you can only store 9 characters in this variable and the 10th character will be a null terminating character.

So, If we have need to get the value of an Environment Variable, we need to make an array of it's size + 1. 

### Return Value

If the function succeeds, it returns the size of characters that it stored in the address of `lpBuffer` variable(not including the size of the null terminating character).

If `lpBuffer` is not large enough to hold the value of the requested Environment Variable's value, it will return the size required to store the value including the null terminator and nothing will be done with the `lpBuffer` parameter.

# 2. SetEnvironmentVariable

`SetEnvironmentVariable` function is used to set an Environment Variable.
Calling this function will not change the value of this Variable for a any other process, it will only last in the process that created it until the process is not killed.
As we learned before, A child process inherits it's Environment Variables from the parent process.
So, If a process sets an Environment Variable and then creates a child process, then only the child process will have that Environment Variable set.

The Function Signature of this one looks like this:

```cpp
BOOL SetEnvironmentVariable(
  LPCTSTR lpName,
  LPCTSTR lpValue
);
```

The return type of the function is the same as the last one, So, It will just return a number.

The first parameter is `lpName`, which of course takes the name of the Environment Variable that you want to create / change.

The second parameter is `lpValue`, which will be the value for the Environment Variable in the first parameter.

## What are we supposed to do about this function?

As I told you, we can't change an Environment Variable for every process using `SetEnvironmnetVariable` function. Then, how are we supposed to change an environment variable programmatically?

Well, We always had an option from the start, that I already told you about.

We can change the Environment Variables by changing the registry keys, right?

So..., What if we were able to change the registry keys and write a wrapper that does this for us?

It would be fun, right?

That's exactly what we are going to do in the next part of the blog, I could have included it in this same part but It will make the blog post huge and boring.

In this blog, we are just going to see the use of the function `GetEnvironmentVariable` and see how can we use it, It will also be fun ;)

I could also have included the usage `SetEnvironmentVariable` function by showing you a process setting an Environment Variable but for that, we would need to read about another function and that will also increase the size of the blog, but I Promise to write blog posts on that functions too!

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

The comments of the code pretty much explains what's going on, but, I'm still going to explain what's this code doing.

First, We declare the Variable `com_name`, which we will use to store the value of the Environment Variable, the size of this variable is 200 and it can store 199 characters and a null terminator.

Next, we call the function `GetEnvironmentVariable` to get the value of the Environment Variable `COMPUTERNAME` and store it in `com_name` variable.

And, as you have learnt that we have to specify the size of the variable in the third argument, which is `nSize`, We also do that by specifying 200.

Then we simply use `puts` to print the Environment Variable.

The output of the code is pretty simple and expected, it just prints the value of the Environment Variable and nothing else, which looks like this:

![https://i.ibb.co/5MVZZcV/image.png](https://i.ibb.co/5MVZZcV/image.png)

Awesome!!

But, this one was too small, let look at a bigger one and make something more cool.

# Code Example - 2

Now, we are going to add some more functionality to our little program, this time, we will ask the user to enter the name of the Environment Variable and we will also check if the Environment Variable exits or not.

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

Then we ask the user to input the name of the Variable whose value they wants to know, and stores the input in the `env_name` variable.

Then we call our almighty function, `GetEnvironmentVariable` with `env_name` variable as the first parameter, which stores the name of the Environment Variable whose value the user wants to know.

We pass the `env_value` variable as the second parameter, which is the variable that will store the value of the Environment Variable(if it succeeds).

And, we specify the last and third parameter as the size of the second parameter.

After all this, comes the interesting part. If you read my blogs frequently(which you should), you probably already know the usage of the `GetLastError`.

If you don't know read my blogs frequently, let me explain you the function `GetLastError`, it's used to get the last error, but what "last"?

"Last" is referring to the last function that you called, So, In this program, I have called the `GetEnvironmentVariable` function.

So, this function will return the error code of the error that occurred in the `GetEnvironmentVariable` function and we can lookup this error codes here:  

[System Error Codes (0-499) (WinError.h) - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/debug/system-error-codes--0-499-)

but, if you look at the code, I have compared the return value of `GetLastError` with the constant `ERROR_ENVVAR_NOT_FOUND` which is defined in `winerror.h` which is actually `203` in decimal.

The reason why I am comparing this with `ERROR_ENVVAR_NOT_FOUND` constant or the value `203` is because it's the return value that `GetLastError` function return when we try to access a Environment Variable that doesn't exits.

Wait, How do I even know that?

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

If you have understood the blog until now, you probably have understood that this code is trying to understand a non-existing environment variable then it's calling `GetLastError` to get the error code of the last error and then printing it.

The output of this code looks like this:

![https://i.ibb.co/Sx7Rznr/image.png](https://i.ibb.co/Sx7Rznr/image.png)

Okay, now you know what error code the `GetEnvironmentVariable` returns when the Environment Variable does not exits, Now, How can I know which constant to compare it to?

Well, to know that, I'll refer you again to this page: 

[System Error Codes (0-499) (WinError.h) - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/debug/system-error-codes--0-499-)

You can search the error code there and find which constant you should use, in this case, It looked like this:

![https://i.ibb.co/kxpQ5K1/image.png](https://i.ibb.co/kxpQ5K1/image.png)

You can use pretty much the same approach to find error codes for any other function.

Let's get back to the program that I made, After I compared it with the constant, I displayed the error message exited with the return code of the error, which will be 203 in this case.

And that was all for this one, I hope you learned something new, I will be back with the second part of this blog very soon :)
