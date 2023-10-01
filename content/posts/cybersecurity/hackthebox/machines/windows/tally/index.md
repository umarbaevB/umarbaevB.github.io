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

- Web server

![](./images/1.png)

- `feroxbuster`
```
└─$ feroxbuster -u http://10.10.10.59/_layouts -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -k 

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.10.59/_layouts
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔓  Insecure              │ true
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
301      GET        2l       10w      151c http://10.10.10.59/_layouts => http://10.10.10.59/_layouts/
301      GET        2l       10w      158c http://10.10.10.59/_layouts/images => http://10.10.10.59/_layouts/images/
301      GET        2l       10w      155c http://10.10.10.59/_layouts/inc => http://10.10.10.59/_layouts/inc/
301      GET        2l       10w      158c http://10.10.10.59/_layouts/styles => http://10.10.10.59/_layouts/styles/
301      GET        2l       10w      158c http://10.10.10.59/_layouts/mobile => http://10.10.10.59/_layouts/mobile/
```

- It's a `sharepoint`
  - We can enum it using this [post](https://resources.bishopfox.com/resources/tools/sharepoint-hacking-diggity/attack-tools/)

![](./images/2.png)

- `Documents` have 1 item

![](./images/3.png)

- It contains the password for `ftp`
  - `UTDRSCH53c"$6hys`

![](./images/4.png)

- `Site Pages` also contain 1 item
  - I had to add `tally` to `/etc/hosts` to see the content
  - We have possible usernames: `rahul`, `sarah`, `tim`
  - And also `ftp_user` account

![](./images/5.png)

![](./images/6.png)

- `ftp`
  - `ftp_user:UTDRSCH53c"$6hys`
```
└─$ ftp 10.10.10.59
Connected to 10.10.10.59.
220 Microsoft FTP Service
Name (10.10.10.59:kali): ftp_user
331 Password required
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||49816|)
125 Data connection already open; Transfer starting.
08-31-17  11:51PM       <DIR>          From-Custodian
10-01-17  11:37PM       <DIR>          Intranet
08-28-17  06:56PM       <DIR>          Logs
09-15-17  09:30PM       <DIR>          To-Upload
09-17-17  09:27PM       <DIR>          User
226 Transfer complete.
ftp> 
```
- - Let's download everything
```
─$ wget -r 'ftp://ftp_user:UTDRSCH53c"$6hys@10.10.10.59'
--2023-10-01 20:33:13--  ftp://ftp_user:*password*@10.10.10.59/
           => ‘10.10.10.59/.listing’
Connecting to 10.10.10.59:21... connected.
Logging in as ftp_user ... Logged in!
==> SYST ... done.    ==> PWD ... done.
==> TYPE I ... done.  ==> CWD not needed.
==> PASV ... done.    ==> LIST ... done.
...
```


## Foothold


## User

## Root