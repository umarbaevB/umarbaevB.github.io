---
title: "[HTB] Machine: Acute"
description: "[HTB] Machine: Acute"
date: 2023-09-27
menu:
  sidebar:
    name: "[HTB] Machine: Acute"
    identifier: htb-machine-Acute
    parent: htb-machines-windows
    weight: 10
hero: images/acute.png
tags: ["HTB"]
---

# Acute
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.145 -T4 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-27 18:20 BST
Nmap scan report for 10.10.11.145 (10.10.11.145)
Host is up (0.12s latency).
Not shown: 65534 filtered tcp ports (no-response)
PORT    STATE SERVICE
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 33.16 seconds
```
```
└─$ nmap -Pn -p443 -sC -sV 10.10.11.145 -T4 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-27 18:21 BST
Nmap scan report for 10.10.11.145 (10.10.11.145)
Host is up (0.095s latency).

PORT    STATE SERVICE  VERSION
443/tcp open  ssl/http Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
| ssl-cert: Subject: commonName=atsserver.acute.local
| Subject Alternative Name: DNS:atsserver.acute.local, DNS:atsserver
| Not valid before: 2022-01-06T06:34:58
|_Not valid after:  2030-01-04T06:34:58
| tls-alpn: 
|_  http/1.1
|_ssl-date: 2023-09-27T17:21:14+00:00; -41s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -41s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.34 seconds

```
- Web server

![]

- `feroxbuster`
```
└─$ feroxbuster -u https://atsserver.acute.local -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -k --depth 2

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ https://atsserver.acute.local
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔓  Insecure              │ true
 🔃  Recursion Depth       │ 2
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET      818l     1728w    26105c https://atsserver.acute.local/Acute/core.css
200      GET        2l       15w     1426c https://atsserver.acute.local/Acute/wp-embed.js
301      GET        2l       10w      167c https://atsserver.acute.local/aspnet_client => https://atsserver.acute.local/aspnet_client/
200      GET        2l      378w    11224c https://atsserver.acute.local/Acute/jquery-migrate.js

```

- Possible usernames
  - And `doc` file with default password
  - And info about remote training on https://atsserver.acute.local/Acute_Staff_Access

![](./images/2.png)

![](./images/3.png)

```
└─$ exiftool New_Starter_CheckList_v7.docx                                 
ExifTool Version Number         : 12.64
File Name                       : New_Starter_CheckList_v7.docx
Directory                       : .
File Size                       : 35 kB
File Modification Date/Time     : 2023:09:27 18:34:18+01:00
File Access Date/Time           : 2023:09:27 18:34:25+01:00
File Inode Change Date/Time     : 2023:09:27 18:34:19+01:00
File Permissions                : -rw-r--r--
File Type                       : DOCX
File Type Extension             : docx
MIME Type                       : application/vnd.openxmlformats-officedocument.wordprocessingml.document
Zip Required Version            : 20
Zip Bit Flag                    : 0x0006
Zip Compression                 : Deflated
Zip Modify Date                 : 1980:01:01 00:00:00
Zip CRC                         : 0x079b7eb2
Zip Compressed Size             : 428
Zip Uncompressed Size           : 2527
Zip File Name                   : [Content_Types].xml
Creator                         : FCastle
Description                     : Created on Acute-PC01
Last Modified By                : Daniel
Revision Number                 : 8
Last Printed                    : 2021:01:04 15:54:00Z
Create Date                     : 2021:12:08 14:21:00Z
Modify Date                     : 2021:12:22 00:39:00Z
Template                        : Normal.dotm
Total Edit Time                 : 2.6 hours
Pages                           : 3
Words                           : 886
Characters                      : 5055
Application                     : Microsoft Office Word
Doc Security                    : None
Lines                           : 42
Paragraphs                      : 11
Scale Crop                      : No
Heading Pairs                   : Title, 1
Titles Of Parts                 : 
Company                         : University of Marvel
Links Up To Date                : No
Characters With Spaces          : 5930
Shared Doc                      : No
Hyperlinks Changed              : No
App Version                     : 16.0000
```

## Foothold
- User list based on `exiftool` hint
  - We also computer name `Acute-PC01`
```
AWallace
CHall
EDavies
IMonks
JMorgan
LHopkins
```

## User