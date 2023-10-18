---
title: "Cara Menghubungkan Instance MySQL di Alibaba Cloud ECS ke Komputer Lokal"
date: 2023-10-19T01:01:13+07:00
# weight: 1
# aliases: ["/first"]
categories: "hello"
tags: ["ECS", "Alibaba Cloud"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
# description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

## Siapkan MySQL Database pada ECS Instance
Untuk requirement berikut, bisa ikuti Hands-on Labs pada web alibabacloud.com dengan link berikut 
[Deploy MySQL Database in 10 Minutes](https://intl.aliyun.com/lab/dl-1686306541559?spm=a3c0i.28268686.8054513620.8.e4b679aeVHffpH)

## Cek ECS Security Group Rules
Salah satu fungsi bagian security ini mengatur IP address / komputer mana yang bisa mengakses instance ECS. Untuk MySQL port yang biasanya digunakan adalah 3306. Jadi harus menambahkan inbound rule yang membolehkan IP address komputer mengakses port MySQL. Berikut langkah-langkahnya

### Masuk ke Alibaba Cloud Console:
Buka web browser, Buka Alibaba Cloud Console (https://home.console.aliyun.com/).

### Akses Dashboard ECS:
Setelah berhasil masuk, pergilah ke dashboard Elastic Compute Service (ECS).

### Pilih Instance ECS:
Di dashboard ECS, pilih instance ECS yang menjalankan DB MySQL (tutorial hands-on lab sebelumnya).

### Lihat bagian Security Groups:
Pada halaman detail instance ECS, scroll ke bawah hingga bagian "Security Groups" Di bawah bagian ini, Akan ada subbagian yang disebut "Seucurity Group." Seharusnya ada satu atau lebih security group yang terkait dengan instance ECS Anda.

### Edit Aturan Security Group:
Klik pada kelompok keamanan yang terkait dengan instance ECS. Seharusnya ini akan mengarahkan ke halaman detail security group.

### Tambahkan Aturan Masuk (Inbound Rule):
Di halaman detail kelompok keamanan, navigasi ke tab Inbound. Lalu klik tombol "Add Rule" untuk menambahkan aturan baru yang mengizinkan lalu lintas masuk ke port MySQL (biasanya 3306).

### Buat Aturan Masuk (Inbound Rule):
Untuk membuat aturan masuk, klik tombol "Tambahkan Aturan Masuk" atau "Otorisasi Aturan Kelompok Keamanan." Anda perlu mengonfigurasi aturan dengan detail berikut:

    Protokol: Pilih "MySQL/AliDB" atau "TCP Kustom."
    Rentang Port: Tetapkan ke "3306" untuk MySQL.
    Alamat IP Sumber: Masukkan alamat IP publik laptop Anda. Jika alamat IP Anda sering berubah, Anda dapat menggunakan "0.0.0.0/0" untuk mengizinkan lalu lintas dari sumber apa pun, tetapi ini kurang aman.

### Simpan dan Terapkan Aturan:
Setelah mengonfigurasi aturan masuk, klik tombol "OK" atau "Otorisasi" untuk menyimpan perubahan dan menerapkan aturan baru.


Ini akan mengizinkan lalu lintas masuk ke port 3306 ke instans ECS Anda dari alamat IP sumber yang ditentukan, yaitu laptop Anda. Setelah aturan ini diterapkan, Anda seharusnya dapat terhubung ke basis data MySQL pada instans ECS Anda menggunakan MySQL Workbench dari laptop Anda.

Harap dicatat bahwa umumnya lebih baik untuk keamanan jika Anda menentukan alamat IP aktual laptop Anda daripada menggunakan "0.0.0.0/0" untuk membatasi akses hanya ke laptop Anda. Selain itu, pastikan kelompok keamanan dan aturan firewall Anda aman, dan pertimbangkan untuk menggunakan SSH tunnel untuk keamanan tambahan saat mengakses basis data Anda secara remote.


## Error access denied saat akan connect
Saat mencoba connect instance MySQL di ECS dengan mariadb cli pada laptop dengan command berikut
``` bash
mysql -h 172.17.68.190 -u moanrisy -p
```
Akan muncul
Error: RROR 1045 (28000): Access denied for user 'moanrisy'@'117.103.71.94' (using password: YES)

Penyebab error ini ada beberapa seperti salah username password, IP address public instance MySQL yang salah dll.
Tapi pada kasus saya, errornya ada pada hostname laptop 'moanrisy' yang tidak punya akses.


Solusinya buka console, klik All operations (pojok kanan atas), lalu workbench.
![workbench](/posts/connect-mysql-ecs/images/workbench.png)

Ini akan meredirect ke terminal ECS.

Lalu login ke mysql dengan user root, beri akses ke database yang dipilih dan passwordnya.
``` bash
mysql -uroot -p;
GRANT ALL PRIVILEGES ON your_database.* TO 'moanrisy'@'%' IDENTIFIED BY 'your_password';
FLUSH PRIVILEGES;
```

note*
This command grants all privileges on 'your_database' to the user 'moanrisy' from any host ('%'). Please replace 'your_database' and 'your_password' with the actual database name and password you intend to use.

Keep in mind that allowing connections from any IP address with '%' can be a security risk, especially if your MySQL server is accessible from the public internet. Make sure to secure your MySQL server and database by using strong passwords and consider restricting access to specific trusted IP addresses if possible.

## Cara mengetahui IP public / private instance MySQL ECS
Pada workbench, keluar dari mysql dengan exit. Lalu exit sekali lagi untuk keluar terminal workbench. Lalu muncul popup yang salah satu tombolnya 'Back to logon'.

![instance-login.png](/posts/connect-mysql-ecs/images/instance-login.png)

## Demo workbench ECS DB MySQL sync dengan local cli app mariadb (Click to play)
[![demo](/posts/connect-mysql-ecs/images/thumbnail-play.png)](/posts/connect-mysql-ecs/images/demo-connect-mysql-ecs.mp4)
