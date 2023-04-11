---
title: "Windows Kernel Driver Dev"
last_modified_at: 2023-04-12T12:00:00-01:00
categories:
  - Windows
tags:
  - Windows Kernel Driver
---

In this post I will detail the steps I had to get through to setup a Kernel Driver Development environment.

# First Steps

This page contains the information needed to setup an environment for testing and testing Windows Kernel Drivers

Everything here is a summary from this page: [/windows-hardware/drivers/gettingstarted/provision-a-target-computer-wdk-8-1](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/provision-a-target-computer-wdk-8-1) 

## Host Required Software

First things first, we need to install these on the host machine:

- [Visual Studio](https://learn.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk#download-and-install-the-windows-11-version-22h2-wdk)
- [Windows SDK](https://developer.microsoft.com/en-us/windows/downloads/windows-sdk/)
- [Windows Driver Kit (WDK)](https://learn.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk)

## Setting up the VM

Once that’s installed, we need to figure out a way for debugging stuff, so VMs to the rescue. This can be done on whatever virtualization software you want (VMWare, Virtual Box, Hyper-V). I’m choosing Hyper-V because why not.

Basically,

- Install [WDK](https://learn.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk) on the ***target VM***
- Run ***WDK Test Setup*** tool located on `C:\Program Files (x86)\Windows Kits\10\Remote\x64\WDK Test Target Setup x64-x64_en-us.msi`
- Now we need to make sure that your ***target VM*** can reach to the host and vice versa. To do so it’s necessary to have an ***internal switch*** with the ***vLAN Identification for management operating system*** set to ***enabled*** and then use this new switch for the VM in question. If everything goes well you should be able to ping the VM from your host machine.

## Setting up Visual Studio

This bit is easy … just open your Visual Studio project go to ***Extensions -> Driver -> Test -> Configure Device***, then select ***Add Device*** and put the IP or the hostname in the ***Network Hostname***.

> If you decide to put the IP instead of the hostname, make sure that the IP of the VM is static

Once that’s done … the driver can be deployed on the target by clicking on ***Build -> Deploy Solution***.

## Setting up the debugger

For more information about setting up WinDbg, read: [windows-hardware/drivers/debugger/setting-up-network-debugging-of-a-virtual-machine-host](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/setting-up-network-debugging-of-a-virtual-machine-host)

The debugger I’m going to be using is ***WinDbg*** . There are a couple of ways of connecting this debugger to the target, but I’m going to be using the network one

> if you don’t have ***kdnet*** on the target machine, you can copy it from your local machine to the target. It’s usually under `C:\Program Files (x86)\Windows Kits\10\Debuggers\x64` . Create a folder ***C:\KDNET*** on the target and copy both ***kdnet.exe*** and ***VerifiedNICList.xml*** to there

> The `<debugport >` must be between *50000-50039*

On the target machine, run the following command `kdnet <yourhostip> <debugport>` . Once this command runs you should see something like the output below. Save that somewhere on your host because you are going to need this information.

```
C:\>kdnet <YourIPAddress> <YourDebugPort> 

Enabling network debugging on Microsoft Hypervisor Virtual Machine.
Key=3u8smyv477z20.2owh9gl90gbxx.3sfsihzgq7di4.nh8ugnmzb4l7

To debug this vm, run the following command on your debugger host machine.
windbg -k net:port=50005,key=3u8smyv477z20.2owh9gl90gbxx.3sfsihzgq7di4.nh8ugnmzb4l7

Then restart this VM by running shutdown -r -t 0 from this command prompt.
```

Now, open WinDbg as an administrator on the host machine and go to ***File -> Kernel Debug***. In the window opened put the port and the key used in the port above.