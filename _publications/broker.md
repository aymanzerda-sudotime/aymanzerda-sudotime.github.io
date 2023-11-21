---
title: "HackTheBox: Broker"
collection: publications
permalink: /publication/broker
excerpt: 'This is a retired machine from HackTheBox'
date: 2023-11-21
venue: '11-21'
#paperurl: 'http://academicpages.github.io/files/paper1.pdf'
#citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---

![header](/images/broker_header.png)

**LINK:** [Broker](https://app.hackthebox.com/machines/Broker)

---

## Enumeration : 

* Let's start off with an nmap scan :

```bash
nmap -p- 10.10.11.243                                                       
Starting Nmap 7.80 ( https://nmap.org ) at 2023-11-21 16:58 EST
Nmap scan report for 10.10.11.243
Host is up (0.097s latency).
Not shown: 65526 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
1883/tcp  open  mqtt
5672/tcp  open  amqp
8161/tcp  open  patrol-snmp
39751/tcp open  unknown
61613/tcp open  unknown
61614/tcp open  unknown
61616/tcp open  unknown
```
* Full Scna : 

```bash
nmap -p 22,80,1883,5672,8161,39751,61613,61614,61616 -sCV 10.10.11.243
Starting Nmap 7.80 ( https://nmap.org ) at 2023-11-21 17:00 EST
Nmap scan report for 10.10.11.243
Host is up (0.092s latency).

PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)                                                    
80/tcp    open  http       nginx 1.18.0 (Ubuntu)
| http-auth:
| HTTP/1.1 401 Unauthorized\x0D
|_  basic realm=ActiveMQRealm
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Error 401 Unauthorized
1883/tcp  open  mqtt
|_mqtt-subscribe: ERROR: Script execution failed (use -d to debug)
5672/tcp  open  amqp?
|_amqp-info: ERROR: AQMP:handshake expected header (1) frame, but was 65
| fingerprint-strings:
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GetRequest, HTTPOptions, RPCCheck, RTSPRequest, SSLSessionReq, TerminalServerCookie:        
|     AMQP
|     AMQP
|     amqp:decode-error
|_    7Connection from client using unsupported AMQP attempted
8161/tcp  open  http       Jetty 9.4.39.v20210325
| http-auth:
| HTTP/1.1 401 Unauthorized\x0D
|_  basic realm=ActiveMQRealm
|_http-server-header: Jetty(9.4.39.v20210325)
|_http-title: Error 401 Unauthorized
39751/tcp open  tcpwrapped
61613/tcp open  unknown
| fingerprint-strings:
|   HELP4STOMP:
|     ERROR
|     content-type:text/plain
|     message:Unknown STOMP action: HELP
|     org.apache.activemq.transport.stomp.ProtocolException: Unknown STOMP action: HELP                                                                       
|     org.apache.activemq.transport.stomp.ProtocolConverter.onStompCommand(ProtocolConverter.java:258)                                                        
|     org.apache.activemq.transport.stomp.StompTransportFilter.onCommand(StompTransportFilter.java:85)                                                        
|     org.apache.activemq.transport.TransportSupport.doConsume(TransportSupport.java:83)
|     org.apache.activemq.transport.tcp.TcpTransport.doRun(TcpTransport.java:233)
|     org.apache.activemq.transport.tcp.TcpTransport.run(TcpTransport.java:215)
|_    java.lang.Thread.run(Thread.java:750)                                    
61614/tcp open  http       Jetty 9.4.39.v20210325                              
| http-methods:                                                                
|_  Potentially risky methods: TRACE                                           
|_http-server-header: Jetty(9.4.39.v20210325)                                  
|_http-title: Site doesn't have a title.                                       
61616/tcp open  apachemq   ActiveMQ OpenWire transport                         
| fingerprint-strings:
|   NULL:
|     ActiveMQ
|     TcpNoDelayEnabled
|     SizePrefixDisabled
```

* The scan reveals 9 open ports : 
    * ``OpenSSH`` server running on port 22.
    * ``nginx`` web server running on port 80 but it's returing a 401 unauthorized access.
    * ``ActiveMQ`` is a messaging broker running on ports ``61613`` and ``61616``, These ports are associated with the AMQP (Advanced Message Queuing Protocol) and MQTT (Message Queuing Telemetry Transport) communication protocols, respectively running on port ``1883`` and ``5672``.

* Visiting ``http://10.10.11.243``, we have a basic HTTP authentication.

![broker1](/images/broker1.png)

* trying the default credentials ``admin:admin``, we successefully logged in.

![broker2](/images/broker2.png)

* the ``Manage ActiveMQ broker`` link lead to the ``/admin`` page which reveals the Broker's version.

![broker3](/images/broker3.png)

* After a quick google search on this version, i found out it's vulnerable to remote code execution.

![broker4](/images/broker4.png)

* For more information about this specific vulnerability, please refer to the provided links below.

    - [CVE-2023-46604 | AttackerKB](https://attackerkb.com/topics/IHsgZDE3tS/cve-2023-46604/rapid7-analysis?referrer=etrblog)

    - [Huntress](https://www.huntress.com/blog/critical-vulnerability-exploitation-of-apache-activemq-cve-2023-46604)

# User Flag : 

* I found this a [Python POC](https://github.com/evkl1d/CVE-2023-46604) that exploitsthis vulnerability. In essence, the script exploits a deserialization vulnerability within ActiveMQ, leveraging a Spring gadget to load a remote XML file with the capability to execute programs. The ``poc.xml`` file serves as a conventional bash reverse shell, make sure to modify the ``ip address``.

![broker6](/images/broker6.png)

* Before running the python script, let's host a python web server.

![broker5](/images/broker5.png)

* Let"s start a netcat listener.

```bash
nc -lvnp 9001
```

* Time to execute the python script.

![broker7](/images/broker7.png)

* Now let's grab the ``User`` flag.

![broker8](/images/broker8.png)


## Root Flag : 

* Checking what commands we can execute with elevated privileges.

![broker9](/images/broker9.png)

* We can run ``nginx``as root without password.

* With ``sudo nginx`` we can create a webserver on the box with root privileges

    - [nginx local root exploit](https://darrenmartynie.wordpress.com/2021/10/25/zimbra-nginx-local-root-exploit/)

* Let's create a config file called ``nginx.conf``.

![broker10](/images/broker10.png)

* Now all we have to do is to start the webserver using ``nginx`` with the ``-c`` option and specify the path to the ``nginx.conf`` file.

```bash
sudo /usr/sbin/nginx -c /tmp/nginx.conf 
```

* Now we can query this webserver and read root files.

![broker11](/images/broker11.png)





















