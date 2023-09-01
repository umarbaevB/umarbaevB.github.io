---
title: "[HTB] Machine: Haystack"
description: "[HTB] Machine: Haystack"
date: 2023-06-23
menu:
  sidebar:
    name: "[HTB] Machine: Haystack"
    identifier: htb-machine-haystack
    parent: htb-machines-linux
    weight: 10
hero: images/haystack.png
tags: ["HTB", "steganography", "elasticsearch", "ssh", "kibana", "cve-2018-17246", "javascript", "lfi", "logstash", "herokuapp"]
---

# Haystack
## Enumeration
- `nmap`
```
└─$ nmap -sC -sV -Pn 10.10.10.115
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-23 19:36 BST
Nmap scan report for 10.10.10.115 (10.10.10.115)
Host is up (0.55s latency).
Not shown: 922 filtered tcp ports (no-response), 75 filtered tcp ports (host-unreach)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 2a8de2928b14b63fe42f3a4743238b2b (RSA)
|   256 e75a3a978e8e728769a30dd100bc1f09 (ECDSA)
|_  256 01d259b2660a9749205f1c84eb81ed95 (ED25519)
80/tcp   open  http    nginx 1.12.2
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: nginx/1.12.2
9200/tcp open  http    nginx 1.12.2
|_http-title: Site doesn't have a title (application/json; charset=UTF-8).
| http-methods: 
|_  Potentially risky methods: DELETE
|_http-server-header: nginx/1.12.2

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 89.56 seconds
```
- Web server

![](./images/1.png)

- Port `9200`

![](./images/2.png)

## Foothold
- Let's download the image and check it
  - Run:
    -  `file needle.jpg`
    -  `xxd needle.jpg`
    -  `strings needle.jpg`
 - You will notice a base64 string

![](./images/3.png)

![](./images/4.png)

![](./images/5.png)

- Nothing else
  - Let's check `elasticsearch`
  - `curl http://10.10.10.115:9200/_cat/indices?v`

![](./images/6.png)

- Let's check `bank` and `quotes` indeces
  - `curl -s -X GET "http://10.10.10.115:9200/bank/_search?size=1000" | jq -c '.hits.hits[]'`
  - `curl -s -X GET "http://10.10.10.115:9200/quotes/_search?size=1000" | jq -c '.hits.hits[]'`
  - We don't find a clue that we had from image in `bank` index
  - But we find it in `quotes`

![](./images/7.png)

![](./images/8.png)

- Let's decode them

![](./images/9.png)

- SSH to box as `security`

![](./images/10.png)

## User
- Enumerate for privesc

![](./images/11.png)

![](./images/12.png)

![](./images/13.png)

- So let's port forward `5601` 
  - `ssh -L 5601:localhost:5601 security@10.10.10.115`

![](./images/14.png)

![](./images/15.png)

- We have a `LFI` in the kibana
  - https://github.com/mpgn/CVE-2018-17246

![](./images/16.png)

![](./images/17.png)

![](./images/18.png)

## Root
- We have a logstash running as root

![](./images/19.png)

![](./images/20.png)

- Let's open `conf` files
  - According to `input.conf` every 10 seconds checks `/opt/kibana/logstash_*` and inputs files to `filter.conf` which eventually get executed based on `output.conf`

![](./images/21.png)

- Let's exploit the exec plugin by crafting the payload that passes the filter
  - Launch listener and wait for connection 
  - Rooted

![](./images/22.png)

![](./images/23.png)
