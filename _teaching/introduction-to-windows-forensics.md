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

> What files were last accessed ?

```bash
HCKU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer 
```
├── \ComDlg32
│   ├── LastVisitedPidIMRU
│   ├── OpenSavePidIMRU
├── \RecentDocs
├── \RunMRU
├── \TypedPaths
├── \UserAssist

> What programs were specified to start upon loggin ?

```bash
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

> Find a file that it once existed on the system :

```bash
HKCU\SOFTWARE\Microsoft\Windows\Shell
```
├── \BagMRU
├── \Bags


   
