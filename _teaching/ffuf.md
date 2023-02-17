---
title: "ffuf"
collection: posts
type: "Info"
permalink: /posts/ffuf
venue: "Red Teaming"
date: 2023-02-17
location: "City, Country"
---

## Description : 
FFUF or “Fuzz Faster you Fool” is an open source web fuzzing tool, intended for discovering elements and content within web applications, or web servers.

---
Hidden Directories : 
```bash
ffuf -u http://X.X.X.X/FUZZ -w /usr/share/seclists/Discovery/Web-Content/big.txt
```

Extensions : 
```bash
ffuf -u http://X.X.X.X/indexFUZZ -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt
```

Exclude Extensions : 
```bash
ffuf -u http://X.X.X.X/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -e .php,.txt
```

Matching/Filtering : 
**match http status code :**
```bash
ffuf -u http://X.X.X.X/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt -mc 200
```
**filter http status code :**
```bash
ffuf -u http://X.X.X.X/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt -fc 403
```

Discovering vulnerable parametres :
```bash
ffuf -u 'http://X.X.X.X/api/items?FUZZ' -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -fw 39
```
Finding subdomains :
```bash
ffuf -u http://FUZZ.X.X.X.X -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```
---
[ffuf reposiroty](https://github.com/ffuf/ffuf)
[Seclists repository](https://github.com/danielmiessler/SecLists)