---
title: "Volatility"
collection: posts
type: "Info"
permalink: /posts/volatility
venue: "Blue Team"
date: 2023-02-27
location: "City, Country"
---

![volatility](/images/volatility3.jpg)

## Description :
Volatility is a free memory forensics tool developed and maintained by Volatility Foundation, commonly used by malware and SOC analysts within a blue team or as part of their detection and monitoring solutions.

--- 

### installation :
```bash
git clone https://github.com/volatilityfoundation/volatility3.git
cd volatility3
python3 vol.py -h
```

---

* Identify image info and profiles :
```bash
python3 vol.py -f 'dump.vmem' windows.info
python3 vol.py -f 'dump.vmem' Imageinfo
```

* Listing processes and connections :
```bash
python3 vol.py -f 'dump.vmem' windows.pslist
```

* Some malware, typically rootkits in an attempt to hide their processes, unlike itself from the list but we can find them :

```bash
python3 vol.py -f 'dump.vmem' windows.psscan
```

* Listing all processes based on their parent process id :
```bash
python3 vol.py -f 'dump.vmem' windows.pstree
```

* identify all memory structures with a network connection :
```bash
python3 vol.py -f 'dump.vmem' windows.netstat
```

* identify injected processes : 
```bash
python3 vol.py -f 'dump.vmem' windows.malfind
```

* can for drivers present on the system at the time of extraction :
```bash
python3 vol.py -f dump.vmem windows.modules
or 
python3 vol.py -f dump.vmem windows.driverscan
```

* Hash dump : 
```bash
python3 vol.py -f dump.vmem --profile=X hashdump
```

* Last shutdown : 
```bash
python3 vol.py -f dump.vmem --profile=X shutdowntim
```

--- 
Some rooms from Tryhackme where you can practice :
* [Memory forensics](https://tryhackme.com/room/memoryforensics)
* [Volatility](https://tryhackme.com/room/volatility)
* [REMnux](https://tryhackme.com/room/malremnuxv2)

---

[Volatility repository](https://github.com/volatilityfoundation/volatility3)