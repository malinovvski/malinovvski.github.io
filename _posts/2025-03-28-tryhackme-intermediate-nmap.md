---
title: "TryHackMe - Intermediate Nmap"
date: 2025-03-28 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [ssh, nmap]

---

## Introduction

This CTF challenge is a straightforward test of basic reconnaissance skills using **Nmap**. The goal is to identify open ports, discover services running on the target machine, and leverage any discovered information to gain access. This type of challenge is great for beginners looking to refine their scanning techniques and understand the importance of enumeration in cybersecurity.


## Reconnaissance

The first step in any reconnaissance process is to run an Nmap scan to gather information about the target system. I used the following command:

```console
> nmap -p- -sC -sV -A 10.10.56.174
```

Here's a breakdown of the options:
- **-p-**: Scans all 65,535 ports, ensuring nothing is missed

-  **sC**: Runs default scripts, which can reveal additional details about services.

- **sV**: Detects service versions to identify potential vulnerabilities.

- **-A**: Enables OS detection, version detection, script scanning, and traceroute.

## Analyzing the scan results

```console
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-28 07:51 EDT
Nmap scan report for 10.10.56.174
Host is up (0.068s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7d:dc:eb:90:e4:af:33:d9:9f:0b:21:9a:fc:d5:77:f2 (RSA)
|   256 83:a7:4a:61:ef:93:a3:57:1a:57:38:5c:48:2a:eb:16 (ECDSA)
|_  256 30:bf:ef:94:08:86:07:00:f7:fc:df:e8:ed:fe:07:af (ED25519)
2222/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 6f:90:b4:d4:32:8f:a0:54:6d:88:12:3a:32:1c:9e:d3 (RSA)
|   256 cc:6d:d6:03:d1:b9:7b:21:d5:6e:ce:57:9e:af:83:dd (ECDSA)
|_  256 d1:b9:85:81:6e:cf:38:1e:4f:45:5c:8a:52:9e:fa:35 (ED25519)
31337/tcp open  Elite?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe: 
|     In case I forget - user:pass
|_    ubuntu:Dafdas!!/str0ng
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port31337-TCP:V=7.95%I=7%D=3/28%Time=67E68D9C%P=x86_64-pc-linux-gnu%r(N
SF:ULL,35,"In\x20case\x20I\x20forget\x20-\x20user:pass\nubuntu:Dafdas!!/st
SF:r0ng\n\n")%r(GetRequest,35,"In\x20case\x20I\x20forget\x20-\x20user:pass
SF:\nubuntu:Dafdas!!/str0ng\n\n")%r(SIPOptions,35,"In\x20case\x20I\x20forg
SF:et\x20-\x20user:pass\nubuntu:Dafdas!!/str0ng\n\n")%r(GenericLines,35,"I
SF:n\x20case\x20I\x20forget\x20-\x20user:pass\nubuntu:Dafdas!!/str0ng\n\n"
SF:)%r(HTTPOptions,35,"In\x20case\x20I\x20forget\x20-\x20user:pass\nubuntu
SF::Dafdas!!/str0ng\n\n")%r(RTSPRequest,35,"In\x20case\x20I\x20forget\x20-
SF:\x20user:pass\nubuntu:Dafdas!!/str0ng\n\n")%r(RPCCheck,35,"In\x20case\x
SF:20I\x20forget\x20-\x20user:pass\nubuntu:Dafdas!!/str0ng\n\n")%r(DNSVers
SF:ionBindReqTCP,35,"In\x20case\x20I\x20forget\x20-\x20user:pass\nubuntu:D
SF:afdas!!/str0ng\n\n")%r(DNSStatusRequestTCP,35,"In\x20case\x20I\x20forge
SF:t\x20-\x20user:pass\nubuntu:Dafdas!!/str0ng\n\n")%r(Help,35,"In\x20case
SF:\x20I\x20forget\x20-\x20user:pass\nubuntu:Dafdas!!/str0ng\n\n")%r(SSLSe
SF:ssionReq,35,"In\x20case\x20I\x20forget\x20-\x20user:pass\nubuntu:Dafdas
SF:!!/str0ng\n\n")%r(TerminalServerCookie,35,"In\x20case\x20I\x20forget\x2
SF:0-\x20user:pass\nubuntu:Dafdas!!/str0ng\n\n")%r(TLSSessionReq,35,"In\x2
SF:0case\x20I\x20forget\x20-\x20user:pass\nubuntu:Dafdas!!/str0ng\n\n")%r(
SF:Kerberos,35,"In\x20case\x20I\x20forget\x20-\x20user:pass\nubuntu:Dafdas
SF:!!/str0ng\n\n")%r(SMBProgNeg,35,"In\x20case\x20I\x20forget\x20-\x20user
SF::pass\nubuntu:Dafdas!!/str0ng\n\n")%r(X11Probe,35,"In\x20case\x20I\x20f
SF:orget\x20-\x20user:pass\nubuntu:Dafdas!!/str0ng\n\n")%r(FourOhFourReque
SF:st,35,"In\x20case\x20I\x20forget\x20-\x20user:pass\nubuntu:Dafdas!!/str
SF:0ng\n\n")%r(LPDString,35,"In\x20case\x20I\x20forget\x20-\x20user:pass\n
SF:ubuntu:Dafdas!!/str0ng\n\n")%r(LDAPSearchReq,35,"In\x20case\x20I\x20for
SF:get\x20-\x20user:pass\nubuntu:Dafdas!!/str0ng\n\n")%r(LDAPBindReq,35,"I
SF:n\x20case\x20I\x20forget\x20-\x20user:pass\nubuntu:Dafdas!!/str0ng\n\n"
SF:)%r(LANDesk-RC,35,"In\x20case\x20I\x20forget\x20-\x20user:pass\nubuntu:
SF:Dafdas!!/str0ng\n\n")%r(TerminalServer,35,"In\x20case\x20I\x20forget\x2
SF:0-\x20user:pass\nubuntu:Dafdas!!/str0ng\n\n");
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 143/tcp)
HOP RTT      ADDRESS
1   67.42 ms 10.8.0.1
2   67.42 ms 10.10.56.174

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 77.21 seconds
                                                                                                     
```
The scan revealed three open ports:

- **22/tcp**: Running OpenSSH 8.2p1

- **2222/tcp**: Another instance of OpenSSH 8.2p1

- **31337/tcp**: A service with an interesting response`

One notable finding from port 31337 was the following message:

This is a clear indication that credentials were leaked through this service. The username is ubuntu, and the password is Dafdas!!/str0ng.
```
In case I forget - user:pass
ubuntu:Dafdas!!/str0ng
```


## Gaining access via SSH

With valid credentials in hand, I attempted to log into the machine via SSH:

```console
> ssh ubuntu@10.10.56.174
```
Once logged in, I started an interactive shell session for better usability:
```console
> script -qc /bin/bash /dev/null
```
## Conclusion

This CTF challenge was a simple yet effective demonstration of how poor security practices, such as exposing credentials through an open service, can lead to unauthorized access.