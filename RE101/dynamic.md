---
layout: default
permalink: /RE101/section6/
title: Dynamic Analysis
---
[Go Back to Reverse Engineering Malware 101](https://nobarxtx.github.io/RE101/)

# Section 6: Dynamic Analysis #

![alt text](https://nobarxtx.github.io/RE101/images/hackerman.gif "hackerman")

## LAB 3
Dynamic analysis is a deeper analysis of the program to understand hidden functionality not understood statically. The static analysis will serve as a guide for stepping through the program in a debugger.

Open the unpacked malware into the **x32dbg.exe** (referred as x64dbg) debugger and **IDAfree**.

--- 

### Rebasing the disassembler

Typically programs start at **004010000** but your debugger might start the program at a different address. You will need to rebase the program's address in the disassembler. In x64dbg, after you hit run or **F9**, it will stop you at the EntryPoint. Scroll up to find the very first address, this is the address that you will need to rebase. 

Edit->Segements->Rebase Program.

![alt text](https://nobarxtx.github.io/RE101/images/dyn2.png "Victim and Sniffer")

--- 

### Finding the starting point

You will need to sync the debugger and disassembler addresses so you can follow along in both. Let's start with the function offset **xxxx1530**.
* In IDA, open the functions tab and look for function xxxx1530. Where xxxx should match your rebase address ( If rebase is **0190**1000, then **0190**1530 ).
* In x64dbg, CTRL+G to jump to a specific address xxxx1530.

![alt text](https://nobarxtx.github.io/RE101/images/dyn3.png "IDAmain")
![alt text](https://nobarxtx.github.io/RE101/images/dyn4.png "x64dbg Jump")

---

### XOR Decode Function

Remember use the F2(breakpoint), F7(Step Into), F8(Step Over), F9(Run) keys to navigate through the debugger. If you accidentally run past the end the of the program you can always restart by clicking ![alt text](https://nobarxtx.github.io/RE101/images/restart.png "restart").

![alt text](https://nobarxtx.github.io/RE101/images/dyn6.png "xordecode")

In **IDA**, get the offset of the XorDecode function you saved prior.

![alt text](https://nobarxtx.github.io/RE101/images/dyn8.png "xordecode")

In **x64bdg** find that same offset and add a comment that it is the Xor Decode function. Set a breakpoint using **F2** on that function. Then run the program until the breakpoint using **F9**. Step into that function using **F7**.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE101/images/dyn5.gif "xordecode")](https://nobarxtx.github.io/RE101/images/dyn5.gif)

Navigate down to the loop that does the Xor Encoding. Place a breakpoint on the same instructions shown below. Right click on the EBX register and select Follow in Dump. This location is where the decoded string will be stored. After you set your break points, press **F9** to get to the start of the loop, then step through the loops until you see the decoded string in the dump.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE101/images/dyn9.png "xordecode")](https://nobarxtx.github.io/RE101/images/dyn9.png)

---

### Navigating to the Internet Request

We want to manipulate the control flow instructions so that we can get to the network connection API call. We know that the program will first **copy** and then **delete** itself after it checks if the file doesn't exists using GetFileAttributes API. Continue to step to the **jne** (jump if not equal) instruction. By double clicking the **ZF flag** we can manipulate the result 1 to 0. This means it will make the jump past the Copfile API.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE101/images/dyn10.gif "ZF Flag")](https://nobarxtx.github.io/RE101/images/dyn10.gif)

Once you get past the delete API, there is that weird string you saw during static analysis. Step over (**F8**) the XorDecode function and notice the EAX register. It is the URL that was in the internet traffic from the triage analysis. 

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE101/images/dyn11.png "Nav to Internet")](https://nobarxtx.github.io/RE101/images/dyn11.png)

---

### Manipulate the HTTP request outcome

The VM was not connected to the internet but instead InetSim. What will happen when you manipulate the control flow to get past the internet connection failure? Go ahead and step past the internet connection and manipulate the control flow flag ZF to do so.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE101/images/dyn12.gif "Nav past Internet")](https://nobarxtx.github.io/RE101/images/dyn12.gif)

It must have been a very funny joke. **l** **m** **a** **o**

---

### There is a message for you

It seems that the malware was waiting for the word **lmao** to display a message. Navigate to the Messagebox api. Set a breakpoint on and after the function call, this will ensure that it will prevent you from skipping any hidden functionality. Go ahead and press **F9** to run the MessageBox function.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE101/images/dyn13.gif "Yo this is Dope")](https://nobarxtx.github.io/RE101/images/dyn13.gif)

---

### Extracting the Resource

The CFF explorer from the triage analysis revealed that there was a resource called **BIN**. Step through the program to get the location of the loaded resource after **LockResource**. Remember function return the output in register **EAX**.  Notice `mov edi,eax` is where the output is stored in **EDI**.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE101/images/dyn14.png "ResourceLoad")](https://nobarxtx.github.io/RE101/images/dyn14.png)

---

### Crypto Function

We can assume that the malware is going to decrypt this string based on the function arguments for [CryptStringToBinary](https://msdn.microsoft.com/en-us/library/windows/desktop/aa380285.aspx).

```C++
BOOL WINAPI CryptStringToBinary(
  _In_    LPCTSTR pszString, //Arg 1
  _In_    DWORD   cchString,
  _In_    DWORD   dwFlags, // Arg 3 Format of the string converted
  _In_    BYTE    *pbBinary,
  _Inout_ DWORD   *pcbBinary,
  _Out_   DWORD   *pdwSkip,
  _Out_   DWORD   *pdwFlags
);
```

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE101/images/dyn15.png "CryptString")](https://nobarxtx.github.io/RE101/images/dyn15.png)

We know that Arg 1 is register **EDI** which is the resource we just loaded into memory and Arg 3 is 1. The CryptStringToBinary dwflag `0x00000001` means `CRYPT_STRING_BASE64`. Dump the address of EDI into one of the dump windows. This data definitely looks like base64 encoded strings. Step over these functions until past the second CryptStringToBinary call. The result will be placed in register **ESI**. Dump the address in the ESI register. Notice anything weird about the first 3 characters?

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE101/images/dyn16.png "Post CryptString")](https://nobarxtx.github.io/RE101/images/dyn16.png)

---

### CreateFile and ShellExecute

Step over the create and write file functions to save the decrypted resource to the file system. Note that this file is saved as **icon.gif**. Next step until the start of the arguments for the ShellExecute call. It looks as if it's using the environment to open the newly created file. The program will finally be done. Open the image and record what you see.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE101/images/dyn17.gif "ShellExecute")](https://nobarxtx.github.io/RE101/images/dyn17.gif)

---

### Finale

Go to the URL in the icon.gif.

[Section 5 <- Back](https://nobarxtx.github.io/RE101/section5) | [Next -> Finale](https://nobarxtx.github.io/RE101/section6.1)
