---
title: Making your first Bootable Operating System!
author: Mr. Rc
date: 2021-03-30 11:33:00 +0800
categories: [Programming]
tags: [Assembly, Operating system]
math: false
mermaid: false
---

**Hey there,**    
**Mr. Rc here. Today in this blog, we are going to deep dive into the stuff that happens when you boot your computer and how can you make something like that.**

## Prerequisites
- **Basic understanding of BIOS and firmware.**
- **Basic understanding of assembly language.**
- **Basic understanding of stack.**

## Boot sector
>A boot sector is the sector of a persistent data storage device (e.g., hard disk, floppy disk, optical disc, etc.) which contains machine code to be loaded into random-access memory (RAM) and then executed by a computer system's built-in firmware (e.g., the BIOS).    Usually, the very first sector of the hard disk is the boot sector, regardless of sector size (512 or 4096 bytes) and partitioning flavor (MBR or GPT).    The purpose of defining one particular sector as the boot sector is inter-operability between various firmwares and various operating systems.
*~ Wikipedia*

## Bootloader
**At the very core of a computer booting is what we refer to as the boot
loader. The boot loader physically reads the first sector or sector 0 from
your HD or other media to ultimately bootstrap an OS.
When the computer boots it reads the first sector which is exactly
0x200 bytes ( hex) or 512 bytes in decimal.
The system that is reading this boot loader is what is referred to as
BIOS which is a basic input output system and it loads in 16-bit mode. It
does this to be compatible with older processors. Modern processors
immediately switch to what we refer to as UEFI which is a more
sophisticated IO system however we will focus on the very basics here
with BIOS. ~ 0xInfection's reversing for everyone course.**

## Making our first bootable operating system.
Before diving in, You should have [nasm](https://www.nasm.us/pub/nasm/releasebuilds/2.15rc12/) and [qemu](https://www.qemu.org/download/) installed in your machine. Both are available for Windows(not xp) and Linux systems.

In linux nasm and qemu can be installed through a single command:
```console
$ sudo apt install nasm; sudo apt install qemu-system-x86
```

As all the installation work is done, Lets starting coding ;) .
 
## Coding time
Lets make function that does nothing more than jumping to itself.
```assembly
callme:
    jmp callme
```
**This isn't the code for our operating system, I rather want you to how to use nasm and qemu and also I want to give you basics about opcodes. So create a file with this code then save it as bootsector.asm and then run this command**
```console
$ nasm bootsector.asm -f bin -o bootloder.bin
```

Before booting our Operating system, do you know what is a hex dump?

I think yes but if no then hexdump just means what its name says.    
Hex: Base 16    
Dump: You already know that.    
**In other words its just a utility that shows the content of a file in hexadecimal(base 16).**

One more and last question for you. Do you know what are opcodes in assembly?    
**In the most simplest term I can say that opcodes are just a hex value for every assembly instruction.**     
We will start by seeing the opcode of our instructions that we wrote in our bootsector.asm file.    
**To do that you should have xxd(linux) or HXD(windows) installed.**     
If you are with xxd, just type
```xxd bootsector.bin``` and here you will be able see it.
If you are in HXD, just open the file and you will be able to see the hexdump.    

**Hexdump**
```
00000000: ebfe                                     ..
```

Here we see some hex values. What are they?     
**Let me explain**    
Do you remember opcodes?    
**This hex values are the opcodes of the instructions that we coded in the bootsector.asm file.**    
Let's dig deeper and find what does this means.    
Before that, do you know what is a nibble, a bit, a byte.
- **A bit = 0 or 1.**
- **A nibble = 4 bits**
- **A byte = 8 bits.**     

If you don't know each hex digit represents 4 bits(a nibble). Here in this hexdump we can see two bytes: eb and fe. ebfe is a infinite loop in assembly, it just calls itself. eb is the opcode of jmp instruction and fe is the hex value of an immediate -2. So ebfe combined makes jmp -2.  jmp -2 means jump to the current instruction pointer - 2.
 
## One step further
As we have come this far and have learned many things, it's time to learn more and then boot our first operating system!!

### More coding
Here is the code of bootsector.asm that we made, now we are going to do some more coding into this and then run our first operating system.
```
loop:
    jmp loop
```

Let's add some more stuff to this piece of code.
```
loop:
    jmp loop

db 0x10
```

