---
title: "Malware analysis: Wannacry"
collection: posts
type: "Info"
permalink: /posts/wannacry
venue: "Blue Teaming"
date: 2023-08-08
location: "City, Country"
---

## Description : 
Today i will make a triage report for the infamous ``Wannacry`` ransomware.

[Wannacry sample](https://github.com/HuskyHacks/PMAT-labs/tree/main/labs/4-1.Bossfight-wannacry.exe)
---

# Executive Summary : 

Wannacry is a ransomware that targets Windows operating systems. It encrypts computer files, modifies the wallpaper and then displays a ransom note in which instructions are given to send payment in bitcoins if you want to decrypt your files. In addition, it possesses persistence mechanisms that allow it to remain on the computer even after a reboot, as well as worm capabilities that allow it to spread to other computers on your network. This malware includes a kill switch mechanism that prevents it from executing on a virtual machine,  when it successfully connects to a specific URL, the malware does not execute.


# High-Level Technical Summary : 

* In the first stage, the ransomware will attempt to contact this URL "hxxp[://]www[.]iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea[.]com".
* If it receives a response from this domain (200 OK), the malware ends its routine and remains inactive, it acts as a ``kill-switch``. 
* If the domain is inaccessible, the program executes its main function, this involves unpacking additional executables hidden in the    file, one of which is an encryptor that encrypts files on the victim machine.
* In addition, a decryptor program is installed on the desktop, which is executed after the infection. The decryption program includes the ransom note and instructions on how to pay the ransom and decrypt system files. The program also displays two timers, indicating deadlines for doubling the ransom amount and the final deadline for paying the ransom and recovering the files.
* The ransomware establishes multiple connections to various IP addresses on port 445, which is typically used for the SMB protocol. It exploits the ``EternalBlue`` vulnerability to propagate.


# Malware composition : 

This version of wannacry contains the following elemnts : 

* Ransomware.wannacry.exe which is the initial executable.
* Unpacked executables after the initial execution

# Basic Static analysis : 

| Hash type | sum                                                               |
| --------  | ------------------------------------------------------------------|
| md5       | db349b97c37d22f5ea1d1841e3c89eb4                                  |
| sha1      | e889544aff85ffaf8b0d0da705105dee7c97fe26                          |
| sha256    | 24d004a104d4d54034dbcffc2a4b19a11f39008a575aa614ea04703480b1022c  |


* Using ``FLOSS`` and ``PEStudio`` and ``capa``, i found several indications of potentially malicious activity.

* The original filename :

![original filename](/images/original_name.png)

* Some interesting strings were found : 

| String                                                         | Indication                                                        |
| ---------------------------------------------------------------|-------------------------------------------------------------------|
| hxxp[://]www[.]iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea[.]com | the defang url                                                    |
| cmd.exe /c "%s"                                                | a command                                                         |
| icacls . /grant Everyone:F /T /C /Q                            | granting access to everyone on the current working directory      |
| attrib +h                                                      | set a directory as hidden                                         |
| !This program cannot be run in DOS mode. (x4)                  | there are other executables or DLL that are loaded in             |
| C:\%s\qeriuwjhrf                                               | suspicious folder name                                            |
| CryptEncrypt,CryptDecrypt,CryptGenKey…                         | cryptography calls                                                |






