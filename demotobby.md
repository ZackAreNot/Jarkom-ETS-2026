# Panduan Demo Tobby — Soal 7 dan 8

**File demo:** `K12_VLSM_OSPF.pkt` · **Kelompok:** 12 · **Skema:** VLSM · **Domain:** `k12.smartcity.lab` · **Link cadangan:** L7 `10.96.241.56/30`

Panduan ini mencakup demo OSPF/failover dan layanan aplikasi. Target soal 7: static route antar-router diganti OSPF, link cadangan L7 antara R-DC dan R-UTARA, router tidak mengirim pesan OSPF ke LAN pengguna, default route hanya di R-PUSAT lalu disebarkan lewat OSPF, dan konektivitas pulih otomatis jika satu link backbone putus. Target soal 8: DNS, web, FTP, dan email berjalan dengan domain `k12.smartcity.lab`. Alokasi waktu sekitar **5 menit per soal**.

## Persiapan

1. Buka `K12_VLSM_OSPF.pkt` di Packet Tracer. Pastikan file yang dibuka adalah file OSPF, bukan master static routing.
2. Tunggu link stabil. Untuk router, klik router → **CLI** → tekan **Enter** → ketik `enable`. Untuk PC, klik PC → **Desktop → Command Prompt**.
3. IP perangkat klien berasal dari DHCP dan bisa berubah. Jalankan `ipconfig /all` pada PC yang dipakai, lalu gunakan alamat IPv4 terbaru sebagai tujuan ping atau traceroute.
4. Siapkan jendela Command Prompt PC-CCTV-KOTA dan CLI R-PUSAT. Untuk demo layanan, siapkan juga PC-PEGAWAI, PC-KANTOR-PELABUHAN, serta PC-SENSOR-LALIN.
5. Layanan berjalan di jaringan simulasi Packet Tracer. `8.8.8.8` adalah server simulasi, bukan bukti akses internet publik.

## Soal 7 — OSPF dan failover (±5 menit)

### Pembuka

> “Di file ini, static route antar-router diganti dengan OSPF area 0. R-PUSAT menyebarkan default route ke internet. L7 menjadi jalur cadangan antara R-DC dan R-UTARA. Saat L1 atau L2 putus, OSPF menghitung jalur alternatif dan memulihkan konektivitas.”

### 7.1 Tunjukkan tetangga dan rute OSPF

Pada **R-PUSAT → CLI** jalankan:

```text
show ip ospf neighbor
show ip route
```

**Yang ditunjukkan:** tetangga OSPF pada backbone berstatus `FULL`. Dalam kondisi normal R-PUSAT melihat lima tetangga: R-DC, R-UTARA, R-BARAT, R-TIMUR, dan R-SELATAN. Pada tabel rute, tunjukkan rute berkode `O` dan default route OSPF `O*E2 0.0.0.0/0`. LAN yang terhubung langsung bisa berkode `C`; tidak semua baris harus berkode O.

Lalu, pada **R-DC → CLI** jalankan:

```text
show ip ospf neighbor
show ip route
```

**Yang dicari:** R-DC bertetangga dengan R-PUSAT melalui L1 dan R-UTARA melalui L7. Tabel rute menunjukkan LAN yang dipelajari lewat OSPF serta default route ke arah R-PUSAT.

### 7.2 Catat jalur normal dan mulai ping kontinu

Di **PC-CCTV-KOTA → Desktop → Command Prompt**, pastikan alamat PC-PEGAWAI dengan `ipconfig /all`. README mencatat `10.96.224.11` sebagai alamat PC-PEGAWAI saat pengujian; bila berbeda, gunakan alamat terbaru.

Catat jalur normal:

```text
tracert 10.96.224.11
```

Jalur normal menuju Balai Kota melewati R-DC dan L1 ke R-PUSAT. Setelah itu jalankan ping berkelanjutan dan biarkan jendela tetap terbuka:

