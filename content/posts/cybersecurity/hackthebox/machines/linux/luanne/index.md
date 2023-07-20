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
tags: ["HTB"]
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
## User


## Root

