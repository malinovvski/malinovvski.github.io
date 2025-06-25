---
title: "TryHackMe - ohSINT"
date: 2025-06-20 12:00:00 +0000
author: scr4tcher
categories: [TryHackMe CTF]
tags: [OSINT]
--- 

In this blog post, I'm documenting my approach to solving the ohSINT challenge from a popular CTF (Capture The Flag) exercise. 
This challenge focuses on Open-Source Intelligence (OSINT), and the goal is to gather as much information as possible about a target based on a single image.

## Analyzing the Image Metadata

The challenge began with a JPEG image named `WindowsXP_1551719014755.jpg`. As always with image-based OSINT challenges, my first step was to inspect the metadata using `exiftool`:

```
┌──(scr4tcher㉿kali)-[~/Desktop/ohSINT]
└─$ exiftool WindowsXP_1551719014755.jpg 
ExifTool Version Number         : 13.10
File Name                       : WindowsXP_1551719014755.jpg
Directory                       : .
File Size                       : 234 kB
File Modification Date/Time     : 2025:06:25 06:58:24-04:00
File Access Date/Time           : 2025:06:25 06:58:28-04:00
File Inode Change Date/Time     : 2025:06:25 06:58:24-04:00
File Permissions                : -rw-rw-r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
XMP Toolkit                     : Image::ExifTool 11.27
GPS Latitude                    : 54 deg 17' 41.27" N
GPS Longitude                   : 2 deg 15' 1.33" W
Copyright                       : OWoodflint
Image Width                     : 1920
Image Height                    : 1080
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 1920x1080
Megapixels                      : 2.1
GPS Latitude Ref                : North
GPS Longitude Ref               : West
GPS Position                    : 54 deg 17' 41.27" N, 2 deg 15' 1.33" W
```

Here’s the output that caught my attention:
```
GPS Latitude     : 54 deg 17' 41.27" N
GPS Longitude    : 2 deg 15' 1.33" W
Copyright        : OWoodflint
```

From this, I learned two important things:

- The image contained GPS coordinates, which point to a specific physical location.

- The copyright metadata referenced the name OWoodflint.


## Investigating the Username

Next, I searched for `OWoodflint` online and quickly found a matching GitHub profile. 
This gave me more leads — in particular, I looked at the profile `README`, which revealed that the user is from **London**. 
I also discovered links to his WordPress site and personal email address.

![1](/images/ohsint/1.jpg)

## Finding Their X profile

Continuing the investigation, I located his X (formerly Twitter) account using the same username.
The profile photo showed a cat, which answered the question:

![1b](/images/ohsint/1b.jpg)

**Question:  What is this user's avatar of?**

> CAT 


## Locating the Wi-Fi Access Point (WAP)

One of the X posts mentioned:

```
"From my house I can get free wifi ;D
Bssid: B4:5D:50:AA:86:41 - Go nuts!"
```

Using the BSSID `B4:5D:50:AA:86:41`, I went to `wigle.net` — a global Wi-Fi database. 
Searching for that BSSID helped me narrow down the SSID of the wireless network he connects to.

> UnileverWiFi


## Gathering More Personal Info

The GitHub page for his `people_finder` project revealed his personal **email address** : 
> OWoodflint@gmail.com

**Question: What site did you find his email address on?**
> Github


## Tracking Travel Activity

While reviewing his WordPress blog, I found a post that mentioned his holiday plans:

![2](/images/ohsint/2.jpg)

**Question : Where has he gone on holiday?**

> New york


## Discovering the Password

Finally, I decided to inspect the source code of one of [his WordPress blog pages](https://oliverwoodflint.wordpress.com/category/uncategorised/)


While scanning the HTML, I noticed a suspicious line that didn’t seem like standard content. That turned out to be his password.

![3](/images/ohsint/3.jpg)

## Summary of Findings

|Question                                           | 	Answer            | 
| :---------------------------                      | :---------------    |
| What is this user's avatar of?                    | cat                 |
| What city is this person in?                      | London              |
| What is the SSID of the WAP he connected to?      | UnileverWiFi        |
| What is his personal email address?               | OWoodflint@gmail.com|
| What site did you find his email address on?      | Github              |
| Where has he gone on holiday?                     | New York            |
| What is the person's password?                    | pennYDr0pper.!      |


## Final Thoughts

This challenge is a great example of how much can be discovered using only publicly available data and a little persistence. From metadata in a single image, I was able to trace a digital footprint across multiple platforms, ultimately revealing sensitive personal information.