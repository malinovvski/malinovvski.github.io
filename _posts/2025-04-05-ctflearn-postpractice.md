---
title: "CTFLearn - POST Practice"
date: 2025-04-05 12:00:00 +0000
author: scr4tcher
categories: [CTFLearn]
tags: [post-request, curl]

---

## Desciption

"This website requires authentication, via POST. However, it seems as if someone has defaced our site. Maybe there is still some way to authenticate? http://165.227.106.113/post.php"

## Solution 

This is a very straightforward CTF challenge, and its solution requires nothing more than the use of the curl command.

At the beginning, I run the following curl command:

```console
curl http://165.227.106.113/post.php?
```
From this, I can see the username (admin) and password (71urlkufpsdnlkadsf) hidden in the HTML comments.

Next, I execute the following curl command to submit the credentials:

```console
curl http://165.227.106.113/post.php -d "username=admin&password=71urlkufpsdnlkadsf"
```

The response gives me the flag:

```
<h1>flag{p0st_d4t4_4ll_d4y}</h1>
```

