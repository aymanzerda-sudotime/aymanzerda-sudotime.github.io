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

7- Real Word Analysis : 

The pinnacle of my internship was putting all these learned techniques to the test by conducting a full analysis of different malware samples from the wild. It was a challenging yet rewarding experience that allowed me to apply my newfound expertise to real-world scenarios.

## Safe Malware Sourcing & Additional Resources :

One of the most critical aspects of malware analysis is safely sourcing malware samples. Handling malware improperly can lead to the inadvertent infection of your actual machine, which is something you definitely want to avoid. Here are some trusted sources where you can find malware samples for analysis:

* [PMAT LABS](https://github.com/HuskyHacks/PMAT-labs) : PMAT LABS is a repository that contains live malware samples specifically created for the PMAT course by TCM. These samples are ideal for educational purposes and allow you to practice analyzing malware in a controlled environment.

* [theZoo](https://github.com/ytisf/theZoo) : theZoo is a fascinating project created to democratize malware analysis. It provides a wide range of malware samples and makes them openly accessible to the public. It's an excellent resource for both beginners and experienced analysts.

* [vxunderground](https://www.vx-underground.org/) : vxunderground is a treasure trove for malware analysts. This website offers a comprehensive collection of malware samples and research materials. It's a valuable resource for anyone looking to dive deep into the world of malware analysis.

* [MalwareBazaar](https://bazaar.abuse.ch/) : MalwareBazaar is a database that hosts a vast repository of submitted malware samples. It's a trusted source for analysts to access a diverse range of malware for research and analysis purposes.

## Sandboxing 

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

## Building your Malware Analysis Lab : 

While automated tools like Cuckoo Sandbox are excellent for initial analysis, they have their limitations. To gain a comprehensive understanding of malicious behaviors, manual analysis is often necessary. Building your malware analysis lab is a valuable step toward achieving this depth of understanding. Here's a brief overview:

* Flare-VM : Flare-VM is a Windows-based virtual machine created by FireEye's Mandiant team. It's pre-configured with numerous security and malware analysis tools, making it a convenient choice for setting up your analysis environment.

* Remnux : created by the malware analysis expert Lenny Zeltser, is a Linux distribution specifically designed for reverse engineering and analyzing malicious software. It contains a vast array of pre-installed tools and scripts tailored to malware analysis tasks.

To get started on building your malware analysis lab, you can follow video tutorials and online resources like this one [Building a Malware Analysis Lab](https://www.youtube.com/watch?v=rmSIm3BKu3Y)




