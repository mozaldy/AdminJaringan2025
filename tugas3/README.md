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

# Tugas 3 - **Rangkuman Filesystem**

# Bab 5: Sistem Berkas

![ikon-sistem-berkas](https://miro.medium.com/v2/resize:fit:752/1*quw0WvsLLCxad3WC6fjQ1Q.png)

Tujuan utama sistem berkas adalah untuk mengatur dan merepresentasikan sumber daya penyimpanan sistem.

Sistem berkas terdiri dari empat komponen utama:

1. **Ruang nama (namespace)** - Cara untuk memberi nama dan mengatur berkas dalam hierarki.
2. **API** - Sekumpulan system call untuk menavigasi dan memanipulasi objek.
3. **Model keamanan** - Skema untuk melindungi, menyembunyikan, dan berbagi objek.
4. **Implementasi** - Perangkat lunak yang menghubungkan model logis dengan perangkat keras.

Sistem berkas berbasis disk yang populer termasuk ext4, XFS, UFS, ZFS, dan Btrfs. Ada juga sistem berkas asing seperti FAT dan NTFS (digunakan oleh Windows) serta ISO 9660 (digunakan oleh CD dan DVD).

Sistem berkas modern bertujuan untuk meningkatkan kecepatan, keandalan, dan menambahkan fitur tambahan di atas fungsionalitas standar.

## Nama Path

Istilah "folder" adalah kebocoran linguistik dari dunia Windows dan macOS. Istilah yang lebih teknis adalah **direktori**. Hindari menggunakan istilah folder dalam konteks teknis.

Daftar direktori yang mengarah ke sebuah berkas disebut **pathname** (nama path). Pathname adalah string yang menunjukkan lokasi berkas dalam hierarki sistem berkas. Pathname dapat berupa **absolut** (misalnya `/home/username/file.txt`) atau **relatif** (misalnya `./file.txt`).

## Pemasangan dan Pelepasan Sistem Berkas (Mounting dan Unmounting)

Sistem berkas terdiri dari beberapa bagian kecil yang disebut "sistem berkas", masing-masing terdiri dari satu direktori dan subdirektori serta berkas-berkasnya. Istilah **pohon berkas** merujuk pada tata letak keseluruhan, sedangkan **sistem berkas** merujuk pada cabang-cabang yang terpasang pada pohon.

Sistem berkas biasanya dipasang ke pohon dengan perintah `mount`. Perintah ini memetakan direktori dalam pohon berkas yang ada, disebut **titik pemasangan** (mount point), ke root dari sistem berkas baru.

Contoh:

```bash
# Memasang sistem berkas di /dev/sda4 ke /users
mount /dev/sda4 /users
```

Linux memiliki opsi pelepasan malas (**umount -l**) yang menghapus sistem berkas dari hierarki penamaan tetapi tidak benar-benar melepaskannya sampai sistem berkas tersebut tidak lagi digunakan.

**umount -f** adalah pelepasan paksa, yang berguna ketika sistem berkas sedang sibuk.

Alih-alih menggunakan **umount -f**, Anda dapat menggunakan **lsof** atau **fuser** untuk mengetahui proses mana yang menggunakan sistem berkas dan kemudian mematikannya.

Contoh:

```bash
# Mencari tahu proses mana yang menggunakan sistem berkas
lsof /home/abdou
```

## Organisasi Pohon Berkas

Sistem UNIX tidak pernah terorganisir dengan baik! Berbagai konvensi penamaan yang tidak kompatibel digunakan secara bersamaan, dan berbagai jenis berkas tersebar secara acak di seluruh ruang nama. **Inilah mengapa sulit untuk meningkatkan sistem operasi**.

Sistem berkas root mencakup setidaknya direktori root dan sekumpulan berkas dan subdirektori minimal. Berkas yang berisi kernel OS biasanya berada di bawah **/boot**, tetapi nama dan lokasinya dapat bervariasi. Di BSD dan beberapa sistem UNIX lainnya, kernel bukan satu berkas melainkan sekumpulan komponen.

**/etc** berisi berkas sistem dan konfigurasi penting. **/sbin** dan **/bin** untuk utilitas penting, dan terkadang **/tmp** untuk berkas sementara. **/dev** secara tradisional merupakan bagian dari sistem berkas root, tetapi saat ini merupakan sistem berkas virtual yang dipasang secara terpisah.

Beberapa sistem menyimpan berkas pustaka bersama dan beberapa hal aneh lainnya, seperti preprosesor C, di direktori **/lib** atau **/lib64**. Yang lain telah memindahkan item-item ini ke **/usr/lib**, terkadang meninggalkan **/lib** sebagai tautan simbolis.

**/usr** dan **/var** juga sangat penting. **/usr** adalah tempat di mana sebagian besar program standar-tetapi-tidak-kritis-sistem disimpan, bersama dengan berbagai barang rampasan lainnya seperti manual daring dan sebagian besar pustaka. FreeBSD menyimpan cukup banyak konfigurasi lokal di bawah **/usr/local**. **/var** berisi direktori spool, berkas log, informasi akuntansi, dan berbagai item lain yang tumbuh atau berubah dengan cepat dan bervariasi pada setiap host. Baik **/usr** dan **/var** harus tersedia untuk memungkinkan sistem benar-benar naik ke mode multi-pengguna.

![pathnames](./data/pathnames.png)

## Jenis Berkas

Sebagian besar implementasi sistem berkas mendefinisikan tujuh jenis berkas:

1. **Berkas biasa (regular files)**
2. **Direktori (directories)**
3. **Berkas perangkat karakter (character device files)**
4. **Berkas perangkat blok (block device files)**
5. **Soket domain lokal (local domain sockets)**
6. **Pipa bernama (named pipes - FIFOs)**
7. **Tautan simbolis (symbolic links)**

Anda dapat menentukan jenis berkas dengan menggunakan perintah **file**.

```bash
$ file /bin/bash
bin/bash: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=33a5554034feb2af38e8c75872058883b2988bc5, for GNU/Linux 3.2.0, stripped
```

Anda juga dapat menggunakan **ls -ld**, flag **-d** memaksa **ls** untuk menampilkan informasi tentang direktori daripada menampilkan isi direktori.

![pengkodean-jenis-berkas](./data/file-type-encoding.png)

**Berkas biasa** terdiri dari serangkaian byte; sistem berkas tidak memaksakan struktur pada isinya. Berkas teks, berkas data, program yang dapat dieksekusi, dan pustaka bersama semuanya disimpan sebagai berkas biasa.

**Direktori** adalah referensi bernama ke berkas lain.

**Tautan keras (hard links)** adalah cara untuk memberikan beberapa nama pada satu berkas. Perintah **ln** membuat tautan keras baru ke berkas yang ada. Opsi **-i** pada **ls** menyebabkannya menampilkan jumlah tautan keras ke setiap berkas.

Contoh:

```bash
$ ln /etc/passwd /tmp/passwd
```

**Berkas perangkat karakter dan blok**

Berkas perangkat memungkinkan program berkomunikasi dengan perangkat keras dan periferal sistem. Kernel menyertakan (atau memuat) perangkat lunak driver untuk setiap perangkat sistem. Perangkat lunak ini mengurus detail rumit pengelolaan setiap perangkat sehingga kernel itu sendiri dapat tetap relatif abstrak dan independen dari perangkat keras.

Perbedaan antara perangkat karakter dan blok sangat halus dan tidak layak ditinjau secara detail.

Berkas perangkat dicirikan oleh nomor perangkat utama dan minornya. Nomor utama mengidentifikasi driver yang mengontrol perangkat, dan nomor minor biasanya memberi tahu driver unit fisik mana yang akan dialamatkan. Misalnya, nomor perangkat utama 4 pada sistem Linux menunjukkan driver serial. Port serial pertama (**/dev/tty0**) akan memiliki nomor perangkat utama 4 dan nomor perangkat minor 0, port serial kedua (**/dev/tty1**) akan memiliki nomor perangkat utama 4 dan nomor perangkat minor 1, dan seterusnya.

Di masa lalu, **/dev** adalah direktori generik dan perangkat dibuat dengan **mknod** dan dihapus dengan **rm**. Saat ini, direktori **/dev** biasanya dipasang sebagai jenis sistem berkas khusus, dan isinya secara otomatis dikelola oleh kernel bersama dengan daemon tingkat pengguna.

**Soket domain lokal** adalah cara bagi proses untuk berkomunikasi satu sama lain. Mereka mirip dengan soket jaringan, tetapi terbatas pada host lokal.

**Pipa bernama** seperti soket domain lokal memungkinkan proses yang berjalan untuk berkomunikasi satu sama lain dalam host yang sama.

**Tautan simbolis** atau tautan lunak menunjuk ke berkas berdasarkan nama. Mereka adalah cara untuk memberikan beberapa nama pada satu berkas, tetapi lebih fleksibel daripada tautan keras. Mereka dapat menunjuk ke berkas di sistem berkas yang berbeda, dan mereka dapat menunjuk ke direktori.

Misalnya, direktori **/usr/bin** seringkali merupakan tautan simbolis ke **/bin**. Ini adalah cara untuk menjaga sistem berkas root tetap kecil dan memudahkan berbagi perangkat lunak yang sama di antara beberapa host.

```bash
$ ln -s /bin /usr/bin

$ ls -l /usr/bin
lrwxrwxrwx 1 root root 4 Mar  1  2020 /usr/bin -> /bin
```

## Atribut Berkas

Dalam model sistem berkas Unix dan Linux, setiap berkas memiliki seperangkat sembilan bit izin, yang menentukan siapa yang dapat membaca, menulis, dan mengeksekusi berkas. Bersama dengan tiga bit lainnya yang terutama memengaruhi operasi program yang dapat dieksekusi, bit-bit ini membentuk mode berkas.

Dua belas bit mode ini disimpan bersama dengan empat bit informasi tipe berkas. Empat bit tipe berkas diatur ketika berkas dibuat dan tidak dapat diubah, tetapi pemilik berkas dan superuser dapat memodifikasi dua belas bit mode dengan perintah **chmod**.

![atribut-berkas](https://cdn.storyasset.link/nlFtWFR5rySdmletT0jhDUQ0tXl2/ms-yxhcfoletf.jpg)

### Bit izin

Bit izin dibagi menjadi tiga kelompok dengan tiga bit masing-masing. Kelompok pertama dari tiga bit adalah untuk pemilik berkas, kelompok kedua adalah untuk grup berkas, dan kelompok ketiga adalah untuk semua orang lainnya.

Anda dapat menggunakan nama **Hugo** untuk mengingat urutan kelompok: **u** untuk pemilik (user), **g** untuk grup, dan **o** untuk yang lain (others).

Dimungkinkan juga untuk menggunakan notasi oktal (basis 8) karena setiap digit dalam notasi oktal mewakili tiga bit. Tiga bit teratas (dengan nilai oktal 400, 200, dan 100) mewakili pemilik berkas, tiga bit tengah (dengan nilai oktal 40, 20, dan 10) mewakili grup berkas, dan tiga bit terbawah (dengan nilai oktal 4, 2, dan 1) mewakili semua orang lainnya.

Pada berkas biasa, bit baca memungkinkan berkas untuk dibaca, bit tulis memungkinkan berkas untuk dimodifikasi atau dipotong; namun, kemampuan untuk menghapus atau mengganti nama (atau menghapus dan kemudian membuat ulang!) berkas dikendalikan oleh izin pada direktori induknya, di mana pemetaan nama-ke-ruang-data dipertahankan.

Bit eksekusi memungkinkan berkas untuk dieksekusi. Ada dua jenis berkas yang dapat dieksekusi: binari, yang dijalankan langsung oleh CPU, dan skrip, yang harus ditafsirkan oleh program seperti shell atau Python. Secara konvensi, skrip dimulai dengan baris **shebang** yang memberi tahu kernel interpreter mana yang akan digunakan.

```bash
#!/usr/bin/perl
```

Berkas yang dapat dieksekusi non-binari yang tidak menentukan interpreter diasumsikan sebagai skrip **sh**.

Kernel memahami sintaks *#!* (shebang) dan bertindak langsung. Namun, jika interpreter tidak ditentukan secara lengkap dan benar, kernel akan menolak berkas tersebut. Shell kemudian mengambil alih dan mencoba menafsirkan berkas sebagai skrip shell.

Untuk direktori, bit eksekusi (sering disebut bit pencarian atau pemindaian) memungkinkan direktori untuk dimasuki atau dilewati saat pathname dievaluasi, tetapi tidak untuk daftar isinya. Kombinasi bit baca dan eksekusi memungkinkan direktori untuk dibaca dan isinya dicantumkan. Kombinasi bit tulis dan eksekusi memungkinkan berkas dibuat, dihapus, dan diganti namanya dalam direktori.

### Bit setuid dan setgid

Bit dengan nilai oktal 4000 dan 2000 adalah bit setuid dan setgid. Ketika bit setuid diatur pada berkas, pemilik berkas sementara diubah menjadi pemilik berkas ketika berkas dieksekusi. Ketika bit setgid diatur pada berkas, grup berkas sementara diubah menjadi grup berkas ketika berkas dieksekusi.

Ketika diatur pada direktori, bit setgid menyebabkan berkas yang baru dibuat dalam direktori mengambil kepemilikan grup dari direktori daripada grup default dari pengguna yang membuat berkas. Ini memudahkan berbagi berkas di antara sekelompok pengguna.

### Bit sticky

Bit dengan nilai oktal 1000 adalah bit sticky. Ketika diatur pada direktori, bit sticky mencegah pengguna menghapus atau mengganti nama berkas yang bukan milik mereka. Ini berguna untuk direktori seperti **/tmp** yang dibagikan di antara banyak pengguna.

### ls: daftar dan periksa berkas

Perintah **ls** mencantumkan berkas dan direktori. Ini juga dapat digunakan untuk memeriksa atribut berkas dan direktori.

Opsi **-l** menyebabkan **ls** menampilkan format panjang, yang mencakup mode berkas, jumlah tautan keras ke berkas, pemilik berkas, grup berkas, ukuran berkas dalam byte, waktu modifikasi berkas, dan nama berkas.

Semua direktori memiliki setidaknya dua tautan keras: satu dari direktori itu sendiri (entri **..**) dan satu dari direktori induknya (entri **.**).

Output **ls** sedikit berbeda untuk berkas perangkat. Misalnya:

```bash
$ ls -l /dev/tty0
crw--w---- 1 root tty 4, 0 Mar  1  2020 /dev/tty0
```

**c** di awal baris menunjukkan bahwa berkas tersebut adalah berkas perangkat karakter. **4, 0** di akhir baris adalah nomor perangkat utama dan minor.

### chmod: ubah izin

Perintah **chmod** mengubah mode berkas. Anda dapat menggunakan notasi oktal atau notasi simbolis.

![pengkodean-izin](./data/permissions-encoding.png)

Contoh sintaks mnemonik chmod:

| Spesifikasi | Arti                                                                        |
| ----------- | --------------------------------------------------------------------------- |
| u+w         | Tambahkan izin tulis untuk pemilik berkas                                   |
| ug=rw,o=r   | Memberikan izin r/w kepada pemilik dan grup, dan izin r kepada orang lain   |
| a-x         | Hapus izin eksekusi untuk semua pengguna                                    |
| ug=srx, o=  | Atur bit setuid, setgid, dan sticky untuk pemilik dan grup (r/x)           |
| g=u         | Jadikan izin grup sama dengan izin pemilik                                  |

**Tips**: Anda juga dapat menentukan mode yang akan ditetapkan dengan menyalin mode dari berkas lain dengan opsi **--reference** (misalnya **chmod --reference=berkas_sumber berkas_target**).

### chown: ubah kepemilikan

Perintah **chown** mengubah pemilik dan grup berkas. Opsi **-R** menyebabkan **chown** mengubah kepemilikan isi berkas secara rekursif.

```bash
$ chown -R abdou:users /home/abdou
```

### chgrp: ubah grup

Perintah **chgrp** mengubah grup berkas. Opsi **-R** menyebabkan **chgrp** mengubah grup isi berkas secara rekursif.

```bash
$ chgrp -R users /home/abdou
```

### umask: atur izin default

Perintah **umask** mengatur izin default untuk berkas dan direktori baru. Perintah **umask** adalah bit mask yang dikurangkan dari izin default untuk menentukan izin sebenarnya.

Contoh:

```bash
$ umask 022
```

| Oktal | Biner | Izin | Oktal | Biner | Izin |
| ----- | ----- | ---- | ----- | ----- | ---- |
| 0     | 000   | rwx  | 4     | 100   | -wx  |
| 1     | 001   | rw-  | 5     | 101   | -w-  |
| 2     | 010   | r-x  | 6     | 110   | --x  |
| 3     | 011   | r--  | 7     | 111   | ---  |

Misalnya, **umask 027** memungkinkan rwx untuk pemilik, rx untuk grup, dan tidak ada izin untuk orang lain.