---
layout: default
permalink: /flareon4/challenge3/
title: Challenge 3 greek_to_me.exe
---

[Go Back to All Challenges](https://nobarxtx.github.io/flareon4)

# Challenge 3: greek_to_me.exe #

Knowing the Flare theme, they do a great job at giving you hints based on the filename. One can only guess that they are expecting non-standard english alphabet charaters as input.

Remember to verify that it's an windows executable and see what is does when you run it. When you run the executable you'll notice that it requires a different kind of input than something simply on the console.

When you open the binary in a disassembler you will see this kind of structure:

![alt text](https://nobarxtx.github.io/flareon4/images/ch3_diagram.png "diagram")

The executable will start up a listening socket on port 2222 and waits to recieve data in a small buffer. That buffer is transfered to a low byte register (i.e. al, bl, cl, dl). This means it's one ascii character ranging from 0 to 255 ordinal.

The hint here is the value tested against 0xFB5E. There is a function that processes the input before it performs the test. This is where you need to recreate the algorithm and it's all down hill from there. Function offset 4011E6 is where you need to break down the algorithm.

### So let's go over the assembly: ###

## First loop ##

The first loop preps the answer array.
![alt text](https://nobarxtx.github.io/flareon4/images/ch3_loop.png "first loop")

In python it looks like this:
```
answer = stored_bytes[:]
for i in range(size):
    x = char ^ answer[i]
    x = (x + 0x22) & 0xff
    answer[i] = x
```

## Main algorithm ##
The 4011E6 will then take the answer array in reverse and do some bit shifting while chaining addition based on 2 bytes because of the usage of ax and cx. This value will eventually equal to 0xFB5E. 
![alt text](https://nobarxtx.github.io/flareon4/images/ch3_aglo.png "algorithm")

Once the answer array runs through the algorithm, and passes the check against 0xFB5E, it will jump to the beginning of the newly modified answer array. This will end up being assembly instructions. If you are making a simple python script, you can use capstone to simply disassemble the new bytes. However, you can always just perform normal debugging.

The script below brute forces the character "flag" and also disassembles the modified bytes.

```
from capstone import *
from capstone.x86 import *
stored_bytes = bytearray(b'\
\x33\xe1\xc4\x99\x11\x06\x81\x16\xf0\x32\x9f\xc4\x91\x17\x06\x81\
\x14\xf0\x06\x81\x15\xf1\xc4\x91\x1a\x06\x81\x1b\xe2\x06\x81\x18\
\xf2\x06\x81\x19\xf1\x06\x81\x1e\xf0\xc4\x99\x1f\xc4\x91\x1c\x06\
\x81\x1d\xe6\x06\x81\x62\xef\x06\x81\x63\xf2\x06\x81\x60\xe3\xc4\
\x99\x61\x06\x81\x66\xbc\x06\x81\x67\xe6\x06\x81\x64\xe8\x06\x81\
\x65\x9d\x06\x81\x6a\xf2\xc4\x99\x6b\x06\x81\x68\xa9\x06\x81\x69\
\xef\x06\x81\x6e\xee\x06\x81\x6f\xae\x06\x81\x6c\xe3\x06\x81\x6d\
\xef\x06\x81\x72\xe9\x06\x81\x73\x7c')

size = len(stored_bytes)
for char in range(0, 255):
    answer = stored_bytes[:]
    for i in range(size):
        x = char ^ answer[i]
        x = (x + 0x22) & 0xff
        answer[i] = x

    answer = answer[::-1]  # reverse order
    a = 0xff  # 255
    b = 0xff  # 255
    offset = 0x14  # 20
    i = size-1
    while i >= 0:
        j = max(i - offset, -1)
        while i > j:
            a = a + (answer[i] & 0xff)
            b = b + a
            i -= 1
        a = ((a >> 8) & 0xff) + (a & 0xff)
        b = ((b >> 8) & 0xff) + (b & 0xff)
    b = (b << 8) & 0xff00
    final = b | a
    if final == 0xfb5e:
        print("This is the flag: %s\n" % hex(char))
        # print disassembly with capstone
        print("This is the new code:")
        CODE = bytes(answer[::-1])
        md = Cs(CS_ARCH_X86, CS_MODE_64)
        md.detail = True
        for insn in md.disasm(CODE, 0x1000):
            c = ''
            if len(insn.operands) > 0:
                for i in insn.operands:
                    if i.type == X86_OP_IMM:
                        c = chr(i.value.imm)
            print("0x%x:\t%s\t%s ; %s" % (insn.address,
                                          insn.mnemonic,
                                          insn.op_str, c))
```

[Challenge 2 <- Back](https://nobarxtx.github.io/flareon4/challenge2) | [Next -> Challenge 4](https://nobarxtx.github.io/flareon4/challenge4)
