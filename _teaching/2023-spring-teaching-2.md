---
title: "WPScan"
collection: posts
type: "Info"
permalink: /posts/2023-spring-teaching-2
venue: "Red Teaming"
date: 2023-02-13
location: "City, Country"
---

## Description:
WordPress Security Scanner(WPScan) is used to test WordPress installations and WordPress-powered websites, it enumerates details and checks them against its database of vulnerabilities and exploits.

![WPScan](/images/wpscan_logo.png)
---
Checks for themes :
```bash
wpscan --url http://example.com/ --enumerate t
```

Checks for Plugins :
```bash
wpscan --url http://example.com/ --enumerate p
```

Checks for users :
```bash
wpscan --url http://example.com/ --enumerate u
```

Checks for vulnerable plugins :
```bash
wpscan --url http://example.com/ --enumerate vp
```

Password attacks :
```bash
wpscan --url http://example.com/ --passwords WORDLIST --usernames USERNAME
```

Adjusting WPScan's Aggressiveness (WAF) :
```bash
--plugins-detection (aggressive or passive)
```

Bypassing simple WAFs :
```bash
--random-user-agent
```
---
Some rooms from Tryhackeme where you can practice using WPScan :
* [Blog](https://tryhackme.com/room/blog)
* [ColddBox: Easy](https://tryhackme.com/room/colddboxeasy)
* [All in One](https://tryhackme.com/room/allinonemj)
* [Jeff](https://tryhackme.com/room/jeff)
* [Jack](https://tryhackme.com/room/jack)

--- 
[WPScan Repository](https://github.com/wpscanteam/wpscan)