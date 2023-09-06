---
title: "My Summer Internship"
collection: posts
type: "Info"
permalink: /posts/summer-internship
venue: "Malware Analysis"
date: 2023-09-04
location: "City, Country"
---

## Description : 
During my internship at Techso Group, I had the opportunity to delve into the intriguing world of malware analysis. Over the course of the summer, I gained a wealth of knowledge and hands-on experience that shed light on the intricacies of this ever-evolving field. In this blog post,  I'll share the key takeaways and insights that shaped my understanding of malware analysis.

1-  Malware Analysis - A Tricky Endeavor : 

From day one, I realized that malware analysis is not for the faint of heart. It's a challenging and ever-evolving field where cybercriminals constantly adapt and innovate. To unravel their nefarious creations, I needed to arm myself with knowledge and tools.

2-  The Importance of Sandboxing : 

Handling malware samples requires a secure environment to prevent any unintended consequences. Sandboxing was the key to safely studying malware without compromising the integrity of the host system. Learning how to set up and use a sandbox was a crucial step in the process.

3- Safely Obtaining Malware Samples

4- Basic malware analysis : 

I learned the fundamentals of malware analysis, such as examining strings, inspecting Windows API calls, identifying packed malware, and uncovering host-based signatures. These skills provided me with insights into the malware's behavior and purpose.

5- Advanced malware analysis : 

Using tools like Cutter and x64dbg to dive deep into malware samples. This allowed me to control execution flow and defeat anti-analysis techniques, giving me a deeper understanding of how malware operates.

6- Assembly Language : 

Learning assembly language was a game-changer. It opened doors to advanced analysis methods, enabling me to dissect malware at the lowest level. Understanding assembly language was like deciphering the code of the digital underworld.

7- Real World Analysis : 

The pinnacle of my internship was putting all these learned techniques to the test by conducting a full analysis of different malware samples from the wild. It was a challenging yet rewarding experience that allowed me to apply my newfound expertise to real-world scenarios.

--------

## Safe Malware Sourcing & Additional Resources :

One of the most critical aspects of malware analysis is safely sourcing malware samples. Handling malware improperly can lead to the inadvertent infection of your actual machine, which is something you definitely want to avoid. Here are some trusted sources where you can find malware samples for analysis:

