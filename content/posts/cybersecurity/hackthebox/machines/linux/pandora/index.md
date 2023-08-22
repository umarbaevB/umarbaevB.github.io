---
title: "[HTB] Machine: Pandora"
description: "[HTB] Machine: Pandora"
date: 2023-08-18
menu:
  sidebar:
    name: "[HTB] Machine: Pandora"
    identifier: htb-machine-Pandora
    parent: htb-machines-linux
    weight: 10
hero: images/pandora.png
tags: ["HTB"]
---

# Pandora
## Enumeration
- `nmap`
```
└─$ nmap -sV -sC -Pn 10.10.11.136 -T4                                         
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-22 19:34 BST
Warning: 10.10.11.136 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.10.11.136 (10.10.11.136)
Host is up (0.15s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE    SERVICE  VERSION
22/tcp   open     ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 24:c2:95:a5:c3:0b:3f:f3:17:3c:68:d7:af:2b:53:38 (RSA)
|   256 b1:41:77:99:46:9a:6c:5d:d2:98:2f:c0:32:9a:ce:03 (ECDSA)
|_  256 e7:36:43:3b:a9:47:8a:19:01:58:b2:bc:89:f6:51:08 (ED25519)
80/tcp   open     http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Play | Landing
1503/tcp filtered imtc-mcs
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.99 seconds
```
```
└─$ sudo nmap -sU 10.10.11.136 -T4
[sudo] password for kali: 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-22 19:38 BST
Warning: 10.10.11.136 giving up on port because retransmission cap hit (6).
Stats: 0:02:58 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 20.77% done; ETC: 19:52 (0:11:15 remaining)
Nmap scan report for 10.10.11.136 (10.10.11.136)
Host is up (0.15s latency).
Not shown: 984 closed udp ports (port-unreach)
PORT      STATE         SERVICE
17/udp    open|filtered qotd
161/udp   open          snmp
162/udp   open|filtered snmptrap
682/udp   open|filtered xfr
1013/udp  open|filtered unknown
1031/udp  open|filtered iad2
19140/udp open|filtered unknown
19222/udp open|filtered unknown
20154/udp open|filtered unknown
21298/udp open|filtered unknown
29823/udp open|filtered unknown
30263/udp open|filtered unknown
32768/udp open|filtered omad
40019/udp open|filtered unknown
49503/udp open|filtered unknown
55043/udp open|filtered unknown
```
- Web server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://panda.htb  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x php,txt --no-error
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://panda.htb
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php,txt
[+] Timeout:                 10s
===============================================================
2023/08/22 19:58:05 Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 274]
/assets               (Status: 301) [Size: 307] [--> http://panda.htb/assets/]
/.php                 (Status: 403) [Size: 274]
/server-status        (Status: 403) [Size: 274]
```
- `ffuf`
```
└─$ ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u 'http://panda.htb/' -H 'Host: FUZZ.panda.htb'  -fs 33560

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://panda.htb/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.panda.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response size: 33560
________________________________________________

:: Progress: [19966/19966] :: Job [1/1] :: 120 req/sec :: Duration: [0:03:08] :: Errors: 0 ::
```
- `snmpwalk`
  - `apt install snmp-mibs-downloader`
  - comment line in `/etc/snmp/snmp.conf`
  - `snmpbulkwalk`
```
└─$ snmpbulkwalk -Cr1000 -c public -v2c 10.10.11.136 > snmpbulkwalk.out
```
- `SNMP` analysis


## Foothold

## User

## Root