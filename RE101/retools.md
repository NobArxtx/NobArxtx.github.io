---
layout: default
permalink: /RE101/section3/
title: RE Tools
---
[Go Back to Reverse Engineering Malware 101](https://nobarxtx.github.io/RE101/)

# Section 3: Reverse Engineering (RE) Tools #


## Disassemblers

* [Ida](https://www.hex-rays.com/products/ida/)
  * Free (Used in this workshop)
  * Pro (Most Popular)
* [Radare](https://www.radare.org)
* [Capstone](http://www.capstone-engine.org/)

---

## Debuggers

* [x64dbg](http://x64dbg.com/) (Used in this workshop)
* [Immunity](https://www.immunityinc.com/products/debugger/)
* [OllyDbg](http://www.ollydbg.de/)  (Most Popular)
* [WinDbg](https://developer.microsoft.com/en-us/windows/hardware/windows-driver-kit)

---

## Decompilers

* [Snowman](https://derevenets.com/) (Integrated with x64dbg)
* [dotPeek](https://www.jetbrains.com/decompiler/) .NET decompiler

---

## Information Gathering

* [CFF Explorer](http://www.ntcore.com/exsuite.php) - PE header parser (Used in this workshop)
* [PE Explorer](http://www.heaventools.com/overview.htm) - PE inspection tool (Used in this workshop)
* [BinText](https://www.mcafee.com/hk/downloads/free-tools/bintext.aspx) - Extract string from a binary
* [Sysinternals Suite](https://technet.microsoft.com/en-us/sysinternals/bb842062.aspx) (Used in this workshop)
  * procmon
  * procexplorer
* [InetSim: Internet Services Simulation Suite](http://www.inetsim.org/downloads.html) (Used in this workshop)
* [Yara: pattern matching rule engine](https://virustotal.github.io/yara/)
* [Wireshark](https://www.wireshark.org/download.html) - network sniffing (Used in this workshop)
* [API Monitor](http://www.rohitab.com/downloads)

### Helpful Websites

* [virustotal.com](https://www.virustotal.com/) - free service that analyzes suspicious files and URLs 
* [malwr.com](https://malwr.com/) - Malwr is a free malware analysis service
* [hyrbid-analysis](https://www.hybrid-analysis.com/) - free malware analysis service
* [whois.domaintools.com](http://whois.domaintools.com/) - look up domains
* [robtex.com](https://www.robtex.com/) - free DNS lookup tool 
* [www.debuggex.com](https://www.debuggex.com/) - Online Visual Regex Tester

---
  
## Support

* [HxD Hex Editor](https://mh-nexus.de/en/hxd/) (Used in this workshop)
* [Python](https://www.python.org/downloads/) - used for automating tasks

---

## Tools Used in the Workshop

### Disassembler: IdaFree

![alt text](https://nobarxtx.github.io/RE101/images/IdaFree.gif "IdaFree Layout")

* **Visual Modes**
  * **Graph Mode** - control flow diagram
  * **Text Mode** - default view of disassembled code
* **Command Cheatsheet**
  * Please refer to this [Ida cheatsheet](https://nobarxtx.github.io/RE101/idacheatsheet.html)
* **Common Commands**

| Action | Command |
| --- | --- |
| Jump to xref to operand | X |
| Jump to address | G |
| Enter comment	| Shift+; |

---

### Debugger: x64dbg

![alt text](https://nobarxtx.github.io/RE101/images/x64dbg.gif "x64dbg Layout")

**Common Commands**

| Action | Command |
| --- | --- |
| Enter comment	| ; |
| BreakPoint	| F2 |
| Step into	| F7 |
| Step over	| F8 |
| Run	| F9 |
| Edit Instruction | Space |

### Keyboard Layout for IdaFree and x64dbg

![alt text](https://nobarxtx.github.io/RE101/images/keyboarddbg.gif "Keyboard Layout")

---

## Information Gathering: CFF Explorer

* Parses the PE headers
* Explores Resources
* Unpacks UPX

![alt text](https://nobarxtx.github.io/RE101/images/CFFexplorer.gif "CFF Explorer")

## Information Gathering: Sysinternals Suite

* advanced system utilities
* **ProcMon** - Monitor processes/thread, files system, network, and registry activity on the system
* **ProcExp** - Monitor processes running on the system

![alt text](https://nobarxtx.github.io/RE101/images/procmon.png "ProcExp")


[Section 2.1 <- Back](https://nobarxtx.github.io/RE101/section2.1) | [Next -> Section 4](https://nobarxtx.github.io/RE101/section4)