* [PMAT LABS](https://github.com/HuskyHacks/PMAT-labs) : PMAT LABS is a repository that contains live malware samples specifically created for the PMAT course by TCM. These samples are ideal for educational purposes and allow you to practice analyzing malware in a controlled environment.

* [theZoo](https://github.com/ytisf/theZoo) : theZoo is a fascinating project created to democratize malware analysis. It provides a wide range of malware samples and makes them openly accessible to the public. It's an excellent resource for both beginners and experienced analysts.

* [vxunderground](https://www.vx-underground.org/) : vxunderground is a treasure trove for malware analysts. This website offers a comprehensive collection of malware samples and research materials. It's a valuable resource for anyone looking to dive deep into the world of malware analysis.

* [MalwareBazaar](https://bazaar.abuse.ch/) : MalwareBazaar is a database that hosts a vast repository of submitted malware samples. It's a trusted source for analysts to access a diverse range of malware for research and analysis purposes.

--------

## Sandboxing :

Sandboxing is an essential technique in malware analysis, providing a secure environment to execute and analyze malicious software without compromising the integrity of the host system. This approach allows analysts to observe the behavior of malware in a controlled setting. Here are some insights into sandboxing and the tools I explored:

### Cuckoo Sandbox : 

Cuckoo Sandbox is a powerful open-source tool designed to automate the analysis of malicious files on various operating systems, including Windows, macOS, and Linux. While setting up Cuckoo Sandbox can be challenging, the benefits are substantial. It provides detailed reports on malware behavior and activities.

Resources for setting up Cuckoo Sandbox:

* Official documentation: [Cuckoo Sandbox Documentation](https://cuckoo.readthedocs.io/en/latest/)
* Video tutorial: [Cuckoo Sandbox Installation Guide](https://www.youtube.com/watch?v=fbt4fk5qiow)
* GitHub Demo: [Cuckoo Install Demo](https://github.com/OpenSecureCo/Demos/blob/main/Cuckoo%20Install)

> When configuring Cuckoo Sandbox, you might encounter various errors, but don't be discouraged. Google is your friend, and online communities can provide valuable assistance.

### Online Sandboxing Services : 

If setting up Cuckoo Sandbox seems daunting, several online sandboxing services can help you analyze malware without the need for complex installations. Some notable options include:

* [Hybrid Analysis](https://hybrid-analysis.com/)
* [Tria.ge](https://tria.ge/)
* [Cuckoo Sandbox Online](https://cuckoo.cert.ee/) (though not as robust as the offline version)
* [any.run](https://any.run/)

-----

## Building your Malware Analysis Lab : 

While automated tools like Cuckoo Sandbox are excellent for initial analysis, they have their limitations. To gain a comprehensive understanding of malicious behaviors, manual analysis is often necessary. Building your malware analysis lab is a valuable step toward achieving this depth of understanding. Here's a brief overview:

* Flare-VM : Flare-VM is a Windows-based virtual machine created by FireEye's Mandiant team. It's pre-configured with numerous security and malware analysis tools, making it a convenient choice for setting up your analysis environment.

* Remnux : created by the malware analysis expert Lenny Zeltser, is a Linux distribution specifically designed for reverse engineering and analyzing malicious software. It contains a vast array of pre-installed tools and scripts tailored to malware analysis tasks.

To get started on building your malware analysis lab, you can follow video tutorials and online resources like this one [Building a Malware Analysis Lab](https://www.youtube.com/watch?v=rmSIm3BKu3Y)

-----

## Basic Static Analysis : 

Static analysis involves examining the properties of malware without executing it. This approach is crucial for understanding the potential risks and characteristics of a given piece of malicious code.

* Fingerprinting Malware Through Hashes : Hashes are unique identifiers generated from the contents of a file. There are various methods to compute hashes, with the most commonly used ones being ``md5sum``, ``sha1sum``, and ``sha256sum``. These commands were readily available within tools like Flare-VM and REMnux. Once you have the hash, you can submit it to services like [VirusTotal](https://www.virustotal.com/gui/home/search) to check if it has already been analyzed and identified as malicious.

* String search : Another essential technique in static analysis is string searching within the malware. This involves extracting arrays of characters from the malware's file to discover valuable information. During this process, I learned to look for specific artifacts, such as:
   * Windows Functions and APIs
   * IP Addresses, URLs, or Domains
   * Miscellaneous strings like Bitcoin addresses or text used in message boxes

Flare-VM provided a useful utility called ``strings`` to perform this task. However, some malware authors employ techniques to obfuscate strings, making them harder to detect. To tackle this challenge, I utilized ``FLOSS``, which is included with Flare-VM, to deobfuscate and extract hidden strings that would otherwise remain concealed.

[PEStudio](https://www.winitor.com/download) is a versatile and indispensable asset in the malware analysis toolkit. It offers a comprehensive overview of a file, providing valuable information about hashes, strings, API calls, libraries, and imports. This tool significantly simplifies the process of dissecting malicious binaries.

* Signatures-Based Detection : Security researchers often use signatures to identify patterns in files and detect malicious behavior. Two significant tools that I explored were Yara rules and CAPA:

  * Yara rules : help identify information based on binary and textual patterns. I found a treasure trove of Yara rules in this [repository](https://github.com/Yara-Rules/rules), which proved immensely helpful for my analysis.

  * CAPA: another tool included with Flare-VM, assists in identifying the capabilities of a PE file. It reads files and identifies behaviors based on signatures, such as imports, strings, mutexes, and other artifacts present within the file.

------

## Basic Dynamic Analysis :

No matter how cunning malware can be in evading static analysis, it inevitably leaves traces when it executes. These traces, if scrutinized effectively, can unveil its malicious intent. This is where dynamic analysis comes into play.

the key aspects we need to pay attention to when conducting such an analysis: 

* Process Activities : When malware is executed, it initiates a flurry of activities within the host operating system. Examining these processes is a pivotal part of dynamic analysis. Here are some crucial aspects to consider 
   * **Child Processes:** Malware often spawns new processes. Analyzing whether a process creates child processes can provide essential insights into its behavior.
   * **Imported DLLs:** Take note of the dynamic link libraries (DLLs) a process imports. These libraries can reveal the functions the malware intends to use.
   * **User Context:** Determine which user account is running the process. Malware may attempt to execute in the context of specific users for stealth or privilege escalation.

> Tools like ``Process Hacker`` and ``Procmon`` can be invaluable for dissecting processes during dynamic analysis.

* Network Activities: Malware frequently establishes network connections to carry out its nefarious activities, such as downloading additional payloads, communicating with command and control servers, or exfiltrating data.

> ``Wireshark`` and ``TCPView`` are powerful network analysis tools that allow you to capture and analyze packets, giving you visibility into the traffic generated by the malware.

* Registry Activities:

Registry modifications are a favorite tactic among malware authors to ensure persistence and steal data. Certain registry keys are particularly attractive to attackers, including:
   * HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
   * HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce
   * HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
   * HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce

> To detect changes made by malware to these registry keys, tools like ``Regshot`` can be invaluable. ``Regshot`` captures snapshots of the registry before and after malware execution, helping you pinpoint any suspicious alterations.

----

## Advanced Static Analysis : 

Advanced static analysis is a crucial aspect of malware analysis, as it enables analysts to dissect malicious code without executing it. This approach is particularly important when dealing with sophisticated malware that is designed to evade traditional detection methods. Here are some of the key steps involved in performing advanced static analysis of malware:

* Identifying the Entry Point of the Malware: This is the portion of the code where execution begins when the malware is activated. Understanding the entry point helps analysts gain insight into the malware's functionality and behavior.

* Analyzing System Calls: Malware often interacts with the operating system through system calls. By monitoring and analyzing the system calls made by the malware, analysts can uncover important clues about its intentions. This step can reveal whether the malware is trying to access sensitive data, manipulate files, or perform other malicious actions.

* Control Flow Graph Analysis: This graph illustrates the possible paths that the code can take during execution. Analyzing the control flow graph helps analysts identify the malware's execution path and understand how different parts of the code interact with each other.

> There are several powerful tools available for this purpose like ``Cutter``, ``IDA PRO`` and ``Ghidra``, each with its own unique features and capabilities. 

----- 

## Advanced Dynamic Analysis : 

* malware authors are well aware that their malicious creations will be scrutinized by cybersecurity experts. To counter this, they employ various evasion techniques designed to make their malware as elusive as possible. These evasion techniques can range from obfuscating code to employing anti-analysis tricks.

* To successfully analyze and combat malware, it is imperative for a malware analyst to understand and overcome these evasion techniques. This involves gaining control over the malware's execution to closely monitor its behavior and identify its malicious activities.

* Debuggers play a pivotal role in providing malware analysts with the level of control necessary to closely examine the behavior of a running program. Interactive debugging enables analysts to monitor changes in registers, variables, and memory regions as each instruction is executed, shedding light on the malware's inner workings.

* In the world of malware analysis, choosing the right debugger can significantly impact the effectiveness of your analysis. There are several options available, each with its strengths and capabilities. Some of the popular choices include ``Windbg``, ``Ollydbg``, ``IDA``, and ``Ghidra``. However, during my internship, I primarily relied on ``x32dbg`` and ``x64dbg`` due to their user-friendly interfaces and robust features.

------

## Real world Analysis : 

To demonstrate what i learned, I chose the infamous ransomware WannaCry as my subject of study. If you haven't already, you can read the detailed post about [WannaCry](https://aymanzerda-sudotime.github.io/posts/wannacry).

---- 

Here's a list of the resources I found particularly useful :

[Practical Malware Analysis & Triage by TCM Security](https://academy.tcm-sec.com/p/practical-malware-analysis-triage)

[Intro to malware analysis](https://tryhackme.com/room/intromalwareanalysis)

[Basic static analysis](https://tryhackme.com/room/staticanalysis1)

[Basic dynamic analysis](https://tryhackme.com/room/basicdynamicanalysis)

[Advanced static analysis](https://tryhackme.com/room/advancedstaticanalysis)

[Debugging](https://tryhackme.com/room/advanceddynamicanalysis)

[Anti-Reverse Engineering](https://tryhackme.com/room/antireverseengineering)

[X86 Assembly](https://tryhackme.com/room/x86assemblycrashcourse)

[PE Headers](https://tryhackme.com/room/dissectingpeheaders)




