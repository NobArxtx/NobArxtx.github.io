---
layout: default
permalink: /RE102/section5.2/
title: Setup
---
[Go Back to Reverse Engineering Malware 102](https://nobarxtx.github.io/RE102/)

# Section 5.2: Evasion Techniques #

## Anti-Automation ##

Before you continue to `loc_401CCA`, there were some Anti-Automation behaviors that were not discussed from Section 3.1. The calls to GetForegroundWindow, Sleep, and GetForegroundWindow indicate that the malware is deploying various anti-automation techniques to ensure that there is an actual user changing the state of the foreground window. Typically in automated sandbox testing there is no user interaction unless they accounted to build that into their VM.

![alt text](https://nobarxtx.github.io/RE102/images/Section3.1_record_interesting.png "Section3.1_record_interesting")

---

## Anti-Debugging ##

If you remember from Section 3.1 and 3.2, there were many calls to OutputDebugString. Instead of directly calling for IsDebuggerPresent, calling to OutputDebugString and checking the success or failure is another technique to check if there is a debugger running. It’s a simple tactic to make reverse engineering and debugging the malware harder.

---

## VM Evasion ##

There are many resources for a developer to identify if the process is running in a Virtual Machine. Paranoid Fish or pafish is one of the more well-known automated VM identification scripts available. You can view the code here: [https://github.com/a0rtega/pafish](https://github.com/a0rtega/pafish). 

Every VM distro has their own filesystem and registry indicators. Products such as VMware and VirtualBox often have software installed to help with host to guest sharing. Hardware simulation will contain strings and naming related to the VM product.  Some malware will change their behavior if they find out they are running inside a VM.

In IDA, start back at `loc_401CCA` where you will be able to identify some VM Evasion techniques.

---

### Checking Hardware Device ###

Earlier in this section, there was an anti-analysis technique of pushing strings to the stack. In function `sub_4029E7` until you are in function `sub_402274`, you can see that it is pushing **H** and **A** in the the screenshot below.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_hardware.gif "Section5.2_hardware")](https://nobarxtx.github.io/RE102/images/Section5.2_hardware.gif)

Go ahead and go through all the strings that are being pushed to the stack. It should com out to:

```
HARDWARE\DEVICEMAP\Scsi\Scsi Port 0\Scsi Bus 0\Target Id 0\ Logical Unit Id 0\Identifier
```

At the very end of the function it jumps to `loc_404777` where it calls `sub_403F73`. This is where the shellcode pushes strings **vmware, qemu,** and **vbox**. The malware is checking for registry artifacts to see if it’s running inside a VM. In the debugger, set a breakpoint and run/step into `00406AB6` within function `sub_4037FD`. This is where the call to RegOpenKeyEx happens. 

![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_checkregistry.png "Section5.2_checkregistry")

If you follow the stack argument DWORD in the dump you can see the full strings. To view this, right click on the stack argument and select **Follow DWORD in dump**.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_hardwarestrings.png "Section5.2_hardwarestrings")](https://nobarxtx.github.io/RE102/images/Section5.2_hardwarestrings.png)

Open regedit.exe in Windows and verify that this registry key exists under HKEY_LOCAL_MACHINE. If this key exists RegOpenKeyEx will return 0, if not 2. In the debugger, Step over **F8** this function call. Fortunately this VM was built with an IDE instead of scsi hardware. You can verify this by looking at Virtualbox’s storage settings.

![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_vboxstoragesettings.png "Section5.2_vboxstoragesettings")

If the VM you are working in does happen to have this registry key, you can always bypass the check. You can either get rid of the artifacts themselves or patch the binary. Put a breakpoint at `00404977` so that you won’t miss this next jump. When you are debugging you can modify the **ZF flag** so that `jz loc_404D01` will fail and continue onto the next check.

![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_checkbypass.png "Section5.2_checkbypass")

---

### Check the System Bios ###

Continue to step **F7** to function `sub_4021FE` at `00404982`. This function is using the same anti-analysis technique by pushing strings onto the stack. The strings **vbox** and **qemu** are used to check the value in another registry key. Step through the rest of this function to reveal the full string until you reach `004047A7`.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_Hardware2.gif "Section5.2_Hardware2")](https://nobarxtx.github.io/RE102/images/Section5.2_Hardware2.gif)

The registry key that you should have recovered from the stack is:

```
HARDWARE\Description\System\SystemBiosVersion\SystemBiosVersion
```

Verify in the registry using regedit.exe that this registry key exists. It looks like **vbox** does exist in the SystemBiosVersion string. You will need to modify the jumps in order to bypass this check in order to continue.

![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_systembiosregistry.png "Section5.2_systembiosregistry")

There are 2 places where you can choose to modify the jump:

* Right after the registry key check function:

![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_biosjump.png "Section5.2_biosjump")

* or Right after `sub_4021FE`:

![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_biosjump2.png "Section5.2_biosjump2")

If you modified either of the jump calls above while debugging you should have reached `loc_4010FE` and `sub_4029F1`. Below, you can see how to modify the second jump.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_ModifyFlags.gif "Section5.2_ModifyFlags")](https://nobarxtx.github.io/RE102/images/Section5.2_ModifyFlags.gif)

---

### Check for VM drivers ###

Just like the previous functions, the strings a pushed to the stack. Look for the instruction `call    dword ptr [ebx+0B0h]` which is where you set a breakpoint at `00405248`. In the debugger, this will call GetSystemDirectory which will return %system32%. Keep stepping through this function to get the full paths of the files it is checking for.

* %system32%\drivers\vmmouse.sys
* %system32%\drivers\vmhgfs.sys
* %system32%\drivers\VBoxMouse.sys
* %system32%\drivers\VBoxGuest.sys

Keep stepping through function `sub_4029F1` until you get back to `0040110B` where `jnz sub_401117` and force the jump to `sub_401117`.

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_ModifyJump2.gif "Section5.2_ModifyJump2")](https://nobarxtx.github.io/RE102/images/Section5.2_ModifyJump2.gif)

---

### Check for VM DLLs ###

Step into`sub_401117` and keep going through instructions until you reach some interesting immediate values. Go ahead and convert the immediate values at `00405884` into strings. 

![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_sandboxiedll.png "Section5.2_sandboxiedll")

This function is checking for sbiedll.dll which is a DLL used by the Sandboxie sandbox. If you are working with Vbox, this DLL will not exist so you won’t need to bypass the jump. Keep working your way through this function because it’s not done with all the checks.

---

### Check the Physical Drive ###

In IDA, look into `sub_406FCC` at `0040218D` after the sandboxie DLL check. Based on the logic below you might not need to step into this function. You can always force the jump to `loc_402192` and skip over `sub_406FCC`. For the purposes of recognizing VM evasion, you should step through this function.


![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_PhyicalDriveCheck.png "Section5.2_PhyicalDriveCheck")

Put a breakpoint at `00404403` where the instruction `call dword ptr ds:[esi+98]` because this is the next API call. It tries to call CreateFile the PhysicalDrive0 in order to read it. 

*Click to Enlarge*
[![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_PhysicalDriveapicall.png "Section5.2_PhysicalDriveapicall")](https://nobarxtx.github.io/RE102/images/Section5.2_PhysicalDriveapicall.png)

Because the above check failed, it will perform another device check. Keep stepping through the program until you reach 00406266 where the second API call is `call dword ptr ds:[esi+94]`. It is calling DeviceIoControl where it will check the \\.\PhysicalDrive0 for the following strings:

* qm00001
* virtual
* array
* vbox
* vmware
* 00000000000000000001

Here is the API function as reference:

```
BOOL WINAPI DeviceIoControl(
  _In_        HANDLE       hDevice,
  _In_        DWORD        dwIoControlCode,
  _In_opt_    LPVOID       lpInBuffer,
  _In_        DWORD        nInBufferSize,
  _Out_opt_   LPVOID       lpOutBuffer,
  _In_        DWORD        nOutBufferSize,
  _Out_opt_   LPDWORD      lpBytesReturned,
  _Inout_opt_ LPOVERLAPPED lpOverlapped
);
```

After DeviceIOControl is called do not take the jump after at `00405778` or `loc_405778`. This will cause the device to close and return back to the main function where the sandboxie DLL was checked.

![alt text](https://nobarxtx.github.io/RE102/images/Section5.2_deviceIOcontroljump.png "Section5.2_deviceIOcontroljump")

This jump should land you at `loc_402192` or `00402192`. **Congratulations!** You have made it past several VM evasion techniques. The next section will go over identifying a packer.

[Section 5.1 <- Back](https://nobarxtx.github.io/RE102/section5.1) | [Next -> Section 6](https://nobarxtx.github.io/RE102/section6)
