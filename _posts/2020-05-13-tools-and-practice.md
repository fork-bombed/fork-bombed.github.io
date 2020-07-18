---
title: Tools and Practice
author: Murphy
date: 2020-05-13 09:00:00 +0100
categories: [Reverse Engineering]
tags: [tools, crackme]
---

In order to start reversing my first program, I'll need (at least) two tools: a disassembler and a debugger. A disassembler converts machine code into assembly language and will allow me to view the program's instructions without actually running it (*static analysis*). A debugger will allow me to view and alter the program's memory while it's running (*dynamic analysis*). This is very powerful as I can set breakpoints and view data in registers and on the stack as each instruction is executed.

There are plenty of disassemblers and debuggers out there and it can be hard to choose the right one, so I've compiled a list of the most popular ones and my first impressions of them.

## IDA

IDA is a disassembler and debugger, which comes in handy as you can do static and dynamic analysis from the same program. On top of this, it comes with a decompiler which converts the assembly code into a high-level pseudocode (resembling C) to make it easier to analyse files.

![IDA Screenshot]({{ "/assets/img/1/ida_screenshot.png" | relative_url }})

In my opinion, IDA has one of the best interfaces for viewing the control flow of a program. It has a useful graph view that allows you to see where certain subroutines jump to and easily find cross-references. Unfortunately, the free version of IDA doesn't come with a decompiler or debugger, and plugins may not work as expected. With licenses upwards of 500 USD for IDA Pro, it just doesn't seem like a viable option for me as a student.

You can download IDA Freeware here: <https://www.hex-rays.com/products/ida/support/download_freeware/>

## Ghidra

