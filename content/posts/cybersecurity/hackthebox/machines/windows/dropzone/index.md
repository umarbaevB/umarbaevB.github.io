---
title: "[HTB] Machine: Dropzone"
description: "[HTB] Machine: Dropzone"
date: 2023-09-17
menu:
  sidebar:
    name: "[HTB] Machine: Dropzone"
    identifier: htb-machine-Dropzone
    parent: htb-machines-windows
    weight: 10
hero: images/dropzone.png
tags: ["HTB"]
---

# Dropzone
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.90 -T4 --min-rate 5000 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-17 10:42 BST
Nmap scan report for 10.10.10.90 (10.10.10.90)
Host is up.
All 65535 scanned ports on 10.10.10.90 (10.10.10.90) are in ignored states.
Not shown: 65535 filtered tcp ports (no-response)

Nmap done: 1 IP address (1 host up) scanned in 65.20 seconds
```
```
└─$ sudo nmap -p- -sU 10.10.10.90 -T4 --min-rate 5000  
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-17 10:51 BST
Nmap scan report for 10.10.10.90 (10.10.10.90)
Host is up (0.098s latency).
Not shown: 65533 open|filtered udp ports (no-response)
PORT     STATE SERVICE
69/udp   open  tftp
1042/udp open  afrog

Nmap done: 1 IP address (1 host up) scanned in 39.89 seconds

```
```
└─$ sudo nmap -p69,1042 -sC -sV -sU 10.10.10.90 -T4 --min-rate 5000 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-17 10:55 BST
Nmap scan report for 10.10.10.90 (10.10.10.90)
Host is up (0.19s latency).

PORT     STATE         SERVICE VERSION
69/udp   open          tftp    SolarWinds Free tftpd
| tftp-version: 
|   If you know the name or version of the software running on this port, please submit it to dev@nmap.org along with the following information: 
|     opcode: 5
|     errcode: 1
|     length: 43
|     rport: 1044
|_    errmsg: Could not find file 'C:\BMAuRGtQ.NIU'.
1042/udp open|filtered afrog

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 123.24 seconds

```

## Root
- `tftp`
```
└─$ tftp 10.10.10.90 
tftp> ?
tftp-hpa 5.2
Commands may be abbreviated.  Commands are:

connect         connect to remote tftp
mode            set file transfer mode
put             send file
get             receive file
quit            exit tftp
verbose         toggle verbose mode
trace           toggle packet tracing
literal         toggle literal mode, ignore ':' in file name
status          show current status
binary          set mode to octet
ascii           set mode to netascii
rexmt           set per-packet transmission timeout
timeout         set total retransmission timeout
?               print help information
help            print help information
```
- We can send and get files that exist
```
└─$ tftp 10.10.10.90         
tftp> put test.txt
tftp> get test.txt
tftp> get something.txt
Error code 1: Could not find file 'C:\something.txt'.
tftp> 

```

- But we can't access 'hives'
```
└─$ tftp 10.10.10.90  
tftp> get \windows\system32\config\sam
Error code 1: The process cannot access the file 'C:\windows\system32\config\sam' because it is being used by another process.
```

- Let's gather `OS` info
  - It's `Windows XP SP3`
```
tftp> get \windows\system32\eula.txt
tftp> get \windows\system32\license.rtf
Error code 1: Could not find file 'C:\windows\system32\license.rtf'.
```
```
└─$ cat '\windows\system32\eula.txt'
END-USER LICENSE AGREEMENT FOR MICROSOFT 
SOFTWARE

MICROSOFT WINDOWS XP PROFESSIONAL EDITION 
SERVICE PACK 3

IMPORTANT-READ CAREFULLY: This End-User 
License Agreement ('EULA') is a legal 
agreement between you (either an individual 
or a single entity) and Microsoft Corporation 
or one of its affiliates ('Microsoft') for 
the Microsoft software that accompanies this 
EULA, which includes computer software and 
may include associated media, printed 
materials, 'online' or electronic 
documentation, and Internet-based services 
('Software'). An amendment or addendum to 
this EULA may accompany the Software. 

Some of the terms have changed since the 
initial release of Windows XP Service Pack 2. 
The changes include:
...
```
- 