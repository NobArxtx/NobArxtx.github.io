---
layout: default
permalink: /RE102/section5.1/
title: Setup
---
[Go Back to Reverse Engineering Malware 102](https://nobarxtx.github.io/RE102/)

# Section 5.1: Debugging #

Debugging should be your last resort, as it can be time consuming. However, for the sake of teaching, I will go over it anyway. Be sure to take a snapshot of your VM before you begin debugging. This snapshot will come in handy when you accidently run the malware sample. 

**Tip:** Take frequent snapshots to save your debugging work, this way you won’t lose your place.

## Create the Breakpoints ##

Open decrypted_shellcode.exe in x32dbg.

At this point you should have recorded the following functions along with their corresponding locations:

1. `sub_402B1C` @ `00401D9B` - The function the loads the libraries
2. `sub_4014AA` @ `0040560B` - The function that checks for sample, sandbox, and virus
3. `jz loc_405272` @ `004019E4` - The jump to modify the EFlags if necessary
4. `jnz loc_405277` @ `0040526C` - The jump to modify the EFlags if necessary
5. `loc_405272` @ `00405272` - The location that calls the unknown API
6. `loc_405277` @ `00405277` - The location that calls the unknown API

Place a breakpoint with x32dbg using the command line. Example: `bp 00401D9B`

![alt text](https://nobarxtx.github.io/RE102/images/Section5.1_breakpoint.png "Section5.1_breakpoint")

Now press **F9** to run the program to breakpoints until you reach `004019E4`.

![alt text](https://nobarxtx.github.io/RE102/images/Section5.1_4019E4.png "Section5.1_4019E4")

Scroll down to check out offset `00405272`. Looks like the `[esp+1C]` is using Path Windows APIs to check the strings against sample, sandbox, and virus. Since your exe name is and path does not contain these words, it will not take the jump. Thus, no need  to change the flags or patch the instruction. Keep pressing **F8** (to step over the instruction) until you reach the offset `00405277`.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE102/images/Section5.1_405272.png "Section5.1_405272")](https://nobarxtx.github.io/RE102/images/Section5.1_405272.png)

**Congrats!** You bypassed the first evasion technique deployed by this malware . Now that you know what these API calls are, you should be renaming the subroutines in your IDA with appropriate labels.

---

## Adding Resources ##

Step **F7** the program until you reach the next function `sub_40487D`. Be sure to record the arguments pushed onto the stack. Step Into **F7** function `sub_40487D`. Next step until you reach `00401632` and look down to `00401645`. The calls to GetModuleHandle and FindResource indicate that the malware is  about to access a resource.

This is typically how you get a resource from an exe:

```
HMODULE hModule = GetModuleHandle(NULL); // get the handle to self (exe)
HRSRC hResource = FindResource(hModule, MAKEINTRESOURCE(RESOURCE_ID), RESOURCE_TYPE); 
HGLOBAL hMemory = LoadResource(hModule, hResource);
DWORD dwSize = SizeofResource(hModule, hResource);
LPVOID lpAddress = LockResource(hMemory);
```

When you turned the shellcode into an exe it did not include any resources. Remember that the original exe is where this shellcode gets executed. So, you will need to get the resource from the original exe and import them into shellcode exe. The argument passed to the function `sub_40487D` was 0xE38 which is 1000 in decimal. If you keep stepping through function `sub_40487D` you will see the routine above, and notice that the argument to find the resource is 1000.

```
HRSRC WINAPI FindResource(
  _In_opt_ HMODULE hModule,
  _In_     LPCTSTR lpName, //ID of the resource
  _In_     LPCTSTR lpType
);
```

![alt text](https://nobarxtx.github.io/RE102/images/Section5.1_LoadResource.png "Section5.1_LoadResource")

Close x32dbg while you edit the decrypted_shellcode.exe.

Open up the original exe in CFF explorer and look for the resource 1000. Next export this resource under Resource Editor, right-click and Save Resource (RAW). Take a moment and look at the data of this resource. Hint: looks like more junk data. 

![alt text](https://nobarxtx.github.io/RE102/images/Section5.1_resource.png "Section5.1_resource")

Once you exported the resource 1000, open the decrypted_shellcode.exe with CFF explorer. In the Resource Editor add Add Custom Resource (Raw) with the id of 1000. It should mirror the original exe. Afterwards open decrypted_shellcode.exe with x32dbg again. Navigate back to function `sub_40487D` or just set a breakpoint at `0040487D` and run until that function.

---

## Saving Junk and Chunks in Memory ##

Keep stepping until you reach `0040416F` where you will see that the resource is being placed into a new memory allocation. Remember that VirtualAlloc is typically followed by a ‘mov instruction’. After the VirtualAlloc function is returned make sure you note the address of the newly allocated memory. As before, this function will put the address of the allocated memory in the `eax` register (the returned value).

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE102/images/Section5.1_savingresource.png "Section5.1_savingresource")](https://nobarxtx.github.io/RE102/images/Section5.1_savingresource.png)

Once you are done stepping through function `sub_40487D` step until you reach `loc_4014C2`.

![alt text](https://nobarxtx.github.io/RE102/images/Section5.1_allocate318.png "Section5.1_allocate318")

The size 0x318 is a common theme for the next couple of function calls. This is where you will see another combo  of VirtualAlloc and `mov`.  It will store the first 0x318 bytes into the newly allocated memory.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE102/images/Section5.1_Virtualloc318.png "Section5.1_Virtualloc318")](https://nobarxtx.github.io/RE102/images/Section5.1_Virtualloc318.png)

![alt text](https://nobarxtx.github.io/RE102/images/Section5.1_move_decrypt.png "Section5.1_move_decrypt")

Does function `sub_403BC2` look familiar? Here is the breakdown:

```
Arg_0 CopiedData+offset 20h
Arg_1 0x2F8 size
Arg_2 CopiedData
Arg_3 0x20 size
```

Why offset 0x20? Here is the dump of the CopiedData:

![alt text](https://nobarxtx.github.io/RE102/images/Section5.1_20bytesof318.png "Section5.1_20bytesof318")

At this point it’s too early to guess what this data does.

```
78 95 4D 26 0A C4 55 94 74 AF 5A 78 33 71 58 EB CD 05 B3 D6 5A B7 D6 05 43 D8 1A 7D 4A B6 EA 10
```

In IDA, glance through function `sub_403BC2`. There are 3 hints that give away what this function is doing.

* The use of 0x100 and 0x20
* Multiple loops
* The use of XOR

If you remember from the previous Section 4, multiple loops and the use XOR is indicative of being some kind of crypto algorithm. There is a theme of crypto here, but there is just a slight difference. The use of anding a value with `800000FFh` is also a form of modulo for `X mod 256`. Earlier we saw that the modified RC4 algorithm was using a delphi mod function instead.

As you might have guessed, it looks like the malware is using RC4 again, but you might want to step through the algorithm to confirm if it’s the correct RC4 or the modified RC4 like from Section 4. Once you have, you will notice that the first 32 bytes (0x20) decrypted the rest of the CopiedData 760 bytes (0x2F8). Be sure to save the address of this memory in your notes, and rename the functions in IDA as you will need to go back to them for Section 6.


Step through until you reach `loc_401CCA` and continue to the next page.

[Section 5 <- Back](https://nobarxtx.github.io/RE102/section5) | [Next -> Section 5.2](https://nobarxtx.github.io/RE102/section5.2)
