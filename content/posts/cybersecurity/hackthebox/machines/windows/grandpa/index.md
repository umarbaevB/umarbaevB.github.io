---
title: "[HTB] Machine: Grandpa"
description: "[HTB] Machine: Grandpa"
date: 2023-06-04
menu:
  sidebar:
    name: "[HTB] Machine: Grandpa"
    identifier: htb-machine-grandpa
    parent: htb-machines-windows
    weight: 10
hero: images/grandpa.png
tags: ["HTB"]
---

# Grandpa
## Enumeration
- ```Nmap```
```
└─$ nmap -sC -sV -Pn 10.10.10.14
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-04 18:44 BST
Nmap scan report for 10.10.10.14 (10.10.10.14)
Host is up (0.093s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|   Server Type: Microsoft-IIS/6.0
|   WebDAV type: Unknown
|   Server Date: Sun, 04 Jun 2023 17:44:06 GMT
|_  Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
| http-ntlm-info: 
|   Target_Name: GRANPA
|   NetBIOS_Domain_Name: GRANPA
|   NetBIOS_Computer_Name: GRANPA
|   DNS_Domain_Name: granpa
|   DNS_Computer_Name: granpa
|_  Product_Version: 5.2.3790
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.37 seconds
```
## Root
