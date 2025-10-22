 Jarkom-Modul-2-2025-K-16

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
Angin dari luar mulai berhembus ketika Eonwe membuka jalan ke awan NAT. Pastikan jalur WAN di router aktif dan NAT meneruskan trafik keluar bagi seluruh alamat internal sehingga host di dalam dapat mencapai layanan di luar menggunakan IP address.
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
Kabar dari Barat menyapa Timur. Pastikan kelima klien dapat saling berkomunikasi lintas jalur (routing internal via Eonwe berfungsi), lalu pastikan setiap host non-router menambahkan resolver 192.168.122.1 saat interfacenya aktif agar akses paket dari internet tersedia sejak awal.
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
Para penjaga nama naik ke menara, di Tirion (ns1/master) bangun zona <xxxx>.com sebagai authoritative dengan SOA yang menunjuk ke ns1.<xxxx>.com dan catatan NS untuk ns1.<xxxx>.com dan ns2.<xxxx>.com. Buat A record untuk ns1.<xxxx>.com dan ns2.<xxxx>.com yang mengarah ke alamat Tirion dan Valmar sesuai glosarium, serta A record apex <xxxx>.com yang mengarah ke alamat Sirion (front door), aktifkan notify dan allow-transfer ke Valmar, set forwarders ke 192.168.122.1. Di Valmar (ns2/slave) tarik zona <xxxx>.com dari Tirion dan pastikan menjawab authoritative. pada seluruh host non-router ubah urutan resolver menjadi IP dari ns1.<xxxx>.com → ns2.<xxxx>.com → 192.168.122.1. Verifikasi query ke apex dan hostname layanan dalam zona dijawab melalui ns1/ns2.
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
