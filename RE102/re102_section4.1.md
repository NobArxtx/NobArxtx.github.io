---
layout: default
permalink: /RE102/section4.1/
title: Setup
---
[Go Back to Reverse Engineering Malware 102](https://nobarxtx.github.io/RE102/)

# Section 4.1: Identifying the Decryption Algorithm #

Now it’s time to dig deeper and follow the assembly one step at a time. From the previous page you recorded what are the arguments and variables used in function `sub_45B5AC`:

```
eax = data_size
edx= key_size
ecx = key
sub_45B5AC( 0x100, 0xBEE2, junk2, 0x1F) 
```

Now that you are in the `sub_45B5AC` function, IDA labels the arguments as:

```
Arg_0 = 0x100
Arg_4 = 0xBEE2
Arg_8 = junk2
Arg_C = 0x1F 
```

At `loc_45B5C9`, the registers that saved the key and sizes are moved into base pointer offsets:

```
45b5cb:    mov [ebp-0xc], ecx ; Key
45b5ce:    mov [ebp-0x8], edx ; Size of Key
45b5d1:    mov [ebp-0x4], eax ; Size of Shellcode 
```

Remember the stack structure from RE101, local variables grow to lower addresses and parameters grow to higher addresses:

![alt text](https://nobarxtx.github.io/RE102/images/Section4.1_TheStackFrame2.png "Section4.1_TheStackFrame2")

Now that you have all the important variables, you can statically trace through this function in IDA to discover it’s algorithm.

---

## Loop 1: Saving the Key on the Stack ##

Arg_C is 0x1F (31 dec) bytes, which is one byte less than the size of our key. Since arrays start from 0, as you can guess this represents **key_size - 1**. This gets saved into register `ebx`

![alt text](https://nobarxtx.github.io/RE102/images/Section4.1_loop1.png "Section4.1_loop1")

If you are not familiar with mathematical equivalent of bitwise operations, it is important to know shift operations can be a form of multiplication or division. For example, when you see `shr ebx, 2`, it means that the content of the ebx register is getting divided by 4. This is 31 divided by 4. Why 4? Because when you shift n bits of an unsigned binary number, it has the effect of dividing it by 2^n (rounding towards 0). As it loops through the Key (ecx) is pushes/saves 4 byte chunks onto the stack. It should look something like this:

```
00183BCC  3669C7AF  
00183BD0  CBD60266  
00183BD4  0C33A849  
00183BD8  973AD4C1  
00183BDC  C868B780  
00183BE0  820B3D00  
00183BE4  2C9BED2C  
00183BE8  F94D125D   
```

---

## Loop 2: Fill the Stack 0x100 characters ##

This next loop fills the stack starting at `[ebp+var_418]`. It loops for 0x100 times or 256 decimal while incrementing ebx from 0 to 255.

![alt text](https://nobarxtx.github.io/RE102/images/Section4.1_loop2.png "Section4.1_loop2")

At this stage the question that you need to ask yourself is what crypto algorithm uses 256 bytes with a key size of 32 bytes? You can also even narrow it down to only symmetric key algorithms, since this function is way too simple be an asymmetric key algorithm.

So let’s create the pseudo code for this loop:

```
int ebx = 0;
int length = 256 // 0x100
While (ebx < 256)
{
    push(ebx)
    ebx++
}
```

This is what the stack should look like:

![alt text](https://nobarxtx.github.io/RE102/images/Section4.1_256bytes.png "Section4.1_256bytes")

---

## Loop 3: Functions applied to 0x100 characters ##

In the same location on the stack [ebp+var_418], the loop processes the data again, but introduces the usage of function `sub_405268`. This function takes 3 inputs. 

The first call to `sub_405268`:

1. 0
2. [ebp-0x8] Size of Key which is 32 decimal
3. eax

The second call to `sub_405268`:

1. 0
2. Arg_0 which is 0x100, 256 decimal
3. eax

![alt text](https://nobarxtx.github.io/RE102/images/Section4.1_loop3.png "Section4.1_loop3")

When you enter function `sub_405268`, you will notice that there are a bunch of arithmetic instructions. This function is actually a modulo function. Tip: the Pro version of IDA marks function `sub_405268` as the Delphi library function System::llmod.

Rename function `sub_405268` to “mod”.

At the end of the loop, the are some move instructions. Can you guess what is going on here?

* `ebx` is being incremented by 1, let ebx be i where i = 0
* `esi` is being incremented by 4, in other words let esi be j where j = 0.
* `[ebp+var_418]` is location of the 0-256 characters created, let it be array[]
* `ebp+var_C` is the pointer to the Key
* `ebp+var_D` is a temporary location on the stack

So let’s make the pseudo code for this loop:

```
int i = 0; //eax
int j = 0;
int temp, a, b, c;
while (i < 0x100)
{
    i = mod( 0, 0x20, i );
    a = Key[ i ]; // eax, [edx+eax]
    b = j+Array[i]; // edi, [esi]
    c = a+b; // add eax, edi
    j = mod (0, 0x100, c);
    
    //swap
    temp = Array[i];
    Array[i] = Array[j];
    Array[j] = temp;
    i++;
}
```

Let’s see if you can identify this crypto algorithm. Try google searching for “symmetric mod 256”. Your first hit might be RC4 from wikipedia.

![alt text](https://nobarxtx.github.io/RE102/images/face.jpg "face")

Check out that Key-scheduling algorithm on the RC4 wikipedia page. Notice any similarities from Loop 2 and Loop 3?

![alt text](https://nobarxtx.github.io/RE102/images/Section4.1_keyschedule.png "Section4.1_keyschedule")

---

## Loop 4: Loop through Junk2 data ##

Looks like this algorithm is RC4 256. On your own, try to trace through the second part of the RC4 algorithm with the fourth loop. Be extra careful in assigning the variables, because there is an error here and you may not find it right away until you start decrypting.

Section 4.2 will go over decrypting that junk2 data.

[Section 4 <- Back](https://nobarxtx.github.io/RE102/section4) | [Next -> Section 4.2](https://nobarxtx.github.io/RE102/section4.2)