```text
ping -t 10.96.224.11
```

### 7.3 Putus L1, tunjukkan pemulihan lewat L7, lalu pulihkan link

Saat ping terus berjalan, pindah ke **R-PUSAT → CLI** dan matikan interface L1 (`Serial0/0/0`):

```text
enable
configure terminal
interface Serial0/0/0
shutdown
end
```

Kembali ke PC-CCTV-KOTA. Tunjukkan beberapa `Request timed out` (RTO) saat OSPF menghitung ulang rute. Tunggu hingga `Reply from` muncul kembali. Konvergensi dapat memerlukan puluhan detik. Hentikan ping dengan **Ctrl+C** setelah reply pulih.

Kemudian periksa jalur baru:

```text
tracert 10.96.224.11
```

**Jalur yang diharapkan saat L1 mati:** R-DC → L7 menuju R-UTARA (`10.96.241.58`) → L2 menuju R-PUSAT (`10.96.241.37`) → PC-PEGAWAI. Hop tepat pada output dapat berbeda karena tampilan simulasi, tetapi jalur semestinya memakai L7.

Pulihkan L1 dari **R-PUSAT → CLI** sebelum mengakhiri demo:

```text
enable
configure terminal
interface Serial0/0/0
no shutdown
end
```

Tunggu interface kembali `up/up`, lalu pastikan tetangga pulih:

```text
show ip interface brief
show ip ospf neighbor
```

R-PUSAT kembali melihat lima tetangga `FULL`. Bila link sudah pulih dan semua tetangga kembali `FULL`, konfigurasi dapat disimpan:

```text
write memory
```

**Kesimpulan lisan:**

> “Saat L1 putus, ping sempat timeout karena OSPF sedang konvergen. Setelah itu reply kembali dan traceroute menunjukkan jalur alternatif melalui L7. L1 sudah saya sambungkan kembali dan tetangga OSPF kembali FULL.”

> **Penting:** Jangan menyimpan konfigurasi saat L1 masih `shutdown`. Jika waktu demo sangat terbatas, fokuskan failover pada L1; jangan mulai uji L2 sebelum L1 dipulihkan.

### Pertanyaan yang mungkin ditanyakan — Soal 7

- **Apa bedanya OSPF dengan static routing?** Static route ditulis manual pada router, sedangkan OSPF bertukar informasi rute dengan tetangga dan menghitung ulang jalur saat topologi berubah.
- **Apa arti neighbor `FULL`?** Router sudah membentuk adjacency OSPF dan menyelaraskan informasi link-state dengan tetangganya.
- **Mengapa ada `O*E2 0.0.0.0/0`?** Itu default route eksternal yang berasal dari R-PUSAT dan disebarkan ke router lain melalui OSPF.
- **Apa fungsi L7?** L7 menghubungkan R-DC dengan R-UTARA sebagai jalur alternatif agar trafik tetap dapat menuju R-PUSAT saat L1 atau L2 terputus.
- **Mengapa ping sempat RTO?** Setelah link putus, OSPF perlu mendeteksi perubahan dan menghitung ulang rute. Selama konvergensi, sebagian paket bisa hilang; setelah rute alternatif terpasang, reply pulih.
- **Bagaimana membuktikan jalur berubah?** Bandingkan `tracert` sebelum dan sesudah L1 diputus; jalur setelahnya semestinya melewati L7 melalui R-UTARA.

### Catatan demo Soal 7

- Sebelum mulai, pastikan L1 dan L7 aktif, rute OSPF sudah muncul, dan neighbor berstatus `FULL`.
- Tunjukkan urutan bukti: neighbor/routing table normal → `tracert` normal → `ping -t` → RTO saat L1 diputus → reply pulih → `tracert` lewat L7 → L1 disambungkan kembali.
- Beri waktu OSPF untuk konvergen. Akhiri dengan memastikan L1 `up/up` dan lima neighbor R-PUSAT kembali `FULL`; simpan hanya setelah kondisi normal.

