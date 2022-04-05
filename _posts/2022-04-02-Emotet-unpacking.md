---
title: "Emotet Analysis : unpacking part 1"
author: Player-V
date: 2022-04-02 02:26:00 -0500
categories: [Reverse engeneering, Malware Analysis]
tags: [unpacking, analysis]
---


## Introduction

That's will be my first post in the blog, i will make a series of post about Emotet malware, we're going to digg deep in emotet. The first part is about unpacking the malware fire up your virtual machine and let's start.

## Triage

Opening the sample in CFF explorer shows that we're dealing with 32 bit binary.

![CFF-Explorer1](https://pl-v.github.io/plv//assets/Emotet-part1/Hex-View/1.PNG){: width="700" height="400" }

Let's check import section.

![CFF-Explorer3](https://pl-v.github.io/plv//assets/Emotet-part1/Hex-View/3.PNG){: width="700" height="600" }

There is just one imported library which is Kernel32 that's the first sign which indicate that the binary could be packed, another indicator the two API functions:

1. [VirtualAlloc](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc) 
2. [VirtualProtect](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualprotect)

The VirtualAlloc function allocate memory and the VirtualProtect function changes the protection on a region of committed pages in the virtual address space, most of time those two functions are used by malware during the unpacking process. Let's open the binary on Die(Detect-It-Easy) to make sure  that the malware is packed.

![Die1](https://pl-v.github.io/plv//assets/Emotet-part1/Die/1.PNG){: width="700" height="300" }

The status bar says that it's 91% packed and we have a high entropy in the `.text` section, so that's a strong indication that the malware is packed and should be unpacked for further analysis.


## IDA 

Now that we're sure that our sample is packed, let's open it in IDA and find the function which is responsible for unpacking. 

![IDA1](https://pl-v.github.io/plv//assets/Emotet-part1/IDA/1.PNG){: width="700" height="300" }

Click on `Imports` to reveal all the functions used by the binary.

![IDA2](https://pl-v.github.io/plv//assets/Emotet-part1/IDA/2.PNG){: width="700" height="300" }

Search for VirtualAlloc and double click on it.

![IDA3](https://pl-v.github.io/plv//assets/Emotet-part1/IDA/3.PNG){: width="700" height="300" }

VirtualAlloc function is used two times by the same function `sub_1001AFF0`, double click on `sub_1001AFF0` and scroll down notice that the first function called after VirtualAlloc is `sub_10022C40`, so maybe we've found our unpacking function. to make sure let's open it on `Xdbg` and figure out. 

## Unpacking

Open your `X32dbg` and paste the sample there for debugging.

![XDBG1](https://pl-v.github.io/plv//assets/Emotet-part1/Xdbg/1.PNG){: width="700" height="300" }

Make a breakpoint on VirtualAlloc and hit run.

![XDBG2](https://pl-v.github.io/plv//assets/Emotet-part1/Xdbg/2.PNG){: width="700" height="300" }

Xdbg will keep running untill it hit the breakpoint, then click two times on `Execute till run`. 

![XDBG3](https://pl-v.github.io/plv//assets/Emotet-part1/Xdbg/3.PNG){: width="700" height="300" }

Notice that the EAX register contain the address of the allocated memory, right click on that value and click on `Follow in Dump`.

![XDBG4](https://pl-v.github.io/plv//assets/Emotet-part1/Xdbg/4.PNG){: width="700" height="300" }
