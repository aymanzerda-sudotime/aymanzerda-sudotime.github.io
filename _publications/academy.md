---
title: "Academy | Hackthebox "
collection: publications
permalink: /publication/academy
excerpt: 'This is a retired machine from HackTheBox'
date: 2023-08-16
venue: '08-16'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/academy-header.png)

**LINK:** [Academy](https://app.hackthebox.com/machines/Academy)

---

## Enumeration : 

* Let's scan the target with nmap :

```bash
nmap -A -p- 10.10.10.165
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-16 06:19 EDT
Nmap scan report for 10.10.10.165
Host is up (0.19s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 ubuntu 4ubuntu0.1 (Ubuntu Linux) (protocol 2.0)
| ssh-hostkey: 
|   3072 aa99a81668cd41ccf96c8401c759095c (RSA)
|   256 93dd1a23eed71f086b58470973a388cc (ECDSA)
|_  256 9dd6621e7afb8f5692e637f110db9bce (ED25519)
80/tcp open  http    Apache httpd 2.4.41((ubuntu)) 
|_http-server-header: Apache/2.4.41
|_http-title: Did not follow redirect to http://academy.htb/
33060/tcp open   mysqlx?
```

* The scan reveals 3 open ports : 
    * ``OpenSSH`` server running on port 22.
    * ``Apache`` web server running on port 80.
    * ``MySQL`` server running on port 33060

* Visiting the website on port 80 redirects us to ``http://academy.htb``, let's add it to ``/etc/hosts``

```bash
echo "10.10.10.215 academy.htb" >> /etc/hosts
```
![academy1](/images/academy1.png)

* The webiste has a ``login`` and ``register`` page, after registering the website redirects us to a welcome page then a login page.

* I spent some times looking for potential features that can be exploited but unfortunately there wasn't.

* Let's see if we can find any hidden directories or files.

```bash
└─# gobuster dir -u http://academy.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x html,php
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://academy.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              html,php
[+] Timeout:                 10s
===============================================================
2023/08/16 11:23:50 Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 276]
/.html                (Status: 403) [Size: 276]
/images               (Status: 301) [Size: 311] [--> http://academy.htb/images/]
/index.php            (Status: 200) [Size: 2117]
/home.php             (Status: 302) [Size: 55034] [--> login.php]
/login.php            (Status: 200) [Size: 2627]
/register.php         (Status: 200) [Size: 3003]
/admin.php            (Status: 200) [Size: 2633]
```
* the interesting page is ``admin.php``, browsing to it reveals a new login page, i tried to login with the registered user but it didnt work.

* Back to the registration page, let's try to create a new user but this time we will intercept the request with ``burpsuite``.

![academy2](/images/academy2.png)

![academy3](/images/academy3.png)

* We can see a third parameter ``roleid``, as the name suggests it might be the parameter that is responsible for the user's permission, let's try to change it from 0 to 1 and see if we can login with the new user at ``/admin.php``.

![academy4](/images/academy4.png)

![academy5](/images/academy5.png)

* After logging we are prompted with this ``Academy Launch Planner``, we have now a new hostname so let's add it to ``/etc/hosts``.

```bash
echo "10.10.10.215 dev-staging-01.academy.htb" >> /etc/hosts
```

* Visiting ``dev-staging-01.academy.htb``, we have a lot of indicators that ``Laravel`` framework is running behind the scenes, i found the APP key but i couldn't find the version.

![academy6](/images/academy6.png)

* I did some research and i found that a specific version of ``Laravel`` is vulnerable to ``RCE``.

> [CVE-2018-15133](https://github.com/aljavier/exploit_laravel_cve-2018-15133)

* Even though i wasn't sure about the version, the exploit that i found explains that it's possible to get remote code execution if we have the Laravel API key so obviously thats why they gave us the key.

![meme](/images/academy-meme.png)


* Let's run the exploit and see if we can execute a command.

![academy7](/images/academy7.png)

* It's working, let's run it with the interactive option and spawn a reverse shell. But first lets start a netcat listener.

```bash
nc -lvnp 4444
```

![academy8](/images/academy8.png)

# User Flag : 

* I wasn't able to read the ``user.txt`` file.

![academy9](/images/academy9.png)

* I found the main application at ``/var/www/html/academy`` which contains the ``.env`` configuration file.

![academy10](/images/academy10.png)

* these credentials didnt work with ``MySQL``, but i was able to switch to ``cry0l1t3`` user with the password that i found earlier.

![academy11](/images/academy11.png)

# Root Flag : 

* ``cry0l1t3`` is part of a group named ``adm`` 

![academy12](/images/academy12.png)

* A quick research reveals that the ``adm`` groups can read many log files.

![academy13](/images/academy13.png)

* Instead of checking all of those log files which is quite challenging, i found this tool called ``aureport`` that produces summary reports of the audit system logs, it has an option to check the logs for the terminal sessions.

![academy14](/images/academy14.png)

* The ``TTY`` report reveals the password for the user ``mrb3n``.

![academy15](/images/academy15.png)

* Let's check the ``sudo`` permissions for the user ``mrb3n``.

![academy16](/images/academy16.png)

* the user ``mrb3n`` can run ``composer`` as root. time to pay [GTFObins](https://gtfobins.github.io/gtfobins/composer/#sudo) a visit.

![academy17](/images/academy17.png)

* After inputting these commands, i got a shell as ``root``

![academy18](/images/academy18.png)









