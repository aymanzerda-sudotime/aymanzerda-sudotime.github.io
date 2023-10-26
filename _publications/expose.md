---
title: "TryHackMe: Expose"
collection: publications
permalink: /publication/expose
excerpt: 'This is a subscriber only room from TryHackMe'
date: 2023-09-19
venue: '09-19'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/expose-header.png)

**LINK:** [Expose](https://tryhackme.com/room/expose)

---

## Enumeration : 

* The initial scan of the target revealed 6 open ports

```bash
# nmap -sV -sC -T4 -p- 10.10.191.104
Starting Nmap 7.93 ( https://nmap.org ) at 2023-09-18 08:18 EDT
Nmap scan report for 10.10.191.104
Host is up (0.11s latency).
Not shown: 65529 closed tcp ports (reset)
PORT      STATE    SERVICE                 VERSION
21/tcp    open     ftp                     vsftpd 2.0.8 or later
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.18.0.245
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp    open     ssh                     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 54b1deb55cd1c66348b9425dc5a726cf (RSA)
|   256 0466e91b7928de2252c68c488d259339 (ECDSA)
|_  256 b58adc38ec28fd7249e239f0ff9fe5a6 (ED25519)
53/tcp    open     domain                  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
1337/tcp  open     http                    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: EXPOSED
|_http-server-header: Apache/2.4.41 (Ubuntu)
1883/tcp  open     mosquitto version 1.6.9
35224/tcp filtered unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

* I initiated a directory scan using ``ffuf`` on the web server at port 1337, which yielded several interesting directories.

```bash
ffuf -u http://10.10.197.217:1337/FUZZ -w /usr/share/seclists/Discovery/Web-Content/big.txt


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.197.217:1337/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

[Status: 403, Size: 280, Words: 20, Lines: 10, Duration: 684ms]
    * FUZZ: .htaccess

[Status: 403, Size: 280, Words: 20, Lines: 10, Duration: 185ms]
    * FUZZ: .htpasswd

[Status: 301, Size: 321, Words: 20, Lines: 10, Duration: 183ms]
    * FUZZ: admin

[Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 175ms]
    * FUZZ: admin_101

[Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 119ms]
    * FUZZ: javascript

[Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 121ms]
    * FUZZ: phpmyadmin

[Status: 403, Size: 280, Words: 20, Lines: 10, Duration: 135ms]
    * FUZZ: server-status

:: Progress: [20476/20476] :: Job [1/1] :: 315 req/sec :: Duration: [0:01:29] :: Errors: 0 ::
```
* While i explored the directories, i was unable to find any useful information regarding the ``/admin`` page.
* However, i noticed something intriguing on ``/admin_101``,  the username field was prefilled, indicating that this might be the correct login page.

![expose1](/images/expose1.png)

## User Flag : 

* Lacking valid credentials, i considered a SQL injection attack as our next step.
* i captured the login request with Burp Suite and saved the request to use SQLMap for our attack

![expose2](/images/expose2.png)

* Running SQLMap with the captured request, we successfully dumped the database.

```bash
sqlmap -r req --dump
```

![expose3](/images/expose3.png)

* From the database, i obtained the password for the admin portal and discovered two URLs: `/file1010111/index.php` and `/upload-cv00101011/index.php`.

* i found a hash for `/file1010111/index.php`, which might be crackable.

![expose4](/images/expose4.png)

* As i provided the cracked password, i received another hint to fuzz for parameters.

![expose5](/images/expose5.png)

* Visiting `/upload-cv00101011/index.php` gives us a hint about the password being the name of the machine user starting with the letter Z.

* back to `/file1010111/index.php`, let's check the source code

![expose6](/images/expose6.png)

* the 2nd hint suggests to use `file` or `View` as get parameters, it indicates that maybe this page is vulnerable to local file inclusion and we can read files on the system.

* By adding `?file=../../../../../etc/passwd` to the URL, i successfully read the `/etc/passwd` file and identify a username starts with letter `Z` which is `zeamkish`.

![expose7](/images/expose7.png)

* Upon visiting `/upload-cv00101011/index.php` after providing the username, i encountered a file upload page.

![expose8](/images/expose8.png)

* Examining the source code, i discovered a simple JavaScript filter allowing only JPG or PNG files.

![expose9](/images/expose9.png)

* A quick search on bypassing file upload restrictions led me to a useful [resource](https://null-byte.wonderhowto.com/how-to/bypass-file-upload-restrictions-web-apps-get-shell-0323454/)

* I downloaded a [PHP reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php), renamed it to shell.phpD.jpg, and captured the upload request in Burp Suite.

* Switching to hex format and changed the letter "D" to a null byte `00`

![expose10](/images/expose10.png)

![expose11](/images/expose11.png)

* Starting a netcat listenner and visiting `/upload-cv00101011/upload_thm_1001` and clicking on `shell.php` triggered the payload.

```bash
nc -lvnp 1234
```

![expose12](/images/expose12.png)

* After gaining a shell, i upgraded it by following specific commands

![expose13](/images/expose13.png)

* In `zeamkish's` home directory, i located the flag file that i couldn't access. However, i found the credentials stored for the user in `ssh_cred.txt`

![expose14](/images/expose14.png)

* Logging in as zeamkish via `SSH`, i obtained the user flag.

## Root Flag: 

* Privilege escalation proved to be straightforward. i identified the find command with the SUID bit set

```bash
find / -perm -u=s -type f 2>/dev/null
```

![expose15](/images/expose15.png)

* A visit to [GTFOBins](https://gtfobins.github.io/gtfobins/find/#suid) showed us how to use it to escalate our privileges to root.

![expose16](/images/expose16.png)







