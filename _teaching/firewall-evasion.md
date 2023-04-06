---
title: "Firewall Evasion"
collection: posts
type: "Info"
permalink: /posts/firewall-evasion
venue: "Red Teaming"
date: 2023-04-06
location: "City, Country"
---

## Description : 

Firewall evasion is the process of bypassing or circumventing security measures implemented by a firewall in order to gain unauthorized access to a network or system. It involves finding and exploiting vulnerabilities in the firewall's rules or configuration to allow malicious traffic to pass through undetected.

---

* Hide a scan with decoys :

```bash
nmap -sS -D DECOY_IP1,DECOY_IP2,... <target>
```

> Decoys are used to conceal the true source IP address of the Nmap scan. By sending packets from multiple decoy IP addresses in addition to the real source IP address, it becomes difficult for a target system to determine the true origin of the scan

* Hide a scan with random decoys : 

```bash
nmap -sS -D RND,RND,ME <target>
```

* Use an HTTP/SOCKS4 proxy to relay connections : 

```bash
nmap -sS --proxies PROXY_URL <target>
```

> Asks Nmap to establish TCP connections with a final target through supplied chain of one or more HTTP or SOCKS4 proxies. Proxies can help hide the true source of a scan or evade certain firewall restrictions

* Spoof source MAC address :

```bash
nmap -sS --spoof-mac MAC_ADDRESS <target>
```

> Asks Nmap to use the given MAC address for all of the raw ethernet frames it sends

* Spoof source IP Address :

```bash
nmap -S IP_ADDRESS <target>
```

> Specify the IP address of the interface you wish to send packets through

* Specify source port number : 

```bash
nmap -g PORT_NUMBER or nmap --source-port PORT_NUMBER
``` 

* Fragment IP data into 8 bytes :

```bash
nmap -sS -f <target>
```

>The idea is to split up the TCP header over several packets to make it harder for packet filters, intrusion detection systems, and other annoyances to detect what you are doing.

* Fragment IP data into 16 bytes

```bash
nmap -sS -ff <target>
```

* Fragment packets with given MTU :

```bash
nmap -sS --mtu VALUE <target>
```

> Specify your own offset size


* Specify packet length :

```bash
nmap -sS --data-length NUM <target>
```

> It slows things down a little, but can make a scan slightly less conspicuous.

* Set IP time-to-live field

```bash
nmap -sS --ttl VALUE <target>
```

* Send packet with a wrong TCP/UDP checksum :

```bash
nmap -sS --badsum <target>
```

> Since virtually all host IP stacks properly drop these packets, any responses received are likely coming from a firewall or IDS that didn't bother to verify the checksum


### References : 
* https://nmap.org/book/man-bypass-firewalls-ids.html
* https://tryhackme.com/room/redteamfirewalls
