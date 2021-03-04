---
layout: default
permalink: /flareon4/challenge5/
title: Challenge 5 pewpewboat.exe
---

[Go Back to All Challenges](https://nobarxtx.github.io/flareon4)

# Challenge 5: pewpewboat.exe #

This was a fun challenge that required you to play mutiple rounds of a battleship game in order to reveal the answer. Remember to always check the file header before starting. 

## Triage ##

As you can see this is not a windows PE binary, but instead an ELF binary:

```
00000000: 7f45 4c46 0201 0100 0000 0000 0000 0000  .ELF............
00000010: 0200 3e00 0100 0000 e00a 4000 0000 0000  ..>.......@.....
00000020: 4000 0000 0000 0000 7033 0100 0000 0000  @.......p3......
00000030: 0000 0000 4000 3800 0900 4000 1d00 1c00  ....@.8...@.....
00000040: 0600 0000 0500 0000 4000 0000 0000 0000  ........@.......
00000050: 4000 4000 0000 0000 4000 4000 0000 0000  @.@.....@.@.....
00000060: f801 0000 0000 0000 f801 0000 0000 0000  ................
00000070: 0800 0000 0000 0000 0300 0000 0400 0000  ................
00000080: 3802 0000 0000 0000 3802 4000 0000 0000  8.......8.@.....
00000090: 3802 4000 0000 0000 1c00 0000 0000 0000  8.@.............
000000a0: 1c00 0000 0000 0000 0100 0000 0000 0000  ................
```

I typically use my ubtunu VM for ELF binary analysis. Here is the **file** command output:

```
ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=580d3cee15362410c9e7b0ae44d65d57deb52912, stripped
```

Here is some console output:

![alt text](https://nobarxtx.github.io/flareon4/images/ch5_run.png "run")

## Getting Started ##

Since I am more of a windows RE person, I had to relearn using GDB. I recommend using GDB for the debugging portion. Here is a simple [GDB Cheatsheet](http://darkdust.net/files/GDB%20Cheat%20Sheet.pdf)
```
sudo apt-get install gdb
```

In the disassembly, be sure to look for:

* large chunks of referenced junk data
* input validation functions

You want to map out the execution flow as much as possible, because you will need to know the exact offsets to set your break points if you use GDB.

You should have mapped out something like this:

![alt text](https://nobarxtx.github.io/flareon4/images/ch5_diagram.png "diagram")

## Getting started with GDB ##

1. Run GDB
```
gdb pewpewboat.exe
```

2. Set a breakpoint at the main function. Break at pointer 0x403D86 (main)
```
break *0x403D86
```

3. Run and it will stop at the next break point
```
run
```

4. Continue to running to the next break point
```
continue
```

5. Step into to the next instruction
```
si
```

6. Display disassembly for 20 instructions for the $pc regsiter
```
x/20i $pc
```
![alt text](https://nobarxtx.github.io/flareon4/images/ch5_gdb.png "gdb")

**Display Registers**
```
info registers
```
![alt text](https://nobarxtx.github.io/flareon4/images/ch5_registers.png "registers")

**Display 20 Hex Values**
```
x/20x $rbp-0x30
```

## Export Referenced data ##

Be sure to extract the referenced data at offset **6050E0**

## Seeds and Xoring ##

Each pewpew map is decoded with a seed value in function 40304F. The initial seed value is in generated in function offset `403034`. If you look up values **41C64E6D** and **3039** you will find various other code that uses this seed generator values like John's code [here](https://github.com/fideliscyber/indicators/blob/master/dga-scripts/vawtrak-dga.py). After every map is decoded it generates a new seed value which it will use to decode the next map.

From the start function:

![alt text](https://nobarxtx.github.io/flareon4/images/ch5_decode.png "decode")

Inside the decode function 40304F:

![alt text](https://nobarxtx.github.io/flareon4/images/ch5_innerdecode.png "inner decode")

## Useless Function ##

If you saw this ouput call for **NotMD5Hash** you really don't need it. You can just simply patch the usage of it.

![alt text](https://nobarxtx.github.io/flareon4/images/ch5_patch.png "patch")

Patch the binary by changing 0x74 to 0x75 which will change JZ to JNZ:
```
< 00003750: 0048 89ce 4889 c7e8 c4d2 ffff 85c0 7405  .H..H.........t.
---
> 00003750: 0048 89ce 4889 c7e8 c4d2 ffff 85c0 7505  .H..H.........u.
```

## Breaking Down the Pewpew Map Data ##

The first 8 bytes are the coordinates for the battelships while the next 8 bytes is the initial seed.
![alt text](https://nobarxtx.github.io/flareon4/images/ch5_parsedmap.png "parsed map")

### Coordinates ###

![alt text](https://nobarxtx.github.io/flareon4/images/ch5_andcoords.png "coords")

You can see where the coordinates are referenced yourself by setting a beak point at `4039E9`
```
break *0x4039E9
```
Hit continue once and it will ask you to enter a coordinate on the pewpew map and will break back at 4039E9. This is where you will want to look at the contents of [rbp+var_58]. Use these commands to view to contents.
```
>>> x/x $rbp-0x58
0x7fffffffdd58:	0x00614010
>>> x/16x 0x00614010
0x614010:	0x08087800	0x00080878	0x00000200	0x00000000
0x614020:	0x59cfcf4f	0xef6e3dba	0x00000020	0x65533242
0x614030:	0x6e616d61	0x63655220	0x74697572	0x83dc5f00
0x614040:	0x557b1484	0x7e520297	0x63603e62	0x65579bcb
>>> x/x $rax+0x8
0x614018:	0x00000800
```

You will begin to notice that $rax+0x8 is the coordinate you entered while $rbp-0x58 is the hardcorded coordinates in the map data.

|Code|Coord|
| --- | --- |
|0x1| A1 |
|0x2| A2 |
|0x4| A3 |
|0x8| A4 |
|0x10| A5 |
|0x20| A6 |
|0x40| A7 |
|0x80| A8 |
|0x100| B1 |
|0x200| B2 |
|0x400| B3 |
|0x800| B4 |
|0x1000| B5 |
|0x2000| B6 |
|0x4000| B7 |
|0x8000| B8 |
|0x10000| C1 |
|0x20000| C2 |
|0x40000| C3 |
|0x80000| C4 |
|0x100000| C5 |
|0x200000| C6 |
|0x400000| C7 |
|0x800000| C8 |
|0x1000000| D1 |
|0x2000000| D2 |
|0x4000000| D3 |
|0x8000000| D4 |
|0x10000000| D5 |
|0x20000000| D6 |
|0x40000000| D7 |
|0x80000000| D8 |
|0x100000000| E1 |
|0x200000000| E2 |
|0x400000000| E3 |
|0x800000000| E4 |
|0x1000000000| E5 |
|0x2000000000| E6 |
|0x4000000000| E7 |
|0x8000000000| E8 |
|0x10000000000| F1 |
|0x20000000000| F2 |
|0x40000000000| F3 |
|0x80000000000| F4 |
|0x100000000000| F5 |
|0x200000000000| F6 |
|0x400000000000| F7 |
|0x800000000000| F8 |
|0x1000000000000| G1 |
|0x2000000000000| G2 |
|0x4000000000000| G3 |
|0x8000000000000| G4 |
|0x10000000000000| G5 |
|0x20000000000000| G6 |
|0x40000000000000| G7 |
|0x80000000000000| G8 |
|0x100000000000000| H1 |
|0x200000000000000| H2 |
|0x400000000000000| H3 |
|0x800000000000000| H4 |
|0x1000000000000000| H5 |
|0x2000000000000000| H6 |
|0x4000000000000000| H7 |
|0x8000000000000000| H8 |

### Updating the seed 4030AF ###

Here is where the seed manipulation is happenning:

![alt text](https://nobarxtx.github.io/flareon4/images/ch5_seedmanip.png "seed manip")

It takes the coordinates and uses this to update the initial seed. If you were to input **B4**: B being x and 4 being y. 

```
seed = init_seed + ((y*0x593) + (x*0x1E01) + (x * y + 0x14a1)) & 0xffffffffffffffff
```

Once you have decoded the last level it will ask you to combine all the letters that were created from the coordinates. The hint here is the number **13** hidden between the PEWs.

```
Aye!PEWYouPEWfoundPEWsomePEWlettersPEWdidPEWya?PEWToPEWfindPEWwhatPEWyou'rePEWlookingPEWfor,PEW
you'llPEWwantPEWtoPEWre-orderPEWthem:PEW9,PEW1,PEW2,PEW7,PEW3,PEW5,PEW6,PEW5,PEW8,PEW0,PEW2,PEW3,PEW5,PEW6,PEW1,PEW4.PEWNextPEWyouPEWletPEW13PEWROTPEWinPEWthe
PEWsea!PEWTHEPEWFINALPEWSECRETPEWCANPEWBEPEWFOUNDPEWWITHPEWONLYPEWTHEPEWUPPERPEWCASE.
```

You will need to shuffle around the letters and use ROT13 to decode them.
```
OHGJURERVFGUREHZ -> BUTWHEREISTHERUM
```

![alt text](https://nobarxtx.github.io/flareon4/images/ch5_finale.png "finale")

Here is the code used to decode the pewpew map data:

```
import struct
import hexdump


pewmap = [[0x1, "A1"], [0x2, "A2"], [0x4, "A3"], [0x8, "A4"], [0x10, "A5"], [0x20, "A6"], [0x40, "A7"], [0x80, "A8"],
[0x100, "B1"], [0x200, "B2"], [0x400, "B3"], [0x800, "B4"], [0x1000, "B5"], [0x2000, "B6"], [0x4000, "B7"], [0x8000, "B8"],
[0x10000, "C1"], [0x20000, "C2"], [0x40000, "C3"], [0x80000, "C4"], [0x100000, "C5"], [0x200000, "C6"], [0x400000, "C7"], [0x800000, "C8"],
[0x1000000, "D1"], [0x2000000, "D2"], [0x4000000, "D3"], [0x8000000, "D4"], [0x10000000, "D5"], [0x20000000, "D6"], [0x40000000, "D7"], [0x80000000, "D8"],
[0x100000000, "E1"], [0x200000000, "E2"], [0x400000000, "E3"], [0x800000000, "E4"], [0x1000000000, "E5"], [0x2000000000, "E6"], [0x4000000000, "E7"], [0x8000000000, "E8"],
[0x10000000000, "F1"], [0x20000000000, "F2"], [0x40000000000, "F3"], [0x80000000000, "F4"], [0x100000000000, "F5"], [0x200000000000, "F6"], [0x400000000000, "F7"], [0x800000000000, "F8"],
[0x1000000000000, "G1"], [0x2000000000000, "G2"], [0x4000000000000, "G3"], [0x8000000000000, "G4"], [0x10000000000000, "G5"], [0x20000000000000, "G6"], [0x40000000000000, "G7"], [0x80000000000000, "G8"],
[0x100000000000000, "H1"], [0x200000000000000, "H2"], [0x400000000000000, "H3"], [0x800000000000000, "H4"], [0x1000000000000000, "H5"], [0x2000000000000000, "H6"], [0x4000000000000000, "H7"], [0x8000000000000000, "H8"]]
size = 0x240
seed = 0x3b1ee5f6b3d99ff7

def gen_seed(seed):
    seed = ((seed * 0x41C64E6D) + 0x3039) & 0xFFFFFFFFFFFFFFFF
    return seed


with open("referenced_data.bin", "rb") as fp:
    data = fp.read()
    dst = ""
    i = 0
    while i < len(data):
        k = 0
        while k < size and (k+i) < len(data):
            seed = gen_seed(seed)
            buff = data[k+i]
            dst = dst + chr((ord(buff) ^ (seed & 0xff)) & 0xff)
            k = k + 1
        print "old_seed: %s" % hexdump.dump(dst[16+i:24+i])
        seed_int = struct.unpack("<Q", dst[16+i:24+i])[0]
        print_coords = ""
        for coordinates in pewmap:
            coords = struct.unpack("<Q", dst[i:8+i])[0]
            result = coords & coordinates[0]

            while result != 0x0:
                if result == 0x1:
                    result = result + 0x1
                result = result & 0x1
                result >> 0x1

                print_coords = print_coords + " " + coordinates[1]
                x = ord(coordinates[1][0])
                y = ord(coordinates[1][1])
                seed_int = (seed_int + ((y * 0x593) + x * 0x1E01) + (y * x + 0x14a1)) & 0xFFFFFFFFFFFFFFFF
                seed = seed_int
        print "coordinates: ", print_coords
        print "new_seed: ", hexdump.dump(struct.pack('<Q', seed))
        dst = dst[:16+i] + struct.pack('<Q', seed) + dst[24+i:]
        print dst[i:400+i]
        print "\n"
        i += size
        open("output.bin", "wb").write(dst)
```

[Challenge 4 <- Back](https://nobarxtx.github.io/flareon4/challenge4) | [Next -> Challenge 6](https://nobarxtx.github.io/flareon4/challenge6)
