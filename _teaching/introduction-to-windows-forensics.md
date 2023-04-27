---
title: "Introduction to windows forensics"
collection: posts
type: "Info"
permalink: /posts/introduction-to-windows-forensics
venue: "Blue Teaming"
date: 2023-04-24
location: "City, Country"
---

## Description : 

I am excited to announce that I will be starting a new series of posts on Windows forensics. In this series, I will be diving into the world of digital forensics and exploring the various tools and techniques used to investigate and analyze Windows systems.

My goal is to publish a new post in this series every week, so be sure to check back regularly for new content.

----

* A hive is a logical group of keys, subkeys, and values in the registry that has a set of supporting files loaded into memory when the operating system is started or a user logs in

## Registry Location : 

```bash
C:\Windows\system32\config
```

![locaton](/images/intro1.png)

* We have another registry named *NTUSER.DAT* located in the user home directory.

* These files are protected, you can use *FTK Imager* to grape them .

> [Download FTK Imager](https://accessdata-ftk-imager.software.informer.com/3.1/)


* *Registry Explorer* allows Windows registry hives to be interrogated and parsed for a wide variety of forensic artifacts

> [Download Registry Explorer](https://www.sans.org/tools/registry-explorer/)


* After you open the *Registry Explorer* you can load the Hive that you grabbed using *FTK Imager*

---

* Some important artifacts that you should check :

> **What files were last accessed ?**

```bash
HCKU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer 
```
![windowsforensics1](/images/windowsforensics1.png)


> **What programs were specified to start upon loggin ?**

```bash
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

> **Find a file that once existed on the system :**

```bash
HKCU\SOFTWARE\Microsoft\Windows\Shell
```
![Windowsforensics2](/images/windowsforensics2.png)


> **What USB devices were plugged into a system?**

```bash

HKLM\SYSTSEM\CurrentControlSet\Enum\USBSTOR  --> Class ID / Serial number
HKLM\SYSTSEM\CurrentControlSet\Enum\USB  --> VID/PID

HKLM\SOFTWARE\Microsoft\Windows Portable Devices\Devices --> find serial number and then look for FriendlyName to obtain the Volume name of the USB device

HKLM\SYSTEM\MountedDevices --> find serial number to obtain the drive letter and Volume GUID of the usb device

NTUSER.DAT\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Mountpoints2 --> USB times like First time device is connected, Last time device is connected, Removal time
```

> **General information about the device:**

```bash
HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation
HKLM\SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName


HKLM\SYSTEM\CurrentControlSet\services\LanmanServer\Shares --> Display all open shares on a system


HKLM\SYSTEM\CurrentControlSet\Control\FileSystem --> look for NtfsDisableLastAccessUpdate, which is set to 0x1 by default, which means that access time stamps are turned off by default

HKLM\SYSTEM\CurrentControlSet\services\Tcpip\Parameters\Interfaces --> Display interfaces and their associated IP address configuration (note the interfaces GUID)
```

> **Network Information:**

```bash

HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList
```

![windowsforensics3](/images/windowsforensics3.png)


> **Link Files:**

```bash
C:\username\AppData\Roaming\Microsoft\Windows\Recent
C:\username\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations
C:\username\AppData\Roaming\Microsoft\Windows\Recent\CustomDestinations
```

> **Prefetcher:**

```bash

c:\Windows\Prefetch --> Prefetch files are an important type of evidence, which provide detailed information about the programs that were run on a computer.

```




   
