---
title: "[HTB] Machine: Reel2"
description: "[HTB] Machine: Reel2"
date: 2023-10-04
menu:
  sidebar:
    name: "[HTB] Machine: Reel2"
    identifier: htb-machine-Reel2
    parent: htb-machines-windows
    weight: 10
hero: images/reel2.png
tags: ["HTB"]
---

# Reel2
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.210  --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-03 19:51 BST
Nmap scan report for 10.10.10.210 (10.10.10.210)
Host is up (0.10s latency).
Not shown: 65521 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
443/tcp  open  https
5985/tcp open  wsman
6004/tcp open  X11:4
6005/tcp open  X11:5
6006/tcp open  X11:6
6007/tcp open  X11:7
6008/tcp open  X11:8
6010/tcp open  x11
6011/tcp open  x11
6012/tcp open  x11
6021/tcp open  x11
6135/tcp open  unknown
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 44.05 seconds

```
```
└─$ nmap -Pn -p80,443,5985,6001-6012,6021,6165,8080 -sC -sV 10.10.10.210  --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-03 19:56 BST
Nmap scan report for 10.10.10.210 (10.10.10.210)
Host is up (0.11s latency).

PORT     STATE    SERVICE    VERSION
80/tcp   open     http       Microsoft IIS httpd 8.5
|_http-server-header: Microsoft-IIS/8.5
|_http-title: 403 - Forbidden: Access is denied.
443/tcp  open     ssl/http   Microsoft IIS httpd 8.5
| ssl-cert: Subject: commonName=Reel2
| Subject Alternative Name: DNS:Reel2, DNS:Reel2.htb.local
| Not valid before: 2020-07-30T10:12:46
|_Not valid after:  2025-07-30T10:12:46
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
|_ssl-date: 2023-10-03T18:57:18+00:00; -43s from scanner time.
5985/tcp open     http       Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
6001/tcp open     ncacn_http Microsoft Windows RPC over HTTP 1.0
6002/tcp open     ncacn_http Microsoft Windows RPC over HTTP 1.0
6003/tcp filtered X11:3
6004/tcp filtered X11:4
6005/tcp filtered X11:5
6006/tcp open     msrpc      Microsoft Windows RPC
6007/tcp open     msrpc      Microsoft Windows RPC
6008/tcp open     msrpc      Microsoft Windows RPC
6009/tcp filtered X11:9
6010/tcp open     ncacn_http Microsoft Windows RPC over HTTP 1.0
6011/tcp filtered x11
6012/tcp open     msrpc      Microsoft Windows RPC
6021/tcp filtered x11
6165/tcp filtered unknown
8080/tcp open     http       Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.2.32)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.2.32
|_http-title: Welcome | Wallstant
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -43s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 70.28 seconds

```