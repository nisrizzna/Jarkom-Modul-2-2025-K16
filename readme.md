### MEMBER
1. Muhammad Ardiansyah Tri Wibowo - 5027241091
2. Nisrina Bilqis - 5027241054

### AKSES SOAL
https://docs.google.com/document/d/1MF4eaywFiqu8w4JgL1dZvVqAOsIoyvLM26ET7PhR9cg/edit?tab=t.0

### Soal 1
Pada soal pertama yaitu
```
Di tepi Beleriand yang porak-poranda, Eonwe merentangkan tiga jalur: Barat untuk Earendil dan Elwing,
Timur untuk Círdan, Elrond, Maglor, serta pelabuhan DMZ bagi Sirion, Tirion, Valmar, Lindon, Vingilot.
Tetapkan alamat dan default gateway tiap tokoh sesuai glosarium yang sudah diberikan.
```
Pada tahap ini, kami melakukan konfigurasi alamat IP dan default gateway untuk setiap node sesuai dengan topologi jaringan yang diberikan. Eonwe berfungsi sebagai router utama yang menghubungkan tiga subnet: Barat, Timur, dan DMZ.

#### Topologi & Alokasi IP:
Eonwe (Router):
- eth1 (Barat): 192.219.1.1
- eth2 (Timur): 192.219.2.1
- eth3 (DMZ): 192.219.3.1
Subnet Barat (Gateway: 192.219.1.1):
- Earendil: 192.219.1.2
- Elwing: 192.219.1.3
Subnet Timur (Gateway: 192.219.2.1):
- Cirdan: 192.219.2.2
- Elrond: 192.219.2.3
- Maglor: 192.219.2.4
Subnet DMZ (Gateway: 192.219.3.1):
- Sirion: 192.219.3.2 
- Tirion (ns1): 192.219.3.3
- Valmar (ns2): 192.219.3.4
- Lindon: 192.219.3.5
- Vingilot: 192.219.3.6

#### Script
Contoh script di Cirdan
```
# /etc/network/interfaces

auto eth0
iface eth0 inet static
    address 192.219.2.2
    netmask 255.255.255.0
    gateway 192.219.2.1
```
  
<img width="893" height="765" alt="image" src="https://github.com/user-attachments/assets/74e91b39-407e-4f6e-afdb-40f80990e284" />

### Soal 2
Pada soal kedua yaitu
```
Angin dari luar mulai berhembus ketika Eonwe membuka jalan ke awan NAT. Pastikan jalur WAN di router aktif dan NAT meneruskan trafik keluar bagi seluruh alamat internal
 sehingga host di dalam dapat mencapai layanan di luar menggunakan IP address.
```
Untuk memungkinkan semua host internal mengakses internet, kami mengaktifkan NAT pada router Eonwe. Ini dilakukan dengan menambahkan aturan POSTROUTING menggunakan iptables untuk "menyamarkan" (masquerade) semua trafik keluar dengan IP publik Eonwe.

#### Script
Script di Eonwe
```
# Mengaktifkan IP forwarding
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p

# Menambahkan aturan NAT
iptables -t nat -A POSTROUTING -o <NAMA_INTERFACE_WAN> -j MASQUERADE
```
### Soal 3
Pada soal ketiga yaitu
```
Kabar dari Barat menyapa Timur. Pastikan kelima klien dapat saling berkomunikasi lintas jalur (routing internal via Eonwe berfungsi), lalu pastikan setiap host non-router
 menambahkan resolver 192.168.122.1 saat interfacenya aktif agar akses paket dari internet tersedia sejak awal.
```
Kami memastikan Eonwe dapat merutekan paket antar-subnet internal (Barat, Timur, DMZ) dengan mengaktifkan IP forwarding. Selain itu, semua host non-router dikonfigurasi untuk menggunakan resolver 192.168.122.1 agar dapat menginstall paket dari internet.

Verifikasi Routing Internal: Kami melakukan ping dari Cirdan (Timur) ke Elrond (Timur) untuk menguji konektivitas dasar, yang berhasil.

Konfigurasi Resolver Awal: File /etc/resolv.conf di semua klien diatur sebagai berikut.
```
nameserver 192.168.122.1
```

### Soal 4
Pada soal keempat yaitu
```
Para penjaga nama naik ke menara, di Tirion (ns1/master) bangun zona <xxxx>.com sebagai authoritative dengan SOA yang menunjuk ke ns1.<xxxx>.com dan catatan NS untuk ns1.
<xxxx>.com dan ns2.<xxxx>.com. Buat A record untuk ns1.<xxxx>.com dan ns2.<xxxx>.com yang mengarah ke alamat Tirion dan Valmar sesuai glosarium, serta A record apex
<xxxx>.com yang mengarah ke alamat Sirion (front door), aktifkan notify dan allow-transfer ke Valmar, set forwarders ke 192.168.122.1. Di Valmar (ns2/slave) tarik zona
<xxxx>.com dari Tirion dan pastikan menjawab authoritative. pada seluruh host non-router ubah urutan resolver menjadi IP dari ns1.<xxxx>.com → ns2.<xxxx>.com →
192.168.122.1. Verifikasi query ke apex dan hostname layanan dalam zona dijawab melalui ns1/ns2.
```
Tahap ini adalah inti dari penamaan di jaringan kami. Tirion dikonfigurasi sebagai DNS Master (ns1) dan Valmar sebagai DNS Slave (ns2).

