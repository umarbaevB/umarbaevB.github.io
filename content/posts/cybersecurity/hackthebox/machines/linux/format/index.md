---
title: "[HTB] Machine: Format"
description: "[HTB] Machine: Format"
date: 2023-10-28
menu:
  sidebar:
    name: "[HTB] Machine: Format"
    identifier: htb-machine-Format
    parent: htb-machines-linux
    weight: 10
hero: images/format.png
tags: ["HTB"]
---

# Format
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.213 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-30 18:41 GMT
Warning: 10.10.11.213 giving up on port because retransmission cap hit (10).
Nmap scan report for microblog.htb (10.10.11.213)
Host is up (0.18s latency).
Not shown: 63979 closed tcp ports (conn-refused), 1553 filtered tcp ports (no-response)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3000/tcp open  ppp

Nmap done: 1 IP address (1 host up) scanned in 114.64 seconds

```
```
└─$ nmap -Pn -p22,80,3000 -sC -sV 10.10.11.213 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-30 18:43 GMT
Nmap scan report for microblog.htb (10.10.11.213)
Host is up (0.21s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 c3:97:ce:83:7d:25:5d:5d:ed:b5:45:cd:f2:0b:05:4f (RSA)
|   256 b3:aa:30:35:2b:99:7d:20:fe:b6:75:88:40:a5:17:c1 (ECDSA)
|_  256 fa:b3:7d:6e:1a:bc:d1:4b:68:ed:d6:e8:97:67:27:d7 (ED25519)
80/tcp   open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: 404 Not Found
3000/tcp open  http    nginx 1.18.0
|_http-title:  Microblog
|_http-server-header: nginx/1.18.0
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.96 seconds
```

- `vhosts`
```
└─$ wfuzz -u http://microblog.htb/ -H 'Host: FUZZ.microblog.htb' -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt --hw 11 
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://microblog.htb/
Total requests: 19966

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000111:   200        83 L     306 W      3973 Ch     "app"
000001619:   200        42 L     434 W      3731 Ch     "sunny" 
```

- `gobuster`
```
└─$ gobuster dir -u http://microblog.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,js --no-error

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://microblog.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt,html,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================

```
```
└─$ sudo vim /etc/hosts      
[sudo] password for kali: 
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u http://app.microblog.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,js --no-error
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://app.microblog.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,js,php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 3976]
/login                (Status: 301) [Size: 169] [--> http://app.microblog.htb/login/]
/register             (Status: 301) [Size: 169] [--> http://app.microblog.htb/register/]
/logout               (Status: 301) [Size: 169] [--> http://app.microblog.htb/logout/]

```
```
└─$ gobuster dir -u http://sunny.microblog.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,js --no-error
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://sunny.microblog.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt,html,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 169] [--> http://sunny.microblog.htb/images/]
/index.php            (Status: 200) [Size: 3732]
/content              (Status: 301) [Size: 169] [--> http://sunny.microblog.htb/content/]
/edit                 (Status: 301) [Size: 169] [--> http://sunny.microblog.htb/edit/]

```

- `microblog.htb`

![](./images/1.png)

- `app.microblog.htb`

![](./images/2.png)

- `sunny.microblog.htb`

![](./images/3.png)
