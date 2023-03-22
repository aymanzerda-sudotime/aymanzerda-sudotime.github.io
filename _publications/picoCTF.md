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