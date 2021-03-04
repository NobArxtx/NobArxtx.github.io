---
layout: default
permalink: /RE101/section1/
title: Fundamentals
---
[Go Back to Reverse Engineering Malware 101](https://nobarxtx.github.io/RE101/)

# Section 1: Fundamentals #

## Environment Setup ##

In this section you will be setting up a safe virtual malware analysis environment. The virtual machine (VM) that you will be running the malware on should not have internet access nor network share access to the host system. This VM will be designated as the **Victim VM**. On the other hand, the **Sniffer VM** will have a passive role in serving and monitoring the internet traffic of the Victim VM. This connection remains on a closed network within virtualbox.

### Installing VirtualBox ###

For windows and osx, follow the instructions in the install binary.

| Windows | Mac OSX | Linux |
| --- | --- | --- |
| [![alt text](https://nobarxtx.github.io/RE101/images/VBwin.png "Windows Virualbox")](http://download.virtualbox.org/virtualbox/5.1.14/VirtualBox-5.1.14-112924-Win.exe) | [![alt text](https://nobarxtx.github.io/RE101/images/VBmac.png "OSX Virtualbox")](http://download.virtualbox.org/virtualbox/5.1.14/VirtualBox-5.1.14-112924-OSX.dmg) | [![alt text](https://nobarxtx.github.io/RE101/images/Vblinux.png "Linux Virtualbox")](https://www.virtualbox.org/wiki/Linux_Downloads) |

### Download Victim and Sniffer VMs ###

Please use the utility [7zip](http://www.7-zip.org/download.html). Unzip the files with 7zip below and in VirtualBox **File->Import** Appliance targeting the .ova file. 


[Victim VM](https://drive.google.com/open?id=0B_0DJl2kuzoNZkpveEtiMWJKWDA)

* MD5sum: b84f0cdb7acc00aeb9effcee84b85f65 **Updated 9/11/2017**
* OS: Windows 7 Service Pack 1
* Architecture: Intel 32bit
* Username: victim
* Password: re1012017
* IP Address: 192.168.0.2
* Gateway: 192.168.0.1
* Zip size 3.96G, Final size required 10.1G

**Note:** If the VM is rebooting on you, open a command prompt with admin privileges and run "slmgr /rearm", then reboot. It should reset the VM's trial version.

[Sniffer VM](https://drive.google.com/open?id=0B_0DJl2kuzoNT3IwNElLV3VRdms)

* MD5sum: fc69591b0ce1cdc84fc5c63d02d30d5f **Updated 9/11/2017**
* OS: Ubuntu 16.04.2 LTS Desktop
* Architecture: Intel 64bit
* Username: sniffer
* password re1012017
* IP Address: 192.168.0.1
* Gateway: 192.168.0.1
* Zip size 2.08G, Final size required 6G

---

### Post Install Instructions ###

**Note:** If you are having problems getting the VM to run, revert to the AnalysisReady snapshot, then right-click on the VM and select discard the saved state.

1. Install VirtualBox CD on both VMs: Devices->Insert Guest Additions CD Image
  * If it doesn't auto appear, navigate to the CD Drive to install
  * Follow install directions from the Guest Additions Dialog
  * Note: it will require install privileges so insert passwords for each VM
  * Shutdown Both VMs after you have installed the Guest Additions CD.
2. Victim VM: Devices->Drag and Drop->Bidrectional 
3. Victim VM: Devices->Shared Clipboard->Bidirectional
4. Both VMs: Devices->Network->Network Settings
  *  Select Attached to `Internal Network`
  *  Name should mirror both VMs. Default is `intnet`
5. Run/Play both VMs to verify network connectivity
  * **Important** While running, take a snapshot of each VM and name each "Clean". This will save a clean slate for you to revert the VM image back to.
6. Sniffer VM: Ensure `inetsim` is running
  * Open terminal and run: `ps -ef | grep inetsim`
  * If no output, run: `/etc/init.d/inetsim start`
  * Run the ps command again to confirm it's running.
  * Expected output: ![alt text](https://nobarxtx.github.io/RE101/images/VerifyInetsim.png "ps output")
7. Victim VM: test connection to Sniffer VM
  * In the search bar, type `cmd.exe` to open terminal
  * Run command: `ping 192.168.0.1`
  * Expected output: ![alt text](https://nobarxtx.github.io/RE101/images/PingGateway.png "Ping Output")
8. Sniffer VM: Devices->Shared Folders->Shared Folders Settings
  * On your Host, create a folder called `sniffershare`
  * In virtual box select Add New Shared Folder icon and navigate to the folder you just created (sniffershare)
  * In Sniffer VM, open the terminal and run command:`mkdir ~/host; sudo mount -t vboxsf -o uid=$UID,gid=$(id -g) sniffershare ~/host`

[Intro <- Back](https://nobarxtx.github.io/RE101/intro) | [Next -> Anatomy of PE](https://nobarxtx.github.io/RE101/section1.2)
