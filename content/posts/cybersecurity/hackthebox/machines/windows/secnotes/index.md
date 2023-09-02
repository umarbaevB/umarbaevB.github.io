---
title: "[HTB] Machine: SecNotes"
description: "[HTB] Machine: SecNotes"
date: 2023-09-01
menu:
  sidebar:
    name: "[HTB] Machine: SecNotes"
    identifier: htb-machine-SecNotes
    parent: htb-machines-windows
    weight: 10
hero: images/secnotes.png
tags: ["HTB"]
---

# SecNotes
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.97 -T4                                                      
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-02 18:49 BST
Nmap scan report for 10.10.10.97 (10.10.10.97)
Host is up (0.092s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
445/tcp  open  microsoft-ds
8808/tcp open  ssports-bcast
```
```
└─$ nmap -Pn -p80,445,8808 -sC -sV 10.10.10.97 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-02 18:54 BST
Nmap scan report for 10.10.10.97 (10.10.10.97)
Host is up (0.13s latency).

PORT     STATE SERVICE   VERSION
80/tcp   open  http      Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
| http-title: Secure Notes - Login
|_Requested resource was login.php
|_http-server-header: Microsoft-IIS/10.0
445/tcp  open  10.10.100 Windows 10 Enterprise 17134 microsoft-ds (workgroup: HTB)
8808/tcp open  http      Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows
Service Info: Host: SECNOTES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 2h19m24s, deviation: 4h02m30s, median: -35s
| smb-os-discovery: 
|   OS: Windows 10 Enterprise 17134 (Windows 10 Enterprise 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: SECNOTES
|   NetBIOS computer name: SECNOTES\x00
|   Workgroup: HTB\x00
|_  System time: 2023-09-02T10:53:49-07:00
| smb2-time: 
|   date: 2023-09-02T17:53:53
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
```

- Web Server

![](./images/1.png)

- Port `8808`

![](./images/2.png)
