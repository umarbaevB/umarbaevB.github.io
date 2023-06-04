---
title: Miscellaneous
menu:
  notes:
    name: Miscellaneous
    identifier: notes-misc
    parent: notes-footprinting-and-enumeration
    weight: 20
---
# Miscellaneous Options
<!-- Miscellaneous Options -->
{{< note title="Miscellaneous Options">}}
- `for i in {1..255}; do (ping -c 1 192.168.1.${i} | grep "bytes from" &); done` - Scan for hosts in subnet
- `nc -w 1 192.168.110.52 1-10000 -v -z -n 2>&1 | grep succeeded` - Scan host range of ports using `nc`
- `for i in {1..65535}; do (echo > /dev/tcp/192.168.1.1/$i) >/dev/null 2>&1 && echo $i is open; done` - Scan host range of ports
- `gobuster dir -u https://10.10.10.7/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -k` - `gobuster` ignore certificate
- `ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u 'http://<website>.com' -H 'Host: FUZZ.<website>.com'` - `ffuf` vhost enumeration
{{< /note >}}
