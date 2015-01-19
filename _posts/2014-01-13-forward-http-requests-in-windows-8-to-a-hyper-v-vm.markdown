---
title: "How to Forward HTTP Requests in Windows 8 to a Hyper-V VM"
tags: 
  - Hyper-V
  - Request Forwarding
  - Windows 8
---

Complicated title — that's pretty much, what I was looking for on Google, yet with only little success. At first, I found some instructions for Windows Server 2008R2 and 2012. After modifying the search query, some instructions for exporting a network connection in VMWare showed up. There were plenty of tutorials, but none of them quite what I was looking for.

The story:
----------

I have created a virtual machine on my Windows 8 via Hyper V and installed the Ubuntu 13.10 Beta. It runs nicely - full screen is missing, but except that, everything works. I have a default LAMP-Stack installed in Ubuntu, and also some CMSs set up, where I try to do something out of it. While this works great on my machine directly and I can do everything, it isn't possible for others from the same LAN, to access that CMS. So the question was, ...

### How can others connect to my VM?

After the described less successful attempts to find the answer on Google, I stopped for a moment to think. I remembered, that I'm using putty for tunnelling connections over the network __AND__ - guess what. Putty can also accept connections from remote hosts.

### To use Putty to forward requests from remote hosts to the VM you can do the following:

0. Type in Host Name (or IP address) and make sure that your VM has an SSH-Daemon running
1. [Set up forwarding in Connection->SSH->Tunnels](https://www.google.com/search?q=set+up+putty+port+forwarding)
2. Check the box _"Local ports accept connections from other hosts"_
3. Open the connection and log-in
4. You might get a warning from your firewall, about putty opening a port. Just allow it and you are finished.
