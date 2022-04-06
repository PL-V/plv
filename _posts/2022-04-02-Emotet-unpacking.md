---
title: "Emotet Analysis Part 1: Unpacking"
author: Player-V
date: 2022-04-02 02:26:00 -0500
categories: [Reverse engeneering, Malware Analysis]
tags: [unpacking]
---

![CFF-Explorer3](https://pl-v.github.io/plv/assets/Emotet-part1/Emotet.jpg){: width="700" height="600" }

## Introduction

That's will be my first post in the blog, i will make a series of posts about [Emotet](https://www.malwarebytes.com/emotet).

[Emotet](https://www.malwarebytes.com/emotet) is a Trojan that is primarily spread through spam emails (malspam), we're going to digg deep in the anlysis of this Trojan, the first part is about unpacking the malware then we will try to analyse the different modules and techniques used by the malware to compromise a machine, so fire up your virtual machine and let's start.

## Triage

The first thing i always do before opening a sample in `IDA` or `Xdbg` is opening the binary first in a hex editor, in my case i will use [CFF Explorer](https://ntcore.com/?page_id=388), so opening the sample in CFF explorer shows that we're dealing with 32 bit binary.

![CFF-Explorer1](https://pl-v.github.io/plv/assets/Emotet-part1/Hex-View/1.PNG){: width="400" height="400" }

Let's check import section, the malware use only one library which is `Kernel32` that's the first sign which indicate that we're dealing with packed binary.

![CFF-Explorer3](https://pl-v.github.io/plv/assets/Emotet-part1/Hex-View/3.PNG){: width="700" height="1200" }

Two intersting API functions are used:

1. [VirtualAlloc](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc) 
2. [VirtualProtect](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualprotect)

The [VirtualAlloc](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc) function allocate memory while the [VirtualProtect](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualprotect) function changes the protection on a region of committed pages in the virtual address space, most of time those two functions are used by malware during the unpacking process. To make sure that our sample is packed Let's open the binary on [Die(Detect-It-Easy)](https://github.com/horsicq/Detect-It-Easy).

![Die1](https://pl-v.github.io/plv/assets/Emotet-part1/Die/1.PNG){: width="700" height="300" }

The status bar says that it's 91% packed and `.text` section has a high entropy, that's a strong indication that the malware is packed and we should unpack it for further analysis.


## IDA 

Now that we're sure that our sample is packed, let's open it in `IDA` and try to find the function which is responsible for unpacking. 

![IDA1](https://pl-v.github.io/plv/assets/Emotet-part1/IDA/1.PNG){: width="700" height="300" }

Click on `Imports` to reveal all the functions used by the binary.

![IDA2](https://pl-v.github.io/plv/assets/Emotet-part1/IDA/2.PNG){: width="700" height="300" }

Search for [VirtualAlloc](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc) and double click on it.

![IDA3](https://pl-v.github.io/plv/assets/Emotet-part1/IDA/3.PNG){: width="700" height="300" }

[VirtualAlloc](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc) function is used two times by the same function `sub_1001AFF0`, double click on `sub_1001AFF0` and scroll down we notice that the first function called after [VirtualAlloc](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc) is `sub_10022C40`, so maybe we've found our unpacking function. to make sure let's open it on `Xdbg` and figure out. 

## Unpacking

Open your `X32dbg` and paste and paste your sample to it.

![XDBG1](https://pl-v.github.io/plv/assets/Emotet-part1/Xdbg/1.PNG){: width="700" height="300" }

Place a breakpoint on [VirtualAlloc](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc) and hit run.

![XDBG2](https://pl-v.github.io/plv/assets/Emotet-part1/Xdbg/2.PNG){: width="700" height="300" }

`Xdbg` will keep running untill it hit the breakpoint, after  click two times on `Execute till run`. 

![XDBG3](https://pl-v.github.io/plv/assets/Emotet-part1/Xdbg/3.PNG){: width="700" height="300" }

Check the `EAX` register it contain the return adress address of the allocated memory by [VirtualAlloc](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc), right click on that value and click on `Follow in Dump`.

![XDBG4](https://pl-v.github.io/plv/assets/Emotet-part1/Xdbg/4.PNG){: width="700" height="300" }

As we said earlier that the function after [VirtualAlloc](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc) is responsible for unpacking, step over it and keep your eyes on the dump window at the bottom.

![XDBG4](https://pl-v.github.io/plv/assets/Emotet-part1/Xdbg/6.PNG){: width="700" height="300" }

After executing `sub_10022C40` function we can finally see our unpacked malware, dump it and save it somewhere in your machine. 

![XDBG4](https://pl-v.github.io/plv/assets/Emotet-part1/Xdbg/7.PNG){: width="700" height="300" }

Right click on dump windows and `Follow in memory map`.

![XDBG4](https://pl-v.github.io/plv/assets/Emotet-part1/Xdbg/8.PNG){: width="700" height="300" }

Another right click on the address of the unpacked binary then `Dump memory to file`.

![XDBG4](https://pl-v.github.io/plv/assets/Emotet-part1/Xdbg/9.PNG){: width="700" height="300" }

Now that we have our sample unpacked and ready for analysis let's open it in `X32dbg`. 

![XDBG4](https://pl-v.github.io/plv/assets/Emotet-part1/Xdbg/9.PNG){: width="700" height="300" }

It seems that our unpacked binary is missed and it should be fixed.
## Fixing

To fix the unpacked binary there are several methods to do that, we will use `LordPE` to automate the fixing, so all we should do is to open LordPe and click on options, then uncheck `Wipe Relocation` and `Rebuild ImportTable` options, finally click on normal then `OK`

![XDBG4](https://pl-v.github.io/plv/assets/Emotet-part1/Fixing/1.PNG){: width="700" height="300" }

Drag your unpacked sample to `LordPe` and it will be fixed automatically.

![XDBG4](https://pl-v.github.io/plv/assets/Emotet-part1/Fixing/2.PNG){: width="700" height="300" }

Finally open your fixed binary in `X32dbg` and notice that it's more readble right now.

![XDBG4](https://pl-v.github.io/plv/assets/Emotet-part1/Fixing/3.PNG){: width="700" height="300" }

## Reference

1. [New Emotet 11/2021 - Reverse Engineering VBA Obfuscation + Unpacking](https://www.reverse-engineer.net/view/courses/training/329541-new-emotet-11-2021-reverse-engineering-vba-obfuscation-unpacking-emotet)


2. [How to Unpack Malware with x64dbg](https://www.varonis.com/blog/x64dbg-unpack-malware)
