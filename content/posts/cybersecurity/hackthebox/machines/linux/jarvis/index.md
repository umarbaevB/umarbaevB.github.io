---
title: "[HTB] Machine: Jarvis"
description: "[HTB] Machine: Jarvis"
date: 2023-10-25
menu:
  sidebar:
    name: "[HTB] Machine: Jarvis"
    identifier: htb-machine-Jarvis
    parent: htb-machines-linux
    weight: 10
hero: images/jarvis.png
tags: ["HTB"]
---

# Jarvis
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.143 --min-rate 5000                                                            
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-26 19:46 BST
Nmap scan report for 10.10.10.143 (10.10.10.143)
Host is up (0.21s latency).
Not shown: 65478 filtered tcp ports (no-response), 55 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
64999/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 28.66 seconds
```
```
└─$ nmap -Pn -p22,80,64999 -sC -sV 10.10.10.143 --min-rate 5000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-26 19:47 BST
Nmap scan report for 10.10.10.143 (10.10.10.143)
Host is up (0.24s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:f3:4e:22:36:3e:3b:81:30:79:ed:49:67:65:16:67 (RSA)
|   256 25:d8:08:a8:4d:6d:e8:d2:f8:43:4a:2c:20:c8:5a:f6 (ECDSA)
|_  256 77:d4:ae:1f:b0:be:15:1f:f8:cd:c8:15:3a:c3:69:e1 (ED25519)
80/tcp    open  http    Apache httpd 2.4.25 ((Debian))
|_http-title: Stark Hotel
|_http-server-header: Apache/2.4.25 (Debian)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
64999/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.25 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.80 seconds

```

- Web server

![](./images/1.png)

- Port `64999`

![](./images/2.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.143 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php,html -t 50
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.143
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 277]
/.html                (Status: 403) [Size: 277]
/images               (Status: 301) [Size: 313] [--> http://10.10.10.143/images/]
/index.php            (Status: 200) [Size: 23628]
/nav.php              (Status: 200) [Size: 1333]
/footer.php           (Status: 200) [Size: 2237]
/css                  (Status: 301) [Size: 310] [--> http://10.10.10.143/css/]
/js                   (Status: 301) [Size: 309] [--> http://10.10.10.143/js/]
/fonts                (Status: 301) [Size: 312] [--> http://10.10.10.143/fonts/]
/phpmyadmin           (Status: 301) [Size: 317] [--> http://10.10.10.143/phpmyadmin/]
/connection.php       (Status: 200) [Size: 0]
/room.php             (Status: 302) [Size: 3024] [--> index.php]
/sass                 (Status: 301) [Size: 311] [--> http://10.10.10.143/sass/]
```