Konfigurasi di Tirion (ns1): File /etc/bind/named.conf.local diubah untuk mendeklarasikan zona k16.com dan mengizinkan transfer ke Valmar.

```
zone "k16.com" {
    type master;
    file "/etc/bind/k16/k16.com";
    notify yes;
    also-notify { 192.219.3.4; }; // IP Valmar
    allow-transfer { 192.219.3.4; }; // Izinkan transfer ke Valmar
};
```
Konfigurasi di Valmar (ns2): Valmar diatur untuk mengambil (menarik) zona k16.com dari Tirion.
```
# /etc/bind/named.conf.local di Valmar
zone "k16.com" {
    type slave;
    masters { 192.219.3.3; }; // IP Tirion
    file "/var/lib/bind/k16.com";
};
```
Penataan Ulang Resolver Klien: Resolver pada semua host non-router diubah urutannya menjadi ns1 -> ns2 -> resolver publik.

```
nameserver 192.219.3.3
nameserver 192.219.3.4
nameserver 192.168.122.1
```

<img width="1502" height="759" alt="image" src="https://github.com/user-attachments/assets/9f7ebc93-4170-4eec-9214-a988ad6cf259" />

### Soal 5
Pada soal kelima yaitu
```
“Nama memberi arah,” kata Eonwe. Namai semua tokoh (hostname) sesuai glosarium, eonwe, earendil, elwing, cirdan, elrond, maglor, sirion, tirion, valmar, lindon, vingilot,
dan verifikasi bahwa setiap host mengenali dan menggunakan hostname tersebut secara system-wide. Buat setiap domain untuk masing masing node sesuai dengan namanya (contoh:
eru.<xxxx>.com) dan assign IP masing-masing juga. Lakukan pengecualian untuk node yang bertanggung jawab atas ns1 dan ns2
```
Setiap node diberikan hostname yang sesuai dan A record ditambahkan ke DNS server untuk semua node.
Penambahan A Records di Tirion: Sebuah script dijalankan di Tirion untuk menambahkan A records untuk semua host ke dalam file zona /etc/bind/k16/k16.com.

```
; Contoh A records di file zona k16.com
eonwe      IN  A   192.219.1.1   ; dan IP lainnya
earendil   IN  A   192.219.1.2
cirdan     IN  A   192.219.2.2
sirion     IN  A   192.219.3.2
...
```

<img width="1225" height="679" alt="image" src="https://github.com/user-attachments/assets/8ac1ae95-6a9e-4835-98da-9fdcd41eee99" />


<img width="1231" height="533" alt="image" src="https://github.com/user-attachments/assets/c811fdd1-b486-48b9-b739-fd0689bd58b8" />

### Soal 6
Pada soal keenam yaitu
```
Lonceng Valmar berdentang mengikuti irama Tirion. Pastikan zone transfer berjalan, Pastikan Valmar (ns2) telah menerima salinan zona terbaru dari Tirion (ns1). Nilai serial SOA di keduanya harus sama
```
Kami memastikan bahwa Valmar (ns2) berhasil menerima salinan zona dari Tirion (ns1) dengan membandingkan nomor serial SOA pada kedua server.
Script Pengecekan: Sebuah script dijalankan dari Cirdan untuk melakukan query dig ke Tirion dan Valmar, lalu membandingkan serialnya.
Hasil Pengecekan: Hasilnya menunjukkan bahwa nomor serial di kedua server sama, yaitu 2025100401, yang mengonfirmasi bahwa zone transfer berhasil.


<img width="1589" height="515" alt="image" src="https://github.com/user-attachments/assets/93b36a9c-7959-434b-8af9-41f00ad407e0" />

### Soal 7
Pada soal ketujuh yaitu
```
Peta kota dan pelabuhan dilukis. Sirion sebagai gerbang, Lindon sebagai web statis, Vingilot sebagai web dinamis. Tambahkan pada zona <xxxx>.com A record untuk sirion.<xxxx>.com (IP Sirion), lindon.<xxxx>.com (IP Lindon), dan vingilot.<xxxx>.com (IP Vingilot). Tetapkan CNAME :
www.<xxxx>.com → sirion.<xxxx>.com, 
static.<xxxx>.com → lindon.<xxxx>.com, dan 
app.<xxxx>.com → vingilot.<xxxx>.com. 
Verifikasi dari dua klien berbeda bahwa seluruh hostname tersebut ter-resolve ke tujuan yang benar dan konsisten.
```

