---
layout: default
permalink: /RE102/section3/
title: Setup
---
[Go Back to Reverse Engineering Malware 102](https://nobarxtx.github.io/RE102/)

# Section 3: Creating Travel Directions #

![alt text](https://nobarxtx.github.io/RE102/images/Section3_intro.gif "intro")

Now it's time for static analysis by looking at the disassembly. The point of this section is to create a map of the execution flow of the malware. The easiest way to accomplish this is by starting somewhere in the middle and then working your way backwards. Working backwards helps you create a more accurate route because you can see the forks that led to your current position. You can anticipate the right or left path, in assembly, it’s jump or not-to-jump.

Starting somewhere in the middle means picking an interesting function to look at or where a string is referenced. Many malware reverse engineers want to start at interesting API functions like the imports mentioned in Section 2. 

## Understanding Post-Compiled Structure ##

Remember that this sample is Borland Delphi code. This means we will see many functions building up the Delphi libraries. These libraries are organized like object-oriented classes. Each class has an initialization function as well as references to class functions. A Delphi app will sequentially load these structures where libraries are loaded before the main function coded by the malware author. Makes sense, right? In order to use the library, you have to load them first. 

![alt text](https://nobarxtx.github.io/RE102/images/Section3_delphi.gif "Section3_delphi")

The diagram above is a high-level view of how a Delphi app executes each library class. There is a pointer to a hardcoded array/list of these classes which is passed to InitExe function and then the StartExe function. It will loop through this list initializing, executing, and storing pointers to functions for later use. I have identified Main Functions as the possible interesting functions we want to look at. Below the is the disassembly equivalent of the diagram.

*Click Image to Enlarge*
[![alt text](https://nobarxtx.github.io/RE102/images/Section3_initexe.png "initexe")](https://nobarxtx.github.io/RE102/images/Section3_initexe.png)

*Click Image to Enlarge*
[![alt text](https://nobarxtx.github.io/RE102/images/Section3_array1.png "array1")](https://nobarxtx.github.io/RE102/images/Section3_array1.png)
[![alt text](https://nobarxtx.github.io/RE102/images/Section3_array2.png "array2")](https://nobarxtx.github.io/RE102/images/Section3_array1.png)

---

## Where to Start? ##

So we have some options to start working backwards:

1. Where was that junk data was referenced.
2. Choose an import function (i.e VirtualAlloc).
3. Choose a function that is not loading a library.

So the goal here is making the route between the StartExe and choices 1,2, or 3. So let’s pick option 1 and start Lab 1 on the next page. 


[Section 2.1 <- Back](https://nobarxtx.github.io/RE102/section2.1) | [Next -> Section 3.1](https://nobarxtx.github.io/RE102/section3.1)