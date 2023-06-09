---
title: "[HTB] Machine: Sense"
description: "[HTB] Machine: Sense"
date: 2023-06-07
menu:
  sidebar:
    name: "[HTB] Machine: Sense"
    identifier: htb-machine-sense
    parent: htb-machines-linux
    weight: 10
hero: images/sense.png
tags: ["HTB"]
---

# Sense
## Enumeration
- `nmap`
```
└─$ nmap -sC -sV 10.10.10.60
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-09 16:21 BST
Nmap scan report for 10.10.10.60 (10.10.10.60)
Host is up (0.11s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
80/tcp  open  http     lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Did not follow redirect to https://10.10.10.60/
443/tcp open  ssl/http lighttpd 1.4.35
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=Common Name (eg, YOUR name)/organizationName=CompanyName/stateOrProvinceName=Somewhere/countryName=US
| Not valid before: 2017-10-14T19:21:35
|_Not valid after:  2023-04-06T19:21:35
|_http-title: Login
|_http-server-header: lighttpd/1.4.35

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.55 seconds
```
- d
```
┌──(kali㉿kali)-[~/Documents/tasks]
└─$ gobuster dir -u https://10.10.10.60/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -t 50 -x php -k
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://10.10.10.60/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
2023/06/09 16:31:53 Starting gobuster in directory enumeration mode
===============================================================
/includes             (Status: 301) [Size: 0] [--> https://10.10.10.60/includes/]
/themes               (Status: 301) [Size: 0] [--> https://10.10.10.60/themes/]
/css                  (Status: 301) [Size: 0] [--> https://10.10.10.60/css/]
/xmlrpc.php           (Status: 200) [Size: 384]
/stats.php            (Status: 200) [Size: 6690]
/classes              (Status: 301) [Size: 0] [--> https://10.10.10.60/classes/]
/help.php             (Status: 200) [Size: 6689]
/javascript           (Status: 301) [Size: 0] [--> https://10.10.10.60/javascript/]
/system.php           (Status: 200) [Size: 6691]
/index.php            (Status: 200) [Size: 6690]
/edit.php             (Status: 200) [Size: 6689]
/widgets              (Status: 301) [Size: 0] [--> https://10.10.10.60/widgets/]
/status.php           (Status: 200) [Size: 6691]
/installer            (Status: 301) [Size: 0] [--> https://10.10.10.60/installer/]
/~                    (Status: 403) [Size: 345]
/exec.php             (Status: 200) [Size: 6689]
/license.php          (Status: 200) [Size: 6692]
/tree                 (Status: 301) [Size: 0] [--> https://10.10.10.60/tree/]
/graph.php            (Status: 200) [Size: 6690]
/wizards              (Status: 301) [Size: 0] [--> https://10.10.10.60/wizards/]
/wizard.php           (Status: 200) [Size: 6691]
/interfaces.php       (Status: 200) [Size: 6695]
/~sys~                (Status: 403) [Size: 345]
Progress: 38640 / 53170 (72.67%)[ERROR] 2023/06/09 16:34:54 [!] parse "https://10.10.10.60/besalu\t.php": net/url: invalid control character in URL
Progress: 41449 / 53170 (77.96%)[ERROR] 2023/06/09 16:35:07 [!] parse "https://10.10.10.60/error\x1f_log": net/url: invalid control character in URL
[ERROR] 2023/06/09 16:35:07 [!] parse "https://10.10.10.60/error\x1f_log.php": net/url: invalid control character in URL
Progress: 53045 / 53170 (99.76%)
===============================================================
2023/06/09 16:35:48 Finished
===============================================================
```
## Foothold/User

## Root