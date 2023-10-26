---
title: "TryHackMe: Capture "
collection: publications
permalink: /publication/capture
excerpt: 'This is a free machine from TryHackMe'
date: 2023-05-08
venue: '05-08'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![Header](/images/capture-header.png)

**LINK :**  [Capture room](https://tryhackme.com/room/capture)

---

* Let's scan our target.

```bash
# nmap -sC -sV -T4 10.10.78.63
Starting Nmap 7.92 ( https://nmap.org ) at 2023-05-06 16:33 
Nmap scan report for 10.10.78.63
Host is up (0.10s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Werkzeug/2.2.2 Python/3.8.10
```
* We have one open port which is HTTP server running on port 80.

* After you navigate to the web page you will be redirected to a login page **http://10.10.X.X/login**

![Capture1](/images/capture1.png)

* After you try to login using a random username, you will see an error message "The user 'X' does not exist", so the applicaton is vulnerable to username enumeration.

![Capture2](/images/capture2.png)

* Since the the room comes with two attached files that contains usernames list and passwords list, we will use them to find a valid username then we will start brute forcing the password with that valid username.

* After 10 unsuccessful login attempts, a captcha will be enabled.

![capture3](/images/capture3.png)

* I created a python script to automate the process of username enumeration, password brute forcing and solving mathematical-based captcha.

* You can find the script here [Python script](https://github.com/aymanzerda-sudotime/Tryhackme-Capture)

* So basically this python script automates the process of enumerating the username, if the response contains "Captcha enabled", the eval() method will answer the mathematical operation.


> The eval() method parses the expression passed to this method and runs python expression (code) within the program.

![username-enumeration](/images/capture-username.png)


* After successfully identifying a valid username, the script will initiate a brute force attack to find the password associated with that username.

![password-bruteforcing](/images/capture-password.png)

* Use these credentials to login and get your flag

![flag](/images/capture-flag.png)