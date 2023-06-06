---
title: "Windows forensics 3 : System Resource Utilization Monitor"
collection: posts
type: "Info"
permalink: /posts/srum
venue: "Blue Teaming"
date: 2023-06-06
location: "City, Country"
---

## Description :

Today we're going to see how wen can retrive a wealth of information about all the activities that occur on your machine.

---

## What is SRUM:
* The Windows System Resource Usage Monitor (aka *SRUM*) is designed to track system resource utilization things like CPU cycles,network activity,power consumption.

## File Location:

```bash
c:\Windows\System32\sru\SRUMDB.dat
```

## How to copy the file in order to parse it:

* You can grab SRUMDB.dat using *FTK Imager*

> [Download FTK Imager](https://accessdata-ftk-imager.software.informer.com/3.1/)

## How to parse the SRUMDB.dat file : 

* SRUM Dump extracts information from the System Resource Utilization Management Database and creates a Excel spreadsheet.

> [SRUM Dump](https://github.com/MarkBaggett/srum-dump)

## How to use srum_dump:

```bash
srum_dump.exe -i SRUMDB.dat -o outfile.xlsx -t SRUM_TEMPLATE.xlsx -r SOFTWARE
```

![SRUM](/images/srum.png)

* You can get user name from SID using this command

```bash
wmic useraccount get name,sid
```

--- 

[more infos about SRUM](https://isc.sans.edu/diary/System+Resource+Utilization+Monitor/21927)



