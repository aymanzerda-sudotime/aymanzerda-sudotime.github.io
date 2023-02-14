---
title: "Basic Static Analysis"
collection: publications
permalink: /publication/2023-02-14-paper-title-number-1
excerpt: 'This is a subscriber only room from Tryhackme'
date: 2023-02-14
venue: ' '
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---
![header](/images/header.png)

**Link:** [Basic static Analysis](https://tryhackme.com/room/staticanalysis1)
## Description: 
* Learn basic malware analysis techniques without running the malware.

* FLARE VM is attached to this room for performing practical tasks.

---
<span style="color: red;">TASK3: String search : </span>
* Malware authors use several techniques to obfuscate the key parts of their code. These techniques often render a string search ineffective
* **FLOSS** uses several techniques to deobfuscate and extract strings that would not be otherwise found using a string search
  
>there is a directory named 'mal' with malware samples 1 to 6. Use floss to identify obfuscated strings found in the samples named 2, 5, and 6. Which of these samples contains the string 'DbgView.exe'?


```bash
C:\Users\Administrator\Desktop>floss --no-static-strings C:\Users\Administrator\Desktop\mal\6
```

![answer](/images/task3.png)

---
<span style="color: red;">TASK4: Fingerprinting malware : </span>

* The imphash is a hash of the function calls/libraries that a malware sample imports and the order in which these libraries are present in the sample
* We can use **PEstudio** to calculate the Imphash of a sample.

* A Fuzzy Hash is calculated by dividing a file into pieces and calculating the hashes of the different pieces
* We can use **SSDEEP** to calculate the fuzzy hash.

>which of the samples has the same imphash as file 3?

**mal3** imphash:
![file3](/images/impash-of-file-3.png)
**mal1** imphash:
![file1](/images/impash-of-file-1.png)


>Using the ssdeep utility, what is the percentage match of the above-mentioned files?

```bash
ssdeep -l -r -d C:\Users\Administrator\Desktop\mal\1 C:\Users\Administrator\Desktop\mal\3
```
![matches](/images/matches.png)

---
<span style="color: red;">TASK5: Signature based Detection : </span>

* Signatures are a way to identify if a particular file has a particular type of content.

* **Capa** is a tool used to identify the behavior of the file based on signatures such as imports, strings, mutexes, and other artifacts present in the file.

>How many matches for anti-VM execution techniques were identified in the sample?

```bash
C:\Users\Administrator\Desktop>capa  mal\4
```
![anti-vm](/images/anti-Vm.png)

>What MBC behavior is observed against the MBC Objective 'Anti-Static Analysis'?

![MBC](/images/mbc-behavior.png)

>At what address is the function that has the capability 'Check HTTP Status Code'?

```bash
C:\Users\Administrator\Desktop>capa -vv mal\4 > result.txt
```

![http](/images/http.png)

---
<span style="color: red;">TASK5: Leveraging the PE header : </span>

* The PE header contains information about the libraries that a PE file uses and the functions it imports from those libraries.

>Open the sample Desktop\mal\4 in PEstudio. Which library is blacklisted?

![last task](/images/last-task.png)
