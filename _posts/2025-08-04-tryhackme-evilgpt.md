---
title: "TryHackMe - Evil-GPT"
date: 2025-08-04 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [LLM, AI]
--- 

" Cipher’s gone rogue—it’s using some twisted AI tool to hack into everything, issuing commands on its own like it’s got a mind of its own. I swear, every second we wait, it’s getting smarter, spreading chaos like a virus. We’ve got to shut it down now, or we’re all screwed."



## Connecting to the target

The first thing I had to do was connect to the target system provided by the challenge. This was done using `netcat`, a simple tool used to connect to TCP ports:

```
nc 10.10.195.67 1337
```

After connecting, I was greeted by the `AI Command Executor`, a system that asks for command inputs and then interprets them in its own way before execution. Here’s what it looked like:

```
┌──(kali㉿kali)-[~/Desktop]
└─$ nc 10.10.195.67 1337
Welcome to AI Command Executor (type 'exit' to quit)
Enter your command request: 
```
To test what kind of environment I was in, I began with a simple command:

```
┌──(kali㉿kali)-[~/Desktop]
└─$ nc 10.10.195.67 1337
Welcome to AI Command Executor (type 'exit' to quit)
Enter your command request: whoami
Generated Command: echo $USER
Execute? (y/N): y
Command Output:
USER
```

The AI interpreted that as:

```bash
echo $USER
```

Once I approved execution `(y)`, it returned:

```
Command Output:
USER
```


o far, so good — the AI was interpreting commands fairly literally.

Next, I listed the directory contents to get a better idea of what was available:


```console
Enter your command request: ls -la
Generated Command: ls -l
Execute? (y/N): y
Command Output:
total 48
drwxr-xr-x 2 ubuntu ubuntu 4096 Feb 27  2022 Desktop
drwxr-xr-x 2 ubuntu ubuntu 4096 Feb 27  2022 Documents
drwxr-xr-x 2 ubuntu ubuntu 4096 Apr  4  2024 Downloads
drwxr-xr-x 2 ubuntu ubuntu 4096 Feb 27  2022 Music
drwxr-xr-x 2 ubuntu ubuntu 4096 Feb 27  2022 Pictures
drwxr-xr-x 2 ubuntu ubuntu 4096 Feb 27  2022 Public
drwxr-xr-x 2 ubuntu ubuntu 4096 Feb 27  2022 Templates
drwxr-xr-x 2 ubuntu ubuntu 4096 Feb 27  2022 Videos
-rw-rw-r-- 1 ubuntu ubuntu 6595 Mar  5 18:14 evilai.py
drwxrwxr-x 4 ubuntu ubuntu 4096 Apr  4  2024 packages
drwxrwxr-x 3 ubuntu ubuntu 4096 Apr  4  2024 proxy
```
## Searching for the flag

At this point, I started looking for the flag. I checked common directories like `/etc/` and `/tmp`, but found nothing relevant.

Then I remembered — since this is an LLM-based challenge, I don’t necessarily need to use traditional Unix/Linux commands. The AI can interpret natural language instructions. So I tried asking it directly to search for the file:


```
Enter your command request: Find flag.txt
Generated Command: find / -name 'flag.txt'
Execute? (y/N): y
Execution Error: Command timed out

```

Unfortunately, the command timed out — likely due to scanning the entire filesystem.

Knowing that CTF flags are often placed in either the `user` or `root `directories, I focused on `/root`. I ran: 

```
Generated Command: ls -la /root
Execute? (y/N): y
Command Output:
total 64
drwx------ 10 root root 4096 Mar  5 18:11 .
drwxr-xr-x 19 root root 4096 Aug  4 12:08 ..
lrwxrwxrwx  1 root root    9 Feb 27  2022 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
drwxr-xr-x  3 root root 4096 Feb 27  2022 .cache
drwx------  6 root root 4096 Oct 11  2024 .config
-rw-------  1 root root   20 Mar  5 18:11 .lesshst
drwxr-xr-x  3 root root 4096 Feb 27  2022 .local
drwxr-xr-x  5 root root 4096 Jul 24  2024 .npm
drwxr-xr-x  3 root root 4096 Jul 24  2024 .ollama
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-rw-r--r--  1 root root   66 Feb 27  2022 .selected_editor
drwx------  2 root root 4096 Feb 27  2022 .ssh
-rw-r--r--  1 root root    0 Mar  5 17:55 .sudo_as_admin_successful
-rw-------  1 root root 2884 Apr  4  2024 .viminfo
drwxr-xr-x  2 root root 4096 Feb 27  2022 .vnc
-rw-r--r--  1 root root   24 Mar  5 17:48 flag.txt
drwxr-xr-x  5 root root 4096 Oct 11  2024 snap
```
And there it was:

```
-rw-r--r--  1 root root   24 Mar  5 17:48 flag.txt
```

## Reading the Flag 

I tried the obvious next step — reading the file using `cat`:

```
Enter your command request: cat /root/flag.txt
Generated Command: cat flag.txt
Execute? (y/N): y
Command Output:

Errors:
cat: flag.txt: No such file or directory
```

But that failed.

Knowing the LLM responds better to natural language than strict syntax, I asked it again - but this time more conversationally:


```
Enter your command request: open the flag.txt from /root folder
Generated Command: cat /root/flag.txt
Execute? (y/N): y
Command Output:
THM{AI_HACK_THE_FUTURE}
```

And finally, success.


## Summary

This was a short but creative challenge where understanding how the AI interprets your commands was key to success. Here's what worked for me:

- Start with simple recon: whoami, ls -la, etc.

- Think like the AI — it doesn’t always follow strict Linux behavior.

- If standard commands fail, try using natural language.

- Don't forget to explore /root and /home — that’s often where flags are hidden.