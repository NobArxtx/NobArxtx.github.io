---
layout: default
permalink: /RE102/section4/
title: Setup
---
[Go Back to Reverse Engineering Malware 102](https://nobarxtx.github.io/RE102/)

# Section 4: Identifying Encryption #

![alt text](https://nobarxtx.github.io/RE102/images/Section4_intro.gif "intro")

This section will focus on generically recognizing encryption routines. In the previous section, you left off at `sub_45B5AC`. As you might be able to guess, this malware is using an encryption algorithm here. The give aways are:

* Suspicious function arguments (e.g., large amounts of bytes used for allocation)
* Multiple loops
* Usage of XOR
* Unusual instructions (e.g., NOP)

---

## Suspicious Function Arguments ##

To decrypt data that is encrypted the malware needs:

1. Key
2. Encrypted Data (a.k.a ciphertext)
3. Destination for Decrypted Data

Let’s take a look at the arguments for `sub_45B5AC`. Remember in section 1.3 of RE 101, it explained that assembly function calls have their arguments pushed onto the stack in reverse order. To learn about the reason behind this you can check out this [article](https://en.wikipedia.org/wiki/Calling_convention). In the image below, you can see it’s pushing 4 times and saving 3 objects in 3 different registers (ecx, edx, eax).

![alt text](https://nobarxtx.github.io/RE102/images/Section4_functionargs.png "Section4_functionargs")

### Delphi calling convention ###

Remember that this is delphi so the calling convention uses registers eax, ecx, edx as first 3 arguments. The rest of the arguments are pushed on stack in reverse order.

```
push    3
push    4
mov     ecx, 2
mov     edx, 1
xor     eax, eax
call    function
xor     eax,eax
retn    10
```

Based on previous sections, it should be already obvious to you what these values mean. You know that the malware recently called VirtualAlloc, and moved **junk 2** of size 0x65E4 into the new memory stored it in `[ebp+var_BEEB]`. If you click on `unk_45CCB4`, you will see that this data is only 0x20 (32 dec) bytes. So, the pseudo code for this function would be:

```
eax = size_of_junk2
edx = size_of_small_junk
ecx = small_junk unk_45CCB4
sub_45B5AC( 0x100, 0xBEE2, junk2, 0x1F) 
```

Let’s rename it all:

```
eax = data_size
edx= key_size
ecx = key
decrypt(0x100, 0xBEE2, encrypted_data, 0x1F)
```

Now all you need to know is what 0x100 and 0xBEE2 represent, and you might not know until you start to break down the decrypt function. 

**Hint:** 0xBEE2 is 48,866 bytes. This is large enough to be a new executable. 

---

## Multiple Loops ##

Cryptographic algorithms are often grouped into two major categories: symmetric and asymmetric. Most of these algorithms in order to perform some sort of shuffling to the plaintext need to loop over each or blocks of characters. Let’s take a look at a structure used in many symmetric block cipher algorithms:

![alt text](https://nobarxtx.github.io/RE102/images/Section4_cipher.png "Section4_cipher")

For every subkey K in this algorithm, it has to loop through each K to XOR and Swap. In the disassembly you will be able to see this looping, incrementing, and swapping action going on. Now let’s look at `sub_45B5AC`.

![alt text](https://nobarxtx.github.io/RE102/images/Section4_looping.png "Section4_looping")

There are actually multiple loops happening in this function. Section 4.1 will go over how identifying this algorithm. This section focuses on just recognizing usage of crypto.

---

## Usage of XOR ##

Bitwise operator, XOR, is the bare bone of symmetric key encryption algorithms. Like in the block cipher algorithm above the circle with a cross inside represents the XOR symbol. When reversing assembly code to identify the usage of cryptographic algorithms, you typically want to look for XOR instruction with 2 different registers.

**Note:** Do not mistake instructions such as xor eax, eax for usage of crypto, because they are usually used for clearing out a register (e.g., eax in this case). 

In function `sub_45B5AC`, `xor [esi], al`, is another nice indicator of encryption usage. 

---

## Suspicious Instructions ##

In the beginning of this section, it mentioned you need to be suspicious of NOP instructions; however, they are not indicators for usage of cryptographic algorithms. They usually show  that the malware author did not want the function to be analyzed or detected. Inserting NOPs changes the patterns of the bytecode of a binary, and makes it harder for AV’s signatures to detect those patterns. As an analyst, when I see these NOPs, I can usually tell that I am in the right spot (or a spot that the malware author does not want me to be), so I start digging deeper.

The next subsection will go over identifying which cryptographic algorithm this malware is using.

[Section 3.2 <- Back](https://nobarxtx.github.io/RE102/section3.2) | [Next -> Section 4.1](https://nobarxtx.github.io/RE102/section4.1)
