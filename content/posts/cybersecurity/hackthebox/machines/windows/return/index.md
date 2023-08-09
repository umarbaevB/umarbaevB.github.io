---
title: "[HTB] Machine: Return"
description: "[HTB] Machine: Return"
date: 2023-08-08
menu:
  sidebar:
    name: "[HTB] Machine: Return"
    identifier: htb-machine-Return
    parent: htb-machines-windows
    weight: 10
hero: images/return.png
tags: ["HTB"]
---

# Return
## Enumeration
- `nmap`
```
└─$ nmap -sC -sV -Pn 10.10.11.108
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-09 18:52 BST
Nmap scan report for 10.10.11.108 (10.10.11.108)
Host is up (0.19s latency).
Not shown: 990 closed tcp ports (conn-refused)
PORT    STATE SERVICE       VERSION
53/tcp  open  domain        Simple DNS Plus
80/tcp  open  http          Microsoft IIS httpd 10.0
|_http-title: HTB Printer Admin Panel
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp  open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-08-09 18:11:46Z)
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
445/tcp open  microsoft-ds?
464/tcp open  kpasswd5?
593/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp open  tcpwrapped
Service Info: Host: PRINTER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-08-09T18:12:01
|_  start_date: N/A
|_clock-skew: 18m01s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.64 seconds

```