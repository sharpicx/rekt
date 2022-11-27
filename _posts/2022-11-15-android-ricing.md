---
layout: post
title: "Android Ricing"
date: "2022-11-27"
categories: unixporn
---
 
## Apa itu Android Ricing

> Android ricing refers to changing up the UI of a device running an Android OS to better suit one's taste. This could be done for many reasons, such as to make using the device more practical, or to make it suit the user's tastes. (*[wiki.installgentoo.com](https://wiki.installgentoo.com/wiki/Android_ricing)*, Diakses pada 27 November).

Menurut Wikinya Gentoo, Android ricing merupakan upaya untuk mengganti atau mengubah User Interface perangkat yang berjalan pada sistem operasi Android untuk dicocokkan sesuai dengan selera orang. Dan ini bisa dilakukan dengan banyak alasan, seperti untuk menggunakan perangkat lebih praktis, atau untuk membuat perangkat selaras dengan selera pengguna dan masih banyak lagi.

## Android Ricing Community
* <https://forum.xda-developers.com>
* <https://t.me/redminote8indonesia>
* <https://t.me/infinixhot8>
* <https://t.me/naz_dev>
* <https://t.me/androidwithlinux_discussion>
* <https://t.me/xdroid_update>
* <https://t.me/ancientid>
* And, if you're researcher. I know y'all got skill to dig all those stuff by dorking it via tons of search engines on the internet.

## Development

### ROM / Read Only Memory (Android)
  ROM merupakan media penyimpanan internal (**internal storage**) di mana firmware dan aplikasi bawaan tersimpan (*[ngelag.com](https://ngelag.com/apa-itu-rom-pada-smartphone-android/)*, 2022). Firmware merupakan salah satu perangkat lunak yang mana ia akan disimpan dalam `read-only` format dan tidak akan bisa diubah ketika tidak dialiri dengan listrik (*[qwords.com](qwords.com/blog/apa-itu-firmware)*, 2020). ROM adalah sebuah hardware yang bertugas sebagai media penyimpanan internal dan data yang ada di dalamnya hanya bisa dibaca (read). Namun melalui proses *flashing*, maka ROM ini bisa ditulis (write) / diisi dengan data (*[ngelag.com](https://ngelag.com/apa-itu-rom-pada-smartphone-android/)*, 2022). Usually, ROM itu refleksi dari kata "Sistem Operasi" kalau anda berada dalam komunitas, tidak jarang juga para developer ROM menggunakan nama "OS" didalam *project*nya. Jadi, OS, ROM, bahkan firmware ini secara literal mengandung makna yang sama bagi "mereka".

* ### Macam-macam ROMs.
  * **Stock ROM** adalah firmware dasar pada smartphone android yang dibuat agar sebuah smartphone android bisa bekerja. Stok ROM biasanya memiliki desain dasar tanpa ada branding sedikitpun. Intinya Stock ROM dibuat hanya sampai smartphone android dan hardware yang ada didalamnya bisa bekerja (*[ngelag.com](https://ngelag.com/apa-itu-rom-pada-smartphone-android/)*, 2022).

  * **Custom ROM** adalah firmware smartphone android yang dirilis secara tidak resmi oleh seorang pengembang yang mungkin memiliki fitur tampilan baru, fitur baru hingga peningkatan performa. Custom ROM biasanya dibuat ketika sebuah smartphone pabrikan tertentu memiliki kelemahan dalam kinerjanya, sehingga untuk menutupi kelemahan tersebut dibutuhkan perubahan terhadap ROM atau Firmwarenya (*[ngelag.com](https://ngelag.com/apa-itu-rom-pada-smartphone-android/)*, 2022). 

  * **Branded Stock ROM** Branded Stock ROM pada dasarnya adalah sebuah stock ROM yang dicustom dirubah sesuai dengan keinginan produsen atau pemegang merek smartphone Android tertentu. Perubahan disini biasanya meliputi desain atau tampilan, aplikasi-aplikasi tertentu hingga perubahan yang dilakukan agar performa smartphone Android meningkat (*[ngelag.com](https://ngelag.com/apa-itu-rom-pada-smartphone-android/)*, 2022).

Pada dasarnya, menggunakan Custom ROM dapat mempercepat dan meringankan penggunaan smartphone, karena terhindar dari adanya bloatware atau service-service yang ditanampaksakan oleh Manufaktur. 

### Kernel

Kernel adalah modul utama dari sistem operasi sebuah hardware. Tugas utamanya adalah untuk memberi layanan kepada aplikasi dan bagian lain dari OS, lalu memuatnya di dalam memori utama.([glints.com](https://glints.com/id/lowongan/kernel-adalah/), Diakses pada 27 November 2022).

Kernel android itu didasarkan pada branches of linux kernel LTS (*long-term support*). Mulai pada tahun 2021, Android menggunakan versi 4.14, 4.19 or 5.4 dari kernel linux. Kernel yang sebenarnya itu bergantung pada perangkat individualnya ([wikipedia.com](https://en.wikipedia.org/wiki/Android_(operating_system)), Diakses pada 27 November 2022).

Google Engineer, yaitu Patrick Brady pernah menyatakan dalam konferensi pengembang perusahaan bahwa "Android bukanlah Linux". Dengan *Computerworld* menambahkan bahwa "Izinkan saya membuatnya sederhana untukmu, Tanpa Linux di sana tidak ada android". Ars Technica pernah menulis bahwa "Meskipun Android itu dibuat diatasnya kernel Linux, platform ini memiliki sedikit kesamaan dengan tumpukan desktop linux yang konvensional" ([wikipedia.com](https://en.wikipedia.org/wiki/Android_(operating_system)), Diakses pada 27 November 2022).

Android adalah distribusi Linux yang bergantung pada Linux Foundation, dan Android tidak menggunakan [GNU C Library](https://www.gnu.org/software/libc/) tetapi ia menggunakan [Bionic](https://en.wikipedia.org/wiki/Bionic_(software)), yaitu *glibc* yang dibangun oleh Google sendiri. Android bukanlah sistem operasi tradisional *unix-like* pada umumnya dan tidak menggunakan komponen yang secara tipikal ditemukan di distribusi Linux lainnya ([wikipedia.com](https://en.wikipedia.org/wiki/Android_(operating_system)), Diakses pada 27 November 2022).

* ### custom Kernel made by community
  * <https://www.xda-developers.com/tag/kernel/>
  * <https://www.xda-developers.com/tag/kernel-source/>

--

Dengan *Custom kernel* sendiri, merujuk pada member [forum.xda-developers.com](https://forum.xda-developers.com/t/what-is-a-custom-kernel.2278000/) mengatakan bahwa *custom kernel* itu bukanlah apa-apa tanpa *stock kernel* yang termodifikasi. Diselesaikannya, untuk mencapai fitur-fitur yang mana tidak ditampakkan dalam kernel yang disediakan oleh manufaktur. Anda bisa menemukannya untuk perangkat anda jika seseorang sudah membuat sebuah *custom kernel* untuk "perangkat". Kernel juga perangkat yang spesifik. Dengan lanjutannya, menjaga cadangan dari *stock kernel* itu sangat direkomendasikan. Kernel juga dipanggil sebagai `boot.img` dan itu seharusnya sangat dimungkinkan untuk mengekstraksi dari perangkat menggunakan `adb`.

Jadi, dengan kata lain, membuat cadangan dari *stock kernel* dan *stock rom* itu sangat direkomendasikan, dan hasil studi ini sudah terverifikasi secara empiris dengan pengalaman saya yang diajarkan oleh teman saya, [Mr Clay](https://t.me/mrxclayed). Bisa dilihat hasil ucapannya dalam screenshot dibawah.

![image](https://i.postimg.cc/5NwPwzJ3/image.png)

Saya sebagai orang yang belum paham tentang komponen android, sangat berterima kasih yang sebesar-besarnya dari dia. Dengan adanya keberadaan dia ini, saya antusias untuk riset jauh mengenai sistem operasi Android ini, yang padahal aslinya cuma *distribusi Linux*, hadeh.

--

### Rooting
Sama seperti Linux, sistem operasi Android ini menjalankan akses terhadap administrasi sistem dengan istilah *root* kalau di Windows dinamakan *Administrator* sebagai pengendali utama dari berjalannya komponen, seperti kernel, firmware, dan sebagainya.

Untuk membuka *root access*, Android tidak semudah Linux yang benar-benar tinggal menjalankan perintah `sudo`. Android menggunakan `bootloader` untuk mengunci semua partisi yang sensitif dan semua komponen yang mempunyai status `read-only`. Untuk membuka `bootloader`, sebagai developer harus menyalakan opsi `OEM Unlocking` yang ada pada model perangkat. Proses pembukaan ini, harus me-reset seluruh sistem dengan *factory reset*, dan itu menghapus seluruh *user data*. 

Ada juga cara lain untuk membuka akses `root` pada Android, yaitu dengan mengeksploitasi *security flaws* yang mana ini digunakan oleh *open-source community* untuk menambah kapabilitas dan penyesuaian atau kustomisasi pada perangkat mereka, tapi juga menggunakan *malicious parties* atau pihak jahat untuk menginstall virus dan malware.

### Recovery App 
Merujuk pada pengertian [**Diablo67** di forum XDA-Developer](https://forum.xda-developers.com/t/lotk-android-terms-slang-definitions-laiman-terms-android-guides-updated-07-26-13.1466228/),

> The recovery partition is a boot-mode for your phone that allows you to wipe your settings from the Data partition of the phone (a hard wipe), or perform an update using an update.zip file on the root of the microSD card. It is common (although not necessary) to flash a patched Recovery image, such as TWRP or ClockworkMod Recovery. This allows you to run Nandroid backup from the device, and flash modifications, such as files to the device, essentially becoming a means to install software to the device. Recovery mode is separate from ‘normal’ mode, and can be entered by holding down home whilst turning the phone on.

Partisi recovery itu adalah `boot-mode` untuk smartphone yang mengizinkan untuk membersihkan pengaturan-pengaturan dari partisi data smartphone (*a hard wipe*), atau melaksanakan pembaruan menggunakan file `update.zip` pada rootnya kartu microSD. Secara umum, untuk *ngeflash* suatu *recovery image* yang *patched*, seperti [TWRP](https://twrp.me/), atau [ClockworkMod Recovery](https://www.clockworkmod.com/). Dan ini akan mengizinkan pengguna untuk menjalankan suatu modifikasi-modifikasi untuk diinstalasikan kepada perangkat. `Recovery Mode` itu terpisahkan dari `normal mode`, dan bisa dinyalakan dengan menekan turun tombol `home` dan sedangkan tombol power. Kalau sekarang, menggunakan `volume down` + `power`.

### Launcher
> Collectively, the part of the Android user interface on home screens that lets you launch apps, make phone calls, etc. Is built in to Android, or can be purchased in the Android Market. ([forum.xda-developers.com](https://forum.xda-developers.com/t/lotk-android-terms-slang-definitions-laiman-terms-android-guides-updated-07-26-13.1466228/), 2012 oleh **Diablo67**).

Launcher secara definisi **Diablo67** merupakan bagian dari interface android pada *homescreen* yang mengizinkan anda untuk menjalankan *app*, membuat panggilan telepon, dan lain lain. Ini dibuat dalam android dan bisa dibeli dalam *Android Market*.

Launcher secara definisi saya merupakan *homescreen app drawer* atau pengubah tampilan hingga fungsionalitas dari stock launcher itu sendiri. Biasanya para manufaktur atau developer dari ROM mengembangkan *Launcher* mereka dengan kapabilitas sampai ke fungsionalitas tertentu untuk tujuan tertentu juga. Mungkin untuk *gaming* bisa dikembangkan akselerasi kecepatan RAM (maybe), atau percepatan pada performa.

Ya seperti biasa yang sudah dijelaskan mengenai pembagiannya itu ada

* Stock Launcher
* Custom Launcher

Yang secara definitif kurang lebih sama seperti makna-makna yang sudah dijelaskan di atas.

Di sini launcher sangat tidak dibutuhkan sekali, karena biasanya para *Android ricer* ini orang-orangnya adalah gamer semua jadi untuk perihal kustomisasi sampai ke Launcher sangat jarang orang-orangnya, tidak seperti di *Linux Ricing* yang mana orang-orangnya sangat peduli tinggi terhadap *interface* mereka (dalam pandangan saya selama terjun di grup mereka).

<details>
<img width="300px" src="https://i.postimg.cc/SQPxJ8X1/image.png" style="border-radius: 5%;">
<p>Saya sendiri, dari Januari 2022. Menggunakan Launchernya POCO yang dikembangkan oleh Xiomi</p>
</details>




## My System Information

<style>
table,th,td {
    float: top;
    border: 1px solid black;
    margin-bottom: 20px;
  }
img {
    box-shadow: 10px 10px 8px #aaa;
  }
</style>

Name | Information |
-----|--------------|
Device|Infinix Hot 8 X650C|
GSI Type|phhGSI v316 CrDroid|
Android Version|[11](https://www.android.com/intl/en_ca/android-11/)|
OS|[CrDroid](https://crdroid.net/) v7.16|
Kernel Version|4.9.117+|

<details>
<img width="300px" src="https://i.postimg.cc/fyLMqWVb/image.png" style="border-radius: 5%;">
<img width="300px" src="https://i.postimg.cc/rpjzwwt8/image.png" style="border-radius: 5%;">
<img width="300px" src="https://i.postimg.cc/4xqm0xhy/image.png" style="border-radius: 5%;">
<img width="300px" src="https://i.postimg.cc/tJJbyFKj/image.png" style="border-radius: 5%;">
</details>

## Penutup
Android merupakan sistem operasi yang diakui secara konvensional, tetapi secara spesifik Android merupakan turunan Linux dan tidak terpisahkan dari Linux walaupun mesin utama yang didasarkan adalah JVM (Java Virtual Machine). Toh, tetap saja masih turunan Linux mau sesombong apapun, secara historis tidak bisa dipisahkan faktanya bahwa ia masih saudara dengan kernel Linux. Banggalah wahai Linux user!

Saya juga di sini masih belajar bagaimana Android bekerja secara stabil tanpa ada banyak kendala, mempelajari setiap komponen dan developmentalnya. Saya turut senang sekali mendapat ilmu tentang sistem operasi ini.

Ya, seperti itulah.

## Daftar Pustaka
*  Hildenbrand, Jerry (January 23, 2012). "What is a kernel?". Android Central. Archived from the original on May 27, 2017. Retrieved June 20, 2017.
