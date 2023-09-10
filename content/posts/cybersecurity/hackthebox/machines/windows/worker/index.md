---
title: "[HTB] Machine: Worker"
description: "[HTB] Machine: Worker"
date: 2023-09-08
menu:
  sidebar:
    name: "[HTB] Machine: Worker"
    identifier: htb-machine-Worker
    parent: htb-machines-windows
    weight: 10
hero: images/worker.png
tags: ["HTB"]
---

# Worker
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.203 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-10 18:39 BST
Nmap scan report for 10.10.10.203 (10.10.10.203)
Host is up (0.098s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
3690/tcp open  svn
5985/tcp open  wsman
```
```
└─$ nmap -Pn -p80,3690,5985 -sC -sV 10.10.10.203 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-10 18:42 BST
Nmap scan report for 10.10.10.203 (10.10.10.203)
Host is up (0.12s latency).

PORT     STATE SERVICE  VERSION
80/tcp   open  http     Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
3690/tcp open  svnserve Subversion
5985/tcp open  http     Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.87 seconds
```

- Web server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.203/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x txt,asp,aspx --no-error   
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.203/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt,asp,aspx
[+] Timeout:                 10s
===============================================================
2023/09/10 18:47:29 Starting gobuster in directory enumeration mode
===============================================================
===============================================================
2023/09/10 18:48:34 Finished
===============================================================

```

## Foothold
- We have `svn` on port `3690`
  - https://book.hacktricks.xyz/network-services-pentesting/3690-pentesting-subversion-svn-server

```
└─$ svn log svn://10.10.10.203
------------------------------------------------------------------------
r5 | nathen | 2020-06-20 14:52:00 +0100 (Sat, 20 Jun 2020) | 1 line

Added note that repo has been migrated
------------------------------------------------------------------------
r4 | nathen | 2020-06-20 14:50:20 +0100 (Sat, 20 Jun 2020) | 1 line

Moving this repo to our new devops server which will handle the deployment for us
------------------------------------------------------------------------
r3 | nathen | 2020-06-20 14:46:19 +0100 (Sat, 20 Jun 2020) | 1 line

-
------------------------------------------------------------------------
r2 | nathen | 2020-06-20 14:45:16 +0100 (Sat, 20 Jun 2020) | 1 line

Added deployment script
------------------------------------------------------------------------
r1 | nathen | 2020-06-20 14:43:43 +0100 (Sat, 20 Jun 2020) | 1 line

First version
------------------------------------------------------------------------
```
- Let's download the repo

```
└─$ svn checkout svn://10.10.10.203
A    dimension.worker.htb
A    dimension.worker.htb/LICENSE.txt
A    dimension.worker.htb/README.txt
A    dimension.worker.htb/assets
A    dimension.worker.htb/assets/css
A    dimension.worker.htb/assets/css/font
...
```
```
└─$ ls -lha
total 20K
drwxr-xr-x  4 kali kali 4.0K Sep 10 18:58 .
drwxr-xr-x 19 kali kali 4.0K Sep 10 18:58 ..
drwxr-xr-x  4 kali kali 4.0K Sep 10 18:58 dimension.worker.htb
-rw-r--r--  1 kali kali  162 Sep 10 18:58 moved.txt
drwxr-xr-x  4 kali kali 4.0K Sep 10 18:58 .svn
```

- If we check `moved.txt`
```
└─$ cat moved.txt          
This repository has been migrated and will no longer be maintaned here.
You can find the latest version at: http://devops.worker.htb

// The Worker team :)
```

- We also saw `r2` commit from the logs about `Added deployment script`
  - We could have a creds there
  - But I'll start from the `r1`
```
└─$ svn up -r1
Updating '.':
D    moved.txt
Updated to revision 1.
```
```
└─$ svn up -r2
Updating '.':
A    deploy.ps1
Updated to revision 2.
```
- We have creds in `deploy.ps1`
  - But they don't work for `winrm`
```
└─$ cat deploy.ps1      
$user = "nathen" 
$plain = "wendel98"
$pwd = ($plain | ConvertTo-SecureString)
$Credential = New-Object System.Management.Automation.PSCredential $user, $pwd
$args = "Copy-Site.ps1"
Start-Process powershell.exe -Credential $Credential -ArgumentList ("-file $args")
```

- Let's check other `vhosts`
```
└─$ wfuzz -c -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u http://10.10.10.203 -H 'Host: FUZZ.worker.htb' --hh 703
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.203/
Total requests: 100000

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000296:   200        170 L    542 W      6495 Ch     "alpha"
000007691:   200        355 L    1408 W     16045 Ch    "story"
000022566:   401        85 L     329 W      20028 Ch    "devops"
000023339:   200        397 L    1274 W     14803 Ch    "cartoon"
000023462:   200        111 L    398 W      4971 Ch     "lens"
000024714:   200        368 L    1173 W     14588 Ch    "dimension"
000037212:   400        6 L      26 W       334 Ch      "*"
000071250:   200        274 L    871 W      10134 Ch    "twenty"  
```

- Add all of them to `/etc/hosts`
  - `10.10.10.203 worker.htb alpha.worker.htb story.worker.htb devops.worker.htb cartoon.worker.htb lens.worker.htb dimension.worker.htb twenty.worker.htb`

- `dimension.worker.htb`
  - It shows other `vhosts`

![](./images/2.png)

![](./images/3.png)

- If we visit `devops.worker.htb` we have to authenticate
  - Using the creds `nathen:wendel98` from `svn` works

![](./images/4.png)

![](./images/5.png)



## User


## Root