Record DNS tambahan dibuat untuk layanan web. A record dibuat untuk host yang melayani konten, dan CNAME dibuat sebagai alias yang lebih mudah diingat.

#### Konfigurasi di File Zona (Tirion):
```
; A Records
sirion      IN  A   192.219.3.2
lindon      IN  A   192.219.3.5
vingilot    IN  A   192.219.3.6

; CNAME Records
www         IN  CNAME   sirion.k16.com.
static      IN  CNAME   lindon.k16.com.
app         IN  CNAME   vingilot.k16.com.
```
Verifikasi: Kami melakukan ping ke app.k16.com dari Cirdan. Hasilnya menunjukkan bahwa nama tersebut berhasil di-resolve ke vingilot.k16.com dengan IP 192.219.3.6.

<img width="1174" height="790" alt="image" src="https://github.com/user-attachments/assets/412e0efa-8776-4b91-b6fb-b60c7822147e" />

### Soal 8
Pada soal kedelapan yaitu
```
Setiap jejak harus bisa diikuti. Di Tirion (ns1) deklarasikan satu reverse zone untuk segmen DMZ tempat Sirion, Lindon, Vingilot berada. Di Valmar (ns2) tarik reverse zone tersebut sebagai slave, isi PTR untuk ketiga hostname itu agar pencarian balik IP address mengembalikan hostname yang benar, lalu pastikan query reverse untuk alamat Sirion, Lindon, Vingilot dijawab authoritative.
```
Reverse DNS (rDNS) dikonfigurasi agar pencarian berdasarkan alamat IP dapat mengembalikan nama host yang benar. Ini penting untuk verifikasi dan logging.

#### Konfigurasi Reverse Zone di Tirion:
```
# /etc/bind/named.conf.local
zone "3.219.192.in-addr.arpa" {
    type master;
    file "/etc/bind/k16/db.reverse";
    allow-transfer { 192.219.3.4; }; // Allow transfer to Valmar
};
```
Isi file reverse zone:
```
; /etc/bind/k16/db.reverse
...
2   IN  PTR sirion.k16.com.
5   IN  PTR lindon.k16.com.
6   IN  PTR vingilot.k16.com.
```
Verifikasi: Kami melakukan query rDNS untuk IP Sirion (192.219.3.2) dari Tirion. Hasilnya mengembalikan sirion.k16.com., yang menandakan konfigurasi berhasil.

<img width="1350" height="785" alt="image" src="https://github.com/user-attachments/assets/e0135f41-11e5-421b-a066-5d01ea24e289" />

### Soal 9
Pada soal kesembilan yaitu
```
Lampion Lindon dinyalakan. Jalankan web statis pada hostname static.<xxxx>.com dan buka folder arsip /annals/ dengan autoindex (directory listing) sehingga isinya dapat ditelusuri. Akses harus dilakukan melalui hostname, bukan IP.
```
Lindon berfungsi sebagai server web statis. Kami menginstal Nginx dan mengonfigurasinya untuk menyajikan konten dari direktori /annals/ dengan autoindex on.

#### Konfigurasi Nginx di Lindon:
```
# /etc/nginx/sites-available/default
server {
    listen 80;
    server_name static.k16.com lindon.k16.com;

    location /annals/ {
        root /var/www/html;
        autoindex on;
    }
}
```
Verifikasi: Kami mengakses http://static.k16.com/annals/ menggunakan curl dari Cirdan. Outputnya adalah halaman HTML yang berisi daftar file, sesuai harapan.


<img width="1655" height="711" alt="image" src="https://github.com/user-attachments/assets/8bc4d2c2-ae1f-4f32-be24-60f3cff4423a" />

### Soal 10
Pada Soal kesepuluh yaitu
```
Vingilot mengisahkan cerita dinamis. Jalankan web dinamis (PHP-FPM) pada hostname app.<xxxx>.com dengan beranda dan halaman about, serta terapkan rewrite sehingga /about berfungsi tanpa akhiran .php. Akses harus dilakukan melalui hostname.
```

Vingilot menyajikan konten dinamis menggunakan PHP-FPM. Nginx di Vingilot dikonfigurasi untuk meneruskan request .php ke service PHP-FPM dan menerapkan rewrite rule untuk URL yang lebih rapi.

#### Konfigurasi Nginx di Vingilot:
```
server {
    listen 80;
    server_name app.k16.com vingilot.k16.com;
    root /var/www/html;
    index index.php index.html;

    # Rewrite /about to /about.php
    location /about {
        try_files $uri /about.php;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.4-fpm.sock;
    }
}
```
Verifikasi: Kami mengakses http://app.k16.com dan http://app.k16.com/about dari Cirdan. Kedua halaman berhasil ditampilkan.


<img width="994" height="761" alt="image" src="https://github.com/user-attachments/assets/0451e836-41b0-4f08-be17-8d9d6deff899" />


