---
title: "Hijack | TryHackMe"
collection: publications
permalink: /publication/hijack
excerpt: 'This is a free room from TryHackMe'
date: 2023-10-24
venue: '10-24'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header]()

**LINK :** [Devie](https://tryhackme.com/room/devie)

# Enumeration : 

* Let's start by looking for open ports.

```console

```

* We see few open ports : 
    * ``ftp`` on port 21
    * ``ssh`` on port 22
    * ``http`` on port 80
    * ``nfs`` on port 2049

* Upon visiting the web page, we encountered an ``administration`` page that denied us access, indicating that it's accessible only to admins. This information suggests that there might be an admin account we can target.

[image]

* we proceeded to the login page and attempted to log in with a random username. The system's response was "no account for that username." This response exposes a vulnerability allowing us to enumerate valid usernames on the system.

[image]

* We then tried to log in as 'admin' with a random password, receiving the response, "the password you entered is not valid." This confirms the presence of an 'admin' username on the website.

[image]

* Since we did not have any valid credentials for the web application, we decided to investigate the NFS port, a potential point of entry. 

* By listing the available shares, we discovered a globally accessible file-share at '/mnt/share'.

```console
showmount -e 10.10.X.X
```

[image]

* let's try to mount it 

```console
sudo mount -t nfs 10.10.79.63:/mnt/share nfs_share
```
*  when we tried to access it, we received a 'permission denied' message.

[image]

* To address the permission issue, we examined the ownership and permissions of the mounted directory. We discovered that it was owned by a user with the UID 1003.

> NFS lacks authentication and authorization mechanisms. By creating a local user on our machine with the UID and GID set to 1003, we were able to imitate the owner of the NFS share and inherit their permissions over the directory.

* We created a user with the UID 1003 and GID 1003 on our local system to mimic the share's owner.

```console
sudo useradd fakeuser
sudo usermod -u 1003 fakeuser
sudo groupmod -g 1003 fakeuser
```

* now we login as that user and try to access the mounted share 

```console
su fakeuser
```

* Upon accessing the mounted share, we found a text file containing credentials for an FTP server.

[image]

* With the obtained FTP credentials, we logged in to the FTP server.

* let's list the content of this server :

[image]

* we see that there are 2 hidden text files named ``.from_admin.txt`` and ``passwords_list.txt``.

 ``.from_admin.txt`` : 

 [image]

 ``passwords_list.txt`` : 

 [image]


* These files provided us with crucial information:
    * Two potential usernames: 'admin' and 'rick'.
    * Confirmation that one of the passwords from the list corresponds to the 'admin' account.
    * Knowledge that the website has a brute force protection mechanism in place.

* To verify the website's brute force protection, we attempted to log in as 'admin' with passwords from the list. After five failed login attempts, the site blocked access to the account for five minutes.

[image]

* While technically it was possible to brute force the login page, the rate limiting restriction of 5 failed attempts before a 5-minute lockout would make this a time-consuming process.

* The presence of a signup page caught our attention. We registered a new account through the signup process and successfully logged in.

[image]

* After logging in, we inspected the cookie that was set. The cookie's value appeared to be encoded in base64. We attempted to decode it to verify its contents.

* Upon decoding the cookie, we discovered that its format was 'USERNAME:HASH.' To determine the hash type, we employed the 'hash-identifier' tool, which identified it as an MD5 hash.

* The placement of this hash within the cookie value strongly indicated that it represented the password hash. To confirm our suspicion, we hashed the password we had used to log in with MD5.

* Our suspicion was confirmed when the hash of our password matched the hash value found in the cookie. This validated our understanding that the cookie contained the password hash.


