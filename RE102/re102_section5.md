---
layout: default
permalink: /RE102/section5/
title: Setup
---
[Go Back to Reverse Engineering Malware 102](https://nobarxtx.github.io/RE102/)

# Section 5: Evasion Techniques #

![alt text](https://nobarxtx.github.io/RE102/images/Section5_intro.gif "intro")

This section will focus on identifying various evasion techniques as well as working around them during the debugging phase. Now that you will be working with an new executable, you will need to create another road map.

## Control Flow Obfuscation ##

You will notice that the shellcode is broken up into extraneous and unnecessary jumps. This is meant to throw off malware analysts. Malware that has this kind of useless instructions is usually processed with some kind of obfuscation kit (e.g., cryptors). Malware authors rarely write new shellcode and will sell, share, or reuse this code.

Going forward, you should be viewing the disassembly in graph mode, as it makes it easier to understand  the control flow. Below is an example of the flow-chart mode of these jumps.

![alt text](https://nobarxtx.github.io/RE102/images/Section5_ControlFlowObfuscation.png "Section5_ControlFlowObfuscation")

## Where to Start? ##

There are no strings for us to investigate and there are no functions parsed by IDA.. So, you need to start exploring each function one by one finding an interesting piece of code to analyze. This comes with experience. If this seems too daunting, then manual debugging is your next option. The goal is to make a road map of shellcode by working backwards.

## String Obfuscation ##

The first function call `sub_404C1E` doesn’t look like something interesting, so move on to the next function call to `sub_402B1C`. This function is a jump-wrapper for the function `sub_4059A3`.

Notice anything strange about the immediate values being placed onto the stack? These are actually strings. Breaking up strings and  pushing them onto the stack is a common of hiding strings from malware analysts. Go ahead right-click these numbers and convert it to a string (R).

![alt text](https://nobarxtx.github.io/RE102/images/Section5_FunkyStrings.png "Section5_FunkyStrings")

They should look like this afterwards:

![alt text](https://nobarxtx.github.io/RE102/images/Section5_PostStrings.png "Section5_PostStrings")

## Dynamic Library Loading ##

With shellcode or position independent code (PIC), the code needs to load resources and libraries to work with before it performs the nefarious routines. Based on the strings above you can tell that it is going to load these libraries:

* user32
* shell32
* shlwapi
* advapi32

## Access to the Process Environment Block (PEB) ##

After the advapi32 string gets loaded onto the stack, enter the function `sub_405421`.  
This function is accessing the FS segment register at offset 30. This register is commonly used by OS kernels to access thread-specific memory. This specific offset (i.e., `fs:[0x30]`)  points to the Process Environment Block. This is a common shellcode tactic to get handles to loaded windows libraries a.k.a. Modules, specifically the base address of kernel32 from the PEB.

```    
mov     eax, 30h
mov     eax, fs:[eax] ; Get the address of PEB
mov     eax, [eax+0Ch] ; Get the address of PEB_LDR_DATA
mov     eax, [eax+0Ch] ; InLoadOrderModuleList
mov     eax, [eax] ; get the next entry
mov     eax, [eax+18h] ; get Kernel32
```

![alt text](https://nobarxtx.github.io/RE102/images/Section5_PEB.gif "Section5_PEB")

The second instruction `mov eax, [eax+0Ch]` gets the address of the PEB Loader Data from the [PEB](https://msdn.microsoft.com/en-us/library/windows/desktop/aa813706%28v=vs.85%29.aspx) struct. The [PEB_LDR_DATA](https://msdn.microsoft.com/en-us/library/windows/desktop/aa813708(v=vs.85).aspx) contains the struct for the InMemoryOrderModuleList which is where it gets the pointer for Kernel32. **Note:** there are many great shellcode resources available that explain this technique. I just want you to recognize the instruction `fs:[0x30]`.

```
struct PEB_LDR_DATA {
    DWORD Length;                                       ; 0
    BYTE Initialized;                                   ; 4
    void* SsHandle;                                     ; 8
    struct LIST_ENTRY InLoadOrderModuleList;            ; 0ch
    struct LIST_ENTRY InMemoryOrderModuleList;          ; 14h
    struct LIST_ENTRY InInitializationOrderModuleList;  ; 1ch
};
```

Save these functions `sub_402B1C` and `sub_405421` for debugging later. Also include these into your road map for the shellcode executable.

## Checking the Filename and Path ##

Go to the next function `sub_4014AA` which is a wrapper for function `sub_401D36`. Again, this function is using an anti-analysis technique of pushing a string one by one onto the stack. Can you guess what this function is doing?

![alt text](https://nobarxtx.github.io/RE102/images/Section5_NameCheck.png "Section5_NameCheck")

The strings are:

* sample
* sandbox
* virus

It seems the malware author wanted to detect if this executable contained strings related to malware analysis. You will need to debug this function to see which string it’s comparing these values. You will want to avoid this function because you need to get around the anti-analysis detection. Remember that functions return 0 or 1 in `eax` depending on the success or failure. You want this function to fail or return 1 because you want to get around these traps. Below the instruction `cmp eax, 1` and `jz loc_405272` is where the comparison to the return value occurs. During debugging, you would want to force the jump by changing the EFlags.

![alt text](https://nobarxtx.github.io/RE102/images/Section5_checkname.png "Section5_checkname")

## Time to Start Debugging ##

After `jz loc_405272` there is a call to [esp+1Ch] this is actually calling a Windows API call that was loaded there by the loaded library function `sub_402B1C`. It would be tedious to go through those locations by hand, so let’s start debugging.

![alt text](https://nobarxtx.github.io/RE102/images/Section5_startdebugging.png "Section5_startdebugging")

The next page will go over debugging the decrypted_shellcode.exe with x32dbg.

[Section 4.3 <- Back](https://nobarxtx.github.io/RE102/section4.3) | [Next -> Section 5.1](https://nobarxtx.github.io/RE102/section5.1)
