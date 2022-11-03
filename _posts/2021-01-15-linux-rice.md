---
layout: post
title:  "Linux Rice"
date:   2021-01-15 21:03:36 +0530
categories: [unixporn]
---

akhir-akhir ini saya belom bisa nulis writeup CTF. karena waktu yang sibuk ditambah note-note ringkasan solusi writeup simpanan saya banyak yang hilang sejak ganti *Linux Distro* ke [Archlinux](https://archlinux.org). kenapa bisa hilang? karena waktu install ini OS terlalu buru-buru dengan rasa kesal yang menjadi-jadi, ribet nginstallnya (karna belom pro aja).

## Apa itu **Linux Rice**
menurut saya, **Linux Rice** memiliki arti secara istilah berasal dari *Linux* atau *GNU/Linux* yaitu sebuah sistem operasi yang *open-source* alias bisa *bisa dimodifikasi dan bisa dikembangkan siapa saja*. sedangkan *Rice* sendiri memiliki arti *Nasi* atau *Beras*, maksud Rice disini sebenarnya bukan *Nasi* ataupun *Beras*. Tetapi, *RICE stands for Race Inspired Cosmetic Enhancement*. merujuk pada mobil sport buatan Asia yang memiliki banyak modifikasi dan sedikit atau tanpa modifikasi internal. 

## Komunitas Linux Ricing
[r/unixporn](https://reddit.com/r/unixporn), sebuah komunitas untuk pecinta *Unix Customization* yang membahas tentang cara bagaimana mengkustomisasi / mengkonfigurasi sebuah sistem operasi linux. 

## Properti Yang Dibutuhkan. 
* ### Operating System
ini adalah komponen yang paling penting bagi sejuta umat, yaitu, sistem operasi. tanpa sistem operasi semua tidak berjalan semesti yang diinginkan. sedangkan sistem operasi sendiri
adalah sebuah system software yang dimana untuk mengatur hardware dari komputer. dan juga sistem operasi yang digunakan adalah *Linux*, contohnya, [Arch Linux](https://archlinux.org), [Debian](https://debian.org), [Void Linux](https://voidlinux.org), [Gentoo](https://gentoo.org).

* ### Window Manager
adalah sebuah sistem software yang dimana untuk mengatur tampilan dan tata letak dari *window* dalam sistem *Graphical User Interface* / *GUI*. **Window Manager** sendiri dibuat atau didesain untuk membantu menyediakan sebuah *desktop environment*. misalnya seperti, *i3-gaps*, *bspwm*, *dwm*, *awesome*, *openbox*, *fluxbox*, dll. 

* ### Panel / Bar
sebuah program yang digunakan untuk menampilkan status, keadaan dan informasi tentang *Network*, *Persentase Baterai*, *Memory*, *Volume Suara*, *Kecepatan Internet*, *Kalender*, dll. contohnya, [polybar](https://github.com/polybar/polybar), [lemonbar](https://github.com/LemonBoy/bar).

* ### App Launcher
sebuah program yang digunakan untuk mencari software atau program mana yang akan dijalankan walau sebenarnya bisa menggunakan Terminal Emulator. contohnya, [rofi](https://github.com/davatorium/rofi).

* ### Display Manager
adalah program yang digunakan untuk masuk ke layar GUI untuk desktop sistem operasi linux. Setelah masuk ke Graphical User Interface, kendali sepenuhnya diberikan kepada WM. DM menyediakan fitur seperti memilih sesi DE/WM yang akan digunakan dan fitur reboot, halt, suspend. contohnya, [LightDM](https://github.com/canonical/lightdm), [LXDM](http://www.linuxfromscratch.org/blfs/view/8.4/x/lxdm.html).

* ### Window Compositor
suatu program yang berfungsi untuk mengatur transparansi, animasi, bayangan, dll. pada window manager. contohnya [compton](https://github.com/chjj/compton) atau [picom](https://github.com/yshui/picom), dll.

* ### Screen Locker
sebuah program yang digunakan sebagai tampilan lockscreen pada window manager. contohnya, [betterlockscreen](https://github.com/pavanjadhaw/betterlockscreen), [i3lock](https://i3wm.org/i3lock/), dll.

* ### Terminal Emulator
adalah program yang digunakan alias berfungsi sebagai sarana interaksi dengan *Unix-Shell* seperti menjalankan program, mengirim perintah, mengatur sistem, dll. contohnya, [termite](https://github.com/thestinger/termite), [simple-terminal](https://st.suckless.org/), [kitty](https://sw.kovidgoyal.net/kitty/), dan sebagainya.

* ### Notification-Daemon
dan yang terakhir adalah sebuah program yang digunakan atau berfungsi sebagai penampil *pop-up*
notifikasi. contoh, [dunst](https://dunst-project.org/), dan masih banyak lagi.

## Macam-macam Window Managers.
* ### Stacking / Floating
Secara umum maksud dari Window Manager ini adalah bisa ditumpuk antar Program, Software Window-nya. Dan salah satu WM yang menyediakan metafora tradisional desktop yang biasanya digunakan dalam sistem operasi komersial, seperti Windows/OSX. atau contoh lainnya seperti, [Openbox](http://openbox.org/), [xfwm4](https://docs.xfce.org/xfce/xfwm4/start), [Fluxbox](https://github.com/fluxbox/fluxbox).

* ### Tiling
Bisa diartikan dari namanya, Ubin. kiasan yang artinya bisa dipisah-pisahkan sehingga tidak terjadi "overlapping" alias saling tindih. Hampir sangat jarang mempercayakan semuanya kepada mouse karena di WM ini digantikan key-bindings yang sangat luas untuk digunakan. Contoh dari Tiling WM antara lain, [bspwm](https://github.com/baskerville/bspwm), [i3](https://i3wm.org/) / [i3-gaps](https://github.com/Airblader/i3) (fork), [awesome](https://github.com/awesomeWM/awesome), [xmonad](https://github.com/xmonad/xmonad).

* ### Dynamic
Window Manager yang secara dinamis bisa switching into *Tiling* or *Stacking*. seperti yang terkenal, [dwm](https://git.suckless.org/dwm/), dan sebagainya.

## Contoh Ricing Saya
Dengan spesifikasi apa adanya dan resolusi layar yang cukup dari Laptop jadul walau gk jadul-jadul amat. saya tetap nyaman dan merasa cepat saat menggunakan Tiling. intinya, tidak ada keluhan sama sekali :)

![Archent](https://i.postimg.cc/284zKrsd/2021-09-10-18-48.png)
*(screenshot: Friday, 18/09/2021)*

<details>
    <summary>$ More Images (Click Here)...</summary>
First Rice:<br/>
<img src="https://s4.gifyu.com/images/ezgif-6-caef54386e61.gif"><br/>

<img src="https://i.ibb.co/TMNnVDg/2021-03-19-21-19.png" width="600"><br/>

(screenshot: 01/04/2021) <br/>
<img src="https://i.ibb.co/0VMSyST/dude.png" width="600"><br/>

(screenshot: 25/05/2021)<br/>
<img alt="gambarlagi" src="https://i.ibb.co/pfgR4dm/MyLove.png">
    
(screenshot: 31/05/2021)
<img alt="gambargambar" src="https://i.postimg.cc/L413r9sC/image.png">

</details>

* OS: [Arch Linux](https://archlinux.org)
* Panel/Bar: [Polybar](https://github.com/polybar/polybar)
* App Launcher: [Rofi](https://github.com/davatorium/rofi) / [Dmenu](http://tools.suckless.org/dmenu/)
* WM: [Bspwm](https://github.com/baskerville/bspwm)
* Display Manager: [LightDM](https://github.com/canonical/lightdm)
* Screen Locker: [betterlockscreen](https://github.com/pavanjadhaw/betterlockscreen)
* Terminal: [termite](https://github.com/thestinger/termite/) / [kitty](https://github.com/kovidgoyal/kitty)
* Window Compositor: [picom](https://github.com/yshui/picom)
* Notification-Daemon: [dunst](https://dunst-project.org/)

here's my [dotfiles](https://github.com/sharpicx/dotfiles) for more..

## Referensi
* <https://www.urbandictionary.com/define.php?term=Rice%20car>
* <https://en.wikipedia.org/wiki/Operating_system>
* <https://reddit.com/r/unixporn>
* <https://wiki.archlinux.org/title/Window_manager>
