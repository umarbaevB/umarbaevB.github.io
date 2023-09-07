---
title: "[HTB] Machine: Json"
description: "[HTB] Machine: Json"
date: 2023-09-06
menu:
  sidebar:
    name: "[HTB] Machine: Json"
    identifier: htb-machine-Json
    parent: htb-machines-windows
    weight: 10
hero: images/json.png
tags: ["HTB"]
---

# Json
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.158 -T4   
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-07 20:02 BST
Stats: 0:03:53 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 45.70% done; ETC: 20:10 (0:04:37 remaining)
Nmap scan report for 10.10.10.158 (10.10.10.158)
Host is up (0.12s latency).
Not shown: 65521 closed tcp ports (conn-refused)
PORT      STATE SERVICE
21/tcp    open  ftp
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5985/tcp  open  wsman
47001/tcp open  winrm
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49156/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown

```
```
└─$ nmap -Pn -p21,80,135,139,445,5985 -sC -sV 10.10.10.158                                 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-07 20:02 BST
Nmap scan report for 10.10.10.158 (10.10.10.158)
Host is up (0.13s latency).

PORT     STATE SERVICE      VERSION
21/tcp   open  ftp          FileZilla ftpd
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
80/tcp   open  http         Microsoft IIS httpd 8.5
|_http-title: Json HTB
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-09-07T19:02:11
|_  start_date: 2023-09-07T18:55:22
|_clock-skew: mean: -36s, deviation: 0s, median: -36s
| smb2-security-mode: 
|   3:0:2: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: JSON, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:fb:43 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.86 seconds
```