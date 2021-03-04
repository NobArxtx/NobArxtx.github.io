---
layout: default
permalink: /RE102/section3.2/
title: Setup
---
[Go Back to Reverse Engineering Malware 102](https://nobarxtx.github.io/RE102/)

# Section 3.2: Travel Directions #

1. Start
2. `sub_406604` -  Step into InitExe 
3. `sub_403FA0` -  Step into StartExe
4. `sub_403F40` -  Step into This loops through the static list of functions in the references until the main function sub_45B93C
5. `sub_45B93C` - Checks to see if the foreground window has changed
    1. First it will get the foreground window, then sleep for 64h then capture the foreground window again (See Section 5)
    2. `0045B991` - jz should not jump
6. It then tries to check for debug output using string “w4ZUHcHjWZiye735mOUvnkKZ6XwjXIlyrS” (See Section 5)
    1. `0045B9C1` - jl should  jump
    2. `0045B9CB` - jnz should not jump
7. Tries to unsuccessfully load dll AXLzZmdD9HtbQccvaUl8.dll
    1. `0045B9D9` - jnz should not jump
    2. `0045B9DE` - jnz should not jump
8. Tries to find Atom RkLNPKJEBsQUb
9. `sub_45B894` - Step into before_use_junkdata
    1. `GetConsoleCP` - Retrieves the input code page used by the console associated with the calling process. A console uses its input code page to translate keyboard input into the corresponding character value.
    2. `0045B89F` - jz should not jump
    3. Loops for 0x355aef09 times for no reason. Kill the loop by `0045B8AD` jnz to not jump.
    4. `0045B8C4` - jnz should not jump
    5. Loops for 0x5A73350 times for no reason. Kill the loop by setting jnz to not jump.
10. `sub_45B794` - Step into use_junkdata
    1. VirtualAlloc new memory with the size of 0x65E4
    2. Nop instructions indicate foul play (See Section 5)
    3. `dword_45CCB0` value is 0x42B7
    4. `unk_45CCD4` is the Junk data
    5. `sub_407074` - Step over Copy_to_new_mem, loads Junk data of size 65E4 into new memory: Delphi move(source, dest);
    6. `unk_45CCB4` - loads 0x20 byte string
    7. `sub_45B5AC` - do_something_interesting( size of junk data, size of 0x20 byte string, pointer to 0x20 byte string, 0x100, 0x0BEE2, pointer to newly copied memory of junk data)

Let’s save `sub_45B5AC` for the next section.

[Section 3.1 <- Back](https://nobarxtx.github.io/RE102/section3.1) | [Next -> Section 4](https://nobarxtx.github.io/RE102/section4)