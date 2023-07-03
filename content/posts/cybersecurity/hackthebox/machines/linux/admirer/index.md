---
title: "[HTB] Machine: Admirer"
description: "[HTB] Machine: Admirer"
date: 2023-06-29
menu:
  sidebar:
    name: "[HTB] Machine: Admirer"
    identifier: htb-machine-admirer
    parent: htb-machines-linux
    weight: 10
hero: images/admirer.png
tags: ["HTB"]
---

# Admirer
## Enumeration
- `nmap`
```
└─$ nmap -sC -sV -Pn 10.10.10.187
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-03 18:30 BST
Nmap scan report for 10.10.10.187 (10.10.10.187)
Host is up (0.17s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 4a71e92163699dcbdd84021a2397e1b9 (RSA)
|   256 c595b6214d46a425557a873e19a8e702 (ECDSA)
|_  256 d02dddd05c42f87b315abe57c4a9a756 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/admin-dir
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Admirer
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.52 seconds

```
- Web server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.187 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x php,txt --no-error
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.187
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php,txt
[+] Timeout:                 10s
===============================================================
2023/07/03 18:33:05 Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 313] [--> http://10.10.10.187/assets/]
/.php                 (Status: 403) [Size: 277]
/index.php            (Status: 200) [Size: 6051]
/images               (Status: 301) [Size: 313] [--> http://10.10.10.187/images/]
/robots.txt           (Status: 200) [Size: 138]
```

- Nothing on `FTP` 

![](./images/2.png)

## Foothold
- If we check `robots.txt` we find `admin-dir` and possible username `waldo`

![](./images/3.png)

- But we can't visit that page

![](./images/4.png)

- Let's `gobuster` the `admin-dir` path
  - Just like it was stated in the `robots.txt`
```
└─$ gobuster dir -u http://10.10.10.187/admin-dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x php,txt --no-error
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.187/admin-dir
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt,php
[+] Timeout:                 10s
===============================================================
2023/07/03 18:37:24 Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 277]
/contacts.txt         (Status: 200) [Size: 350]
/.php                 (Status: 403) [Size: 277]
/credentials.txt      (Status: 200) [Size: 136]
```

- Let's `curl` those endpoints

![](./images/5.png)

- Creds didn't work for `ssh`, but work for `ftp`
  - Let's download files

![](./images/6.png)
![](./images/7.png)

- Start checking the files
  - `index.php` has creds, but they don't work neither for `ssh` nor `ftp`

![](./images/8.png)

- We also have a new directory `/utility-scripts`
  - Another pair of creds, which don't work 
  - Tried `gobuster` the directory, nothing was found

![](./images/9.png)
![](./images/10.png)

- Digging through `google` resulted in [Adminer](https://www.adminer.org/)
  - Considering the name of the box `Admirer` and `db_admin` file in `/utility-scripts` could be a viable path

![](./images/11.png)

