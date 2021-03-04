---
layout: default
permalink: /RE101/section6.1/
title: Dynamic Analysis
---
[Go Back to Reverse Engineering Malware 101](https://nobarxtx.github.io/RE101/)

# Section 6: Finale #

Congrats, you made it through the workshop. All of your notes and debugging should have gotten you to come up with a similar control flow like the diagram and report below.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE101/images/Diagram.png "diagram")](https://nobarxtx.github.io/RE101/images/Diagram.png)


## Simple Report

Filename: Unkown.exe

Sha256: a635f37c16fc05e554a6c7b3f696e47e8eaf3531407cac27e357851cb710e615

### Summary

This file creates a copy of itself in the %APPDATA% location, sets persistence mechanisms, and beacons to definitely-not-evil.com. If beacon is successful, it will open a messagebox, then decrypt the resource which will then spawn a shell window to open the resource.

### General Characteristics

The file is UPX packed

Import Functions:
* GetEnvironmentVariable
* CopyFile
* DeleteFile
* InternetOpen
* InternetConnect
* HttpOpenRequest
* HttpSendRequest
* MessageBox
* FindResource
* CryptStringToBinary
* CreateFile
* ShellExecute
* CreateProcess


### File System IOC

CreateFile	C:\Users\victim\AppData\Roaming\dope.exe CreateFile icon.gif

### Network IOC

GET /ayy HTTP/1.1 

Content-Type: text/html 

MySpecialHeader: whatever 

User-Agent: definitely-not-evil.com 

Host: definitely-not-evil.com 

Cache-Control: no-cache 

### Registry IOC

RegQueryValue	HKCU\Software\Microsoft\Windows\CurrentVersion\Run\dope

### Behavior & Control Flow

Processes Created dope.exe

1) Starts by decoding xor strings 

2) Checks to see if dope.exe already exists in %APPDATA% 

3) If it doesn't exist create a copy of itself to %APPDATA% as dope.exe 

4) Set the startup registry key 

5) Start the newly copied dope.exe process 

6) Delete the original 

7) Dope.exe will check the registry key if set 

8) Call out to definitely-not-evil.com 

9) If the result is "lmao" it will open a messagebox and extract the resource 

10) Base64 decode the resource 

11) Save decoded resource as icon.gif 

12) Shellexecute to open icon.gif

[Section 6 <- Back](https://nobarxtx.github.io/RE101/section6)
