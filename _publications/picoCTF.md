---
title: "PicoCTF2023 writeups"
collection: publications
permalink: /publication/picoCTF
excerpt: 'PicoCTF 2023 competition organized by Carnegie Mellon University ("CMU")'
date: 2023-03-22
venue: '03-22'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

# Web Exploitation 

## Findme : 100 points

![challenge](/images/findme1.png)

* After you login using the credentials provided in the description, you need to intercept the request using Burp

![findme2](/images/findme2.png)

* You will be redirected to a page with an id that contains the first half of the flag encoded with base64

![findme3](/images/findme3.png)

* The response contains the second half of the flag encoded with base64 

* FLAG : 
![findme4](/images/findme4.png)

## MatchTheRegex : 100 points

![challenge](/images/regex1.png)

* The website contains a box where you can submit inputs

![regex2](/images/regex2.png)

* You will find the answer in the source code. 

![regex3](/images/regex3.png)

* If you submit this input, the page will load a popup windows with the flag 

![regex4](/images/regex4.png)

## SOAP : 100 points 

![challenge](/images/soap1.png)

* After you read the description you realise it's a LFI, let's check the website 

![soap2](/images/soap2.png)

* If you click on *Details* and intercept the request, you will notice that the website is using XML so the first idea came to my mind is XXE vulnerability.

![soap3](/images/soap3.png)

* Let's read the /etc/passwd file

![soap4](/images/soap4.png)

## Java Code Analysis!?! : 300 points

![challenge](/images/java1.png)
 
* Let's login

![java2](/images/java2.png)

* If you check the local storage you will find a *token-payload* and *auth-token*

![java3](/images/java3.png)

* You can decode the *auth-token* using https://jwt.io

![java4](/images/java4.png)

* So You need to change 3 values

* You can find the secret key here 

![java5](/images/java5.png)

* I found this but i think they made a mistake because the role *Admin* with id=4 didn't work. the id should be equal to 0

![java6](/images/java6.png)

![java7](/images/java7.png)

* After you change the tokens, you need to refresh the page and NOW you are admin

![java8](/images/java8.png)

* Go to the admin panel 

![java9](/images/java9.png)

* Change the role of *User* to Admin

![java10](/images/java10.png)

* Now try to login as *User* and check the flag book

![java11](/images/java11.png)

# General Skills

## Chrono : 100 points:

![challenge](/images/chrono1.png)

* We know that the **/etc/crontab** file contains the automated tasks

![chrono2](/images/chrono2.png)


## Money-ware : 100 points

![challenge](/images/money-ware1.png)

* From the description, we know that the *1Mz7153HMuxXTuR2R1t78mGSdzaAtNbBWX* is the bitcoin wallet's address
* After a quick research on google, i found this article that talks about the attack

![money-ware2](/images/money-ware2.png)

* The article mentions the name of the malware 

![money-ware3](/images/money-ware3.png)

## Permissions : 100 points

![challenge](/images/permissions1.png)

* After you login via ssh, you should become root to read files in the root directory
* Let's check the sudo permissions 

![permissions2](/images/permissions2.png)

* Let's visit https://gtfobins.github.io to see what we can do if we are allowed to run *Vi* as superuser by sudo

![permissions3](/images/permissions3.png)

* After we execute the command, we are now **root**

![permission](/images/permissions4.png)

## Repetitions : 100 points

![challenge](/images/repetitions1.png)

* Let's check the content of the file 

![repetitions2](/images/repetitions2.png)

* Obviously it's base64 encoding, we can use https://gchq.github.io/CyberChef/ to decode the file

![repetitions3](/images/repetitions3.png)

* The file was encoded 6 times

## Useless : 100 points

![challenge](/images/useless1.png)

* Let's read the content of this executable 

![useless2](/images/useless2.png)

* As you can see we have a manual option, so let's read the manual of this executable

![useless3](/images/useless3.png)


## Specialer : 300 points

![challenge](/images/specialer1.png)

* The first thing to do is to try to understand the system and see what commands you can use

* I noticed that i can use *cd* command, so i kept using *tab* to complete the command and i was able to know that there is a user called *ctf-player*

![specialer2](/images/specialer2.png)

* Let's see what folders we have using the same trick 

![specialer3](/images/specialer3.png)

* The *ala* folder contains 2 files, i was able to read the files using :

```bash
bash -v [filename]
```
![specialer4](/images/specialer4.png)
