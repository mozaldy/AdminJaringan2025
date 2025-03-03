<p align="center">
  <img src="../Logo_PENS.png" alt="Logo">
</p>

<h1 align="center">Admin Jaringan 2025</h1>

```
Dosen Pengampu: Dr Ferry Astika Saputra ST, M.Sc
Nama: Mohammad Rizaldy Ramadhan
NRP: 3123600011
Kelas: D4 IT A
```

# Tugas 2 - **Rangkuman Process Control**

# Bab 4: Kontrol Proses

![Kontrol Proses](https://www.sifars.com/blog/wp-content/uploads/2019/10/Procs-1024x577-1.png)

## Komponen Proses

Sebuah proses terdiri dari ruang alamat dan sekumpulan struktur data dalam kernel. Ruang alamat adalah sekumpulan halaman memori yang ditandai oleh kernel untuk digunakan oleh proses. (Halaman adalah unit pengelolaan memori, biasanya berukuran 4KiB atau 8KiB.) Halaman ini digunakan untuk menyimpan kode, data, dan stack proses. Struktur data dalam kernel melacak status proses, prioritasnya, parameter penjadwalannya, dan sebagainya.

Bayangkan proses sebagai wadah untuk sekumpulan sumber daya yang dikelola kernel atas nama program yang sedang berjalan. Sumber daya ini termasuk halaman memori yang menyimpan kode dan data program, deskriptor file yang merujuk ke file yang terbuka, dan berbagai atribut yang menggambarkan status proses.

Struktur data internal kernel mencatat berbagai informasi tentang setiap proses:

- Peta ruang alamat proses
- Status saat ini dari proses (berjalan, tidur, dan sebagainya)
- Prioritas proses
- Informasi tentang sumber daya yang digunakan proses (CPU, memori, dan sebagainya)
- Informasi tentang file dan port jaringan yang dibuka oleh proses
- Mask sinyal proses (kumpulan sinyal yang saat ini diblokir)
- Pemilik proses (ID pengguna yang memulai proses)

"Thread" adalah konteks eksekusi dalam sebuah proses. Sebuah proses dapat memiliki banyak thread, yang semuanya berbagi ruang alamat dan sumber daya lainnya. Thread digunakan untuk mencapai paralelisme dalam sebuah proses. Thread juga dikenal sebagai proses ringan karena lebih murah untuk dibuat dan dihancurkan dibandingkan proses.

Sebagai contoh untuk memahami konsep proses dan thread, pertimbangkan server web. Server web mendengarkan koneksi masuk dan kemudian membuat thread baru untuk menangani setiap permintaan masuk. Setiap thread menangani satu permintaan pada satu waktu, tetapi server web secara keseluruhan dapat menangani banyak permintaan secara bersamaan karena memiliki banyak thread. Di sini, server web adalah sebuah proses, dan setiap thread adalah konteks eksekusi terpisah dalam proses tersebut.

### PID: Nomor ID Proses

Setiap proses diidentifikasi oleh nomor ID proses unik, atau PID. PID adalah bilangan bulat yang diberikan kernel kepada setiap proses saat proses tersebut dibuat. PID digunakan untuk merujuk ke proses dalam berbagai panggilan sistem, misalnya, untuk mengirim sinyal ke proses.

Konsep "namespaces" memungkinkan proses yang berbeda memiliki PID yang sama. Namespaces digunakan untuk membuat kontainer, yang merupakan lingkungan terisolasi yang memiliki pandangan sendiri tentang sistem. Kontainer digunakan untuk menjalankan beberapa instance aplikasi pada sistem yang sama, masing-masing dalam lingkungan terisolasi sendiri.

### PPID: Nomor ID Proses Induk

Setiap proses juga dikaitkan dengan proses induk, yaitu proses yang membuatnya. Nomor ID proses induk, atau PPID, adalah PID dari proses induk. PPID digunakan untuk merujuk ke proses induk dalam berbagai panggilan sistem, misalnya, untuk mengirim sinyal ke proses induk.

### UID dan EUID: ID Pengguna dan ID Pengguna Efektif

ID pengguna, atau UID, adalah ID pengguna yang memulai proses. ID pengguna efektif, atau EUID, adalah ID pengguna yang digunakan proses untuk menentukan sumber daya apa yang dapat diakses oleh proses. EUID digunakan untuk mengontrol akses ke file, port jaringan, dan sumber daya lainnya.

## Siklus Hidup Proses

Untuk membuat proses baru, sebuah proses menyalin dirinya sendiri dengan panggilan sistem **fork**. **fork** membuat salinan dari proses asli, dan salinan tersebut sebagian besar identik dengan proses induk. Proses baru memiliki **PID** yang berbeda dan memiliki informasi akuntansi sendiri. (Secara teknis, sistem Linux menggunakan **clone**, superset dari **fork** yang menangani thread dan mencakup fitur tambahan. **fork** tetap ada di kernel untuk kompatibilitas ke belakang tetapi memanggil **clone** secara internal.)

Saat sistem boot, kernel secara mandiri membuat dan menginstal beberapa proses. Yang paling terkenal adalah **init** atau **systemd**, yang selalu menjadi proses nomor 1. Proses ini menjalankan skrip startup sistem, meskipun cara tepatnya hal ini dilakukan sedikit berbeda antara UNIX dan Linux. Semua proses selain yang dibuat oleh kernel adalah keturunan dari proses primordial ini.

### Sinyal

Sinyal adalah cara untuk mengirim notifikasi ke sebuah proses. Mereka digunakan untuk memberi tahu proses bahwa suatu peristiwa tertentu telah terjadi.

Ada sekitar tiga puluh jenis sinyal yang didefinisikan, dan mereka digunakan dalam berbagai cara:

- Mereka dapat dikirim antar proses sebagai sarana komunikasi.
- Mereka dapat dikirim oleh driver terminal untuk membunuh, menginterupsi, atau menangguhkan proses saat tombol seperti <Control-C> dan <Control-Z> ditekan.
- Mereka dapat dikirim oleh administrator (dengan kill) untuk mencapai berbagai tujuan.
- Mereka dapat dikirim oleh kernel ketika sebuah proses melakukan pelanggaran seperti pembagian dengan nol.
- Mereka dapat dikirim oleh kernel untuk memberi tahu proses tentang kondisi "menarik" seperti kematian proses anak atau ketersediaan data pada saluran I/O.

![Sinyal](https://liujunming.top/images/2018/12/71.png)

Sinyal KILL, INT, TERM, HUP, dan QUIT semuanya terdengar seperti memiliki arti yang kurang lebih sama, tetapi penggunaannya sebenarnya cukup berbeda.

- **KILL** tidak dapat diblokir dan mengakhiri proses di tingkat kernel. Sebuah proses tidak pernah benar-benar menerima atau menangani sinyal ini.
- **INT** dikirim oleh driver terminal ketika pengguna mengetik <Control-C>. Ini adalah permintaan untuk menghentikan operasi saat ini. Program sederhana harus keluar (jika mereka menangkap sinyal) atau membiarkan diri mereka dibunuh, yang merupakan default jika sinyal tidak ditangkap. Program yang memiliki baris perintah interaktif (seperti shell) harus berhenti melakukan apa yang mereka lakukan, membersihkan, dan menunggu input pengguna lagi.
- **TERM** adalah permintaan untuk menghentikan eksekusi sepenuhnya. Diharapkan bahwa proses yang menerima akan membersihkan statusnya dan keluar.
- **HUP** dikirim ke proses ketika terminal kontrol ditutup. Awalnya digunakan untuk menunjukkan "hang up" dari koneksi telepon, sekarang sering digunakan untuk menginstruksikan proses daemon untuk mengakhiri dan memulai ulang, seringkali untuk mempertimbangkan konfigurasi baru. Perilaku tepatnya tergantung pada proses spesifik yang menerima sinyal HUP.
- **QUIT** mirip dengan TERM, kecuali bahwa defaultnya adalah menghasilkan core dump jika tidak ditangkap. Beberapa program mengkonsumsi sinyal ini dan menafsirkannya sebagai sesuatu yang lain.

### kill: mengirim sinyal

Seperti namanya, perintah **kill** paling sering digunakan untuk mengakhiri sebuah proses. **kill** dapat mengirim sinyal apa pun, tetapi secara default mengirimkan TERM. **kill** dapat digunakan oleh pengguna biasa pada proses mereka sendiri atau oleh root pada proses apa pun. Sintaksnya adalah:

```bash
kill [-signal] pid
```

di mana *signal* adalah nomor atau nama simbolik dari sinyal yang akan dikirim dan *pid* adalah nomor identifikasi proses dari proses target.

**kill** tanpa nomor sinyal tidak menjamin bahwa proses akan mati, karena sinyal TERM dapat ditangkap, diblokir, atau diabaikan. Perintah **kill -9 pid** dijamin akan membunuh proses karena sinyal KILL tidak dapat ditangkap, diblokir, atau diabaikan.

**killall** membunuh proses berdasarkan nama daripada ID proses. Ini tidak tersedia di semua sistem.
Contoh:

```bash
killall firefox
```

Perintah **pkill** mirip dengan **killall** tetapi menyediakan lebih banyak opsi.

Contoh:

```bash
pkill -u abdoufermat # membunuh semua proses yang dimiliki oleh pengguna abdoufermat
```

## PS: Memantau Proses

Perintah **ps** adalah alat utama administrator sistem untuk memantau proses. Meskipun versi **ps** berbeda dalam argumen dan tampilannya, mereka semua memberikan informasi yang pada dasarnya sama.

**ps** dapat menunjukkan PID, UID, prioritas, dan terminal kontrol dari proses. Ini juga memberi tahu Anda berapa banyak memori yang digunakan proses, berapa banyak waktu CPU yang telah dikonsumsi, dan apa statusnya saat ini (berjalan, berhenti, tidur, dan sebagainya).

Anda dapat mendapatkan gambaran umum yang berguna tentang sistem dengan menjalankan **ps aux**. Opsi **a** memberi tahu **ps** untuk menunjukkan proses semua pengguna, dan opsi **u** memberi tahu untuk memberikan informasi rinci tentang setiap proses. Opsi **x** memberi tahu **ps** untuk menunjukkan proses yang tidak terkait dengan terminal.

```bash
$ ps aux | head -8
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  22556  2584 ?        Ss   2019   0:02 /sbin/init
root         2  0.0  0.0      0     0 ?        S    2019   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        I<   2019   0:00 [rcu_gp]
root         4  0.0  0.0      0     0 ?        I<   2019   0:00 [rcu_par_gp]
root         6  0.0  0.0      0     0 ?        I<   2019   0:00 [kworker/0:0H-kblockd]
root         8  0.0  0.0      0     0 ?        I<   2019   0:00 [mm_percpu_wq]
root         9  0.0  0.0      0     0 ?        S    2019   0:00 [ksoftirqd/0]
```

![penjelasan-proses](./data/process-explanation.png)

Set argumen berguna lainnya adalah **lax**, yang memberikan informasi lebih teknis tentang proses. **lax** sedikit lebih cepat daripada **aux** karena tidak perlu menyelesaikan nama pengguna dan grup.

```bash
$ ps lax | head -8
F   UID   PID  PPID PRI  NI    VSZ   RSS WCHAN  STAT TTY        TIME COMMAND
4     0     1     0  20   0  22556  2584 -      Ss   ?          0:02 /sbin/init
1     0     2     0  20   0      0     0 -      S    ?          0:00 [kthreadd]
1     0     3     2  20   0      0     0 -      I<   ?          0:00 [rcu_gp]
1     0     4     2  20   0      0     0 -      I<   ?          0:00 [rcu_par_gp]
1     0     6     2  20   0      0     0 -      I<   ?          0:00 [kworker/0:0H-kblockd]
1     0     8     2  20   0      0     0 -      I<   ?          0:00 [mm_percpu_wq]
1     0     9     2  20   0      0     0 -      S    ?          0:00 [ksoftirqd/0]
```

Untuk mencari proses tertentu, Anda dapat menggunakan **grep** untuk menyaring output dari **ps**.

```bash
$ ps aux | grep -v grep | grep firefox
```

Kita dapat menentukan PID dari sebuah proses dengan menggunakan **pgrep**.

```bash
$ pgrep firefox
```

atau **pidof**.

```bash
$ pidof /usr/bin/firefox
```

## Pemantauan Interaktif dengan top

Perintah **top** memberikan tampilan real-time dinamis dari sistem yang sedang berjalan. Ini dapat menampilkan informasi ringkasan sistem serta daftar proses atau thread yang saat ini dikelola oleh kernel Linux. Jenis informasi ringkasan sistem yang ditampilkan dan jenis, urutan, dan ukuran informasi yang ditampilkan untuk proses semuanya dapat dikonfigurasi oleh pengguna dan konfigurasi tersebut dapat dibuat persisten di seluruh restart.

Secara default, tampilan diperbarui setiap 1-2 detik, tergantung pada sistem.

Ada juga perintah **htop**, yang merupakan penampil proses interaktif untuk sistem Unix. Ini adalah aplikasi mode teks (untuk konsol atau terminal X) dan memerlukan ncurses. Ini mirip dengan top, tetapi memungkinkan Anda untuk menggulir secara vertikal dan horizontal, sehingga Anda dapat melihat semua proses yang berjalan di sistem, bersama dengan baris perintah lengkapnya. **htop** juga memiliki antarmuka pengguna yang lebih baik dan lebih banyak opsi untuk operasi.

## Nice dan renice: mengubah prioritas proses

**Niceness** adalah petunjuk numerik ke kernel tentang bagaimana proses harus diperlakukan dalam hubungannya dengan proses lain yang bersaing untuk CPU.

Niceness yang tinggi berarti prioritas rendah untuk proses Anda: Anda akan bersikap baik. Nilai rendah atau negatif berarti prioritas tinggi: Anda tidak terlalu baik!

Rentang nilai niceness yang diizinkan bervariasi di antara sistem. Di Linux, rentangnya adalah -20 hingga +19, dan di FreeBSD adalah -20 hingga +20.

Proses prioritas rendah adalah proses yang tidak terlalu penting. Ini akan mendapatkan waktu CPU lebih sedikit daripada proses prioritas tinggi. Proses prioritas tinggi adalah proses yang penting dan harus diberikan lebih banyak waktu CPU daripada proses prioritas rendah.

Misalnya, jika Anda menjalankan pekerjaan intensif CPU yang ingin Anda jalankan di latar belakang, Anda dapat memulainya dengan nilai niceness yang tinggi. Ini akan memungkinkan proses lain untuk berjalan tanpa diperlambat oleh pekerjaan Anda.

Perintah **nice** digunakan untuk memulai proses dengan nilai niceness tertentu. Sintaksnya adalah:

```bash
nice -n nice_val [command]

# Contoh
nice -n 10 sh infinite.sh &
```

Perintah **renice** digunakan untuk mengubah nilai niceness dari proses yang sedang berjalan. Sintaksnya adalah:

```bash
renice -n nice_val -p pid

# Contoh
renice -n 10 -p 1234
```

**Nilai prioritas** adalah prioritas aktual proses yang digunakan oleh kernel Linux untuk menjadwalkan tugas.
Dalam sistem Linux, prioritas adalah 0 hingga 139 di mana 0 hingga 99 untuk real-time dan 100 hingga 139 untuk pengguna.

Hubungan antara nilai nice dan prioritas adalah sebagai berikut:

> nilai_prioritas = 20 + nilai_nice

Nilai nice default adalah 0. Semakin rendah nilai nice, semakin tinggi prioritas proses.

## Sistem Berkas /proc

Versi Linux dari **ps** dan **top** membaca informasi status proses mereka dari direktori **/proc**, sebuah sistem berkas semu di mana kernel mengekspos berbagai informasi menarik tentang status sistem.

Terlepas dari namanya, **/proc** berisi informasi lain selain proses (statistik yang dihasilkan oleh sistem, dll).

Proses direpresentasikan oleh direktori di **/proc**, dan setiap proses memiliki direktori yang dinamai sesuai dengan PID-nya. Direktori **/proc** berisi berbagai file yang memberikan informasi tentang proses, seperti baris perintah, variabel lingkungan, deskriptor file, dan sebagainya.

![informasi-proses](./data/process-information.png)

## Strace dan truss

Untuk mengetahui apa yang dilakukan sebuah proses, Anda dapat menggunakan **strace** di Linux atau **truss** di FreeBSD. Perintah ini melacak panggilan sistem dan sinyal. Mereka dapat digunakan untuk men-debug program atau untuk memahami apa yang dilakukan program.

Misalnya, log berikut dihasilkan oleh strace yang dijalankan terhadap salinan aktif dari top (yang berjalan sebagai PID 5810):

```bash
$ strace -p 5810

gettimeofday({1197646605,  123456}, {300, 0}) = 0
open("/proc", O_RDONLY|O_NONBLOCK|O_LARGEFILE|O_DIRECTORY) = 7
fstat64(7, {st_mode=S_IFDIR|0555, st_size=0, ...}) = 0
fcntl64(7, F_SETFD, FD_CLOEXEC)          = 0
getdents64(7, /* 3 entries */, 32768)   = 72
getdents64(7, /* 0 entries */, 32768)   = 0
stat64("/proc/1", {st_mode=S_IFDIR|0555, st_size=0, ...}) = 0
open("/proc/1/stat", O_RDONLY)           = 8
read(8, "1 (init) S 0 1 1 0 -1 4202752"..., 1023) = 168
close(8)                                = 0

[...]
```

**top** memulai dengan memeriksa waktu saat ini. Kemudian membuka dan memeriksa direktori **/proc**, dan membaca file **/proc/1/stat** untuk mendapatkan informasi tentang proses **init**.

## Proses yang Tidak Terkendali

Terkadang sebuah proses berhenti merespons sistem dan berjalan liar. Proses ini mengabaikan prioritas penjadwalannya dan bersikeras mengambil 100% CPU. Karena proses lain hanya dapat mendapatkan akses terbatas ke CPU, mesin mulai berjalan sangat lambat. Ini disebut proses yang tidak terkendali.

Perintah **kill** dapat digunakan untuk mengakhiri proses yang tidak terkendali. Jika proses tidak merespons sinyal TERM, Anda dapat menggunakan sinyal KILL untuk mengakhirinya.

```bash
kill -9 pid

atau

kill -KILL pid
```

Kita dapat menyelid
