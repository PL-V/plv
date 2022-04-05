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

Let's check import section

![CFF-Explorer3](https://pl-v.github.io/plv//assets/Emotet-part1/Hex-View/3.PNG){: width="700" height="600" }

There is just one imported library which is Kernel32 that's the first sign which indicate that the binary could be packed, another indicator the two API functions:

1. [VirtualAlloc](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc) 
2. [VirtualProtect](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualprotect)

The VirtualAlloc function allocate memory and the VirtualProtect function changes the protection on a region of committed pages in the virtual address space, most of time those two functions are used by malware during the unpacking process. Let's open the binary on Die(Detect-It-Easy) to make sure  that the malware is packed.

![Die1](https://pl-v.github.io/plv//assets/Emotet-part1/Die/1.PNG){: width="700" height="300" }

The status bar says that it's 91% packed and we have a high entropy in the `.text` section, so that's a strong indication that the malware is packed and should be unpacked for further analysis.


## Unpacking
