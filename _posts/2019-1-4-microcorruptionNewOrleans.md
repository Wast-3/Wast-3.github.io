---
layout: post
title: MicroCorruption New Orleans Tutorial
---
Welcome, to Microcorruption. If you're reading this, you're either very, very lost, or have an interest in reverse-engineering and hacking. Microcorruption is a series of challenges that task you with opening a virtual door, by exploiting vulnerabilities in the code of a smart lock. If you can get through all of the challenges, you'll have a good understanding of how programs are executed at their lowest level, how bad programming practices can be exploited, and how buffer-overflows (and other exploitation techniques) work. 

The smart lock is based around the TI MSP430 CPU. The instruction set of the MSP430 is very beginner friendly and you should be able to pick it up easily. You should read the lock manual before starting. You can get in-depth information about instructions by looking up the MSP430 User's Guide. 

After you complete the tutorial, you should have a basic knowledge of what's going on on-screen. It looks overwhelming to start, but once you get the hang of things everything will click together. Right now, we don't have a very clear understanding of what the program is doing, so first let's check out the program disassembly. 

The program starts at `0x4400` and executes an initialization routine. You can safely ignore these, as all they do is set the program up to run properly. Let's start by taking a look at the main function. 

![img1.png](../images/f0048297b344459185baee6b29c859ce.png)

It's pretty simple, and by ignoring assembly specifics it can be broken down to the following:

1) Call the create_password() function
2) Print the string "Enter the password to continue" to output
3) Call the get_password() function
4) Call the check_password() function
5) Test if the value stored in register 15 is equal to zero and set the status register accordingly
6) Make a jump to `0x4462` if the status register indicates r15 is non-zero. If not do nothing
7) If a jump is not made, print the string "Invalid password; try again." to output and end execution.
8) If a jump is made, print "Access Granted!" to output and call unlock_door()

When trying to get a picture of how a program executes, start at the main() function, and look at what functions main() calls. Once you know what can happen, you can start to piece together how. Look for jmp instructions (jnz, jz) and where they point to in the program. You'll see what "paths" a program can take during it's execution. 

This is only the beginning, but you should have a vague idea of what an disassembled program looks like by now. Let's inspect the first function that is called, create_password().

![img2.png](../images/8540db77481c4eb083456b4230c25d9d.png)

The function moves different values into memory byte by byte. I'll break down exactly what happens here.

```
447e:  3f40 0024      mov	#0x2400, r15    ; move the value 0x2400 into register r15.
4482:  ff40 5e00 0000 mov.b	#0x5e, 0x0(r15) ; copy the byte 0x5e, into the address stored in r15, plus an offset of zero.
4488:  ff40 3000 0100 mov.b	#0x30, 0x1(r15) ; copy the byte 0x30, into the address stored in r15, plus an offset of one.
448e:  ff40 2800 0200 mov.b	#0x28, 0x2(r15) ; copy the byte 0x28, into the address stored in r15, plus an offset of two.
4494:  ff40 5f00 0300 mov.b	#0x5f, 0x3(r15) ; copy the byte 0x5f, into the address stored in r15, plus an offset of three.
449a:  ff40 2d00 0400 mov.b	#0x2d, 0x4(r15) ; copy the byte 0x2d, into the address stored in r15, plus an offset of four.
44a0:  ff40 2a00 0500 mov.b	#0x2a, 0x5(r15) ; copy the byte 0x2a, into the address stored in r15, plus an offset of five.
44a6:  ff40 5800 0600 mov.b	#0x58, 0x6(r15) ; copy the byte 0x58, into the address stored in r15, plus an offset of six.
44ac:  cf43 0700      mov.b	#0x0, 0x7(r15)  ; copy the byte 0x7, into the address stored in r15, plus an offset of seven.
44b0:  3041           ret
```

On the very left of each line of assembly, is the address of the code in memory. To the right of that, the machine code of the instruction and arguments. To the right of that, are human readable mnemonics that represent the machine code. 

Values are being copied byte by byte to memory. The base address is held in register 15, 0x2400. This function is suspicious, it's involved with the password and is writing to memory. Let's step through the function and see what happens at memory address 0x2400. 

Go ahead and set a breakpoint at the start of the create_password function. Type `c` into the debugger console. At this point the program will be halted right before the function begins. Before stepping through the function, use the live memory dump to look at memory address `0x2400`. Step forward one instruction. You'll notice that r15 gets set to 2400. Step forward once more, and you'll see the memory contents change based on the executed instructions. To the right, you'll see an ASCII representation of values in memory. Is it possible that the password is right here before our very eyes? Maybe, but I'll leave it to you to find out. 