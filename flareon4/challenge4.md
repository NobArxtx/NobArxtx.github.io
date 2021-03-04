---
layout: default
permalink: /flareon4/challenge4/
title: Challenge 4 notepad.exe
---

[Go Back to All Challenges](https://nobarxtx.github.io/flareon4)

# Challenge 4: notepad.exe #

This challenge was really fun. Trying to disguise the binary as notepad.exe however the start function is pretty obvious that it's not notepad. The bulk of the code you need is actually right before it does normal notepad behavior.

## The Hint ##

The first part of the start function is the biggest hint. It is looking for the folder **%USERPROFILE%\flareon2016challenge**. This means you will need the binaries from the flareon 2016 challenges.

![alt text](https://nobarxtx.github.io/flareon4/images/ch4_hint.png "hint")

## Structure ##

![alt text](https://nobarxtx.github.io/flareon4/images/ch4_diagram.png "ch4 diagram")

The binary is actually performing typical malware-like behaviors that I see all the time:

* Using stack based strings
* Dynamically loading library modules
* Mapping files in memory

Your best bet is to start labeling the API calls in IDA so you know what is going on.

## The PE Header Fun ##

The next hint, is the message box that appears when you have the right exe files in the right order. 

![alt text](https://nobarxtx.github.io/flareon4/images/ch4_messagebox.png "messagebox")

In function offset `01014E20`, CreateFileMappingA and MapViewOfFile are used to place each file into memory in order to read a specific offset of the PE header.

![alt text](https://nobarxtx.github.io/flareon4/images/ch4_header.png "header")

Each header for the a few of the 2016 challenges are processed to contruct the key.bin file. This key is used to decrypt a hardcoded string. Function offset `010146C0` is where all of this takes place.

Hardcoded encrypted answer:

![alt text](https://nobarxtx.github.io/flareon4/images/ch4_answer.png "answer")

Here is the hex for the screenshot above:

```
37 E7 D8 BE 7A 53 30 25 BB 38 57 26 97 26 6F 50 F4 75 67 BF B0 EF A5 7A 65 AE AB 66 73 A0 A3 A1
```

As you can see there is a branch for each challenge:

![alt text](https://nobarxtx.github.io/flareon4/images/ch4_headercheck.png "header check")

These are the timestamps of the challenges:
```
57D1B2A2h       ; Challenge1.exe (Challenge1)
57D2B0F8h       ; Dudelocker.exe (Challenge2)
49180192h       ; kahki.exe (Challenge6)
579E9100h       ; unkown (Challenge3)
```

Each time notepad.exe is run it will check the timestamp value of itself against the next files mentioned above. All you need to do is change the timestamp of notepad.exe for every round. Use your favorite PE header editor like CFF Explorer and make the following modifications each time you run notepad.exe.

1. do not modify
2. modify 48025287 to 57D1B2A2
3. modify 57D1B2A2 to 57D2B0F8
4. modify 57D2B0F8 to 49180192
5. modify 49180192 to 579E9100

![alt text](https://nobarxtx.github.io/flareon4/images/ch4_cff.png "CFF Explorer")

Each time you run notepad.exe it will write a portion of the key to the key.bin file. The key.bin file should look like the following

```
00000000: 558b ec8b 4d0c 5657 8b55 0852 ff15 3020  U...M.VW.U.R..0
00000010: c040 50ff d683 c408 0083 c408 5dc3 cccc 5 .@P.........]...
```

Here is a really simple python script of the streaming xor decryption function

```
string = "\x37\xE7\xD8\xBE\x7A\x53\x30\x25\xBB\x38\x57\x26\x97\x26\x6F\x50\xF4\
\x75\x67\xBF\xB0\xEF\xA5\x7A\x65\xAE\xAB\x66\x73\xA0\xA3\xA1"
key = bytearray(b'\x55\x8b\xec\x8b\x4d\x0c\x56\x57\x8b\x55\x08\x52\xff\x15\x30\
\x20\xc0\x40\x50\xff\xd6\x83\xc4\x08\x00\x83\xc4\x08\x5d\xc3\xcc\xcc')

answer = ''
for i in range(len(string)):
    k = key[i]
    x = chr(k ^ ord(string[i]))
    answer += x
    k = x
print answer
```

[Challenge 3 <- Back](https://nobarxtx.github.io/flareon4/challenge3) | [Next -> Challenge 5](https://nobarxtx.github.io/flareon4/challenge5)
