---
layout: default
permalink: /RE101/section1.2/
title: Fundamentals
---
[Go Back to Reverse Engineering Malware 101](https://nobarxtx.github.io/RE101/)

# Section 1.2: Fundamentals #

## Anatomy of a Windows PE C program ##

Typical windows programs are in the Portable Executable (PE) Format. It’s portable because it contains information, resources, and references to dynamic-linked libraries (DLL) that allows windows to load and execute the machine code. 

![alt text](https://nobarxtx.github.io/RE101/images/Cprogram.gif "C Program")

---

## Windows Architecture ##

In this workshop we will be focusing on user-mode applications.

### User-mode vs. Kernel Mode [[1]][1] ###

- In user-mode, an application starts a user-mode process which comes with its own private virtual address space and handle table

- In kernel mode, applications share virtual address space.

[1]: https://msdn.microsoft.com/en-us/windows/hardware/drivers/gettingstarted/user-mode-and-kernel-mode?f=255&MSPPError=-2147217396

This diagram shows the relationship of application components for user-mode and kernel-mode.
![alt text](https://nobarxtx.github.io/RE101/images/WindowsArch.png "Windows Architecture")

--- 

## PE Header ##

The PE header provides information to operating system on how to map the file into memory.
The executable code has designated regions that require a different memory protection (RWX)
- Read
- Write
- Execute

This diagram shows how this header is broken up.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE101/images/PE32.png "PE 32 Header")](https://nobarxtx.github.io/RE101/images/PE32.png)

Here is a hexcode dump of a PE header we will be working with.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE101/images/PEHeader.gif "PE 32 Header Animated")](https://nobarxtx.github.io/RE101/images/PEHeader.gif)

---

## Memory Layout ##

- **Stack** - region of memory is added or removed using “last-in-first-out” (LIFO) procedure [[2]][2]
- **Heap** - region for dynamic memory allocation [[3]][3]
- **Program Image** - The PE executable code placed into memory
- **DLLs** - Loaded DLL images that are referenced by the PE
- **TEB** - Thread Environment Block stores information about the current running thread(s) [[4]][4]
- **PEB** - Process Environment Block stores information about loaded modules and processes. [[5]][5]

[2]: https://en.wikipedia.org/wiki/Stack_(abstract_data_type)
[3]: https://en.wikipedia.org/wiki/Heap_(data_structure)
[4]: https://en.wikipedia.org/wiki/Win32_Thread_Information_Block
[5]: https://en.wikipedia.org/wiki/Process_Environment_Block

This diagram illustrates how the PE is placed into memory.
![alt text](https://nobarxtx.github.io/RE101/images/Memory.png "PE Memory Layout")

--- 

## The Stack ##

- Data is either pushed onto or popped off of the stack data structure
- **EBP** - Base Pointer is the register that used to store the references in the stack frame

This diagram represents a typical stack frame.
![alt text](https://nobarxtx.github.io/RE101/images/TheStackFrame.png "Stack Frame")

[Environment Setup <- Back](https://nobarxtx.github.io/RE101/section1) | [Next -> x86 Assembly](https://nobarxtx.github.io/RE101/section1.3)
