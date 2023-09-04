---
title: "[HTB] Machine: Outdated"
description: "[HTB] Machine: Outdated"
date: 2023-09-01
menu:
  sidebar:
    name: "[HTB] Machine: Outdated"
    identifier: htb-machine-Outdated
    parent: htb-machines-windows
    weight: 10
hero: images/outdated.png
tags: ["HTB"]
---

# Outdated
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.175 -T4                           
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-04 20:50 BST
Nmap scan report for 10.10.11.175 (10.10.11.175)
Host is up (0.16s latency).
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE
25/tcp    open  smtp
53/tcp    open  domain
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
8530/tcp  open  unknown
8531/tcp  open  unknown
9389/tcp  open  adws
49667/tcp open  unknown
49685/tcp open  unknown
49686/tcp open  unknown
49689/tcp open  unknown
49928/tcp open  unknown
52597/tcp open  unknown

```
```
└─$ nmap -Pn -p25,53,88,135,139,143,389,445,464,587,593,636,2179,3268,3269,5985,8530,8531,9389 -sC -sV 10.10.11.175 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-04 21:06 BST
Nmap scan report for 10.10.11.175 (10.10.11.175)
Host is up (0.18s latency).

PORT     STATE    SERVICE       VERSION
25/tcp   open     smtp          hMailServer smtpd
| smtp-commands: mail.outdated.htb, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
53/tcp   open     domain        Simple DNS Plus
88/tcp   open     kerberos-sec  Microsoft Windows Kerberos (server time: 2023-09-05 03:05:47Z)
135/tcp  open     msrpc         Microsoft Windows RPC
139/tcp  open     netbios-ssn   Microsoft Windows netbios-ssn
143/tcp  filtered imap
389/tcp  open     ldap          Microsoft Windows Active Directory LDAP (Domain: outdated.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-09-05T03:07:12+00:00; +6h59m24s from scanner time.
| ssl-cert: Subject: commonName=DC.outdated.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC.outdated.htb
| Not valid before: 2023-09-05T02:40:55
|_Not valid after:  2024-09-04T02:40:55
445/tcp  open     microsoft-ds?
464/tcp  open     kpasswd5?
587/tcp  filtered submission
593/tcp  open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open     ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: outdated.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-09-05T03:07:13+00:00; +6h59m24s from scanner time.
| ssl-cert: Subject: commonName=DC.outdated.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC.outdated.htb
| Not valid before: 2023-09-05T02:40:55
|_Not valid after:  2024-09-04T02:40:55
2179/tcp filtered vmrdp
3268/tcp open     ldap          Microsoft Windows Active Directory LDAP (Domain: outdated.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-09-05T03:07:12+00:00; +6h59m24s from scanner time.
| ssl-cert: Subject: commonName=DC.outdated.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC.outdated.htb
| Not valid before: 2023-09-05T02:40:55
|_Not valid after:  2024-09-04T02:40:55
3269/tcp open     ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: outdated.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-09-05T03:07:12+00:00; +6h59m24s from scanner time.
| ssl-cert: Subject: commonName=DC.outdated.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC.outdated.htb
| Not valid before: 2023-09-05T02:40:55
|_Not valid after:  2024-09-04T02:40:55
5985/tcp open     http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8530/tcp open     http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title.
| http-methods: 
|_  Potentially risky methods: TRACE
8531/tcp open     unknown
9389/tcp open     mc-nmf        .NET Message Framing
Service Info: Hosts: mail.outdated.htb, DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-09-05T03:06:37
|_  start_date: N/A
|_clock-skew: mean: 6h59m23s, deviation: 0s, median: 6h59m23s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 99.07 seconds

```


## Foothold


## User


## Root