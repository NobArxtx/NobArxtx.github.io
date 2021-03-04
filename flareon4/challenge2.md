---
layout: default
permalink: /flareon4/challenge2/
title: Challenge 2 IgniteMe.exe
---

[Go Back to All Challenges](https://nobarxtx.github.io/flareon4)

# Challenge 2: IgniteMe.exe #

It's always a good idea to triage the binary first:

1. What type of file is it?
2. Run it for shits and giggles


Alright, the **MZ** header is a good sign that this is an actual exe.

```
00000000: 4d5a 9000 0300 0000 0400 0000 ffff 0000  MZ..............
00000010: b800 0000 0000 0000 4000 0000 0000 0000  ........@.......
00000020: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000030: 0000 0000 0000 0000 0000 0000 c000 0000  ................
00000040: 0e1f ba0e 00b4 09cd 21b8 014c cd21 5468  ........!..L.!Th
00000050: 6973 2070 726f 6772 616d 2063 616e 6e6f  is program canno
00000060: 7420 6265 2072 756e 2069 6e20 444f 5320  t be run in DOS 
00000070: 6d6f 6465 2e0d 0d0a 2400 0000 0000 0000  mode....$.......
00000080: 15c3 a2c1 51a2 cc92 51a2 cc92 51a2 cc92  ....Q...Q...Q...
00000090: 8c5d 0792 52a2 cc92 51a2 cd92 55a2 cc92  .]..R...Q...U...
000000a0: 83f9 c893 50a2 cc92 83f9 ce93 50a2 cc92  ....P.......P...
000000b0: 5269 6368 51a2 cc92 0000 0000 0000 0000  RichQ...........
000000c0: 5045 0000 4c01 0300 b73f 5d59 0000 0000  PE..L....?]Y....
```

Output
![alt text](https://nobarxtx.github.io/flareon4/images/ch1_run.png "run it")


## Assumptions ##

1. It needs to take input.
2. It needs to store that input somewhere.
3. It does something with the stored input.
4. It needs to validate the input.


### Find where the input is stored ###

![alt text](https://nobarxtx.github.io/flareon4/images/input.png "input")

Your best bet is looking at the function call right after the **WriteFile** API call. In function offset `4010F0`, you see the call for **ReadFile** which will read the input from the console. It will then store the input at offset `403078`. Here is the instruction that stores one byte at a time (cl is the first byte of register ecx):

```mov byte_403078[eax], cl```

![alt text](https://nobarxtx.github.io/flareon4/images/registers.png "registers")

### Find where the input is used ###

If you remember from my RE101 and RE102 workshops, your best bet is to look for functions looping with the `xor` assembly instruction. In your disassembly, you will find 2 places where xor is used.
1. function at offset 401050 
2. function at offset 401000

In function `401050` you can see where the input is processed through an xor loop. The register **eax** is the input and **ecx** is the key. The key is hard coded somewhere.
![alt text](https://nobarxtx.github.io/flareon4/images/xorloop.png "xorloop")


In function 401000, is a hardcoded value `80070057`. This simple function converts the key to `00700004`. The following reads: xor the first 2 bytes of value 80070057 with itself, rotate left by 4, and shift right by 1 bit.

```
push ebp
mov ebp,esp
mov eax, 80070057
mov edx,eax
xor ax,dx 
rol eax,4
shr ax,1
pop ebp
```

### Find where is the input validated ###

In function 401050, after the input is xored with `0x04` it's checked against a string stored at offset `403000`
Because xoring is symetric, you can easily xor the hardcoded answer with the same key.

Here a simple python script:

```
bytes = "\x0D\x26\x49\x45\x2A\x17\x78\x44\x2B\x6C\x5D\x5E\x45\x12\x2F\x17\x2B\x44\x6F\x6E\x56\x09\x5F\x45\x47\x73\x26\x0A\x0D\x13\x17\x48\x42\x01\x40\x4D\x0C\x02\x69"
key = "\x04"

reversed_bytes = bytes[::-1]

answer = ""

for i in reversed_bytes:
        x = chr(ord(key) ^ ord(i))
        answer += x
        key = x

print answer[::-1]
```

[Challenge 1 <- Back](https://nobarxtx.github.io/flareon4/challenge1) | [Next -> Challenge 3](https://nobarxtx.github.io/flareon4/challenge3)


