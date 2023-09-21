---
title: "[HTB] Machine: Flight"
description: "[HTB] Machine: Flight"
date: 2023-09-20
menu:
  sidebar:
    name: "[HTB] Machine: Flight"
    identifier: htb-machine-Flight
    parent: htb-machines-windows
    weight: 10
hero: images/flight.png
tags: ["HTB"]
---

# Flight
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.187 -T4 --min-rate 5000                                             
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-20 19:23 BST
Nmap scan report for 10.10.11.187 (10.10.11.187)
Host is up (0.099s latency).
Not shown: 65517 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49668/tcp open  unknown
49675/tcp open  unknown
49676/tcp open  unknown
49696/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 43.28 seconds
```
```
└─$ nmap -Pn -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -sC -sV 10.10.11.187 -T4 --min-rate 5000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-20 19:26 BST
Nmap scan report for 10.10.11.187 (10.10.11.187)
Host is up (0.13s latency).

PORT     STATE    SERVICE       VERSION
53/tcp   open     domain        Simple DNS Plus
80/tcp   open     http          Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
|_http-title: g0 Aviation
88/tcp   filtered kerberos-sec
135/tcp  open     msrpc         Microsoft Windows RPC
139/tcp  open     netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open     ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
445/tcp  open     microsoft-ds?
464/tcp  filtered kpasswd5
593/tcp  open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open     tcpwrapped
3268/tcp open     ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
3269/tcp open     tcpwrapped
5985/tcp open     http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open     mc-nmf        .NET Message Framing
Service Info: Host: G0; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-09-21T01:26:00
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 6h59m19s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 56.48 seconds

```

- Web server

![](./images/1.png)

- `vhosts`
```
└─$ wfuzz -u http://flight.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.flight.htb" --hh 7069                      
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://flight.htb/
Total requests: 19966

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000624:   200        90 L     412 W      3996 Ch     "school"  
```

- `gobuster`
  - `feroxbuster -u http://flight.htb -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -k `
```
└─$ gobuster dir -u http://flight.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x txt,php --no-error  
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://flight.htb/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt,php
[+] Timeout:                 10s
===============================================================
2023/09/21 17:49:03 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 333] [--> http://flight.htb/images/]
/Images               (Status: 301) [Size: 333] [--> http://flight.htb/Images/]
/css                  (Status: 301) [Size: 330] [--> http://flight.htb/css/]
/js                   (Status: 301) [Size: 329] [--> http://flight.htb/js/]
/examples             (Status: 503) [Size: 399]
/licenses             (Status: 403) [Size: 418]
/IMAGES               (Status: 301) [Size: 333] [--> http://flight.htb/IMAGES/]
/%20                  (Status: 403) [Size: 299]
/*checkout*.php       (Status: 403) [Size: 299]
/*checkout*           (Status: 403) [Size: 299]
/*checkout*.txt       (Status: 403) [Size: 299]
/CSS                  (Status: 301) [Size: 330] [--> http://flight.htb/CSS/]
/JS                   (Status: 301) [Size: 329] [--> http://flight.htb/JS/]
/phpmyadmin           (Status: 403) [Size: 418]
/webalizer            (Status: 403) [Size: 418]
/*docroot*.php        (Status: 403) [Size: 299]
/*docroot*            (Status: 403) [Size: 299]
/*docroot*.txt        (Status: 403) [Size: 299]
/*.txt                (Status: 403) [Size: 299]
/*                    (Status: 403) [Size: 299]
/*.php                (Status: 403) [Size: 299]
/con                  (Status: 403) [Size: 299]
/con.txt              (Status: 403) [Size: 299]
/con.php              (Status: 403) [Size: 299]

```
```
└─$ gobuster dir -u http://school.flight.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x txt,php --no-error 
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://school.flight.htb/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt,php
[+] Timeout:                 10s
===============================================================
2023/09/21 17:53:03 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 3996]
/images               (Status: 301) [Size: 347] [--> http://school.flight.htb/images/]
/Images               (Status: 301) [Size: 347] [--> http://school.flight.htb/Images/]
/Index.php            (Status: 200) [Size: 3996]
/examples             (Status: 503) [Size: 406]
/styles               (Status: 301) [Size: 347] [--> http://school.flight.htb/styles/]
/licenses             (Status: 403) [Size: 425]
/IMAGES               (Status: 301) [Size: 347] [--> http://school.flight.htb/IMAGES/]
/%20                  (Status: 403) [Size: 306]
/INDEX.php            (Status: 200) [Size: 3996]
/*checkout*.txt       (Status: 403) [Size: 306]
/*checkout*           (Status: 403) [Size: 306]
/*checkout*.php       (Status: 403) [Size: 306]
/phpmyadmin           (Status: 403) [Size: 425]
/webalizer            (Status: 403) [Size: 425]
/Styles               (Status: 301) [Size: 347] [--> http://school.flight.htb/Styles/]
/*docroot*.php        (Status: 403) [Size: 306]
/*docroot*.txt        (Status: 403) [Size: 306]
/*docroot*            (Status: 403) [Size: 306]
/*                    (Status: 403) [Size: 306]
/*.txt                (Status: 403) [Size: 306]
/*.php                (Status: 403) [Size: 306]
/con.txt              (Status: 403) [Size: 306]
/con                  (Status: 403) [Size: 306]
/con.php              (Status: 403) [Size: 306]
/http%3A              (Status: 403) [Size: 306]
/http%3A.txt          (Status: 403) [Size: 306]
/http%3A.php          (Status: 403) [Size: 306]
/**http%3a.txt        (Status: 403) [Size: 306]
/**http%3a            (Status: 403) [Size: 306]
/**http%3a.php        (Status: 403) [Size: 306]
/*http%3A.txt         (Status: 403) [Size: 306]
/*http%3A.php         (Status: 403) [Size: 306]
/*http%3A             (Status: 403) [Size: 306]
/aux                  (Status: 403) [Size: 306]
/aux.txt              (Status: 403) [Size: 306]
/aux.php              (Status: 403) [Size: 306]
/**http%3A.php        (Status: 403) [Size: 306]
/**http%3A            (Status: 403) [Size: 306]
/**http%3A.txt        (Status: 403) [Size: 306]
/%C0                  (Status: 403) [Size: 306]
/%C0.txt              (Status: 403) [Size: 306]
/%C0.php              (Status: 403) [Size: 306]

```

- `smb`
```
└─$ smbclient -N -L //flight.htb                                                                        
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to flight.htb failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

```

- `school.flight.htb`

![](./images/2.png)