## Soal 8 — Layanan aplikasi (±5 menit)

### Ringkasan layanan

| Layanan | Server dan alamat | Nama/akun yang digunakan |
|---|---|---|
| DNS | SRV-DC — `10.96.239.2` | `k12.smartcity.lab` |
| Web | SRV-BALAIKOTA — `10.96.241.2` | `www.k12.smartcity.lab` |
| FTP | SRV-DC — `10.96.239.2` | `ftp.k12.smartcity.lab`, user `cctv` |
| Email SMTP/POP3 | SRV-BALAIKOTA — `10.96.241.2` | `mail.k12.smartcity.lab` |

### 8.1 DNS dan web dari dua wilayah

Pertama, pada **PC-PEGAWAI di Balai Kota → Desktop → Command Prompt**, jalankan:

```text
nslookup www.k12.smartcity.lab
nslookup ftp.k12.smartcity.lab
nslookup mail.k12.smartcity.lab
```

**Yang dijelaskan:** DNS yang menjawab adalah SRV-DC `10.96.239.2`; `www` dan `mail` diterjemahkan ke `10.96.241.2`, sedangkan `ftp` ke `10.96.239.2`.

Buka **Desktop → Web Browser** pada PC-PEGAWAI dan kunjungi:

```text
http://www.k12.smartcity.lab
```

Tunjukkan halaman **Portal Smart City Surabaya** dan nomor kelompok 12. Tekankan bahwa situs dibuka dengan nama domain, bukan alamat IP.

Ulangi pemeriksaan nama dan web dari **PC-KANTOR-PELABUHAN** untuk menunjukkan akses dari wilayah berbeda. Sebelum menguji Pelabuhan, lakukan pemanasan koneksi bila tunnel baru aktif:

```text
ping 10.96.224.11
```

Gunakan IP PC-PEGAWAI terkini dari `ipconfig /all` bila alamatnya berubah. Setelah pemanasan, pada **R-PUSAT → CLI** periksa status IPsec:

```text
show crypto isakmp sa
```

Pastikan sesi dengan peer `10.96.241.54` berstatus `QM_IDLE`. Jika status itu belum muncul, kirim beberapa ping lagi antara PC-PEGAWAI dan PC-KANTOR-PELABUHAN, lalu periksa kembali. Di PC-KANTOR-PELABUHAN, jalankan `nslookup www.k12.smartcity.lab`, lalu buka `http://www.k12.smartcity.lab` di Web Browser.

### 8.2 FTP: login, upload, dan download

FTP menggunakan akun `cctv` pada SRV-DC. Di **PC-CCTV-KOTA → Desktop → Command Prompt**, pastikan `rekaman.txt` sudah tersedia, lalu mulai sesi FTP:

```text
ftp ftp.k12.smartcity.lab
```

Saat diminta, masukkan username `cctv` dan password FTP yang telah dikonfigurasi. Pada prompt `ftp>`, unggah file dan verifikasi:

```text
put rekaman.txt
dir
quit
```

Untuk bukti akses dari wilayah kedua, di **PC-SENSOR-LALIN (Utara) → Desktop → Command Prompt**:

```text
ftp ftp.k12.smartcity.lab
```

Login dengan akun yang sama, lalu unduh file yang baru diunggah:

```text
get rekaman.txt
dir
quit
```

`put` mengunggah file dari CCTV ke server dan `get` mengunduh file dari server ke Utara.

### 8.3 Email dua arah Balai Kota–Pelabuhan

Pada **PC-KANTOR-PELABUHAN**, buka **Desktop → Email** dan kirim email ke:

```text
pegawai@k12.smartcity.lab
```

Pada **PC-PEGAWAI**, klik **Receive**, buka email yang masuk, dan tunjukkan alamat pengirim serta isinya. Untuk membuktikan arah sebaliknya, balas/kirim email dari PC-PEGAWAI ke:

