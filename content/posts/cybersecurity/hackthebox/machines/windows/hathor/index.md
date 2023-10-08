---
title: "[HTB] Machine: Hathor"
description: "[HTB] Machine: Hathor"
date: 2023-10-04
menu:
  sidebar:
    name: "[HTB] Machine: Hathor"
    identifier: htb-machine-Hathor
    parent: htb-machines-windows
    weight: 10
hero: images/hathor.png
tags: ["HTB"]
---

# Hathor
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.147 --min-rate 10000 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-08 18:59 BST
Nmap scan report for 10.10.11.147 (10.10.11.147)
Host is up (0.11s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
49674/tcp open  unknown
49695/tcp open  unknown
49698/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 52.49 seconds
```
```
└─$ nmap -Pn -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -sC -sV 10.10.11.147 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-08 18:59 BST
Nmap scan report for 10.10.11.147 (10.10.11.147)
Host is up (0.28s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Service Unavailable
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-10-08 17:59:12Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: windcorp.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-10-08T18:00:34+00:00; -44s from scanner time.
| ssl-cert: Subject: commonName=hathor.windcorp.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:hathor.windcorp.htb
| Not valid before: 2023-10-08T17:49:24
|_Not valid after:  2024-10-07T17:49:24
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldapssl?
| ssl-cert: Subject: commonName=hathor.windcorp.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:hathor.windcorp.htb
| Not valid before: 2023-10-08T17:49:24
|_Not valid after:  2024-10-07T17:49:24
|_ssl-date: 2023-10-08T18:00:34+00:00; -44s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: windcorp.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=hathor.windcorp.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:hathor.windcorp.htb
| Not valid before: 2023-10-08T17:49:24
|_Not valid after:  2024-10-07T17:49:24
|_ssl-date: 2023-10-08T18:00:34+00:00; -44s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: windcorp.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-10-08T18:00:34+00:00; -44s from scanner time.
| ssl-cert: Subject: commonName=hathor.windcorp.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:hathor.windcorp.htb
| Not valid before: 2023-10-08T17:49:24
|_Not valid after:  2024-10-07T17:49:24
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Service Info: Host: HATHOR; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-10-08T17:59:55
|_  start_date: N/A
|_clock-skew: mean: -44s, deviation: 0s, median: -44s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.19 seconds

```