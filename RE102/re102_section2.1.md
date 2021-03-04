---
layout: default
permalink: /RE102/section2.1/
title: Setup
---
[Go Back to Reverse Engineering Malware 102](https://nobarxtx.github.io/RE102/)

# Section 2.1: Information Gathering Results #

![alt text](https://nobarxtx.github.io/RE102/images/Section2_virustotal.png "virustotal")

I personally just start by looking up the hash on [VirusTotal](https://www.virustotal.com) because most of the triage information is already there. If it’s not on VirusTotal, there are tools in the VMs. You can get the same information by using CFF explorer.

## File Context and Delivery ##

![alt text](https://nobarxtx.github.io/RE102/images/Section2_CFFexp.png "CFFexp")

We don’t have a victim to tell us how this file was delivered. However you can guess the intent by looking at the original filename in the properties **InternalName** and **FileDescription**. As you can see it is posing as the **Anti-Virus Malwarebytes** software. One can only guess that the malware was pretending to be an Anti-Virus so that the victim will trust it. An IT admin might even overlook the process with the name mbam.exe because it will look legit at first glance.

## File Information & Header Analysis ##
We already know that this sample is posing as Anti-Virus Software. How do we know it’s not legit? Because we know that this sample is **not signed** at all or [signed](https://en.wikipedia.org/wiki/Code_signing) by MalwareBytes the company. So that is already a major Red Flag.

Notice that the file size is pretty small, 2.1 MB. Legitimate executables are usually much larger because they have many libraries to ensure that the program has enough resources and support for different execution environments. This file is actually larger than normal malware because of the resource sections, which I will get into later.

## Basic PE information ##
As I explained above, the filesize is small but still larger than most malware. 
That’s because 2 reasons: 

1) It has many resources

2) It was compiled as **Borland Delphi** (`BobSoft Mini Delphi -> BoB / BobSoft`).

Why does it matter how the sample was compiled? Because it will determine how the disassembly will be structured.

There are all different types of programming languages, and each with their supporting libraries. The more high-level the language such as C#, Python, Delphi the more libraries they need to support transposing the language in to assembled code. This info will become more important when you look at its disassembly in Section 3.

### Imports ###
Dynamic Linked Library (DLL) Imports are great way to guess what a malware is going to do.
Looking at the imported functions for `User32.dll` you can see there are many API functions related to User Interaction (i.e. `GetForegroundWindow`, `GetCursorPos`, etc.). While in `Kernel32.dll`, there are many functions for memory manipulation (i.e `VirtualAlloc, VirtualFree`), resource manipulation (i.e. `FindResourceA, SizeofResource`), and possible Anti-Analysis tricks (i.e.`Sleep, GetTickCount`). With the imports from `Advapi32.dll`, we can tell it is going to access registry keys (i.e. `RegOpenKeyExA`). The more malware you examine, the more you will get use to seeing how they API functions are being used.

## Strings Review ##
Strings always provide a good starting point for clues. It may also reveal things that the PE info was not able to provide such as extra  loaded DLLs and API functions. Another hint is looking at junk strings. Junk strings potentially means it is either an image or extra binary data being reference by the sample.  It could potentially be an encoded/encrypted/compressed payload. (muhahaha)

![alt text](https://nobarxtx.github.io/RE102/images/Section2_junkdata.png "junkinthetrunk")

## Web search ##
A string web search is a last resort. I usually use this step to find reports already generated for this family of malware. Unfortunately there is ton of junk data, so we can assume this sample might be packed/encrypted somehow for now.

## Anti-Virus Vendors ##
AV Vendors provide some insight into a sample besides just good or bad. There are also many heuristic and generic detections that might not be useful in telling you what the sample is going to do. Keywords like Trojan or GEN for Generic are not enough to tell you how bad it is. However the keyword Injector gives us a hint that it is doing some kind of code in memory manipulation. As we know from above, this malware has a bunch of junk code, so AV might not have enough binary features to make a definitive signature but still does well in telling you that it looks generally abnormal.

![alt text](https://nobarxtx.github.io/RE102/images/Section2_AVnames.png "Section2_AVnames")

## VM Detonation & Network Information ##
If you followed [RE101 Lab 1](https://nobarxtx.github.io/RE101/section4/) you will know how to do a simple VM detonation to collect the filesystem, process, registry and network information. You will notice that this sample does nothing really special in the VM. So we will need to investigate why.


[Section 2 <- Back](https://nobarxtx.github.io/RE102/section2) | [Next -> Section 3](https://nobarxtx.github.io/RE102/section3)