```text
pelabuhan@k12.smartcity.lab
```

Klik **Receive** pada PC-KANTOR-PELABUHAN dan tunjukkan email balasan masuk. Kedua alamat memakai server `mail.k12.smartcity.lab`; aplikasi email sudah dikonfigurasi dengan akun `pegawai` dan `pelabuhan`.

**Kesimpulan lisan:**

> “Dari Balai Kota dan wilayah lain, DNS menerjemahkan nama layanan dengan benar dan portal web dapat dibuka menggunakan domain. FTP memungkinkan CCTV mengunggah rekaman dan wilayah Utara mengunduhnya. Email berhasil dikirim dan diterima dua arah antara Balai Kota dan Pelabuhan.”

### Pertanyaan yang mungkin ditanyakan — Soal 8

- **Apa fungsi DNS?** DNS menerjemahkan nama domain layanan menjadi alamat IP server. Contohnya `www.k12.smartcity.lab` menjadi `10.96.241.2`.
- **Mengapa web diuji dengan nama domain?** Agar terbukti klien dapat menggunakan DNS untuk menemukan server, bukan hanya mengakses alamat IP secara langsung.
- **Apa bedanya `put` dan `get` pada FTP?** `put` mengunggah file dari klien ke server, sedangkan `get` mengunduh file dari server ke klien.
- **Apa fungsi SMTP dan POP3?** SMTP digunakan untuk mengirim email ke server; POP3 digunakan klien untuk mengambil email dari server.
- **Mengapa sesi IPsec perlu `QM_IDLE` saat menguji Pelabuhan?** `QM_IDLE` menunjukkan sesi ISAKMP/IPsec sudah terbentuk dan siap membawa trafik yang dilindungi di antara kedua peer.
- **Apakah DNS saja cukup agar layanan dapat dipakai?** Tidak. Nama harus diterjemahkan ke IP yang benar, layanan pada server harus aktif, dan harus ada jalur jaringan dari klien ke server.

### Catatan demo Soal 8

- Pastikan layanan DNS, HTTP, FTP, SMTP, dan POP3 sudah aktif sebelum merekam demo; siapkan `rekaman.txt` dan kredensial FTP/email.
- Tunjukkan pengujian dari setidaknya dua wilayah: Balai Kota dan Pelabuhan untuk DNS/web/email, serta CCTV dan Utara untuk upload/download FTP.
- Karena email ditempatkan di Balai Kota dan klien Pelabuhan mengaksesnya melalui tunnel, lakukan pemanasan ping dan pastikan sesi IPsec `QM_IDLE` di R-PUSAT sebelum uji layanan dari Pelabuhan.

## Jika ada kendala saat demo

- **Neighbor belum `FULL`:** periksa `show ip interface brief` dan tunggu link serial `up/up`; pastikan semua link yang sebelumnya diuji sudah `no shutdown`.
- **Ping belum pulih setelah failover:** tunggu konvergensi, cek `show ip ospf neighbor` dan `show ip route` di R-DC/R-PUSAT, serta pastikan L7 `up/up`.
- **Domain tidak terjawab:** cek DNS pada `ipconfig /all` harus `10.96.239.2`, lalu periksa DNS Service dan A Record di SRV-DC.
- **Web/FTP tidak terbuka dengan nama:** ulangi `nslookup`; pastikan layanan HTTP/FTP aktif dan nama yang diketik persis.
- **FTP login gagal:** pastikan layanan FTP On dan gunakan username/password yang sama dengan akun di SRV-DC.
- **Email gagal:** pastikan SMTP dan POP3 On, domain `k12.smartcity.lab` sudah di-Set, akun klien/server cocok, serta jalur Balai Kota–Pelabuhan sudah terbentuk. Ping dari PC-PEGAWAI ke PC-KANTOR-PELABUHAN dapat membantu memanaskan tunnel.

