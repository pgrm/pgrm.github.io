---
layout: post
title: Hyper-V error "IDE/ATAPI Account does not have sufficient privilege to open
  attachment..."
date: '2012-11-22T11:13:00.002+01:00'
tags:
- Administration
- Hyper-V
modified_time: '2012-11-22T11:14:29.242+01:00'
blogger_id: tag:blogger.com,1999:blog-8459046186574607112.post-4855181382494073696
blogger_orig_url: http://emptycode.blogspot.com/2012/11/hyper-v-error-ideatapi-account-does-not.html
---

I have Microsoft Hyper-V running on my laptop (Windows 8). After the installation and configuration was a little bit of a pain (getting an internet connection wasn't that easy for me) it works very well.
Unfortunately I did another major mistake. I wanted to use my VM for classes for testing iptables, and still use it afterwards. But instead of making a snapshot, I did what I was used to do on VirtualBox - copied the file. And once I deleted the destroyed VM and copied my file back, I couldn't start the machine any more. The error displayed was:

> IDE/ATAPI Account does not have sufficient privilege to open attachment...

I've found quite fast an [article on MSDN](http://support.microsoft.com/kb/2249906/en-us?fr=1) but the fix just didn't work for me. What worked was simply 

1. remove the Hard Disc which has the "broken" VHD loaded
2. create a new Hard Disc
3. load the VHD there inside. 

Hyper-V will set all necessary permissions on it's own.