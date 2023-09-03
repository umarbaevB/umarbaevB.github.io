---
title: "[HTB] Machine: Bart"
description: "[HTB] Machine: Bart"
date: 2023-09-01
menu:
  sidebar:
    name: "[HTB] Machine: Bart"
    identifier: htb-machine-Bart
    parent: htb-machines-windows
    weight: 10
hero: images/bart.png
tags: ["HTB"]
---

# Bart
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.81 -T4                            
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-03 16:59 BST
Nmap scan report for 10.10.10.81 (10.10.10.81)
Host is up (0.091s latency).
Not shown: 65534 filtered tcp ports (no-response)
PORT   STATE SERVICE
80/tcp open  http
```
```
└─$ nmap -Pn -p80 -sC -sV 10.10.10.81 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-03 17:04 BST
Nmap scan report for 10.10.10.81 (10.10.10.81)
Host is up (0.22s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 10.0
|_http-title: Did not follow redirect to http://forum.bart.htb/
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
- `vhosts`
```
└─$ ffuf -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u 'http://bart.htb/' -H 'Host: FUZZ.bart.htb' -fs 0

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://bart.htb/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
 :: Header           : Host: FUZZ.bart.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response size: 0
________________________________________________

[Status: 200, Size: 35529, Words: 2090, Lines: 549, Duration: 148ms]
    * FUZZ: forum

[Status: 200, Size: 3423, Words: 385, Lines: 81, Duration: 1330ms]
    * FUZZ: monitor
```
- `gobuster`
  - `forum.bart.htb` had not results
```
└─$ gobuster dir -u http://monitor.bart.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x php,txt --no-error --exclude-length 75
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://monitor.bart.htb/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] Exclude Length:          75
[+] User Agent:              gobuster/3.5
[+] Extensions:              php,txt
[+] Timeout:                 10s
===============================================================
2023/09/03 20:56:29 Starting gobuster in directory enumeration mode
===============================================================
/docs                 (Status: 301) [Size: 152] [--> http://monitor.bart.htb/docs/]
/index.php            (Status: 200) [Size: 3423]
/static               (Status: 301) [Size: 154] [--> http://monitor.bart.htb/static/]
/src                  (Status: 301) [Size: 151] [--> http://monitor.bart.htb/src/]
/Index.php            (Status: 200) [Size: 3423]
/install.php          (Status: 200) [Size: 3714]
/vendor               (Status: 301) [Size: 154] [--> http://monitor.bart.htb/vendor/]
/config.php           (Status: 200) [Size: 0]
/Docs                 (Status: 301) [Size: 152] [--> http://monitor.bart.htb/Docs/]
/cron                 (Status: 301) [Size: 152] [--> http://monitor.bart.htb/cron/]
/INSTALL.php          (Status: 200) [Size: 3714]
/INDEX.php            (Status: 200) [Size: 3423]
```

## Foothold

## User

## Root