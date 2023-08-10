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

This version of wannacry contains the following elements : 

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


* indication that this binary contains three other executables

![executables](/images/indicators.png)

* There are a few cryptographic APIs that are probably used in the encryption routine

![crypto-api](/images/crypto-api.png)

* We know now that the socket api is used for close,receive,send (different types of socket API calls) so maybe the binary may be opening ports and maybe attempting remote connections to different ports.

![network-api](/images/network.png)

* There is some kind of service creation from this binary and perhaps that's a persistence mechanism 

![services](/images/services.png)

* There is an executable inside the ``.rsrc`` section of the file 

![packed-executable](/images/embedded-executable.png)

* Capa was able to find the following indicators 

![capa1](/images/capa1.png)

![capa2](/images/capa2.png)

![capa3](/images/capa3.png)


# Basic dynamic analysis : 

* Wannacry will attempt to contact the URL that we found earlier, if it does make a connection to this weird URL it does not trigger the payload.

* After we have the tcp handshake, we have an http packet that makes a full request to the weird URL.

![weird url](/images/weird-url.png)

* After we stop ``InetSim``, run the following command on Flare-VM :
```bash
ipconfig /flushdns
```

* Now when we run the executable the payload is triggered.

* Let's use ``TCPView`` to see what happens.

![TCPView](/images/smb_connection.png)

* We see a bunch of traffic going out to port 445 to a bunch of remote addresses which means there is no real address connectivity presence to be able to make a connection, so we see how wannacry is trying to propogate itself through network , so this is a ransomware binary but it also has worm capabilities and the way is tring to propogate is through the eternalblue exploit which is an exploit against windows smb of certain versions.


* Let's take a look at the processes created by the malware while executing using ``Procmon``, but first let's apply this filter 

![filter](/images/createfile-filter.png)

![tasksche](/images/tasksche-executable.png)

 * The malware creates a bunch of processes, the interesting one is this ``tasksche.exe``.
 * We can see clearly that the ransomware is unpacking ``tasksche.exe`` with the process tree.

 ![process-tree](/images/process-tree.png)

* The next step is to take a note of the created process's PID and filtering it out as the Parent PID 

![created directory](/images/directory-creation.png)

 * As you can see, the ``tasksche.exe`` is being created again inside a directory with strange name, let's browse to this directory 

 ![hidden directory](/images/hidden-directory.png)

 * After visiting this directory, it appears like this is a staging area for wannacry execution and unpacking all of its packed resources.

* If we open **task manager** and check the services tab, we will find a service with the same name as the directory that was created in ``ProgramData``

![task-manager](/images/task-manager.png)

 * This is a persistence mechanism so if you restart your computer wannacry kicks back and will re-encrypt anything thats been added to the host.

# Advanced static analysis :

* We will use ``Cutter`` to conduct the advanced static analysis, which will confirm what we found in the basic dynamic analysis.

* We see the weird url loaded to ``esi``at the beginning of the program.

![esi](/images/url-moved-to-esi.png)

* We have 3 API calls 

![api calls](/images/api_calls.png)

 * ``InternetOpenA`` prepares the application for future calls.
 * ``InternetOpenUrlA`` tests the connection to www.iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com.
 * If it succeeds in reaching the destination, the program calls ``InternetCloseHandle`` to terminate the Internet connection and exits without malicious action.

 1- ``edi's`` value is compared with its own value, and depending on the result, the program decides which route to take.
 2- If the connection to the url is unsuccessful, we call a function that installs itself as a service, opens up and unpack the rest of the resources and it will kick of the encryption routine.
 3- Otherwise, the program will not run.

* If the program continue to execute, here are the instruction for persistence and the encryption routine.

![instruction](/images/malware-instruction.png)








