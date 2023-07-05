---
title: "Windows forensics 4 : windows MACB timestamps"
collection: posts
type: "Info"
permalink: /posts/macb
venue: "Blue Teaming"
date: 2023-07-04
location: "City, Country"
---

## Description :

Today we're going to look at ``NTFS`` timestamps, this includes how NTFS stores timestamps associated with files, how we can modify a files timestamps using ``timestomp`` and how we can find evidence of timestomp.

---

> What is MACB times ?

* It refers to the timestamps of the latest modification,access,change or creation of a certain file.

![macb1](/images/macb1.png)

* The big question here is why didn't date accessed changed after we accessed the file, the answer is because of this registry key which is set to 1 by default

* Setting this key to 1 means NTFS disables last access update

```console
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem\NtfsDisableLastAccessUpdate
```

---

* timestomp is an anti forensics tool used to set the MACB time of any given file to any value.

> [Download timestomp](https://github.com/limbenjamin/nTimetools)

* How to use the tool : 

```console
timestomp victim.txt -Z "Tuesday 07/04/2022 3:23:43 AM"
```

---

* Analyze MFT tool is a python tool designed the master file table from an ``NTFS`` volume and outputs the results in a comprehensive format.

> [Download Analyze MFT](https://github.com/dkovar/analyzeMFT)

> How Analyze MFT works?

* ``$MFT`` is the most important file because it keeps track of all the files in disk, the thing is there is more than one copy of this f4 timestamps stored withing the MFT

* ``$SI`` or ``$STANDARD_INFORMATION``
* ``$FILE_NAME`` or ``$FN`` 

* When we are looking at files with Windows explorer or cmd or powershell we are interacting with ``$SI``, however there is another copy of the timestamps stored in ``$FN``, so the **ANALYZE MFT** compares the ``$STANDARD_INFORMATION`` timestamps and the ``$FILE_NAME`` timestamps in the Master File Table (MFT).

> How to use Analyze MFT

* You can use **FTKImager to parse the $MFT file.

![MACB2](/images/macb2.png)

* If you switch to the directory where you exported ``$MFT``, you won't see it unless you use this command : 

```console
attrib
```
* or you can delete the hidden option using this command : 

```console
attrib -s -h $MFT
```

> How to run Analyze MFT :

```console
analyzeMFT.py -f $MFT -o output.csv -e
```