Ghidra is an open source disassembler released by the NSA (National Security Agency). The source code is available on the [NSA's GitHub](https://github.com/NationalSecurityAgency/ghidra) which means you can follow issues, updates, and download new versions as soon as they're released. It comes with its own decompiler, and although it may not be as good as IDA Pro's, it is able to handle most things thrown at it.

![Ghidra Screenshot]({{ "/assets/img/1/ghidra_screenshot.png" | relative_url }})

The interface is quite similar to IDA's, but it feels a lot more simplistic, which is nicer for beginners who don't want to be bombarded with a bunch of unknown words. Ghidra comes with a lot of the same features as IDA Pro, which is what makes it so appealing to people looking to get started in reverse engineering. Unfortunately, Ghidra does not come with a debugger and is unable to write patched bytes to a new file, however, the pros greatly outweigh the cons and I would highly recommend Ghidra to any newcomers.

You can download the latest version of Ghidra here: <https://ghidra-sre.org/>

## Radare2

Radare2 is an open source, command-line disassembler and debugger. It supports multiple different architectures and allows scripting in several different languages by making use of its internal API.

![Radare Screenshot]({{ "/assets/img/1/radare_screenshot.png" | relative_url }})

Since radare2 has a command line interface, it can be quite difficult for beginners to understand what's happening. It has hundreds of different commands which can be hard to remember (fortunately, it has some useful help menus). Radare2 has a steep learning curve, but is very powerful once you've mastered the commands and know what you're doing. I've been playing about with this one for a while and can definitely see the appeal, but I just don't know if I have the time to learn radare2 and use it to its full potential. It will come in handy when debugging later on, but for now I'll stick to using Ghidra and IDA for disassembling.

You can download Radare2 here: <https://github.com/radareorg/radare2>

## Debuggers

I haven't started doing any dynamic analysis yet, so I haven't had the chance to use a debugger. However, there are two main debuggers (excluding the ones included in IDA Pro and Radare2) that I will use when I get round to it: GDB and Ollydbg.

GDB is a command line debugger (and disassembler) and requires you to learn the commands like Radare2. I haven't used it before, but I've seen it being used in [LiveOverflow's binary exploitation playlist](https://www.youtube.com/playlist?list=PLhixgUqwRTjxglIswKp9mpkfPNfHkzyeN) and it looks to be pretty powerful.

Ollydbg is exclusively a Windows debugger and has a decent user interface. I have used it a little in the past, but haven't really done any real debugging with it.

You can download GDB here: <https://www.gnu.org/software/gdb/download/>

You can download Ollydbg here: <http://www.ollydbg.de/>

## Practice

I only have a very basic understanding of x86 assembly and little experience with reverse engineering, so I started reading [Practical Reverse Engineering](https://www.amazon.com/dp/1118787315) and following the exercises in the book. I'll start by reversing simple programs and stepping through them one instruction at a time until I fully understand what's happening. Once I have a better foundational understanding, then I'll start moving onto CTF (capture the flag) challenges and crackmes. Reversing CTF challenges generally consist of a program (or multiple) with hidden flags in them. The flags are generally strings in the format `flag{message_here}` and are easily noticeable. Your job is to collect these flags and submit them for points.

A crackme is a program that is designed to be reverse engineered and cracked. They usually ask for a password or code on startup and will print out the flag or secret message. It is your job to try and bypass this check or generate a password/code that works.

There are plenty CTF sites out there such as [HackTheBox](https://hackthebox.eu) and [TryHackMe](https://tryhackme.com), however HackTheBox requires you to complete a small challenge to access the site. You can find crackmes at <https://crackmes.one>, make sure to run them in a virtual machine unless you trust the file!

## First Crackme

To get started, I chose an easy crackme called [easyAF](https://crackmes.one/crackme/5eae2d6633c5d47611746500). I downloaded the file and unzipped it to get the binary file and ran `file easyAF` to get the file type:

```
easyAF: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=c1f484166af165c71e71e8e0e5ebd8f9d6300f0a, for GNU/Linux 3.2.0, not stripped
```

This shows that the file is a 64-bit ELF executable using x86-64 architecture and is to be executed on a Linux system. The first thing I done was boot up my Linux VM (virtual machine) and execute the file to see what kind of inputs and outputs it gives.

![File Output]({{ "/assets/img/1/run_file.png" | relative_url }})

So the program is looking for a password. A simple way to do beat this would be to run `strings` on the program, read any text stored in the binary and guess which one is the password. However, I want to practice my reversing skills, so I opened it up in IDA and tried to figure out what was happening.

![IDA Graph View]({{ "/assets/img/1/ida_flow.png" | relative_url }})

IDA's graph view made it very simple to follow the control flow of the program. There are two strings that stand out to me at the top `"Enter the password: "` and `"pass"`. It's very likely that `pass` is the password, but I decided to follow the instructions step-by-step to get a better understanding of how these strings were being compared.

```
var_60 = byte ptr -60h
var_40 = byte ptr -40h

lea     rax, [rbp+var_60]
mov     rdi, rax
call    basic_string
lea     rax, [rbp+var_60]
lea     rsi, aPass      ; "pass"
mov     rdi, rax
call    assign
lea     rax, [rbp+var_40]
mov     rdi, rax
call    basic_string
lea     rsi, aEnterThePasswo ; "Enter the password: "
lea     rdi, cout
call    basic_ostream
```

From my understanding, this chunk of code is creating space on the stack at `rbp-60h` and storing the string `pass`. It then creates space at `rbp-40h` and prints `"Enter the password: "` out to the screen using `cout`. This part of the code is just initialising variables and printing to the console, but the next bit of code is where the main logic behind the program starts.

```
lea     rax, [rbp+var_40]
mov     rsi, rax
lea     rdi, cin
call    basic_istream
lea     rdx, [rbp+var_60]
lea     rax, [rbp+var_40]
mov     rsi, rdx
mov     rdi, rax
call    compare
test    al, al
jz      short loc_129B
lea     rsi, aWelldone  ; "Welldone!"
lea     rdi, cout
call    basic_ostream
```

Now the program is getting user input using `cin` and storing it on the stack at `rbp-40h` which was previously initialised. It stores pointers to `"pass"` and the user input and stores them in `rsi` and `rdi` respectively which are sent to a function that compares the strings. Upon inspecting the compare function further, it returns 1 if both strings are equal, otherwise it will return 0.

![Compare Function]({{ "/assets/img/1/compare.png" | relative_url }})

When the function returns a value, it stores it in `rdx`. However, in this case we are going to have a return value of 0 or 1, so we use `al` which is the last 8 bits of `rdx`. In the next line, we are doing a TEST on `al`, this just performs a bitwise AND and sets the ZF (zero flag) if the result is 0. So if the strings are not equal, then ZF will be set to 1. To finish it off we have a `jz` which will jump to the instructions that display `"Nope"` if the ZF is set, otherwise it will continue with execution as normal and display `"Welldone!"` to the screen.

As predicted before, `"pass"` is the password for the program, but going through this program step-by-step gave me a better understanding of how it worked.