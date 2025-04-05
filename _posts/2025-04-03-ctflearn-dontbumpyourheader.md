---
title: "CTFLearn - Don't Bump your Head(er)"
date: 2025-04-06 00:30:00 +0000
author: scr4tcher
categories: [CTFLearn]
tags: [header, curl]

---

## Description
"Try to bypass my security measure on this site! http://165.227.106.113/header.php"


## Solution 


First, I attempt to access the page without modifying any headers:

```console
(kali㉿kali)-[~/Desktop]
└─$ curl http://165.227.106.113/header.php                                                        
Sorry, it seems as if your user agent is not correct, in order to access this website. The one you supplied is: curl/8.11.1
<!-- Sup3rS3cr3tAg3nt  -->
```        

This indicates that the server is blocking requests with the default **User-Agent** (in this case, curl/8.11.1), expecting a specific **User-Agent** string, namely  **Sup3rS3cr3tAg3nt**.

Next, I modify the **User-Agent** header to match the required value (Sup3rS3cr3tAg3nt), using the **-H** option in curl:

```console
┌──(kali㉿kali)-[~/Desktop]
└─$ curl http://165.227.106.113/header.php -H "User-Agent: Sup3rS3cr3tAg3nt"
Sorry, it seems as if you did not just come from the site, "awesomesauce.com".
<!-- Sup3rS3cr3tAg3nt  -->
```
It appears that the server is also checking the **Referer** header and is rejecting the request unless the **Referer** is awesomesauce.com.                                                                                          

To bypass this additional restriction, I modify the **Referer** header using the **-e** option to set the referer to awesomesauce.com

```console                 
┌──(kali㉿kali)-[~/Desktop]
└─$ curl http://165.227.106.113/header.php -H "User-Agent: Sup3rS3cr3tAg3nt" -e "awesomesauce.com"
Here is your flag: flag{did_this_m3ss_with_y0ur_h34d}
<!-- Sup3rS3cr3tAg3nt  -->
```