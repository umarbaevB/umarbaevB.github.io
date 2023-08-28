---
title: "[HTB] Machine: Agile"
description: "[HTB] Machine: Agile"
date: 2023-08-05
menu:
  sidebar:
    name: "[HTB] Machine: Agile"
    identifier: htb-machine-Agile
    parent: htb-machines-linux
    weight: 10
hero: images/Agile.png
tags: ["HTB", "flask", "python", "file-read", "werkzeug", "werkzeug-debug", "flask-debug-pin", "python-venv", "selenium", "chrome", "chrome-debug", "sudoedit", "cve-2023-22809"]
---

# Agile
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p22,80,7352,12249,13006,15172,26869,28151,34525,34972,37863,40486,49272,63692 -sC -sV 10.10.11.203 -T4
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-20 15:31 EDT
Nmap scan report for 10.10.11.203 (10.10.11.203)
Host is up (0.18s latency).

PORT      STATE  SERVICE VERSION
22/tcp    open   ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f4bcee21d71f1aa26572212d5ba6f700 (ECDSA)
|_  256 65c1480d88cbb975a02ca5e6377e5106 (ED25519)
80/tcp    open   http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://superpass.htb
|_http-server-header: nginx/1.18.0 (Ubuntu)
7352/tcp  closed swx
12249/tcp closed unknown
13006/tcp closed unknown
15172/tcp closed unknown
26869/tcp closed unknown
28151/tcp closed unknown
34525/tcp closed unknown
34972/tcp closed unknown
37863/tcp closed unknown
40486/tcp closed unknown
49272/tcp closed unknown
63692/tcp closed unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.44 seconds
```
- `gobuster`
```
└─$ gobuster dir -u http://superpass.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://superpass.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/05/20 15:33:55 Starting gobuster in directory enumeration mode
===============================================================
/download             (Status: 302) [Size: 249] [--> /account/login?next=%2Fdownload]
/static               (Status: 301) [Size: 178] [--> http://superpass.htb/static/]
/vault                (Status: 302) [Size: 243] [--> /account/login?next=%2Fvault]
```
- Web server

![](./images/1.png)

## Foothold
- I created a user and was redirected to `/vault`
  - Added a password, saved it and clicked `export`
  - I intercepted the request in `Burp` and played with it

![](./images/4.png)

- While playing around with `/vault` route, the error popped up
  - It indicated the backend tech used: `flask` and `mysql`
  - It looks like the `debug` console is enabled and I tried opening it, but it's protected with `pin`

![](./images/5.png)

![](./images/6.png)

- We know it's a `flask` application
  - And I finally managed to get an `lfi` by playing around with payloads

![](./images/8.png)

![](./images/9.png)

- I was stuck here for a while
  - I couldn't find anything useful, except `config_prod.json` file, which was retrieved via from `/proc/self/environ`
    - `mysql+pymysql://superpassuser:dSA6l7q*yIVs$39Ml6ywvgK@localhost/superpass`

![](./images/10.png)

![](./images/11.png)

- But then I remembered about the `pin` and debug console
  - The following post has all the information we need to crack the `pin`
    - https://www.bengrewell.com/cracking-flask-werkzeug-console-pin/
  - So I basically try replicating the steps
    - I started gathering all the data I needed from box using `lfi`

![](./images/14.png)

- After filling in required info, I ran the script

![](./images/13.png)

- And we have successfully cracked the `pin`
  - Now let's get `rce`
  - And we have our foothold

![](./images/15.png)

![](./images/16.png)

## User #1
- Enumerate the box as usual, but I had `mysql` creds from before
  - I checked the database and found creds for another user 
    - `corum:5db7caa1d13cc37c9fc2`
  - Now we can `ssh`

![](./images/17.png)

![](./images/18.png)

## User #2
- We have another `vhost`
  - But it's available locally
```
server {
    listen 127.0.0.1:80;
    server_name test.superpass.htb;
    location /static {
        alias /app/app-testing/superpass/static;
        expires 365d;
    }
    location / {
        include uwsgi_params;
        proxy_pass http://127.0.0.1:5555;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Protocol $scheme;
    }
}
```
- If we check `/app`, we have `app-testing`
  - Inside we `creds.txt`, but it's not readable
  - By the way, `linpeas` also showed `chromedriver` running in the process list
  - If we check `test_site_interactively.py` in `/app/app-testing/tests/functional`, we can see it uses `creds.txt`
    - It uses `selenium` to load the site

```
corum@agile:/app$ ls -l
total 24
drwxr-xr-x 5 corum     runner    4096 Jan 23 21:50 app
drwxr-xr-x 8 runner    runner    4096 Jan 25 17:29 app-testing
-r--r----- 1 dev_admin www-data    88 Jan 25 00:00 config_prod.json
-r--r----- 1 dev_admin runner      99 Jan 25 15:15 config_test.json
-rwxr-xr-x 1 root      runner     557 Jan 25 17:36 test_and_update.sh
drwxrwxr-x 5 root      dev_admin 4096 Jan 25 17:21 venv
```

- So googling resulted in a nice [post](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/chrome-remote-debugger-pentesting/)
  - Now we need to setup a forwarding
  - And open `chrome` 
    - `chrome://inspect` and `devices` page
    - Add new port in via `Configure`, which will set up a new remote target device `localhost:41829`
    - Click `inspect`
  - And we get our creds: `edwards:d07867c6267dcb5df0af`

![](./images/21.png)

![](./images/20.png)

- `ssh` as edwards

![](./images/22.png)

## Root
- Nothing on `mysql` using creds: `mysql+pymysql://superpasstester:VUO8A2c2#3FnLq3*a9DX1U@localhost/superpasstest`
- But we have `sudo` rights for `sudoedit`
  - https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudoedit-privilege-escalation/

![](./images/27.png)

- We remember a `test_and_update.sh` in `/app` directory
  - It had a hint `# system-wide source doesn't seem to happen in cron jobs`
  - So if we check `$PATH`, we find that each shell is running with python virtual environment

```
corum@agile:~$ echo $PATH
/app/venv/bin:/app/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```
```
edwards@agile:~$ tail -2 /etc/bash.bashrc 
# all users will want the env associated with this application
source /app/venv/bin/activate
```

- If we monitor `pspy` we see that `root` periodically logs in
  - And each time it will source `/app/venv/bin/activate`
  - So we can modify a `/app/venv/bin/activate` via `CVE-2023-22809`
    - https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudoedit-privilege-escalation/
  - And wait for execution

![](./images/28.png)

![](./images/29.png)
