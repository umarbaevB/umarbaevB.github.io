---
title: "[HTB] Machine: Querier"
description: "[HTB] Machine: Querier"
date: 2023-09-06
menu:
  sidebar:
    name: "[HTB] Machine: Querier"
    identifier: htb-machine-Querier
    parent: htb-machines-windows
    weight: 10
hero: images/querier.png
tags: ["HTB"]
---

# Querier
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.125 -T4                                                            
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-07 03:05 BST
Stats: 0:06:52 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 72.17% done; ETC: 03:15 (0:02:39 remaining)
Nmap scan report for 10.10.10.125 (10.10.10.125)
Host is up (0.15s latency).
Not shown: 65521 closed tcp ports (conn-refused)
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
1433/tcp  open  ms-sql-s
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 693.11 seconds
```
```
└─$ nmap -Pn -p135,139,445,1433,5985 -sC -sV 10.10.10.125 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-07 03:18 BST
Nmap scan report for 10.10.10.125 (10.10.10.125)
Host is up (0.13s latency).

PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
1433/tcp open  ms-sql-s      Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   10.10.10.125:1433: 
|     Target_Name: HTB
|     NetBIOS_Domain_Name: HTB
|     NetBIOS_Computer_Name: QUERIER
|     DNS_Domain_Name: HTB.LOCAL
|     DNS_Computer_Name: QUERIER.HTB.LOCAL
|     DNS_Tree_Name: HTB.LOCAL
|_    Product_Version: 10.0.17763
| ms-sql-info: 
|   10.10.10.125:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2023-09-06T19:18:33+00:00; -7h00m00s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2023-09-06T14:05:43
|_Not valid after:  2053-09-06T14:05:43
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-09-06T19:18:24
|_  start_date: N/A
|_clock-skew: mean: -7h00m00s, deviation: 0s, median: -7h00m00s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.61 seconds
```
- `smb`
```
└─$ smbclient -N -L //10.10.10.125

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        Reports         Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.125 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

```