Save it and make a bin file of it.
```nasm bootsector.asm -f bin -o bootsector.bin```
**Now, let's look at the hexdump of this file. It looks like this**:
```
00000000: ebfe 10                                  ...
```
If you don't know ```db(data byte)``` instruction is used for allocate some space and fill it with some data. As a result we can see 10 in our hex dump.

**This isn't enough. Let's add your name to this file.**
```
loop:
    jmp loop

db 0x10
db 'Hello world, Mr. Rc on this side'
```
Let's make a bin file again and look at the hexdump of this file.
```
nasm bootsector.asm -f bin -o bootsector.bin
```
**Hexdump**
```
00000000: ebfe 1048 656c 6c6f 2077 6f72 6c64 2c20  ...Hello world,
00000010: 4d72 2e20 5263 206f 6e20 7468 6973 2073  Mr. Rc on this s
00000020: 6964 65                                  ide
```
**We already know why is thee ``eb`` and ``fe``, but why do we have that other hex values?**    
If you look closely on the right side, you can see it is the ascii translation of what is in the center column. The first three dots indicate that it wasn't able to find the character for those hex values or ascii values. After first 3 bytes you can see there is a 48. 48 in hex = 72 in decimal and 72 is the ascii value of capital H. 65 in decimal is 101 and 101 ascii = small e. It goes for every character to end.

## Booting time
Now that we have done coding and understood many things, we're going to just add two lines to our code and then our first ever operating system will be able to boot.    

### Coding time
Write this into your bootsector.asm file, I will explain the code later.
```
loop:
    jmp loop
db 0x10
db 'Hello world, Mr. Rc on this side'

times 0x1fe-($-$$) db 0
dw 0xaa55
```
It's ok if you don't understand what this means. Let us understand what does this all means to us.    
**We all already know what the first four lines means. Let us understand the last two lines.**    
```times 0x1fe-($-$$) db```
This instruction will simply subtract the left bytes with 0x200 or 512 in decimal. But why 0x200 ?    
That's because we already know
>When the computer boots it reads the first sector which is exactly 0x200 bytes ( hex) or 512 bytes in decimal. ~ Bootloader paragraph    
For this reason we have to pad the file with exactly 512 bytes and that's why we are using that instruction. 

### Magic number
If you look at the code again, we have allocated space for 0xaa55. Why that?    
**0xaa55 is a magic number which is a signature which a CPU looks to identify a boot sector.**

### Hexdump time
We are very close to booting our first operating system, but before that let's just see how it's hexdump looks like because there is something amazing to see.    
Make binary:
```
nasm bootsector.asm -f bin -o bootsector.bin
```
View the hexdump:
```
xxd bootsector.bin
```
Here is how it looks like:
```
00000000: ebfe 1048 656c 6c6f 2077 6f72 6c64 2c20  ...Hello world,
00000010: 4d72 2e20 5263 206f 6e20 7468 6973 2073  Mr. Rc on this s
00000020: 6964 6500 0000 0000 0000 0000 0000 0000  ide.............
00000030: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000040: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000050: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000060: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000070: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000080: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000090: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000a0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000b0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000c0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000d0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000e0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000f0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000100: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000110: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000120: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000130: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000140: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000150: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000160: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000170: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000180: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000190: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000001a0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000001b0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000001c0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000001d0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000001e0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000001f0: 0000 0000 0000 0000 0000 0000 0000 55aa  ..............U.
```
As we can see that the first some bytes that we already analysed are at their normal place and then our whole file is padded with zeros and at last we have that magic number. The last two bytes of this file are the reverse of what we coded. We coded ``aa 55`` and it became ``55 aa``, if you have guessed it right then you're serious with this post. It's because of the endianness which is little endian at this time.

## Booting time
After learning and understanding so much things, finally we're going to boot our Operating system.    
To boot it, make sure you have made a bin file of bootsector.asm. If you've not done that then run this command:
```
nasm bootsector.asm -f bin -o bootsector.bin
```
Then run this command to boot it:
```
qemu-system-x86_64 bootsector.bin
```
**This will run your operating system on the emulator but remember it willn't show anything to us because it's just running a infinite loop. However, We've made our first operating system and we'll go a step further and make this more good.**


