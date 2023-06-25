---
title: "[HTB] Machine: Postman"
description: "[HTB] Machine: Postman"
date: 2023-06-24
menu:
  sidebar:
    name: "[HTB] Machine: Postman"
    identifier: htb-machine-postman
    parent: htb-machines-linux
    weight: 10
hero: images/postman.png
tags: ["HTB"]
---

# Postman
## Enumeration 
- `nmap`
```
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-25 17:52 BST
Nmap scan report for postman (10.10.10.160)
Host is up (0.12s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 46834ff13861c01c74cbb5d14a684d77 (RSA)
|   256 2d8d27d2df151a315305fbfff0622689 (ECDSA)
|_  256 ca7c82aa5ad372ca8b8a383a8041a045 (ED25519)
80/tcp    open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: The Cyber Geek's Personal Website
|_http-server-header: Apache/2.4.29 (Ubuntu)
6379/tcp  open  redis   Redis key-value store 4.0.9
10000/tcp open  http    MiniServ 1.910 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
|_http-server-header: MiniServ/1.910
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.20 seconds
```

- Web server

![](./images/1.png)

- Port `10000`

![](./images/2.png)

- Port `6379`
  - `redis-tools`

![](./images/3.png)

## Foothold
- We can use this [post](https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis) as a guide
  - So after trying several options I stopped on `SSH`: https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis#ssh
  - Generate key if you have not yet: `ssh-keygen -b 2048 -t rsa`
  - Create a `txt` file from key: `(echo -e "\n\n"; cat ~/.ssh/id_rsa.pub; echo -e "\n\n") > key.txt`

![](./images/4.png)

- Connect using `id_rsa`

![](./images/5.png)
## User
- We have another user `Matt`

![](./images/6.png)

- Found a key in `/opt` folder
  - Which is encrypted

![](./images/7.png)

- Let's use `john`

![](./images/8.png)

- But it doesn't work
  - If we check the config file for `ssh`, we see that the `Matt`'s connections are denied

![](./images/9.png)
![](./images/10.png)

- Luckily `Matt` reuses the password as ssh key password

![](./images/11.png)

## Root
- We saw that `webmin` is running as a `root`

![](./images/13.png)

- Let's search exploits for `webmin`

![](./images/12.png)

- Login as `Matt` to `webmin` on port `10000`
  - We can see the version

![](./images/14.png)

- Googling for exploits for that specific version will result in [CVE-2019-12840](https://www.exploit-db.com/exploits/46984)
  - [poc](https://github.com/KrE80r/webmin_cve-2019-12840_poc/blob/master/CVE-2019-12840.py)

![](./images/15.png)

- Let's download poc and run it
  - Rooted

![](./images/16.png)
![](./images/17.png)
