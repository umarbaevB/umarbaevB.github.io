---
title: "[HTB] Machine: Sizzle"
description: "[HTB] Machine: Sizzle"
date: 2023-10-04
menu:
  sidebar:
    name: "[HTB] Machine: Sizzle"
    identifier: htb-machine-Sizzle
    parent: htb-machines-windows
    weight: 10
hero: images/sizzle.png
tags: ["HTB"]
---

# Sizzle
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.103 --min-rate 5000                                                      
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-06 03:19 BST
Nmap scan report for 10.10.10.103 (10.10.10.103)
Host is up (0.11s latency).
Not shown: 65506 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
53/tcp    open  domain
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
5986/tcp  open  wsmans
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49677/tcp open  unknown
49688/tcp open  unknown
49689/tcp open  unknown
49691/tcp open  unknown
49694/tcp open  unknown
49699/tcp open  unknown
49708/tcp open  unknown
49714/tcp open  unknown

```
```
└─$ nmap -Pn -p21,53,80,135,139,443,445,464,593,636,3268,3269,5985,5986,9389,47001 -sC -sV 10.10.10.103 --min-rate 5000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-06 03:19 BST
Nmap scan report for 10.10.10.103 (10.10.10.103)
Host is up (0.10s latency).

PORT      STATE    SERVICE          VERSION
21/tcp    open     ftp              Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp    open     domain           Simple DNS Plus
80/tcp    open     http             Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp   open     msrpc            Microsoft Windows RPC
139/tcp   open     netbios-ssn      Microsoft Windows netbios-ssn
443/tcp   open     ssl/http         Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/html).
|_ssl-date: 2023-10-05T19:21:25+00:00; -7h00m01s from scanner time.
| ssl-cert: Subject: commonName=sizzle.htb.local
| Not valid before: 2018-07-03T17:58:55
|_Not valid after:  2020-07-02T17:58:55
| tls-alpn: 
|   h2
|_  http/1.1
| http-methods: 
|_  Potentially risky methods: TRACE
445/tcp   open     microsoft-ds?
464/tcp   open     kpasswd5?
593/tcp   filtered http-rpc-epmap
636/tcp   filtered ldapssl
3268/tcp  filtered globalcatLDAP
3269/tcp  filtered globalcatLDAPssl
5985/tcp  filtered wsman
5986/tcp  open     ssl/http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_ssl-date: 2023-10-05T19:21:25+00:00; -7h00m01s from scanner time.
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2018-07-02T20:26:23
|_Not valid after:  2019-07-02T20:26:23
| tls-alpn: 
|   h2
|_  http/1.1
9389/tcp  open     mc-nmf           .NET Message Framing
47001/tcp filtered winrm
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -7h00m00s, deviation: 0s, median: -7h00m01s
| smb2-time: 
|   date: 2023-10-05T19:20:48
|_  start_date: 2023-10-05T19:16:02
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 98.35 seconds

```