## Making our OS more awesome.
You've came so far and I can admit it that you're the one who has the capability of sticking with one thing until you fully understand it. Now it's time for making a operating system that boots and asks for input.    
Here is the code that you'll need to do make it happen. Don't get scared by it, I try will explain every line to you.
```
[org 0x7c00]

mov bp, 0xffff
mov sp, bp

call set_video_mode
call get_char_input

jmp $

set_video_mode:
	mov al, 0x03
	mov ah, 0x00
	int 0x10
	ret

get_char_input:
	xor ah, ah
	int 0x16

	mov ah, 0x0e
	int 0x10

	jmp get_char_input

times 0x1fe-($-$$) db 0
dw 0xaa55
```
### Line 1
The first line is a bit scary but ofcourse we are working at very low level and we should see this kind of stuff let me explain its use. The first instruction will simply move our machine code to the address 0x7c00. 0x7c00 is the address which contains all the code that will be loaded into RAM by the bootloader firmware (firmware is simply code that runs
before an OS runs like what we are doing).

## Line 2-5
**On line 2, we are setting our base pointer. If you don't know what is base pointer then In the simplest terms I can say that its kind of a CPU register that contains the address of the base of the stack. We set our base pointer to point at 0xffff and we set stack pointer to base pointer. 0xffff is just some free place to store data before our system boots like 0x7c00.**    
On line 4 and 5, we call two different functions. The first one is ```set_video_mode``` and the other one is ```get_char_input```.    

Let's look at the code of those functions and understand what they do.
```
set_video_mode:
	mov al, 0x03
	mov ah, 0x00
	int 0x10
	ret
```
The first line is just moving the immediate 0x03 to al, then we move 0x00 to ah register and the we use the int instruction with 0x10 value. Let me explain what's this code is all about. If you're familiar with calling conventions, you would quickly recognise that there is no calling convention there which takes arguments from al and ah registers but this isn't a ```call``` it's a ```int``` instruction. The INT n instruction is the general mnemonic for executing a software-generated call to an interrupt handler. Here we are calling it with 0x10. INT 0x10 instruction is used to setup the video mode when the system boots and as it's not a normal call to a function the arguments should be stored in ah and al registers. In INT 10 instruction, we use al register to specify desired video mode. Here is the list of all the modes that we can use:
- AL=0x00 - text mode. 40x25. 16 colors. 8 pages.
- AL=0x03 - text mode. 80x25. 16 colors. 8 pages.
- AL=0x13 - graphical mode. 40x25. 256 colors. 320x200 pixels. 1 page.    

Also, we can specify different screen types by changing the values of registers to this values:
- **AH=0x00: Video mode.**
- **AX=0x1003: Blinking mode**
- **AH=0x13: Write string**
- **AH=0x03: Get cursor position**   
 
Here we have set AH to 0x00 and al to 0x03. That means we have chosen the text mode with 80x25 screen size, which have 16 colors and also we are using the video mode.

Let's dive into the second function(```get_char_input```). The first we are doing is xoring ah. When we xor a number by itself, the result is always zero. So, we're just zeroing our ah register and then we call INT 0x16 instruction. INT 0x16 is used for controlling keyboard related stuff(i.e. reading keystrokes, etc.). Setting ah to 0 means that we want to read input from the keyboard. The next line ```mov ah, 0x0e``` is setting ah register to hex 0e or 14 decimal and then we called INT 0x10 but wait our register value is different this time. When the value of ah is 0e before it we call INT 0x10 it would work like print and it would print whatever the character. At the end we pad the file like we would normally do.

## Booting time
As we've understood everything that what's going on, we can boot out first operating system.    
To do that we have to first make a bin file of our asm file.
```nasm bootsector.asm -f bin -o bootloder.bin```
Then run it with qemu:
```qemu-system-x86_64 bootsector.bin```
Now your operating system has been booted and you can type whatever you want.
![image.png](https://i.ibb.co/k9kR4GG/image.png)
It took a little but finally what we've done is amazing. Now it's time to show this to your friends and twitter. If you read this much and you were able to make this then tag me on twitter, my username is coder_rc .

# Conclusion
**I learned so much and I hope you did too. I got the idea of writing this blog post when I was reading 0xinfection's reversing for everyone book. One of it's part was bootsector basics and after reading it, I was amazed and I thought it would be great if I would be able to teach this to others. I also learnt many things that I probably skipped in those chapters. It took me around 5-6 hour to write this I hope you learned something new.**
