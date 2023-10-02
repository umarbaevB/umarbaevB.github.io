---
title: "[HTB] Machine: BankRobber"
description: "[HTB] Machine: BankRobber"
date: 2023-09-30
menu:
  sidebar:
    name: "[HTB] Machine: BankRobber"
    identifier: htb-machine-BankRobber
    parent: htb-machines-windows
    weight: 10
hero: images/bankrobber.png
tags: ["HTB"]
---

# BankRobber
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.154 --min-rate 10000                     
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 21:23 BST
Nmap scan report for 10.10.10.154 (10.10.10.154)
Host is up (0.11s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
443/tcp  open  https
445/tcp  open  microsoft-ds
3306/tcp open  mysql

Nmap done: 1 IP address (1 host up) scanned in 28.12 seconds

```
```
└─$ nmap -Pn -p80,443,445,3306 -sC -sV 10.10.10.154 --min-rate 10000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 21:27 BST
Nmap scan report for 10.10.10.154 (10.10.10.154)
Host is up (0.13s latency).

PORT     STATE SERVICE      VERSION
80/tcp   open  http         Apache httpd 2.4.39 ((Win64) OpenSSL/1.1.1b PHP/7.3.4)
|_http-server-header: Apache/2.4.39 (Win64) OpenSSL/1.1.1b PHP/7.3.4
|_http-title: E-coin
443/tcp  open  ssl/http     Apache httpd 2.4.39 ((Win64) OpenSSL/1.1.1b PHP/7.3.4)
|_http-server-header: Apache/2.4.39 (Win64) OpenSSL/1.1.1b PHP/7.3.4
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
|_http-title: E-coin
445/tcp  open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3306/tcp open  mysql        MariaDB (unauthorized)
Service Info: Host: BANKROBBER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2023-09-30T20:27:51
|_  start_date: 2023-09-30T20:21:05
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: mean: -3s, deviation: 0s, median: -4s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.68 seconds

```
- `mysql`
```
└─$ mysql -h 10.10.10.154 --port 3306
ERROR 1130 (HY000): Host '10.10.16.9' is not allowed to connect to this MariaDB server
```
- `smb`
```
└─$ smbmap -H 10.10.10.154
[!] Authentication error on 10.10.10.154

```

- Web Server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.154/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50  -x php,txt --no-error
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.154/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php,txt
[+] Timeout:                 10s
===============================================================
2023/10/01 19:41:43 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 8245]
/img                  (Status: 301) [Size: 334] [--> http://10.10.10.154/img/]
/login.php            (Status: 302) [Size: 0] [--> index.php]
/register.php         (Status: 200) [Size: 0]
/user                 (Status: 301) [Size: 335] [--> http://10.10.10.154/user/]
/admin                (Status: 301) [Size: 336] [--> http://10.10.10.154/admin/]
/link.php             (Status: 200) [Size: 0]
/css                  (Status: 301) [Size: 334] [--> http://10.10.10.154/css/]
/Index.php            (Status: 200) [Size: 8245]
/Login.php            (Status: 302) [Size: 0] [--> index.php]
/js                   (Status: 301) [Size: 333] [--> http://10.10.10.154/js/]
/examples             (Status: 503) [Size: 1058]
/notes.txt            (Status: 200) [Size: 133]
/logout.php           (Status: 302) [Size: 0] [--> index.php?msg=Succesfully logged out]
/licenses             (Status: 403) [Size: 1203]
/Register.php         (Status: 200) [Size: 0]
/fonts                (Status: 301) [Size: 336] [--> http://10.10.10.154/fonts/]

```

## Foothold
- http://10.10.10.154/notes.txt
```
- Move all files from the default Xampp folder: TODO
- Encode comments for every IP address except localhost: Done
- Take a break..
```

- Let's register a user

![](./images/2.png)

- We can try transfering the coins
  - But as soon as we try to do that we have a pop up message, stating that admin will review the transaction

![](./images/3.png)

![](./images/4.png)
