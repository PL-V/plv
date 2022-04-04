---
title: "Emotet Analysis : unpacking part 1"
author: Player-V
date: 2022-04-02 02:26:00 -0500
categories: [Reverse engeneering, Malware Analysis]
tags: [unpacking, analysis]
---


## Introduction

That's will be my first post in the blog, i will make a series of post about Emotet malware, we're going to digg deep in emotet. The first part is about unpacking the malware fire up your virtual machine and let's start.

## Unpacking 

Opening the sample in CFF explorer shows that it's a 32 bit binary

![CFF-Explorer1](/assets/Emotet-Part1/Hex-View/1.png){: width="700" height="400" }

It contains 4 sections

![CFF-Explorer1](/assets/Emotet-Part1/Hex-View/2.png){: width="700" height="400" }

Let's check import section

![CFF-Explorer1](/assets/Emotet-part1/Hex-View/3.png){: width="700" height="400" }

There is  