---
layout: default
permalink: /RE102/section6/
title: Setup
---
[Go Back to Reverse Engineering Malware 102](https://nobarxtx.github.io/RE102/)

# Section 6: Identifying Packing #

![alt text](https://nobarxtx.github.io/RE102/images/Section7_intro.gif "intro")

This section will focus on identifying a custom packing routine. Believe it or not this whole shellcode executable is a packer itself. The next several functions will reveal its algorithm, and you will be able to create a simple unpacking script.

## The Bat and Vbs Scripts ##

Before you actually get to the unpacking routine, navigate your way to `loc_4050A0`. There is a function call you might miss. When you are debugging the jump instruction `jz loc_40196B` at `004050A0` will jump over `sub_405463`.  If you want to debug this function just modify the jump here.

![alt text](https://nobarxtx.github.io/RE102/images/Section6_script.png "Section6_script")

Here is a summary of  `sub_405463`:

1. This function allocates memory to store the current filename and %APPDATA% location to determine if the executable already exists there. The giveaways are: 
* VirtualAlloc
* SHGetFolderPath
* GetModuleHandleA
* GetModuleFileNameW
* PathRemoveFileSpec
2. It will then try to create a process from the file stored in %APPDATA%, by calling CreateProcess 
3. Create a .bat file in %APPDATA% where the contents are pushed onto the stack. This file contains the following:
```
start /d "C:\Users\victim\AppData\<exe filename>" 
```

4. Where it will write the hidden .vbs script in the location:
```
C:\\Users\\victim\\AppData\\Roaming\\Microsoft\\Windows\\Start Menu\\Programs\\Startup\\<filename>.vbs
```

This vbs script contains the following:
```
Set WshShell = CreateObject("WScript.Shell")
WshShell.Run chr(34) & "C:\\Users\\victim\\AppData\\Roaming\\<filename>.bat" & Chr(34), 0
Set WshShell = Nothing
```
To see the bat and vbs script get created, force these jump locations to not take the jump branch! This can be done like before, by simply changing the zero flag.
* `00403089`
* `00404652`
* `004048A7`
* `004048B0`
* `00403349`
* `0040507A`

## The Unpacker ##

In IDA, after the call to `sub_405463`, all paths lead to `loc_4057BC`. In the debugger, set a breakpoint at `004057BC` and run to this location.

![alt text](https://nobarxtx.github.io/RE102/images/Section6_paths.png "Section6_paths")

The next routine should look familiar to you. There are multiple values being pushed to the stack before the call to `sub_40651A`.

![alt text](https://nobarxtx.github.io/RE102/images/Section6_compress.png "Section6_compress")

1. The first pushed value is `[esi+60]` which is the location where the first 0x318 bytes of Resource 1000 was decrypted.
2. The second value is 0x1.
3. The third pushed value is one dword at the relative offset 0x64 of that 0x318 bytes.
4. The fourth pushed value is one dword at the relative offset 0xA8 of that 0x318 bytes.
5. The fifth pushed value is the original resource stored in memory.

![alt text](https://nobarxtx.github.io/RE102/images/Section6_offsets.png "Section6_offsets")

The values for 0x0A (10 decimal) and 0x21 (33 decimal) will become important within function `sub_40651A`. Step into **F7** function `sub_40651A`. The first part of the function allocates some memory where it will store the output of the next routine. In the debugger, step over **F8** the VirtualAlloc call and dump the memory location that it returns so that you can monitor the changes.

![alt text](https://nobarxtx.github.io/RE102/images/Section6_VirtualAlloc.png "Section6_VirtualAlloc")

In the debugger, step **F7** through this loop and keep track how values 10 and 33 are used against the resource bytes.

![alt text](https://nobarxtx.github.io/RE102/images/Section6_looping.png "Section6_looping")

The 2 dumps below shows what this routine is actually doing: compression. After the initial byte 0x1, it is removing every 10 bytes, displayed as 0xFF below. The routine will then store the next 33 bytes.

![alt text](https://nobarxtx.github.io/RE102/images/Section6_compressroutine.gif "Section6_compressroutine")

Below is an example of what the first loop through the data looks like. All 10 instances of 0xFF were removed.

![alt text](https://nobarxtx.github.io/RE102/images/Section6_output.png "Section6_output")

After you run through the whole function it will return this new compressed code for the next function call. Be sure to dump this section of memory as a .bin file and name it *compressed.bin*. You should have correctly renamed the RC4 function from earlier in IDA. After function `sub_40651A`, there should be a call to the RC4 decrypt function at `00407165`.

![alt text](https://nobarxtx.github.io/RE102/images/Section6_RC4Decrypt.png "Section6_RC4Decrypt")

If you remember from section 5.1, the key size was 0x20. For this call to RC4Decrypt, the key size is 0x40h at offset 0x2D0 of the decrypted 0x318 bytes. Below is the RC4 key:

![alt text](https://nobarxtx.github.io/RE102/images/Section6_40bytes.png "Section6_40bytes")

```
6F 49 04 00 35 06 03 00 63 49 03 00 89 10 04 00 A2 6C 03 00 F4 D1 02 00 59 88 03 00 25 D4 03 00 74 EF 03 00 0B 6C 03 00 A8 95 03 00 E0 EC 02 00 75 52 04 00 2B FB 02 00 22 C4 03 00 B5 FF 02 00
```

Export this key as a binary file and use the decrypt_shellcode.py script against the compressed.bin and the key.bin.

```
c:\Python27\python.exe <location>\decrypt_shellcode.py  <location>0x40key.bin  <location>\compressed.bin
```
In the debugger, you can step over **F8** the RC4Decrypt function and watch the compressed code change to the output below:

![alt text](https://nobarxtx.github.io/RE102/images/Section6_decrypted.png "Section6_decrypted")

Notice that the output looks like the header of a PE executable. The only difference is that it is missing the MZ header. If you scroll down after the RC4Decrypt function you will see the immediate value 0x544D which is MZ. This is where it will add the MZ header.

![alt text](https://nobarxtx.github.io/RE102/images/Section6_addingMZ.png "Section6_addingMZ")

Step through the rest until you reach a call to `sub_4031A9` at `00404C81`. You will find that it uses CreateProcess to spawn a new process of the newly created PE without dropping it to disk. After you step over the call to CreateProcess, you can open Process Explorer to view the newly created child process.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE102/images/Section6_createprocess.png "Section6_createprocess")](https://nobarxtx.github.io/RE102/images/Section6_createprocess.png)

Now that you know the algorithm, you can create a unpacking script for the resource. The next page will go over the script.

[Section 5.2 <- Back](https://nobarxtx.github.io/RE102/section5.2) | [Next -> Section 6.1](https://nobarxtx.github.io/RE102/section6.1)
