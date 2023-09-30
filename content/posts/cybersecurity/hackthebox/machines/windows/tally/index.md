---
title: "[HTB] Machine: Tally"
description: "[HTB] Machine: Tally"
date: 2023-09-30
menu:
  sidebar:
    name: "[HTB] Machine: Tally"
    identifier: htb-machine-Tallyqqqqqqqqqqq
    parent: htb-machines-windows
    weight: 10
hero: images/tally.png
tags: ["HTB"]
---

# Tally
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.59 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 20:46 BST
Warning: 10.10.10.59 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.59 (10.10.10.59)
Host is up (0.10s latency).
Not shown: 60269 filtered tcp ports (no-response), 5257 closed tcp ports (conn-refused)
PORT      STATE SERVICE
21/tcp    open  ftp
80/tcp    open  http
81/tcp    open  hosts2-ns
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
808/tcp   open  ccproxy-http
1433/tcp  open  ms-sql-s
5985/tcp  open  wsman
15567/tcp open  unknown
32843/tcp open  unknown
32844/tcp open  unknown
32846/tcp open  unknown
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 83.60 seconds

```
```
└─$ nmap -Pn -p21,80,81,135,139,445,808,1433,5985,15567,32843,32844,32846,47001,49664-49670 -sC -sV 10.10.10.59 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 20:51 BST
Nmap scan report for 10.10.10.59 (10.10.10.59)
Host is up (0.10s latency).

PORT      STATE    SERVICE            VERSION
21/tcp    open     ftp                Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open     http               Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-title: Home
|_Requested resource was http://10.10.10.59/_layouts/15/start.aspx#/default.aspx
|_http-generator: Microsoft SharePoint
81/tcp    filtered hosts2-ns
135/tcp   open     msrpc              Microsoft Windows RPC
139/tcp   open     netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open     microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
808/tcp   filtered ccproxy-http
1433/tcp  open     ms-sql-s           Microsoft SQL Server 2016 13.00.1601.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2023-09-30T19:38:29
|_Not valid after:  2053-09-30T19:38:29
| ms-sql-info: 
|   10.10.10.59:1433: 
|     Version: 
|       name: Microsoft SQL Server 2016 RTM
|       number: 13.00.1601.00
|       Product: Microsoft SQL Server 2016
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2023-09-30T19:52:36+00:00; -3s from scanner time.
| ms-sql-ntlm-info: 
|   10.10.10.59:1433: 
|     Target_Name: TALLY
|     NetBIOS_Domain_Name: TALLY
|     NetBIOS_Computer_Name: TALLY
|     DNS_Domain_Name: TALLY
|     DNS_Computer_Name: TALLY
|_    Product_Version: 10.0.14393
5985/tcp  filtered wsman
15567/tcp open     http               Microsoft IIS httpd 10.0
| http-ntlm-info: 
|   Target_Name: TALLY
|   NetBIOS_Domain_Name: TALLY
|   NetBIOS_Computer_Name: TALLY
|   DNS_Domain_Name: TALLY
|   DNS_Computer_Name: TALLY
|_  Product_Version: 10.0.14393
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|   Negotiate
|_  NTLM
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title.
32843/tcp open     http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Service Unavailable
32844/tcp filtered unknown
32846/tcp open     storagecraft-image StorageCraft Image Manager
47001/tcp filtered winrm
49664/tcp filtered unknown
49665/tcp filtered unknown
49666/tcp filtered unknown
49667/tcp filtered unknown
49668/tcp filtered unknown
49669/tcp open     msrpc              Microsoft Windows RPC
49670/tcp open     msrpc              Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: -2s, deviation: 0s, median: -3s
| smb2-time: 
|   date: 2023-09-30T19:52:30
|_  start_date: 2023-09-30T19:38:08

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 68.85 seconds
```