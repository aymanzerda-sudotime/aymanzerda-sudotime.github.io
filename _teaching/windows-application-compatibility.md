---
title: "Windows forensics 2 : windows application compatibility"
collection: posts
type: "Info"
permalink: /posts/windows-application-compatibility
venue: "Blue Teaming"
date: 2023-05-01
location: "City, Country"
---

## Description : 
Let's continue our series of posts on Windows forensics, today we're going to talk about additional artifacts that we can use to determine evidence of application execution.

---

* *Amcache.hve* which is a registry hive, it stores more information and metadata about executed programs including sha-1 hash of the binary that was executed. 

```bash
c:\Windows\AppCompat\Programs\
```
![windows-compatibility](/images/windows-compatibility1.png)

* You can grab amcache.hve using *FTK Imager*

> [Download FTK Imager](https://accessdata-ftk-imager.software.informer.com/3.1/)

* We can parse this artifact using a tool named *AmcacheParser*

> [Download AmcachePaser](https://ericzimmerman.github.io/#!index.md)

```bash
AmcacheParser.exe --csv .\ -f .\Amcache.hve
```

![amcacheparser](/images/amcacheparser.png)


* Another artifact named *AppCompatCache* (aka Shimcache) that is written to registry upon system shutdown

```bash
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache
```

* We can view this artifact using *Registry Explorer* 

> [Download Registry Explorer](https://www.sans.org/tools/registry-explorer/)

![appcompatcache](/images/appcompatcache.png)

* Since the informations are not really readable on the registry explorer, we can use *AppCompatCacheParser* to have a better look

> [Download AppCompatCacheParser](https://ericzimmerman.github.io/#!index.md)

```bash
AppCompatCacheParser.exe --csv .\ -t  -f  .\SYSTEM
```

![AppCompatCacheParser](/images/appcompatcacheparser.png)

----

* [Check the previous post](https://aymanzerda-sudotime.github.io/posts/introduction-to-windows-forensics)
