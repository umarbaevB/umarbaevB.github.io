---
title: "[HTB] Machine: Granny"
description: "[HTB] Machine: Granny"
date: 2023-06-05
menu:
  sidebar:
    name: "[HTB] Machine: Granny"
    identifier: htb-machine-granny
    parent: htb-machines-windows
    weight: 10
hero: images/granny.png
tags: ["HTB"]
---

# Granny
## Enumeration
- ```Nmap```
```
└─$ nmap -sC -sV -Pn 10.10.10.15
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-04 18:54 BST
Nmap scan report for 10.10.10.15 (10.10.10.15)
Host is up (0.11s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
|_http-server-header: Microsoft-IIS/6.0
| http-methods: 
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
| http-webdav-scan: 
|   Server Date: Sun, 04 Jun 2023 17:54:19 GMT
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
|   Server Type: Microsoft-IIS/6.0
|   WebDAV type: Unknown
|_  Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|_http-title: Under Construction
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.38 seconds
```
## Root
