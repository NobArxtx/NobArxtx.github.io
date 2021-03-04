---
layout: default
permalink: /RE102/section4.3/
title: Setup
---
[Go Back to Reverse Engineering Malware 102](https://nobarxtx.github.io/RE102/)

# Section 4.3: Convert the Shellcode Into an Exe #

Now you have the decrypted shellcode. This section will go over converting the shellcode into an executable so that you can view the disassembly in IDA. 

Keep in mind that you don’t need to do this step, but converting it into an executable will help you in debugging and understanding the deployed VM evasion techniques which are heavily discussed in Section 5. You can open the decrypted_shellcode.bin in IDA and notice that the disassembly is not parsing functions properly. The malware author has inserted extraneous assembly instructions to through off malware analysis. 0x78 means assembly instruction `js` or `Jump short if sign (SF=1)`.

```
00000000: 7878 7878 7878 7878 7878 7878 7878 7878  xxxxxxxxxxxxxxxx
00000010: 8b45 088b 4034 55e9 6761 0000 786a 6866  .E..@4U.ga..xjhf
00000020: 8995 34ff ffff 5a6a 7266 8995 36ff ffff  ..4...Zjrf..6...
00000030: 5ae9 2506 0000 7878 7878 7878 7878 7878  Z.%...xxxxxxxxxx
00000040: 7878 7878 7878 7858 6a74 6689 45c8 58e9  xxxxxxxXjtf.E.X.
00000050: 0229 0000 7878 7878 7878 7878 7878 7878  .)..xxxxxxxxxxxx
00000060: 7878 7878 7878 7889 5d88 e8af 3b00 0081  xxxxxxx.]...;...
...
```

There are many tools and scripts available that help you convert shellcode into an exe like [shellcode2exe.py](https://github.com/securedorg/shellcode_tools/blob/master/shellcode2exe.py). However I have found that Hexacon provided a nice easy [tutorial for converting shellcode into an executable](http://www.hexacorn.com/blog/2015/12/10/converting-shellcode-to-portable-executable-32-and-64-bit/). This section will be using this YASM and GoLink to create the executable while using CFF explorer to edit the binary header. 

1. Download Yasm
[yasm-1.3.0-win32.exe](http://www.tortall.net/projects/yasm/releases/yasm-1.3.0-win32.exe)
2. Extract **yasm-1.3.0-win32.exe** and rename it to **yasm.exe**
3. Download GoLink linker
[Golink.zip](http://www.godevtool.com/Golink.zip)
4. Extract golink.exe
5. Create a **decrypted_shellcode.asm** file with the following instructions
```
Global Start 
SECTION 'AyyLmao' write, execute,read 
Start:       
incbin "decrypted_shellcode.bin"  
```
6. From a command line run the following command to assemble the code:
```
yasm.exe -f win32 -o decrypted_shellcode.obj decrypted_shellcode.asm
```
7. Now run the linker
```
golink /ni /entry Start decrypted_shellcode.obj
```
8. Open shellcode.exe with CFF explorer and open the **NT Headers->Optional Headers->AddressOfEntryPoint**. Add the current value to 0x42B7 which was the offset of where the malware was going to return to in function `sub_45B794`. AddressOfEntryPoint should be `000052B7`. This will ensure that IDA knows where to start the disassembly.

Finally, open the decrypted_shellcode.exe into IDA for Section 5.

[Section 4.2 <- Back](https://nobarxtx.github.io/RE102/section4.2) | [Next -> Section 5](https://nobarxtx.github.io/RE102/section5)
