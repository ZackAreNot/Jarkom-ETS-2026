# Laporan Mid-Project Jaringan Komputer
### Implementasi Jaringan Smart City Kota Surabaya
### Komunikasi Data & Jaringan Komputer 2026/2027

**Kelompok:** K-12
**Base IP Kelompok:** `10.96.0.0/16`
**Skema yang dipakai:** VLSM
**Domain internal:** `k12.smartcity.lab`
**Tools:** Cisco Packet Tracer 9.0.1

>Maulana Zaki Putra Zakaria | 5027251009

>I Made Tobby Anantha Adiwijaya | 5027251064

>Michiko Artika Satriyo | 5027251105

---

## Daftar Isi
- [Ringkasan Topologi & Tabel Pengalamatan IP](#ringkasan-topologi--tabel-pengalamatan-ip)
  - [Alasan Memilih VLSM](#alasan-memilih-vlsm)
  - [Tabel VLSM](#tabel-vlsm)
  - [Link Backbone & Alamat Tambahan](#link-backbone--alamat-tambahan)
  - [Tabel Port Router](#tabel-port-router)
  - [Alamat Statis](#alamat-statis)
- [Panduan Dasar Packet Tracer](#panduan-dasar-packet-tracer)
- [Soal 1 – Perancangan Topologi](#soal-1)
- [Soal 2 – Tabel Subnetting](#soal-2)
- [Soal 3 – Static Routing](#soal-3)
- [Soal 4 – DHCP](#soal-4)
- [Soal 5 – NAT](#soal-5)
- [Soal 6 – Tunnel GRE](#soal-6)
- [Soal 7 – Dynamic Routing OSPF](#soal-7)
- [Soal 8 – Layanan Aplikasi Smart City](#soal-8)
- [Kendala & Solusi](#kendala--solusi)

---

## Ringkasan Topologi & Tabel Pengalamatan IP

![Topologi Smart City Surabaya](Screenshot/soal-1/topologi.png)
*Topologi lengkap K12: R-ISP dan SRV-INTERNET di atas, R-PUSAT (Balai Kota) di tengah, lima router wilayah di bawahnya, dan R-PELABUHAN di bawah R-UTARA. Garis merah = link serial backbone L1–L6, garis hitam putus-putus = link tembaga ke ISP, setiap LAN diberi label nama subnet, network ID/prefix, dan jumlah host.*

### Alasan Memilih VLSM

Kebutuhan seluruh LAN dan link adalah **61.752 alamat**, yaitu **94,2%** dari blok `10.96.0.0/16` (65.536 alamat). Ruang sisa hanya 3.784 alamat.

- **VLSM** (urut dari kebutuhan host terbesar) memberi setiap subnet blok terkecil yang cukup, sehingga semuanya muat di dalam `/16`.
- **CIDR berbasis topologi** (setiap router bisa diringkas menjadi satu supernet) **tidak muat** di `/16`. Contoh: R-DC, R-BARAT, R-TIMUR, R-SELATAN, dan R-PELABUHAN masing-masing punya satu LAN `/19` ditambah LAN kecil, jadi supernet tiap router minimal `/18` (16.384 alamat). Lima router × `/18` = 81.920 alamat, sudah lebih besar dari `/16`, belum termasuk sisi R-UTARA yang berisi WiFi Utara `/18`.

Karena itu file Packet Tracer dibangun dengan skema **VLSM**. Tabel CIDR tetap dibuat di spreadsheet (soal 2) sesuai lembar UTS.

### Tabel VLSM

Diurutkan dari kebutuhan host terbesar. Gateway setiap LAN = host pertama.

| Subnet | Router | Kebutuhan Host | Network ID | Prefix | Subnet Mask | Range Host | Broadcast | Gateway | Kapasitas |
| :--- | :--- | ---: | :--- | :---: | :--- | :--- | :--- | :--- | ---: |
| WiFi Publik Utara | R-UTARA | 11.000 | `10.96.0.0` | /18 | `255.255.192.0` | `10.96.0.1` – `10.96.63.254` | `10.96.63.255` | `10.96.0.1` | 16.382 |
| WiFi Publik Selatan | R-SELATAN | 6.500 | `10.96.64.0` | /19 | `255.255.224.0` | `10.96.64.1` – `10.96.95.254` | `10.96.95.255` | `10.96.64.1` | 8.190 |
| CCTV Kota | R-DC | 6.000 | `10.96.96.0` | /19 | `255.255.224.0` | `10.96.96.1` – `10.96.127.254` | `10.96.127.255` | `10.96.96.1` | 8.190 |
| WiFi Publik Timur | R-TIMUR | 5.500 | `10.96.128.0` | /19 | `255.255.224.0` | `10.96.128.1` – `10.96.159.254` | `10.96.159.255` | `10.96.128.1` | 8.190 |
| Sensor & SCADA Pelabuhan | R-PELABUHAN | 5.000 | `10.96.160.0` | /19 | `255.255.224.0` | `10.96.160.1` – `10.96.191.254` | `10.96.191.255` | `10.96.160.1` | 8.190 |
| WiFi Publik Barat | R-BARAT | 4.500 | `10.96.192.0` | /19 | `255.255.224.0` | `10.96.192.1` – `10.96.223.254` | `10.96.223.255` | `10.96.192.1` | 8.190 |
| Pegawai Balai Kota | R-PUSAT | 1.800 | `10.96.224.0` | /21 | `255.255.248.0` | `10.96.224.1` – `10.96.231.254` | `10.96.231.255` | `10.96.224.1` | 2.046 |
| Sensor Banjir Timur | R-TIMUR | 700 | `10.96.232.0` | /22 | `255.255.252.0` | `10.96.232.1` – `10.96.235.254` | `10.96.235.255` | `10.96.232.1` | 1.022 |
| Sensor Lalu Lintas Utara | R-UTARA | 450 | `10.96.236.0` | /23 | `255.255.254.0` | `10.96.236.1` – `10.96.237.254` | `10.96.237.255` | `10.96.236.1` | 510 |
| Puskesmas & Sekolah Barat | R-BARAT | 220 | `10.96.238.0` | /24 | `255.255.255.0` | `10.96.238.1` – `10.96.238.254` | `10.96.238.255` | `10.96.238.1` | 254 |
| Server & Storage DC | R-DC | 180 | `10.96.239.0` | /24 | `255.255.255.0` | `10.96.239.1` – `10.96.239.254` | `10.96.239.255` | `10.96.239.1` | 254 |
| Kantor Pelabuhan | R-PELABUHAN | 120 | `10.96.240.0` | /25 | `255.255.255.128` | `10.96.240.1` – `10.96.240.126` | `10.96.240.127` | `10.96.240.1` | 126 |
| PJU Pintar Selatan | R-SELATAN | 90 | `10.96.240.128` | /25 | `255.255.255.128` | `10.96.240.129` – `10.96.240.254` | `10.96.240.255` | `10.96.240.129` | 126 |
| Server Balai Kota | R-PUSAT | 20 | `10.96.241.0` | /27 | `255.255.255.224` | `10.96.241.1` – `10.96.241.30` | `10.96.241.31` | `10.96.241.1` | 30 |

### Link Backbone & Alamat Tambahan

Link antar-router memakai `/30`. Konvensi: **router atas = host pertama, router bawah = host kedua**.

| Link | Pasangan | Network | Router Atas | Router Bawah | Broadcast |
| :--- | :--- | :--- | :--- | :--- | :--- |
| L1 | R-PUSAT – R-DC | `10.96.241.32/30` | `10.96.241.33` | `10.96.241.34` | `10.96.241.35` |
| L2 | R-PUSAT – R-UTARA | `10.96.241.36/30` | `10.96.241.37` | `10.96.241.38` | `10.96.241.39` |
| L3 | R-PUSAT – R-SELATAN | `10.96.241.40/30` | `10.96.241.41` | `10.96.241.42` | `10.96.241.43` |
| L4 | R-PUSAT – R-TIMUR | `10.96.241.44/30` | `10.96.241.45` | `10.96.241.46` | `10.96.241.47` |
| L5 | R-PUSAT – R-BARAT | `10.96.241.48/30` | `10.96.241.49` | `10.96.241.50` | `10.96.241.51` |
| L6 | R-UTARA – R-PELABUHAN | `10.96.241.52/30` | `10.96.241.53` | `10.96.241.54` | `10.96.241.55` |

Alamat yang diberikan soal (K = 12, tidak dihitung saat UTS):

| Keperluan | Alamat |
| :--- | :--- |
| Link R-PUSAT – R-ISP | `203.0.113.48/30` (4 × 12 = 48) → R-ISP `.49`, R-PUSAT `.50` |
| Server internet | `8.8.8.0/24` → SRV-INTERNET `8.8.8.8`, gateway `8.8.8.1` di R-ISP |
| Tunnel GRE | `192.168.12.0/30` → R-PUSAT `.1`, R-PELABUHAN `.2` |

Alamat terakhir yang terpakai adalah `10.96.241.55`. Link cadangan L7 untuk file OSPF memakai `10.96.241.56/30`.

### Tabel Port Router

Semua router **Cisco 2911**. R-PUSAT dipasangi 3 modul **HWIC-2T** (slot 0/0, 0/1, 0/2), router lain 1 modul HWIC-2T (slot 0/0), R-ISP tanpa modul. Ujung **DCE** serial selalu di router atas.

| Router | Port | Terhubung ke | IP Address |
| :--- | :--- | :--- | :--- |
| R-ISP | G0/0 | R-PUSAT G0/0 (cross-over) | `203.0.113.49/30` |
| | G0/1 | SRV-INTERNET (cross-over) | `8.8.8.1/24` |
| R-PUSAT | G0/0 | R-ISP G0/0 | `203.0.113.50/30` |
| | G0/1 | SW-PEGAWAI G0/1 | `10.96.224.1/21` |
| | G0/2 | SW-SERVER-BALAIKOTA G0/1 | `10.96.241.1/27` |
| | S0/0/0 (DCE) | L1 → R-DC S0/0/0 | `10.96.241.33/30` |
| | S0/0/1 (DCE) | L2 → R-UTARA S0/0/0 | `10.96.241.37/30` |
| | S0/1/0 (DCE) | L3 → R-SELATAN S0/0/0 | `10.96.241.41/30` |
| | S0/1/1 (DCE) | L4 → R-TIMUR S0/0/0 | `10.96.241.45/30` |
| | S0/2/0 (DCE) | L5 → R-BARAT S0/0/0 | `10.96.241.49/30` |
| | S0/2/1 | cadangan (shutdown) | — |
| R-DC | S0/0/0 | L1 → R-PUSAT | `10.96.241.34/30` |
| | G0/0 | SW-CCTV G0/1 | `10.96.96.1/19` |
| | G0/1 | SW-SERVER-DC G0/1 | `10.96.239.1/24` |
| R-UTARA | S0/0/0 | L2 → R-PUSAT | `10.96.241.38/30` |
| | S0/0/1 (DCE) | L6 → R-PELABUHAN S0/0/0 | `10.96.241.53/30` |
| | G0/0 | SW-WIFI-UTARA G0/1 | `10.96.0.1/18` |
| | G0/1 | SW-SENSOR-LALIN G0/1 | `10.96.236.1/23` |
| R-SELATAN | S0/0/0 | L3 → R-PUSAT | `10.96.241.42/30` |
| | G0/0 | SW-WIFI-SELATAN G0/1 | `10.96.64.1/19` |
| | G0/1 | SW-PJU G0/1 | `10.96.240.129/25` |
| R-TIMUR | S0/0/0 | L4 → R-PUSAT | `10.96.241.46/30` |
| | G0/0 | SW-WIFI-TIMUR G0/1 | `10.96.128.1/19` |
| | G0/1 | SW-SENSOR-BANJIR G0/1 | `10.96.232.1/22` |
| R-BARAT | S0/0/0 | L5 → R-PUSAT | `10.96.241.50/30` |
| | G0/0 | SW-WIFI-BARAT G0/1 | `10.96.192.1/19` |
| | G0/1 | SW-PUSKESMAS G0/1 | `10.96.238.1/24` |
| R-PELABUHAN | S0/0/0 | L6 → R-UTARA | `10.96.241.54/30` |
| | G0/0 | SW-SCADA G0/1 | `10.96.160.1/19` |
| | G0/1 | SW-KANTOR-PELABUHAN G0/1 | `10.96.240.1/25` |

### Alamat Statis

Di setiap LAN, alamat `.1`–`.10` dikecualikan dari DHCP: `.1` gateway router, `.2` server (jika ada), sisanya cadangan.

| Perangkat | IP | Gateway | DNS | Peran |
| :--- | :--- | :--- | :--- | :--- |
| SRV-DC | `10.96.239.2/24` | `10.96.239.1` | `10.96.239.2` | DHCP server LAN CCTV, DNS & FTP (soal 8) |
| SRV-BALAIKOTA | `10.96.241.2/27` | `10.96.241.1` | `10.96.239.2` | Web server (soal 8) |
| SRV-INTERNET | `8.8.8.8/24` | `8.8.8.1` | — | Server internet |

---

## Panduan Dasar Packet Tracer

Langkah-langkah di bawah dipakai berulang kali di semua soal, jadi dijelaskan sekali di sini.

### A. Membuka jendela perangkat

Klik ikon perangkat di kanvas (contoh ①: **R-PUSAT**). Jendela perangkat terbuka dengan tab *Physical*, *Config*, *CLI* (router) atau *Desktop* (PC/server/smartphone).

![Klik perangkat di kanvas](Screenshot/umum/langkah-klik-perangkat.png)

### B. Membuka CLI router

Di jendela router, klik tab ① **CLI**. Area ② adalah console IOS.
- Jika muncul `Press RETURN to get started.`, tekan **Enter**.
- Jika router baru ditanya `Would you like to enter the initial configuration dialog? [yes/no]:`, ketik `no` lalu Enter.

![Tab CLI router](Screenshot/umum/langkah-tab-cli.png)

### C. Mode-mode di CLI

Perintah IOS hanya bisa diketik di mode yang tepat. Prompt menunjukkan mode yang sedang aktif.

| Prompt | Mode | Cara masuk | Dipakai untuk |
| :--- | :--- | :--- | :--- |
| `R-PUSAT>` | User EXEC | (awal) | `ping` sederhana |
| `R-PUSAT#` | Privileged EXEC | `enable` | semua perintah `show`, `copy` |
| `R-PUSAT(config)#` | Global config | `configure terminal` | `hostname`, `ip route`, `ip dhcp`, `ip nat`, `access-list` |
| `R-PUSAT(config-if)#` | Interface config | `interface <nama>` | `ip address`, `clock rate`, `no shutdown`, `ip nat inside/outside`, `ip helper-address` |
| `R-PUSAT(dhcp-config)#` | DHCP pool config | `ip dhcp pool <nama>` | `network`, `default-router`, `dns-server` |

`exit` naik satu tingkat, `end` langsung kembali ke `R-PUSAT#`.

### D. Menyimpan konfigurasi router

Setelah selesai mengetik konfigurasi, ketik:

```
end
copy running-config startup-config
```

Saat muncul `Destination filename [startup-config]?`, tekan **Enter**. Konfigurasi tersimpan bila muncul ② `[OK]`. Tanpa langkah ini, konfigurasi hilang saat router di-*reload*.

![Menyimpan konfigurasi](Screenshot/umum/cli-simpan-konfigurasi.png)

### E. Menyimpan file Packet Tracer

Tekan **Ctrl+S** atau klik ikon ① **Save** di toolbar. Lakukan setiap selesai satu tahap.

![Simpan file .pkt](Screenshot/umum/langkah-simpan-file.png)

### F. Membuka Command Prompt PC / Server / Smartphone

Klik perangkat → tab ① **Desktop** → ② **Command Prompt**. (③ **IP Configuration** dipakai di soal 4.) Pada smartphone dan server, Command Prompt adalah ikon ketiga.

![Membuka Command Prompt](Screenshot/soal-3/langkah-buka-command-prompt.png)

---

## Soal 1

>Dikerjakan Oleh Zaki

### Soal
> Bangun topologi di Cisco Packet Tracer sesuai skema yang dipilih (VLSM atau CIDR). Hubungan antar-router (backbone, link L1–L6), termasuk R-ISP dan server 8.8.8.8, wajib mengikuti Gambar 1. Desain setiap lokasi dirancang sendiri (susunan switch, jenis perangkat, media link, penambahan port router) beserta alasannya. Cukup pasang 1 perangkat perwakilan di setiap LAN. Ganti nama bawaan setiap perangkat dan tambahkan label di setiap LAN berisi nama subnet, Network ID/prefix, dan jumlah host.

### Langkah Pengerjaan dan Implementasi

#### Langkah 1 – Memilih perangkat (dan alasannya)

| Bagian | Perangkat | Alasan |
| :--- | :--- | :--- |
| Semua router (8) | **Cisco 2911** + modul **HWIC-2T** | Mengikuti modul praktikum (*06-Tutorial CPT*). Router-PT (IOS 12.2) **tidak punya `interface Tunnel`** dan perintah `crypto` hanya berisi `key`, sehingga GRE (soal 6) dan IPsec (bonus) mustahil. 2911 (IOS 15.1) mendukung GRE bawaan (`ipbasek9`) dan IPsec setelah lisensi `securityk9` aktif. |
| Backbone L1–L6 | Kabel **Serial DCE** | Link antarwilayah kota dimodelkan sebagai link WAN serial seperti di modul. Ujung DCE (pemberi clock) di router atas: R-PUSAT untuk L1–L5, R-UTARA untuk L6. |
| R-PUSAT | 3 × HWIC-2T + 3 port GE bawaan | Butuh 8 interface: 5 serial ke router wilayah (S0/0/0–S0/2/0, S0/2/1 cadangan), G0/0 ke ISP, G0/1 dan G0/2 ke dua LAN Balai Kota. |
| Router wilayah | 1 × HWIC-2T | S0/0/0 ke router atas, G0/0 dan G0/1 ke dua LAN. R-UTARA memakai S0/0/1 untuk L6. |
| Link ke ISP & SRV-INTERNET | Copper Cross-Over | Router–router dan router–server adalah perangkat sejenis (kelompok A). |
| Setiap LAN | 1 switch **2960-24TT** | Router → switch lewat port Gigabit `G0/1` (kabel straight). Switch membuat LAN bisa diperluas ke ribuan host sesuai kebutuhan, walau yang dipasang hanya 1 perwakilan. |
| WiFi Publik (Barat, Utara, Timur, Selatan) | **AccessPoint-PT** + **Smartphone** (wireless) | Pengguna WiFi publik adalah perangkat bergerak. Tiap AP diberi **SSID sendiri** supaya smartphone tidak tersambung ke AP wilayah lain (dengan SSID bawaan `Default`, HP-WIFI-UTARA dan HP-WIFI-SELATAN sempat tersambung ke AP-WIFI-TIMUR). |
| CCTV, Sensor Lalin, Sensor Banjir, SCADA, PJU | **PC-PT** (kabel) | Mewakili kamera/sensor/gateway IoT yang dipasang tetap dan butuh koneksi stabil, sehingga memakai kabel. |
| Pegawai, Puskesmas & Sekolah, Kantor Pelabuhan | **PC-PT** (kabel) | Komputer kerja di gedung. |
| Server Balai Kota, Server & Storage DC, Internet | **Server-PT** (IP statis) | Menjalankan layanan (Web, DNS, DHCP, FTP, email). |

#### Langkah 2 – Meletakkan router 2911 dan memasang modul HWIC-2T

1. Klik **Network Devices → Routers → 2911**, seret ke workspace. Ulangi sampai ada 8 router.
2. Klik router → tab **Physical**.
3. ① Klik tombol power sampai lampu hijau padam (modul hanya bisa dipasang saat router mati).
4. ② Klik **HWIC-2T** di daftar MODULES, lalu seret ke slot kosong ③ di panel belakang.
5. Ulangi: **R-PUSAT 3 modul**, router wilayah 1 modul, R-ISP tidak perlu.
6. Klik tombol power lagi untuk menyalakan router.

![Langkah memasang modul HWIC-2T](Screenshot/soal-1/langkah-modul-hwic2t.png)
*① tombol power, ② modul HWIC-2T, ③ tiga slot HWIC di R-PUSAT yang sudah terisi (menghasilkan Serial0/0/0 sampai Serial0/2/1).*

#### Langkah 3 – Memasang kabel

1. Klik ① **Connections** (ikon petir) di kiri bawah.
2. Pilih kabel. Nama kabel muncul di bawah daftar saat kursor diarahkan ke ikonnya:
   - ② **Serial DCE** (petir merah berjam) → untuk L1–L6. **Klik router atas dulu** (R-PUSAT, atau R-UTARA untuk L6) agar ujung DCE ada di router atas.
   - Kotak kuning kiri **Copper Straight-Through** → router ke switch, switch ke PC/server/AP.
   - Kotak kuning kanan **Copper Cross-Over** → R-PUSAT ↔ R-ISP dan R-ISP ↔ SRV-INTERNET.
3. Klik perangkat pertama → pilih port → klik perangkat kedua → pilih port, sesuai [Tabel Port Router](#tabel-port-router).

![Langkah memilih kabel](Screenshot/soal-1/langkah-pilih-kabel.png)

#### Langkah 4 – Mengganti nama perangkat

Klik perangkat → tab **Config** → ① **Settings** → ② ketik nama di **Display Name** (router/switch juga di **Hostname**). Nama mengikuti lokasi: `R-PUSAT`, `SW-PEGAWAI`, `PC-CCTV-KOTA`, `SRV-DC`, `AP-WIFI-BARAT`, `HP-WIFI-BARAT`, dst. Tiga *Power Distribution Device* yang dibuat otomatis oleh Packet Tracer 9 (di luar kanvas) diganti menjadi `PDU-1`–`PDU-3`.

![Langkah rename dan hostname](Screenshot/soal-1/langkah-rename-hostname.png)

#### Langkah 5 – Konfigurasi IP R-PUSAT lewat CLI

**5.1 Buka CLI R-PUSAT.** Klik ikon **R-PUSAT** → tab **CLI** → tekan Enter (lihat [Panduan A–B](#a-membuka-jendela-perangkat)).

**5.2 Masuk mode konfigurasi dan beri nama router.** Ketik satu per satu:

```
enable
configure terminal
hostname R-PUSAT
no ip domain-lookup
```

| Perintah | Fungsi | Prompt setelahnya |
| :--- | :--- | :--- |
| `enable` | masuk mode privileged | `R-PUSAT#` |
| `configure terminal` | masuk mode konfigurasi global | `R-PUSAT(config)#` |
| `hostname R-PUSAT` | memberi nama router di CLI | `R-PUSAT(config)#` |
| `no ip domain-lookup` | mencegah CLI "macet" mencari DNS saat salah ketik | `R-PUSAT(config)#` |

**5.3 Isi IP ketiga port Gigabit.** Untuk setiap port: masuk interface → beri IP → nyalakan.

```
interface g0/0
ip address 203.0.113.50 255.255.255.252
no shutdown
interface g0/1
ip address 10.96.224.1 255.255.248.0
no shutdown
interface g0/2
ip address 10.96.241.1 255.255.255.224
no shutdown
```

| Perintah | Fungsi |
| :--- | :--- |
| `interface g0/0` | memilih port GigabitEthernet0/0 (ke R-ISP), prompt menjadi `(config-if)#` |
| `ip address 203.0.113.50 255.255.255.252` | IP publik R-PUSAT di link ISP |
| `no shutdown` | menyalakan port (port router mati secara bawaan) |
| `interface g0/1` + IP `10.96.224.1/21` | gateway LAN Pegawai Balai Kota |
| `interface g0/2` + IP `10.96.241.1/27` | gateway LAN Server Balai Kota |

Hasil di CLI:

![CLI R-PUSAT mode konfigurasi dan Gigabit](Screenshot/soal-1/cli-pusat-1-masuk-config-dan-gigabit.png)
*① masuk mode konfigurasi, ② hostname dan `no ip domain-lookup`, ③ IP dan `no shutdown` ketiga port Gigabit.*

**5.4 Isi IP kelima port serial dan clock rate.** R-PUSAT adalah ujung **DCE**, jadi setiap serial diberi `clock rate 64000`.

```
interface s0/0/0
ip address 10.96.241.33 255.255.255.252
clock rate 64000
no shutdown
```

Ulangi pola yang sama untuk port lain:

| Interface | Link | IP Address |
| :--- | :--- | :--- |
| `s0/0/0` | L1 → R-DC | `10.96.241.33 255.255.255.252` |
| `s0/0/1` | L2 → R-UTARA | `10.96.241.37 255.255.255.252` |
| `s0/1/0` | L3 → R-SELATAN | `10.96.241.41 255.255.255.252` |
| `s0/1/1` | L4 → R-TIMUR | `10.96.241.45 255.255.255.252` |
| `s0/2/0` | L5 → R-BARAT | `10.96.241.49 255.255.255.252` |

![CLI R-PUSAT serial dan clock rate](Screenshot/soal-1/cli-pusat-2-serial-clockrate.png)
*① contoh lengkap Serial0/0/0, ② empat serial lain dengan pola yang sama.*

**5.5 Simpan.** `end` → `copy running-config startup-config` → Enter (lihat [Panduan D](#d-menyimpan-konfigurasi-router)).

> **Alternatif GUI:** tab **Config** → ① pilih interface → ② centang **Port Status On** → ③ **Clock Rate 64000** (hanya DCE) → ④ isi IPv4 Address dan Subnet Mask. Kotak ⑤ menunjukkan perintah IOS setara yang dibuat otomatis. Catatan: mengklik interface di tab Config membuat console masuk mode `(config-if)#`, jadi ketik `end` sebelum memakai perintah `show` di CLI.
>
> ![Langkah IP serial dan clock rate GUI](Screenshot/soal-1/langkah-ip-serial-clockrate.png)

#### Langkah 6 – Konfigurasi IP R-UTARA (DCE untuk L6)

R-UTARA adalah DTE di L2 (ke R-PUSAT) tetapi **DCE di L6** (ke R-PELABUHAN), jadi hanya `s0/0/1` yang diberi `clock rate`.

Buka CLI R-UTARA, lalu ketik:

```
enable
configure terminal
hostname R-UTARA
no ip domain-lookup
interface g0/0
ip address 10.96.0.1 255.255.192.0
no shutdown
interface g0/1
ip address 10.96.236.1 255.255.254.0
no shutdown
interface s0/0/0
ip address 10.96.241.38 255.255.255.252
no shutdown
interface s0/0/1
ip address 10.96.241.53 255.255.255.252
clock rate 64000
no shutdown
```

![CLI R-UTARA](Screenshot/soal-1/cli-utara-interface.png)
*① S0/0/0 ke R-PUSAT (DTE, tanpa clock rate), ② S0/0/1 ke R-PELABUHAN (DCE, dengan `clock rate 64000`).*

Simpan dengan `end` → `copy running-config startup-config`.

#### Langkah 7 – Konfigurasi IP router wilayah (contoh R-DC)

Router wilayah adalah sisi **DTE**, jadi tidak perlu `clock rate`. Buka CLI R-DC, lalu ketik:

```
enable
configure terminal
hostname R-DC
no ip domain-lookup
interface g0/0
ip address 10.96.96.1 255.255.224.0
no shutdown
interface g0/1
ip address 10.96.239.1 255.255.255.0
no shutdown
interface s0/0/0
ip address 10.96.241.34 255.255.255.252
no shutdown
```

![CLI R-DC](Screenshot/soal-1/cli-dc-interface.png)
*① hostname dan `no ip domain-lookup`, ② IP ketiga interface.*

Router wilayah lain memakai pola yang sama dengan nilai berikut:

| Router | `interface g0/0` | `interface g0/1` | `interface s0/0/0` |
| :--- | :--- | :--- | :--- |
| R-SELATAN | `10.96.64.1 255.255.224.0` | `10.96.240.129 255.255.255.128` | `10.96.241.42 255.255.255.252` |
| R-TIMUR | `10.96.128.1 255.255.224.0` | `10.96.232.1 255.255.252.0` | `10.96.241.46 255.255.255.252` |
| R-BARAT | `10.96.192.1 255.255.224.0` | `10.96.238.1 255.255.255.0` | `10.96.241.50 255.255.255.252` |
| R-PELABUHAN | `10.96.160.1 255.255.224.0` | `10.96.240.1 255.255.255.128` | `10.96.241.54 255.255.255.252` |

Simpan setiap router dengan `end` → `copy running-config startup-config`.

#### Langkah 8 – Konfigurasi IP R-ISP dan SRV-INTERNET

R-ISP (CLI):

```
enable
configure terminal
hostname R-ISP
no ip domain-lookup
interface g0/0
ip address 203.0.113.49 255.255.255.252
no shutdown
interface g0/1
ip address 8.8.8.1 255.255.255.0
no shutdown
end
copy running-config startup-config
```

SRV-INTERNET (GUI): Desktop → IP Configuration → Static → IP `8.8.8.8`, mask `255.255.255.0`, gateway `8.8.8.1` (caranya sama dengan [soal 4 langkah 1](#langkah-1--ip-statis-server)).

#### Langkah 9 – SSID per Access Point

1. Klik AP → tab **Config** → ① **Port 1** → ② ganti **SSID** (misalnya `WIFI-BARAT`), tekan Enter. ③ Authentication dibiarkan *Disabled* (WiFi publik).

   ![Langkah SSID Access Point](Screenshot/soal-1/langkah-ssid-ap.png)

2. Klik smartphone wilayah yang sama → tab **Config** → ① **Wireless0** → ② isi SSID yang sama. ③ IP Configuration tetap **DHCP**; setelah tersambung ke AP yang benar, smartphone langsung mendapat IP dari subnet wilayahnya.

   ![Langkah SSID Smartphone](Screenshot/soal-1/langkah-ssid-hp.png)

| Access Point | Smartphone | SSID |
| :--- | :--- | :--- |
| AP-WIFI-BARAT | HP-WIFI-BARAT | `WIFI-BARAT` |
| AP-WIFI-UTARA | HP-WIFI-UTARA | `WIFI-UTARA` |
| AP-WIFI-TIMUR | HP-WIFI-TIMUR | `WIFI-TIMUR` |
| AP-WIFI-SELATAN | HP-WIFI-SELATAN | `WIFI-SELATAN` |

#### Langkah 10 – Label topologi

Klik ① **Place Note** (atau tekan **N**) di toolbar, klik area kanvas, lalu tulis nama subnet, network ID/prefix, dan jumlah host. Total 23 label: 14 LAN, L1–L6, link ISP, server internet, dan tunnel GRE.

![Langkah Place Note](Screenshot/soal-1/langkah-place-note-label.png)

### Bukti dan Hasil

1. **Topologi lengkap**: lihat [gambar di bagian Ringkasan](#ringkasan-topologi--tabel-pengalamatan-ip). Backbone sama dengan Gambar 1 soal, semua perangkat sudah diganti namanya, dan setiap LAN berlabel.

2. **Semua interface R-PUSAT aktif**
   - **Node**: `R-PUSAT` → tab CLI → `show ip interface brief`

   ![show ip interface brief R-PUSAT](Screenshot/soal-1/bukti-ip-int-brief-pusat.png)
   *Ketiga port Gigabit dan kelima serial berstatus `up/up` dengan IP sesuai tabel. S0/2/1 sengaja `administratively down` karena cadangan.*

3. **Kabel tersambung ke port yang benar**
   - **Node**: `R-PUSAT` → `show cdp neighbors`

   ![show cdp neighbors R-PUSAT](Screenshot/soal-1/bukti-cdp-neighbors-pusat.png)
   *R-DC di Ser 0/0/0, R-UTARA di Ser 0/0/1, R-SELATAN di Ser 0/1/0, R-TIMUR di Ser 0/1/1, R-BARAT di Ser 0/2/0, R-ISP di Gig 0/0, dan dua switch LAN di Gig 0/1 dan Gig 0/2.*

4. **Sisi DCE dan clock rate benar** (`show controllers serial0/0/0`, `show controllers serial0/0/1`):
   ```
   R-PUSAT  Serial0/0/0 : DCE V.35, clock rate 64000
   R-PUSAT  Serial0/2/0 : DCE V.35, clock rate 64000
   R-UTARA  Serial0/0/1 : DCE V.35, clock rate 64000
   R-DC     Serial0/0/0 : DTE V.35 TX and RX clocks detected
   ```

5. **Pemeriksaan menyeluruh topologi**: tidak ada link yang mati (semua titik link hijau) dan tidak ada IP ganda. File `.pkt` berisi 36 kabel dan 23 label.

---

## Soal 2

>Dikerjakan Oleh Anggota C

### Soal
> Menggunakan VLSM dan CIDR, buat tabel di spreadsheet berisi nama subnet, network ID, prefix, rentang host, alamat broadcast, dan gateway untuk setiap subnet sesuai lembar jawaban UTS. Perbaikan dari UTS wajib dicatat di tabel revisi.

*(Menyusul: tabel VLSM, tabel CIDR, supernet per router, dan tabel revisi di file `.xlsx`.)*

---

## Soal 3

>Dikerjakan Oleh Zaki

### Soal
> Seluruh LAN dapat saling berkomunikasi hanya dengan static route (dan default route untuk arah keluar). Default route hanya boleh mengarah ke router di atasnya / keluar. R-PUSAT wajib punya static route ke semua LAN milik R-DC, R-UTARA, R-PELABUHAN, R-SELATAN, R-TIMUR, dan R-BARAT. R-UTARA wajib punya static route ke semua LAN milik R-PELABUHAN.

### Langkah Pengerjaan dan Implementasi

#### Konsep

- **Router wilayah (R-DC, R-BARAT, R-TIMUR, R-SELATAN) dan R-PELABUHAN adalah *stub***: hanya punya satu jalan keluar, yaitu router di atasnya. Semua tujuan selain LAN-nya sendiri cukup dikirim lewat **satu default route** ke atas.
- **R-UTARA** punya dua arah: ke atas (R-PUSAT) dan ke bawah (R-PELABUHAN). Default route ke atas, ditambah **2 static route** ke LAN Pelabuhan lewat L6.
- **R-PUSAT** adalah persimpangan. Default route-nya sudah dipakai ke ISP, jadi untuk 12 LAN di bawahnya harus ada **static route spesifik** (+1 rute ke link L6). Tanpa rute spesifik, paket ke LAN wilayah ikut default route ke ISP lalu dibuang. Karena *longest prefix match*, rute `/19`, `/24`, dst. selalu menang atas default `/0`.
- Rute ke link **L6 `10.96.241.52/30`** di R-PUSAT diperlukan agar alamat fisik R-PELABUHAN (`10.96.241.54`) bisa dicapai. Alamat ini nanti menjadi *tunnel destination* GRE (soal 6).
- **Next hop selalu IP router seberang** di link yang bersangkutan.
- **R-ISP sengaja tidak diberi rute** ke `10.96.0.0` (syarat soal 5).

Format perintah: `ip route <network tujuan> <subnet mask> <next hop>`

#### Langkah 1 – Static route di R-PUSAT

1. Buka CLI **R-PUSAT**, lalu masuk mode konfigurasi:
   ```
   enable
   configure terminal
   ```
2. Ketik 13 rute ke bawah, satu baris per jaringan tujuan:

   | Perintah | Tujuan | Lewat |
   | :--- | :--- | :--- |
   | `ip route 10.96.96.0 255.255.224.0 10.96.241.34` | LAN CCTV Kota | R-DC (L1) |
   | `ip route 10.96.239.0 255.255.255.0 10.96.241.34` | LAN Server & Storage DC | R-DC (L1) |
   | `ip route 10.96.0.0 255.255.192.0 10.96.241.38` | LAN WiFi Utara | R-UTARA (L2) |
   | `ip route 10.96.236.0 255.255.254.0 10.96.241.38` | LAN Sensor Lalin | R-UTARA (L2) |
   | `ip route 10.96.241.52 255.255.255.252 10.96.241.38` | link L6 | R-UTARA (L2) |
   | `ip route 10.96.160.0 255.255.224.0 10.96.241.38` | LAN SCADA Pelabuhan | R-UTARA (L2) → R-PELABUHAN |
   | `ip route 10.96.240.0 255.255.255.128 10.96.241.38` | LAN Kantor Pelabuhan | R-UTARA (L2) → R-PELABUHAN |
   | `ip route 10.96.64.0 255.255.224.0 10.96.241.42` | LAN WiFi Selatan | R-SELATAN (L3) |
   | `ip route 10.96.240.128 255.255.255.128 10.96.241.42` | LAN PJU Pintar | R-SELATAN (L3) |
   | `ip route 10.96.128.0 255.255.224.0 10.96.241.46` | LAN WiFi Timur | R-TIMUR (L4) |
   | `ip route 10.96.232.0 255.255.252.0 10.96.241.46` | LAN Sensor Banjir | R-TIMUR (L4) |
   | `ip route 10.96.192.0 255.255.224.0 10.96.241.50` | LAN WiFi Barat | R-BARAT (L5) |
   | `ip route 10.96.238.0 255.255.255.0 10.96.241.50` | LAN Puskesmas & Sekolah | R-BARAT (L5) |

3. Ketik default route ke internet:
   ```
   ip route 0.0.0.0 0.0.0.0 203.0.113.49
   ```
4. Simpan: `end` → `copy running-config startup-config` → Enter.

![CLI static route R-PUSAT](Screenshot/soal-3/cli-pusat-static-route.png)
*① 13 static route ke bawah, ② default route ke R-ISP.*

#### Langkah 2 – Static route di R-UTARA

Buka CLI **R-UTARA** → `enable` → `configure terminal`, lalu ketik:

```
ip route 0.0.0.0 0.0.0.0 10.96.241.37
ip route 10.96.160.0 255.255.224.0 10.96.241.54
ip route 10.96.240.0 255.255.255.128 10.96.241.54
```

| Perintah | Fungsi |
| :--- | :--- |
| `ip route 0.0.0.0 0.0.0.0 10.96.241.37` | default route ke atas (R-PUSAT) |
| `ip route 10.96.160.0 255.255.224.0 10.96.241.54` | LAN SCADA lewat R-PELABUHAN |
| `ip route 10.96.240.0 255.255.255.128 10.96.241.54` | LAN Kantor Pelabuhan lewat R-PELABUHAN |

![CLI static route R-UTARA](Screenshot/soal-3/cli-utara-static-route.png)
*① default route ke R-PUSAT, ② dua rute ke LAN Pelabuhan.*

Simpan: `end` → `copy running-config startup-config`.

#### Langkah 3 – Default route di router stub (contoh R-DC)

Buka CLI **R-DC** → `enable` → `configure terminal`, lalu ketik:

```
ip route 0.0.0.0 0.0.0.0 10.96.241.33
```

![CLI default route R-DC](Screenshot/soal-3/cli-dc-default-route.png)

Router stub lainnya:

| Router | Perintah |
| :--- | :--- |
| R-SELATAN | `ip route 0.0.0.0 0.0.0.0 10.96.241.41` |
| R-TIMUR | `ip route 0.0.0.0 0.0.0.0 10.96.241.45` |
| R-BARAT | `ip route 0.0.0.0 0.0.0.0 10.96.241.49` |
| R-PELABUHAN | `ip route 0.0.0.0 0.0.0.0 10.96.241.53` (ke R-UTARA) |

Simpan setiap router: `end` → `copy running-config startup-config`.

> **Alternatif GUI:** tab **Config** → ① **Static** → ② isi Network, Mask, Next Hop → ③ **Add**. Daftar ④ berisi rute yang sudah terpasang.
>
> ![Langkah static route GUI](Screenshot/soal-3/langkah-static-route-gui.png)

### Bukti dan Hasil

#### Tabel routing

1. **R-PUSAT** – `show ip route static`

   ![Route R-PUSAT](Screenshot/soal-3/bukti-route-static-pusat.png)
   *① 13 static route ke 12 LAN wilayah dan link L6, ② default route ke R-ISP `203.0.113.49`.*

2. **R-UTARA** – `show ip route static`

   ![Route R-UTARA](Screenshot/soal-3/bukti-route-static-utara.png)
   *① rute ke LAN SCADA dan Kantor Pelabuhan lewat `10.96.241.54`, ② default ke atas `10.96.241.37`.*

3. **R-DC (router stub)** – `show ip route`

   ![Route R-DC](Screenshot/soal-3/bukti-route-dc.png)
   *① hanya jaringan yang terhubung langsung, ② satu default route ke R-PUSAT `10.96.241.33`.*

#### Uji koneksi (Command Prompt, lihat [Panduan F](#f-membuka-command-prompt-pc--server--smartphone))

Pada setiap gambar: ① balasan `Reply from ...`, ② statistik `0% loss`.

| No | Dari | Ke | Jalur | Hasil |
| :---: | :--- | :--- | :--- | :---: |
| 1 | PC-CCTV-KOTA | Gateway sendiri `10.96.96.1` | dalam LAN | ✅ |
| 2 | PC-CCTV-KOTA | PC-KANTOR-PELABUHAN `10.96.240.11` | R-DC → R-PUSAT → R-UTARA → R-PELABUHAN | ✅ |
| 3 | PC-KANTOR-PELABUHAN | PC-PEGAWAI `10.96.224.11` | R-PELABUHAN → R-UTARA → R-PUSAT | ✅ |
| 4 | PC-KANTOR-PELABUHAN | SRV-DC `10.96.239.2` | R-PELABUHAN → R-UTARA → R-PUSAT → R-DC | ✅ |
| 5 | PC-PJU | PC-SENSOR-LALIN `10.96.236.11` | R-SELATAN → R-PUSAT → R-UTARA | ✅ |
| 6 | HP-WIFI-BARAT (wireless) | SRV-BALAIKOTA `10.96.241.2` | AP → R-BARAT → R-PUSAT | ✅ |

**Uji 1** – PC-CCTV-KOTA → `ping 10.96.96.1`

![Ping CCTV ke gateway](Screenshot/soal-3/ping-cctv-ke-gateway.png)

**Uji 2** – PC-CCTV-KOTA → `ping 10.96.240.11` (jalur terjauh)

![Ping CCTV ke Kantor Pelabuhan](Screenshot/soal-3/ping-cctv-ke-kantor-pelabuhan.png)
*TTL 124 = 128 − 4: paket melewati 4 router.*

**Uji 3** – PC-KANTOR-PELABUHAN → `ping 10.96.224.11`

![Ping Kantor ke Pegawai](Screenshot/soal-3/ping-kantor-ke-pegawai.png)

**Uji 4** – PC-KANTOR-PELABUHAN → `ping 10.96.239.2`

![Ping Kantor ke SRV-DC](Screenshot/soal-3/ping-kantor-ke-srv-dc.png)

**Uji 5** – PC-PJU → `ping 10.96.236.11`

![Ping PJU ke Sensor Lalin](Screenshot/soal-3/ping-pju-ke-sensor-lalin.png)

**Uji 6** – HP-WIFI-BARAT → `ping 10.96.241.2`

![Ping HP Barat ke SRV-BALAIKOTA](Screenshot/soal-3/ping-hp-barat-ke-srv-balaikota.png)

**Uji 7** – PC-CCTV-KOTA → `tracert 10.96.240.11` (melihat router yang dilewati)

![Tracert CCTV ke Kantor Pelabuhan](Screenshot/soal-3/bukti-tracert-cctv-kantor.png)
*① R-DC (`10.96.96.1`) → R-PUSAT (`10.96.241.33`) → R-UTARA (`10.96.241.38`) → R-PELABUHAN (`10.96.241.54`), ② tujuan `10.96.240.11`.*

**Rekap uji ping ke seluruh gateway LAN** (ping 4 paket dari kedua ujung jaringan):

| Dari | Ke 14 gateway LAN | Hasil |
| :--- | :--- | :---: |
| PC-CCTV-KOTA (R-DC) | `10.96.0.1`, `10.96.64.1`, `10.96.96.1`, `10.96.128.1`, `10.96.160.1`, `10.96.192.1`, `10.96.224.1`, `10.96.232.1`, `10.96.236.1`, `10.96.238.1`, `10.96.239.1`, `10.96.240.1`, `10.96.240.129`, `10.96.241.1` | **14/14** ✅ |
| PC-KANTOR-PELABUHAN (R-PELABUHAN) | 14 gateway yang sama | **14/14** ✅ |

*Ping pertama kadang kehilangan 1 paket karena ARP; ulangi sekali bila muncul `Request timed out` di paket pertama.*

---

## Soal 4

>Dikerjakan Oleh Zaki

### Soal
> Interface router dan seluruh server menggunakan IP statis. Perangkat pengguna mendapat IP dinamis secara otomatis, lengkap dengan gateway dan DNS server, tanpa bentrok dengan alamat IP statis. Khusus subnet CCTV Kota (subnet R-DC) mendapatkan alamat dari server DHCP di LAN Server dan Storage DC, bukan dari router.

### Langkah Pengerjaan dan Implementasi

#### Langkah 1 – IP statis server

1. Klik **SRV-DC** → tab ① **Desktop** → **IP Configuration**.
2. Pilih ② **Static**.
3. ③ Isi IPv4 Address `10.96.239.2`, Subnet Mask `255.255.255.0`, Default Gateway `10.96.239.1`, DNS Server `10.96.239.2`.

![IP statis server](Screenshot/soal-4/langkah-ip-statis-server.png)

Lakukan juga untuk **SRV-BALAIKOTA**: `10.96.241.2`, `255.255.255.224`, gateway `10.96.241.1`, DNS `10.96.239.2`.

#### Langkah 2 – DHCP pool di R-PUSAT (LAN Pegawai)

1. Buka CLI **R-PUSAT** → `enable` → `configure terminal`.
2. Kecualikan alamat statis agar tidak dibagikan DHCP:
   ```
   ip dhcp excluded-address 10.96.224.1 10.96.224.10
   ```
3. Buat pool, lalu isi jaringan, gateway, dan DNS-nya:
   ```
   ip dhcp pool DHCP-PEGAWAI
   network 10.96.224.0 255.255.248.0
   default-router 10.96.224.1
   dns-server 10.96.239.2
   exit
   ```
4. Simpan: `end` → `copy running-config startup-config`.

| Perintah | Fungsi | Prompt setelahnya |
| :--- | :--- | :--- |
| `ip dhcp excluded-address 10.96.224.1 10.96.224.10` | `.1`–`.10` tidak pernah dibagikan (gateway, server, cadangan) | `(config)#` |
| `ip dhcp pool DHCP-PEGAWAI` | membuat pool bernama DHCP-PEGAWAI | `(dhcp-config)#` |
| `network 10.96.224.0 255.255.248.0` | jaringan yang dilayani pool ini | `(dhcp-config)#` |
| `default-router 10.96.224.1` | gateway yang diberikan ke klien | `(dhcp-config)#` |
| `dns-server 10.96.239.2` | DNS yang diberikan ke klien (SRV-DC) | `(dhcp-config)#` |
| `exit` | keluar dari pool | `(config)#` |

![CLI DHCP pool R-PUSAT](Screenshot/soal-4/cli-pusat-dhcp-pool.png)
*① excluded-address, ② pool DHCP-PEGAWAI.*

#### Langkah 3 – DHCP pool di router wilayah (contoh R-UTARA, 2 pool)

Router dengan dua LAN pengguna membuat dua pool. Buka CLI **R-UTARA** → `enable` → `configure terminal`, lalu ketik:

```
ip dhcp excluded-address 10.96.0.1 10.96.0.10
ip dhcp pool DHCP-WIFI-UTARA
network 10.96.0.0 255.255.192.0
default-router 10.96.0.1
dns-server 10.96.239.2
exit
ip dhcp excluded-address 10.96.236.1 10.96.236.10
ip dhcp pool DHCP-SENSOR-LALIN
network 10.96.236.0 255.255.254.0
default-router 10.96.236.1
dns-server 10.96.239.2
exit
```

![CLI DHCP pool R-UTARA](Screenshot/soal-4/cli-utara-dhcp-pool.png)
*① pool WiFi Utara, ② pool Sensor Lalu Lintas.*

Router lain memakai pola yang sama (DNS selalu `10.96.239.2`):

| Router | `ip dhcp excluded-address` | `ip dhcp pool` | `network` | `default-router` |
| :--- | :--- | :--- | :--- | :--- |
| R-BARAT | `10.96.192.1 10.96.192.10` | `DHCP-WIFI-BARAT` | `10.96.192.0 255.255.224.0` | `10.96.192.1` |
| R-BARAT | `10.96.238.1 10.96.238.10` | `DHCP-PUSKESMAS` | `10.96.238.0 255.255.255.0` | `10.96.238.1` |
| R-TIMUR | `10.96.128.1 10.96.128.10` | `DHCP-WIFI-TIMUR` | `10.96.128.0 255.255.224.0` | `10.96.128.1` |
| R-TIMUR | `10.96.232.1 10.96.232.10` | `DHCP-SENSOR-BANJIR` | `10.96.232.0 255.255.252.0` | `10.96.232.1` |
| R-SELATAN | `10.96.64.1 10.96.64.10` | `DHCP-WIFI-SELATAN` | `10.96.64.0 255.255.224.0` | `10.96.64.1` |
| R-SELATAN | `10.96.240.129 10.96.240.138` | `DHCP-PJU` | `10.96.240.128 255.255.255.128` | `10.96.240.129` |
| R-PELABUHAN | `10.96.160.1 10.96.160.10` | `DHCP-SCADA` | `10.96.160.0 255.255.224.0` | `10.96.160.1` |
| R-PELABUHAN | `10.96.240.1 10.96.240.10` | `DHCP-KANTOR-PELABUHAN` | `10.96.240.0 255.255.255.128` | `10.96.240.1` |

LAN Server Balai Kota dan LAN Server & Storage DC **tidak diberi pool** karena isinya server ber-IP statis. LAN CCTV juga **tidak** diberi pool di router (lihat langkah 4).

#### Langkah 4 – DHCP LAN CCTV dari SRV-DC (DHCP relay)

LAN CCTV (`10.96.96.0/19`) dan SRV-DC (`10.96.239.0/24`) berbeda subnet. Broadcast DHCP tidak bisa menyeberang router, jadi **R-DC dijadikan DHCP relay**: permintaan DHCP dari LAN CCTV diteruskan sebagai unicast ke SRV-DC.

**4.1 Relay di R-DC.** Buka CLI **R-DC** → `enable` → `configure terminal`, lalu ketik pada interface yang menghadap LAN CCTV:

```
interface g0/0
ip helper-address 10.96.239.2
```

Simpan: `end` → `copy running-config startup-config`.

![CLI helper-address R-DC](Screenshot/soal-4/cli-dc-helper-address.png)
*① `ip helper-address` di G0/0 (gateway LAN CCTV) menunjuk ke SRV-DC.*

**4.2 Pool di SRV-DC.**
1. Klik **SRV-DC** → tab ① **Services** → ② **DHCP**.
2. ③ Service **On**.
3. ④ Isi Pool Name `DHCP-CCTV`, Default Gateway `10.96.96.1`, DNS Server `10.96.239.2`, Start IP Address `10.96.96.11`, Subnet Mask `255.255.224.0`, Maximum Number of Users `8180`.
4. ⑤ Klik **Add** (atau **Save** bila mengedit). Pool tampil di tabel ⑥.
5. Pool bawaan `serverPool` dihapus (pilih → **Remove**) agar server tidak membagikan alamat lain.

![DHCP server CCTV di SRV-DC](Screenshot/soal-4/langkah-dhcp-server-cctv.png)

#### Langkah 5 – Host pengguna memakai DHCP

1. Klik PC → tab ① **Desktop** → **IP Configuration**.
2. Pilih ② **DHCP**.
3. Tunggu beberapa detik sampai ③ IP, mask, gateway, dan DNS terisi otomatis.

![PC memakai DHCP](Screenshot/soal-4/langkah-pc-dhcp.png)

Smartphone sudah memakai DHCP secara bawaan (Config → Wireless0 → IP Configuration **DHCP**, lihat soal 1 langkah 9).

### Bukti dan Hasil

1. **Relay CCTV bekerja** – PC-CCTV-KOTA → `ipconfig /all`

   ![ipconfig PC-CCTV-KOTA](Screenshot/soal-4/bukti-ipconfig-cctv-relay.png)
   *① IP `10.96.96.11/19` dan gateway `10.96.96.1`, ② **DHCP Servers `10.96.239.2`**: alamat diberikan oleh SRV-DC lewat relay, bukan oleh R-DC, ③ DNS `10.96.239.2`.*

2. **Helper-address tersimpan di R-DC** – `show running-config | include helper`

   ![helper-address R-DC](Screenshot/soal-4/bukti-helper-address-dc.png)

3. **Minta ulang IP dari DHCP (PC kabel)** – PC-SENSOR-BANJIR → `ipconfig /renew`

   ![ipconfig renew Sensor Banjir](Screenshot/soal-4/ipconfig-renew-sensor-banjir.png)
   *① IP `10.96.232.11` (di luar `.1`–`.10`), gateway `10.96.232.1`, DNS `10.96.239.2` dari R-TIMUR.*

4. **Minta ulang IP dari DHCP (smartphone wireless)** – HP-WIFI-TIMUR → `ipconfig /renew`

   ![ipconfig renew HP Timur](Screenshot/soal-4/ipconfig-renew-hp-timur.png)
   *① HP mendapat IP dari subnet WiFi Timur, membuktikan HP tersambung ke AP yang benar (SSID `WIFI-TIMUR`).*

5. **Lease DHCP di router** – R-UTARA → `show ip dhcp binding`

   ![DHCP binding R-UTARA](Screenshot/soal-4/bukti-dhcp-binding-utara.png)
   *HP-WIFI-UTARA mendapat `10.96.0.11`, PC-SENSOR-LALIN `10.96.236.11`.*

6. **Rekap seluruh host DHCP** (diambil dari `ipconfig /all` di setiap host):

   | Host | IP | Gateway | DNS | DHCP Server |
   | :--- | :--- | :--- | :--- | :--- |
   | PC-PEGAWAI | `10.96.224.11` | `10.96.224.1` | `10.96.239.2` | `10.96.224.1` (R-PUSAT) |
   | PC-CCTV-KOTA | `10.96.96.11` | `10.96.96.1` | `10.96.239.2` | **`10.96.239.2` (SRV-DC, relay)** |
   | PC-PUSKESMAS | `10.96.238.12` | `10.96.238.1` | `10.96.239.2` | `10.96.238.1` (R-BARAT) |
   | PC-SENSOR-LALIN | `10.96.236.11` | `10.96.236.1` | `10.96.239.2` | `10.96.236.1` (R-UTARA) |
   | PC-SENSOR-BANJIR | `10.96.232.11` | `10.96.232.1` | `10.96.239.2` | `10.96.232.1` (R-TIMUR) |
   | PC-PJU | `10.96.240.140` | `10.96.240.129` | `10.96.239.2` | `10.96.240.129` (R-SELATAN) |
   | PC-SCADA | `10.96.160.12` | `10.96.160.1` | `10.96.239.2` | `10.96.160.1` (R-PELABUHAN) |
   | PC-KANTOR-PELABUHAN | `10.96.240.11` | `10.96.240.1` | `10.96.239.2` | `10.96.240.1` (R-PELABUHAN) |
   | HP-WIFI-BARAT | `10.96.192.11` | `10.96.192.1` | `10.96.239.2` | `10.96.192.1` (R-BARAT) |
   | HP-WIFI-UTARA | `10.96.0.11` | `10.96.0.1` | `10.96.239.2` | `10.96.0.1` (R-UTARA) |
   | HP-WIFI-TIMUR | `10.96.128.11` | `10.96.128.1` | `10.96.239.2` | `10.96.128.1` (R-TIMUR) |
   | HP-WIFI-SELATAN | `10.96.64.11` | `10.96.64.1` | `10.96.239.2` | `10.96.64.1` (R-SELATAN) |

   *Ke-12 host mendapat IP di subnet yang benar, di luar rentang statis `.1`–`.10`, lengkap dengan gateway (host pertama) dan DNS `10.96.239.2`.*

---

## Soal 5

>Dikerjakan Oleh Zaki

### Soal
> Seluruh host dapat mengakses server 8.8.8.8 menggunakan satu IP publik milik R-PUSAT. R-ISP tidak boleh memiliki rute ke alamat privat internal.

### Langkah Pengerjaan dan Implementasi

#### Konsep

Tanpa NAT, paket dari PC sampai ke 8.8.8.8, tetapi balasannya ke alamat `10.96.x.x` dibuang di R-ISP karena R-ISP tidak punya rute ke alamat privat. Solusinya **PAT (NAT overload)** di R-PUSAT: semua alamat `10.96.0.0/16` diterjemahkan ke satu IP publik `203.0.113.50` (G0/0), dibedakan dengan nomor port.

- **inside** = semua interface R-PUSAT yang menghadap jaringan internal (2 LAN + 5 serial).
- **outside** = interface ke ISP (G0/0).
- R-ISP **tidak diubah sama sekali**.

#### Langkah 1 – Buka CLI R-PUSAT dan masuk mode konfigurasi

```
enable
configure terminal
```

#### Langkah 2 – Tentukan alamat yang boleh di-NAT

```
access-list 1 permit 10.96.0.0 0.0.255.255
```

Wildcard `0.0.255.255` mencocokkan seluruh `10.96.0.0/16` dalam satu baris.

#### Langkah 3 – Tandai interface inside

Masuk ke setiap interface yang menghadap jaringan internal dan ketik `ip nat inside`:

```
interface g0/1
ip nat inside
interface g0/2
ip nat inside
interface s0/0/0
ip nat inside
interface s0/0/1
ip nat inside
interface s0/1/0
ip nat inside
interface s0/1/1
ip nat inside
interface s0/2/0
ip nat inside
```

| Interface | Menghadap |
| :--- | :--- |
| `g0/1` | LAN Pegawai Balai Kota |
| `g0/2` | LAN Server Balai Kota |
| `s0/0/0` | L1 → R-DC |
| `s0/0/1` | L2 → R-UTARA (termasuk R-PELABUHAN) |
| `s0/1/0` | L3 → R-SELATAN |
| `s0/1/1` | L4 → R-TIMUR |
| `s0/2/0` | L5 → R-BARAT |

#### Langkah 4 – Tandai interface outside

```
interface g0/0
ip nat outside
exit
```

#### Langkah 5 – Aktifkan PAT

```
ip nat inside source list 1 interface g0/0 overload
```

Artinya: alamat yang cocok dengan `access-list 1` diterjemahkan ke IP milik `g0/0`, dan `overload` membuat banyak host berbagi satu IP itu dengan nomor port berbeda.

![CLI NAT R-PUSAT](Screenshot/soal-5/cli-pusat-nat.png)
*① access-list, ② tujuh interface inside, ③ G0/0 outside, ④ PAT overload.*

#### Langkah 6 – Simpan

`end` → `copy running-config startup-config` → Enter.

### Bukti dan Hasil

1. **Konfigurasi NAT tersimpan** – R-PUSAT → `show running-config | include nat`

   ![Konfigurasi NAT](Screenshot/soal-5/bukti-config-nat-pusat.png)
   *① G0/0 outside, ② tujuh interface inside, ③ PAT `overload` memakai IP G0/0.*

2. **Host internal bisa mengakses 8.8.8.8** (setiap gambar: ① balasan dari 8.8.8.8, ② `0% loss`)

   **Uji 1** – PC-PEGAWAI (LAN Balai Kota) → `ping 8.8.8.8`

   ![Ping internet Pegawai](Screenshot/soal-5/bukti-ping-internet-pegawai.png)

   **Uji 2** – PC-CCTV-KOTA (LAN R-DC) → `ping 8.8.8.8`

   ![Ping internet CCTV](Screenshot/soal-5/ping-internet-cctv.png)

   **Uji 3** – PC-KANTOR-PELABUHAN (LAN terjauh, lewat R-UTARA) → `ping 8.8.8.8`

   ![Ping internet Kantor Pelabuhan](Screenshot/soal-5/ping-internet-kantor.png)

   **Uji 4** – HP-WIFI-SELATAN (wireless) → `ping 8.8.8.8`

   ![Ping internet HP Selatan](Screenshot/soal-5/ping-internet-hp-selatan.png)

3. **Tabel translasi NAT** – R-PUSAT → `show ip nat translations` (setelah ping dari PC-SCADA dan HP-WIFI-TIMUR)

   ![NAT translations](Screenshot/soal-5/bukti-nat-translations.png)
   *Kolom merah (Inside global) = satu IP publik `203.0.113.50` dengan port berbeda; kolom kuning (Inside local) = alamat privat asli `10.96.160.12` (PC-SCADA) dan `10.96.128.11` (HP-WIFI-TIMUR).*

4. **R-ISP tidak punya rute ke alamat privat** – R-ISP → `show ip route`

   ![Route R-ISP](Screenshot/soal-5/bukti-route-isp.png)
   *① tidak ada default route, ② hanya `8.8.8.0/24` dan `203.0.113.48/30`, tidak ada `10.96.x.x`.*

5. **Uji negatif: internet tidak bisa menjangkau alamat privat** – SRV-INTERNET → `ping 10.96.224.11`

   ![Ping dari internet ke alamat privat gagal](Screenshot/soal-5/ping-internet-ke-privat-gagal.png)
   *① R-ISP (`8.8.8.1`) membalas `Destination host unreachable` karena tidak punya rute ke `10.96.0.0`, ② 100% loss. Hasil ini **memang seharusnya gagal**: jaringan internal tidak terekspos ke internet.*

6. **Rekap uji akses internet** (ping 4 paket ke `8.8.8.8`):

   | Sumber | Hasil | | Sumber | Hasil |
   | :--- | :---: | :--- | :--- | :---: |
   | PC-PEGAWAI | ✅ | | HP-WIFI-BARAT | ✅ |
   | PC-CCTV-KOTA | ✅ | | HP-WIFI-UTARA | ✅ |
   | PC-PUSKESMAS | ✅ | | HP-WIFI-TIMUR | ✅ |
   | PC-SENSOR-LALIN | ✅ | | HP-WIFI-SELATAN | ✅ |
   | PC-SENSOR-BANJIR | ✅ | | SRV-DC | ✅ |
   | PC-PJU | ✅ | | SRV-BALAIKOTA | ✅ |
   | PC-SCADA | ✅ | | | |
   | PC-KANTOR-PELABUHAN | ✅ | | **Total** | **14/14** |

---

## Soal 6

>Dikerjakan Oleh 

### Soal
> Bangun tunnel GRE antara R-PUSAT dan R-PELABUHAN. Lalu lintas antara subnet Balai Kota dan subnet Pelabuhan harus melewati tunnel. Jelaskan mengapa router memilih jalur tunnel dan mengapa GRE saja tidak mengenkripsi data. Bonus (+5): enkripsi IPsec.

### Sebelum tunnel
Dari `PC-PEGAWAI`, hasilnya lewat `R-UTARA`
![dokumentasi](Screenshot/soal-6/1.pc-pegawai-before.png)

### Buat tunnel di `R-PUSAT`
![dokumentasi](Screenshot/soal-6/2.r-pusat-tun.png)

| Perintah | Arti |
| :--- | :--- |
| `interface Tunnel0` | membuat interface virtual |
| `ip address 192.168.12.1 ...` | IP tunnel |
| `tunnel source Serial0/0/1` | ujung tunnel di sisi ini, port ke R-UTARA (10.96.241.37) |
| `tunnel destination 10.96.241.54` | ujung tunnel di seberang, alamat fisik R-PELABUHAN |
| `tunnel mode gre ip` | jenis tunnel GRE |

### Buat tunnel di `R-PELABUHAN`
![dokumentasi](Screenshot/soal-6/2.r-pelabuhan-tun.png)

Alamat sumber dan tujuannya dibalik. 

### Cek tunnel sudah hidup
![dokumentasi](Screenshot/soal-6/3.r-pusat-status.png)

### Paksa trafic lewat tunnel (`R-PUSAT`)
Sudah ada dua jalan ke pelabuhan (utara dan tunnel). Route lama harus dilewati tetapi disimpan sebagai cadangan. 

![dokumentasi](Screenshot/soal-6/4.r-pusat-conf.png)

- `no ip route` menghapus route lama.
- dua bars berikutnya membuat route baru lewat tunnel dengan AD 1 (bawaan static route).
- dua baris terakhir memasang kembali route lama dengan AD 200, hanya jadi cadangan.

### Atur arah balik (`R-PELABUHAN`)
![dokumentasi](Screenshot/soal-6/4.r-pelabuhan-conf.png)

Route ini untuk dua subnet Balai Kota: Pegawai (`10.96.224.0/21`) dan Server Balai Kota (`10.96.241.0/27`). Route ini lebih spesifik daripada default route, jadi yang dipilih adalah route tunnel.

### Uji coba
`Tunnel0` dan `line protocol` bernilai `is up` pada `R-PUSAT` dan `R-PELABUHAN`
![dokumentasi](Screenshot/soal-6/5.tunnel0.png)

dua route pelabuhan lewat `192.168.12.2` pada `R-PUSAT`
![dokumentasi](Screenshot/soal-6/6.r-pusat-connect.png)

route balai kota lewat `192.168.12.1` pada `R-PELABUHAN`
![dokumentasi](Screenshot/soal-6/6.r-pelabuhan-connect.png)

tracert dan ping pada `PC-PEGAWAI`
![dokumentasi](Screenshot/soal-6/7.pc-pegawai-ping.png)
![dokumentasi](Screenshot/soal-6/7.pc-pegawai-tracert.png)

ping pada `PC-KANTOR-PELABUHAN`
![dokumentasi](Screenshot/soal-6/7.pc-kantor-pelabuhan-ping.png)

ping pada `PC-SCADA`
![dokumentasi](Screenshot/soal-6/7.pc-scada-ping.png)

### Router memilih jalur tunnel
Di `R-PUSAT` ada dua route ke LAN Pelabuhan dengan prefix yang sama: satu lewat `Tunnel0` (`192.168.12.2`, AD 1) dan satu lewat `R-UTARA` (`10.96.241.38`, AD 200). Router memilih route dengan Administrative Distance **terkecil**, jadi **jalur tunnel yang dipakai**. Route lewat `R-UTARA` tetap ada di konfigurasi sebagai cadangan. Di `R-PELABUHAN`, route ke subnet Balai Kota lewat tunnel dipilih karena lebih spesifik (`/21` dan `/27`) daripada default route `0.0.0.0/0` ke `R-UTARA`, sesuai aturan longest prefix match.

## Soal Bonus
### Mengaktifkan lisensi secruityk9 pada `R-PUSAT` dan `R-PELABUHAN`
```bash
configure terminal
license boot module c2900 technology-package securityk9
```

Simpan dan reload
``` bash
end
copy running-config startup-config
reload
```

### Konfigurasi IPsec di `R-PUSAT`
![dokumentasi](Screenshot/soal-bonus/1.r-pusat-enc.png)

| Bagian | Fungsi |
| ----- | ----- |
| `crypto isakmp policy 10` | aturan negosiasi fase 1: AES-256, hash SHA, otentikasi pakai pre-shared key |
| `crypto isakmp key ... address 10.96.241.54` | kunci rahasia untuk peer R-PELABUHAN (harus sama di kedua sisi) |
| `transform-set TS-GRE` | fase 2: data dienkripsi AES-256 (ESP) dan dijaga integritasnya dengan SHA-HMAC |
| `mode transport` | enkripsi dipasang di paket GRE yang sudah punya IP header sendiri |
| `access-list 101` | trafik yang dienkripsi: hanya paket GRE dari .37 ke .54 |
| `crypto map` | menyatukan peer, transform set, dan ACL |
| `interface Serial0/0/1` | crypto map dipasang di interface tempat paket GRE keluar (sama dengan tunnel source) |

### Konfigurasi IPsec di `R-PELABUHAN`
Isi sama, tapi alamat dibalik dan interface nya `Serial0/0/0`
![dokumentasi](Screenshot/soal-bonus/1.r-pelabuhan-enc.png)

Setelah config, pastikan disimpan `copy running-config startup-config`.

### Uji coba
![dokumentasi](Screenshot/soal-bonus/2.pc-pegawai-ping.png)

Cek state di `R-PUSAT` dengan `show crypto ipsec sa`
![dokumentasi](Screenshot/soal-bonus/3.r-pusat-state.png)

Fase 1: angka `#pkts encrypt` dan `#pkts decrypt` pada ping pertama di `PC-PEGAWAI`.
![dokumentasi](Screenshot/soal-bonus/4.r-pusat-pkts.png)

Fase 2: jika mengirim ping lagi di `PC-PEGAWAI`, angka `#pkts encrypt` dan `#pkts decrypt` akan naik.
![dokumentasi](Screenshot/soal-bonus/4.r-pusat-pkts2.png)

Tunnel tetap jalan 
![dokumentasi](Screenshot/soal-bonus/5.tunnel.png)

Tracert `PC-PEGAWAI` ke `PC-KANTOR-PELAUHAN` masih lewat `192.168.12.2`. 
![dokumentasi](Screenshot/soal-bonus/5.tracert-pegawai-ke-kantor.png)

---

## Soal 7

>Dikerjakan Oleh I Made Tobby Anantha Adiwijaya

### Soal
> Kerjakan di file terpisah. Tambahkan link cadangan L7 antara R-DC dan R-UTARA dengan alamat yang belum digunakan. Ganti static route antar-router dengan OSPF. Router tidak mengirim pesan OSPF ke LAN pengguna, default route hanya dikonfigurasi di R-PUSAT dan disebarkan melalui OSPF, dan konektivitas pulih otomatis jika satu link backbone putus.

### 7.1 Tujuan dan ketentuan soal

Pada soal ini kita diminta agar static route antar-router diganti OSPF, dan dikerjakan di **file terpisah** (`K12_VLSM_OSPF.pkt`, salinan dari file utama), dengan ketentuan:

| Ketentuan | Penerapan |
|---|---|
| Tambah link cadangan L7 antara R-DC dan R-UTARA dengan alamat yang belum dipakai | `10.96.241.56/30` |
| Router tidak mengirim pesan OSPF ke arah LAN pengguna | `passive-interface` pada semua port Gigabit yang menghadap LAN |
| Default route ke internet hanya di R-PUSAT, router lain mendapatkannya lewat OSPF | `ip route 0.0.0.0 0.0.0.0 203.0.113.49` di R-PUSAT + `default-information originate` |
| Jika satu link backbone putus, konektivitas pulih otomatis lewat jalur lain | Diuji dengan memutus L1 dan L2 (bagian 7.6) |

### 7.2 Konsep singkat

- **OSPF** adalah protokol *link-state*: setiap router mengumumkan jaringan yang dimilikinya, lalu semua router menghitung jalur terpendek sendiri. Berbeda dengan static route yang diisi manual, OSPF menyesuaikan diri otomatis saat link putus.
- **Cost** OSPF = 100 Mbps ÷ kecepatan link. Link serial 1,544 Mbps bernilai cost 64, LAN Gigabit bernilai cost 1. Karena itu rute ke LAN yang terhubung langsung ke R-PUSAT bernilai 65 (64 + 1), dan LAN Pelabuhan bernilai 129 (64 + 64 + 1).
- **Jarak administratif (AD)**: static route bernilai 1, OSPF 110. Selama static route masih ada, router memilihnya, sehingga OSPF baru benar-benar dipakai **setelah static route dihapus**. Itu sebabnya penghapusan dilakukan sebagai tahap terpisah, setelah OSPF terbukti membentuk rute yang benar.
- **Wildcard mask** pada perintah `network` adalah kebalikan dari subnet mask. Contoh: `/21` (`255.255.248.0`) menjadi `0.0.7.255` (256 − 248 = 8, wildcard = 8 − 1 = 7).
- **Passive interface**: jaringan pada interface tersebut tetap diumumkan, tetapi router tidak mengirim pesan hello OSPF ke sana, sehingga perangkat pengguna tidak menerima pesan routing.

### 7.3 Tahap 1: Pemasangan link cadangan L7

Alamat L7: `10.96.241.56/30`, R-DC `10.96.241.57` dan R-UTARA `10.96.241.58`.

Langkah-langkahnya:
1. R-UTARA tidak memiliki port serial kosong (dua serial bawaan dipakai L2 dan L6), sehingga tambahkan modul **HWIC-2T** pada slot 1 (router dimatikan, modul dipasang, router dinyalakan kembali). Port baru: `Serial0/1/0`.

   ![Penambahan modul HWIC-2T di R-UTARA](Screenshot/soal-7/tahap-1/1_tambahkan-hwic2t.png)

2. Dipasang kabel **Serial DCE** dari `Serial0/0/1` R-DC (sisi DCE) ke `Serial0/1/0` R-UTARA.

   ![Pembuatan link L7 antara R-DC dan R-UTARA](Screenshot/soal-7/tahap-1/1_buat-link-L7.png)

3. Konfigurasikan IP L7 melalui CLI. Klik router, pilih tab **CLI**, lalu tekan **Enter** sampai prompt muncul. Masukkan blok perintah sesuai router.

   **R-DC** (ujung DCE):

   ```
   enable
   configure terminal
   interface Serial0/0/1
   ip address 10.96.241.57 255.255.255.252
   clock rate 64000
   no shutdown
   end
   write memory
   ```

   **R-UTARA**:

   ```
   enable
   configure terminal
   interface Serial0/1/0
   ip address 10.96.241.58 255.255.255.252
   no shutdown
   end
   write memory
   ```

   Perintah `interface` memilih port serial yang dikonfigurasi. `end` keluar dari mode konfigurasi, lalu `write memory` menyimpan perubahan. `clock rate 64000` hanya dipasang pada ujung DCE, yaitu R-DC.

   ![Konfigurasi dan pengecekan IP di R-DC](Screenshot/soal-7/tahap-1/1_dc-konfigurasi-ip-dan-cek.png)

   ![Konfigurasi dan pengecekan IP di R-UTARA](Screenshot/soal-7/tahap-1/1_utara-konfigurasi-ip-dan-cek.png)

Pemeriksaan: `ping 10.96.241.58` dari R-DC berhasil dan kedua serial berstatus `up/up`.

### 7.4 Tahap 2: Mengaktifkan OSPF di tujuh router

Semua router memakai **proses OSPF 1** dan **area 0** (backbone). Alasan kenapa dibikin satu area adalah agar jaringan hanya terdiri dari delapan router saja dalam satu domain administrasi.

**Router ID** ditetapkan manual agar tidak berubah ketika interface ditambah (misalnya saat L7 dipasang).

**Aturan memilih perintah `network`** ialah satu baris per interface, memakai Network ID dari Tabel VLSM (untuk LAN) atau kelipatan 4 terdekat (untuk link `/30`), dengan wildcard sesuai prefix. Di sini, interface link ke R-ISP (`G0/0` R-PUSAT) sengaja tidak dimasukkan karena R-ISP tidak boleh mengetahui jaringan privat (syarat pada soal 5 hehe).

| Router | Router ID | Perintah `network` (area 0) | Passive interface |
|---|---|---|---|
| R-PUSAT | 1.1.1.1 | `10.96.224.0 0.0.7.255`, `10.96.241.0 0.0.0.31`, `10.96.241.32`, `.36`, `.40`, `.44`, `.48` (semuanya `0.0.0.3`) | G0/1, G0/2 |
| R-DC | 2.2.2.2 | `10.96.96.0 0.0.31.255`, `10.96.239.0 0.0.0.255`, `10.96.241.32 0.0.0.3`, `10.96.241.56 0.0.0.3` | G0/0, G0/1 |
| R-UTARA | 3.3.3.3 | `10.96.0.0 0.0.63.255`, `10.96.236.0 0.0.1.255`, `10.96.241.36`, `.52`, `.56` (semuanya `0.0.0.3`) | G0/0, G0/1 |
| R-PELABUHAN | 4.4.4.4 | `10.96.160.0 0.0.31.255`, `10.96.240.0 0.0.0.127`, `10.96.241.52 0.0.0.3` | G0/0, G0/1 |
| R-BARAT | 5.5.5.5 | `10.96.192.0 0.0.31.255`, `10.96.238.0 0.0.0.255`, `10.96.241.48 0.0.0.3` | G0/0, G0/1 |
| R-TIMUR | 6.6.6.6 | `10.96.128.0 0.0.31.255`, `10.96.232.0 0.0.3.255`, `10.96.241.44 0.0.0.3` | G0/0, G0/1 |
| R-SELATAN | 7.7.7.7 | `10.96.64.0 0.0.31.255`, `10.96.240.128 0.0.0.127`, `10.96.241.40 0.0.0.3` | G0/0, G0/1 |

Masukkan blok konfigurasi berikut pada CLI router yang sesuai. Setiap blok dijalankan **di router yang disebutkan**, satu per satu. Perintah `write memory` menyimpan konfigurasi router tersebut.

**R-PUSAT** (default route ke ISP harus sudah ada agar dapat diumumkan ke router lain):

```
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 203.0.113.49
router ospf 1
router-id 1.1.1.1
network 10.96.224.0 0.0.7.255 area 0
network 10.96.241.0 0.0.0.31 area 0
network 10.96.241.32 0.0.0.3 area 0
network 10.96.241.36 0.0.0.3 area 0
network 10.96.241.40 0.0.0.3 area 0
network 10.96.241.44 0.0.0.3 area 0
network 10.96.241.48 0.0.0.3 area 0
passive-interface GigabitEthernet0/1
passive-interface GigabitEthernet0/2
default-information originate
end
write memory
```

`default-information originate` hanya ada di R-PUSAT karena perintah ini menyebarkan default route (ke R-ISP) kepada router lain lewat OSPF. Router selain R-PUSAT nanti akan muncul kode `O*E2` saat nanti static default mereka dihapus pada langkah berikutnya.

**R-DC:**

```
enable
configure terminal
router ospf 1
router-id 2.2.2.2
network 10.96.96.0 0.0.31.255 area 0
network 10.96.239.0 0.0.0.255 area 0
network 10.96.241.32 0.0.0.3 area 0
network 10.96.241.56 0.0.0.3 area 0
passive-interface GigabitEthernet0/0
passive-interface GigabitEthernet0/1
end
write memory
```

**R-UTARA:**

```
enable
configure terminal
router ospf 1
router-id 3.3.3.3
network 10.96.0.0 0.0.63.255 area 0
network 10.96.236.0 0.0.1.255 area 0
network 10.96.241.36 0.0.0.3 area 0
network 10.96.241.52 0.0.0.3 area 0
network 10.96.241.56 0.0.0.3 area 0
passive-interface GigabitEthernet0/0
passive-interface GigabitEthernet0/1
end
write memory
```

**R-PELABUHAN:**

```
enable
configure terminal
router ospf 1
router-id 4.4.4.4
network 10.96.160.0 0.0.31.255 area 0
network 10.96.240.0 0.0.0.127 area 0
network 10.96.241.52 0.0.0.3 area 0
passive-interface GigabitEthernet0/0
passive-interface GigabitEthernet0/1
end
write memory
```

**R-BARAT:**

```
enable
configure terminal
router ospf 1
router-id 5.5.5.5
network 10.96.192.0 0.0.31.255 area 0
network 10.96.238.0 0.0.0.255 area 0
network 10.96.241.48 0.0.0.3 area 0
passive-interface GigabitEthernet0/0
passive-interface GigabitEthernet0/1
end
write memory
```

**R-TIMUR:**

```
enable
configure terminal
router ospf 1
router-id 6.6.6.6
network 10.96.128.0 0.0.31.255 area 0
network 10.96.232.0 0.0.3.255 area 0
network 10.96.241.44 0.0.0.3 area 0
passive-interface GigabitEthernet0/0
passive-interface GigabitEthernet0/1
end
write memory
```

**R-SELATAN:**

```
enable
configure terminal
router ospf 1
router-id 7.7.7.7
network 10.96.64.0 0.0.31.255 area 0
network 10.96.240.128 0.0.0.127 area 0
network 10.96.241.40 0.0.0.3 area 0
passive-interface GigabitEthernet0/0
passive-interface GigabitEthernet0/1
end
write memory
```

**Cara Memeriksa:**
- `show ip ospf neighbor`: semua tetangga berstatus FULL (R-PUSAT 5 tetangga, R-UTARA 3, R-DC 2, sisanya 1).
- `show ip protocols`: Router ID sesuai, bagian *Routing for Networks* sesuai perintah `network`, dan *Passive Interface(s)* hanya berisi port Gigabit.
- `show ip ospf interface brief`: kolom `Nbrs F/C` pada interface LAN bernilai `0/0`, membuktikan tidak ada tetangga OSPF di sisi LAN.
- Sebelum menghapus static route, `show ip ospf database` di R-DC digunakan untuk memeriksa informasi jaringan L7 yang sudah diterima.

Hasil di R-PUSAT: lima tetangga FULL (`7.7.7.7`, `2.2.2.2`, `3.3.3.3`, `6.6.6.6`, `5.5.5.5`) lewat Serial0/1/0, Serial0/0/0, Serial0/0/1, Serial0/1/1, dan Serial0/2/0. Passive interface hanya `GigabitEthernet0/1` dan `GigabitEthernet0/2`. Cost link serial 64 dan LAN 1.

#### Bukti aktivasi OSPF per router:

![Aktivasi OSPF R-PUSAT](Screenshot/soal-7/tahap-2/2_pusat-aktifkan-ospf.png)

![Aktivasi OSPF R-DC](Screenshot/soal-7/tahap-2/2_dc-aktifkan-ospf.png)

![Aktivasi OSPF R-UTARA](Screenshot/soal-7/tahap-2/2_utara-aktifkan-ospf.png)

![Aktivasi OSPF R-PELABUHAN](Screenshot/soal-7/tahap-2/2_pelabuhan-aktifkan-ospf.png)

![Aktivasi OSPF R-BARAT](Screenshot/soal-7/tahap-2/2_barat-aktifkan-ospf.png)

![Aktivasi OSPF R-TIMUR](Screenshot/soal-7/tahap-2/2_timur-aktifkan-ospf.png)

![Aktivasi OSPF R-SELATAN](Screenshot/soal-7/tahap-2/2_selatan-aktifkan-ospf.png)


#### Bukti pengujian (neighbor, protocol, interface):

![Uji OSPF R-PUSAT](Screenshot/soal-7/tahap-2/uji-coba/2_pusat-tes.png)

![Uji OSPF R-DC](Screenshot/soal-7/tahap-2/uji-coba/2_dc-tes.png)

![Uji OSPF R-UTARA](Screenshot/soal-7/tahap-2/uji-coba/2_utara-tes.png)

![Uji OSPF R-PELABUHAN](Screenshot/soal-7/tahap-2/uji-coba/2_pelabuhan-tes.png)

![Uji OSPF R-BARAT](Screenshot/soal-7/tahap-2/uji-coba/2_barat-tes.png)

![Uji OSPF R-TIMUR](Screenshot/soal-7/tahap-2/uji-coba/2_timur-tes.png)

![Uji OSPF R-SELATAN](Screenshot/soal-7/tahap-2/uji-coba/2_selatan-tes.png)

### 7.5 Tahap 3: Penghapusan static route antar-router

Static route di sini baru boleh untuk dihapus apabila OSPF sudah terbukti membentuk tetangga dan rute yang benar. Di sini, dihapusnya juga harus dengan urutan **router cabang lebih dahulu**, baru router **R-PUSAT terakhir**. Dengan begitu, R-PUSAT tetap menjadi pembanding jika ada rute cabang yang belum benar.

| Router | Dihapus | Dipertahankan |
|---|---|---|
| R-DC, R-BARAT, R-TIMUR, R-SELATAN, R-PELABUHAN | Default route ke router di atasnya | Rute tunnel GRE di R-PELABUHAN |
| R-UTARA | Default route dan 2 rute ke LAN Pelabuhan | n/a |
| R-PUSAT | 13 rute ke LAN dan link di bawahnya, termasuk 2 rute cadangan (AD 200) ke LAN Pelabuhan | Default ke ISP (`203.0.113.49`) dan 2 rute tunnel GRE via `192.168.12.2` |

Pada setiap blok, `show ip route` digunakan untuk memeriksa tabel routing setelah penghapusan. `write memory` menyimpan perubahan pada router tersebut.

#### Router cabang

**R-DC**:

```
enable
show run | include ip route
configure terminal
no ip route 0.0.0.0 0.0.0.0 10.96.241.33
end
show ip route
write memory
```

**R-BARAT**:

```
enable
show run | include ip route
configure terminal
no ip route 0.0.0.0 0.0.0.0 10.96.241.49
end
show ip route
write memory
```

**R-TIMUR**:

```
enable
show run | include ip route
configure terminal
no ip route 0.0.0.0 0.0.0.0 10.96.241.45
end
show ip route
write memory
```

**R-SELATAN**:

```
enable
show run | include ip route
configure terminal
no ip route 0.0.0.0 0.0.0.0 10.96.241.41
end
show ip route
write memory
```

**R-PELABUHAN**:

```
enable
show run | include ip route
configure terminal
no ip route 0.0.0.0 0.0.0.0 10.96.241.53
end
show ip route
write memory
```

**R-UTARA**:

```
enable
show run | include ip route
configure terminal
no ip route 0.0.0.0 0.0.0.0 10.96.241.37
no ip route 10.96.160.0 255.255.224.0 10.96.241.54
no ip route 10.96.240.0 255.255.255.128 10.96.241.54
end
show ip route
write memory
```

#### R-PUSAT

Pada R-PUSAT, hapus rute ke jaringan bawah yang sebelumnya diarahkan melalui router tetangga. Namun, **jangan hapus** default route ke R-ISP dan dua rute tunnel GRE via `192.168.12.2`.

```
enable
show run | include ip route
configure terminal
no ip route 10.96.96.0 255.255.224.0 10.96.241.34
no ip route 10.96.239.0 255.255.255.0 10.96.241.34
no ip route 10.96.192.0 255.255.224.0 10.96.241.50
no ip route 10.96.238.0 255.255.255.0 10.96.241.50
no ip route 10.96.0.0 255.255.192.0 10.96.241.38
no ip route 10.96.236.0 255.255.254.0 10.96.241.38
no ip route 10.96.241.52 255.255.255.252 10.96.241.38
no ip route 10.96.160.0 255.255.224.0 10.96.241.38 200
no ip route 10.96.240.0 255.255.255.128 10.96.241.38 200
no ip route 10.96.128.0 255.255.224.0 10.96.241.46
no ip route 10.96.232.0 255.255.252.0 10.96.241.46
no ip route 10.96.64.0 255.255.224.0 10.96.241.42
no ip route 10.96.240.128 255.255.255.128 10.96.241.42
end
show ip route
show run | include ip route
write memory
```

Setelah penghapusan, `show run | include ip route` di R-PUSAT menampilkan tiga rute yang dipertahankan:

```
ip route 0.0.0.0 0.0.0.0 203.0.113.49
ip route 10.96.160.0 255.255.224.0 192.168.12.2
ip route 10.96.240.0 255.255.255.128 192.168.12.2
```

Di R-DC, `show ip route` menampilkan `Gateway of last resort is 10.96.241.33` dan baris `O*E2 0.0.0.0/0 [110/1]`, membuktikan default route kini berasal dari OSPF. Semua LAN lain berkode `O`.

Pengujian konektivitas setelah penghapusan: ping antarwilayah dan ke `8.8.8.8` berhasil, `tracert` PC-PEGAWAI → PC-KANTOR-PELABUHAN masih melewati tunnel, dan R-ISP tidak memiliki rute ke blok `10.96.x.x` (syarat soal 5).

#### Bukti penghapusan static route:

![Penghapusan default route dan pemeriksaan tabel routing R-DC](Screenshot/soal-7/tahap-3/3_dc-hapus-static-dan-cek.png)

![Penghapusan default route dan pemeriksaan tabel routing R-BARAT](Screenshot/soal-7/tahap-3/3_barat-hapus-static-dan-cek.png)

![Penghapusan default route dan pemeriksaan tabel routing R-TIMUR](Screenshot/soal-7/tahap-3/3_timur-hapus-static-dan-cek.png)

![Penghapusan default route dan pemeriksaan tabel routing R-SELATAN](Screenshot/soal-7/tahap-3/3_selatan-hapus-static-dan-cek.png)

![Penghapusan default route dan pemeriksaan tabel routing R-PELABUHAN](Screenshot/soal-7/tahap-3/3_pelabuhan-hapus-static-dan-cek.png)

![Penghapusan static route dan pemeriksaan tabel routing R-UTARA](Screenshot/soal-7/tahap-3/3_utara-hapus-static-dan-cek.png)

![Penghapusan route di R-PUSAT dan pemeriksaan akhir rute yang tersisa (1)](Screenshot/soal-7/tahap-3/3_pusat-hapus-static-dan-cek-1.png)

![Penghapusan route di R-PUSAT dan pemeriksaan akhir rute yang tersisa (2)](Screenshot/soal-7/tahap-3/3_pusat-hapus-static-dan-cek-2.png)

![Pemeriksaan tabel routing R-ISP tanpa rute privat](Screenshot/soal-7/tahap-3/3_isp-cek.png)

#### Uji konektivitas setelah OSPF:

![PC-PEGAWAI ke internet](Screenshot/soal-7/tahap-3/uji-coba/3_pc-pegawai-internet.png)

![PC-KANTOR-PELABUHAN ke internet](Screenshot/soal-7/tahap-3/uji-coba/3_pc-pelabuhan-internet.png)

![Traceroute PC-PEGAWAI ke PC-KANTOR-PELABUHAN](Screenshot/soal-7/tahap-3/uji-coba/3_pc-pegawai-trace-ke-pc-pelabuhan.png)

![Traceroute PC-KANTOR-PELABUHAN ke PC-PEGAWAI](Screenshot/soal-7/tahap-3/uji-coba/3_pc-pelabuhan-trace-ke-pc-pegawai.png)

![PC-CCTV-KOTA ke PC-KANTOR-PELABUHAN lewat L7](Screenshot/soal-7/tahap-3/uji-coba/3_pc-cctv-ke-pc-pelabuhan-tes-L7.png)

### 7.6 Tahap 4: Uji failover (pemulihan otomatis)

Uji ini dilakukan untuk membuktikan bahwa OSPF memindahkan trafik ke jalur alternatif apabila link utama mati. Di sini kita lakukan uji putuskan L1 dan putuskan L2 secara terpisah, dan tidak lupa untuk selalu **pulihkan link yang diuji sebelum memulai uji berikutnya**. Pada topologi terbaru, hanya link L1 dan L2 yang memiliki jalur alternatif melewati L7. Link lain seperti L3–L6 menuju router yang hanya memiliki satu sambungan.

#### Persiapan

1. Pastikan semua link sudah aktif setelah uji sebelumnya. Di R-PUSAT, periksa interface dan tetangga OSPF:

   ```
   enable
   show ip interface brief
   show ip ospf neighbor
   ```

   Interface serial yang digunakan harus `up/up`; R-PUSAT normalnya melihat lima tetangga berstatus `FULL`.

2. Siapkan dua jendela Packet Tracer: **Command Prompt** pada PC sumber dan **CLI R-PUSAT**. Jangan pernah, sekali lagi, JANGAN PERNAH menyimpan konfigurasi ketika ada interface yang masih `shutdown`.

3. Di setiap uji, catat `tracert` sebelum link diputus. Sesudah jalur pulih, hentikan ping kontinu dengan **Ctrl+C**, dan jalankan `tracert` lagi, lalu pulihkan interface.

#### Uji A: putus L1 antara R-PUSAT dan R-DC

Sumber: **PC-CCTV-KOTA**. Tujuan: **PC-PEGAWAI**, `10.96.224.11`. L1 berada pada `Serial0/0/0` R-PUSAT.  
Jangan lupa untuk `ipconfig` saat baru memasuki Command Prompt.  

1. Di **PC-CCTV-KOTA → Desktop → Command Prompt**, rekam jalur normal:

   ```
   tracert 10.96.224.11
   ```

   Jalur normal melewati gateway CCTV `10.96.96.1`, lalu R-PUSAT pada `10.96.241.33`.

2. Di Command Prompt PC yang sama, mulai ping terus-menerus dan biarkan berjalan:

   ```
   ping -t 10.96.224.11
   ```

3. Buka **CLI R-PUSAT** dan matikan interface L1:

   ```
   enable
   configure terminal
   interface Serial0/0/0
   shutdown
   end
   ```

4. Pada Command Prompt PC sebelumnya yang digunakan untuk `ping -t`, akan muncul beberapa `Request timed out` saat OSPF menghitung ulang jalur. Di sini kita bisa menunggu sekitar **40 detik** atau bisa lebih cepat (Bisa juga pake percepat waktu Packet Tracer). Nah, setelah balasan `Reply from 10.96.224.11` muncul kembali, tekan **Ctrl+C** untuk menghentikan ping dan menampilkan statistik.

5. Di PC-CCTV-KOTA, periksa jalur baru setelah `shutdown`:

   ```
   tracert 10.96.224.11
   ```

   Jalurnya seharusnya memutar dari R-DC melalui L7 ke R-UTARA (`10.96.241.58`), lalu melalui L2 ke R-PUSAT (`10.96.241.37`).
   
   Di **CLI R-DC**, tabel rute juga dapat diperiksa dengan menggunakan:

   ```
   enable
   show ip ospf neighbor
   show ip route
   ```

6. Coba ping ke Internet, setelah `shutdown`:

   ```
   ping 8.8.8.8
   ```

7. Pulihkan L1 di R-PUSAT sebelum lanjut:

   ```
   enable
   configure terminal
   interface Serial0/0/0
   no shutdown
   end
   ```

   Tunggu sampai interface kembali `up/up` dan lima tetangga OSPF R-PUSAT kembali `FULL`:

   ```
   show ip interface brief
   show ip ospf neighbor
   ```

| Kondisi | Jalur `tracert` yang diinginkan |
|---|---|
| L1 aktif | `10.96.96.1` (R-DC) → `10.96.241.33` (R-PUSAT lewat L1) → `10.96.224.11` |
| L1 mati | `10.96.96.1` → `10.96.241.58` (R-UTARA lewat L7) → `10.96.241.37` (R-PUSAT lewat L2) → `10.96.224.11` |

#### Bukti Uji A:

![Ping kontinu yang pulih setelah L1 diputus](Screenshot/soal-7/tahap-4/L1-cctv-ke-pegawai/4_L1-ping-continuous.png)

![Traceroute sebelum L1 diputus](Screenshot/soal-7/tahap-4/L1-cctv-ke-pegawai/4_L1-trace-sebelum.png)

![Traceroute melalui L7 setelah L1 diputus](Screenshot/soal-7/tahap-4/L1-cctv-ke-pegawai/4_L1-trace-setelah-shutdown.png)

![Perintah shutdown dan no shutdown pada L1](Screenshot/soal-7/tahap-4/L1-cctv-ke-pegawai/4_L1-Serial000-shutdown-and-noshutdown.png)

![Pemeriksaan rute di R-DC saat L1 mati](Screenshot/soal-7/tahap-4/L1-cctv-ke-pegawai/4_L1-cek-dc.png)

![Koneksi internet tetap berjalan saat L1 mati](Screenshot/soal-7/tahap-4/L1-cctv-ke-pegawai/4_L1-ping-internet-while-shutdown.png)

#### Uji B: putus L2 antara R-PUSAT dan R-UTARA

Sumber: **PC-SENSOR-LALIN**. Tujuan: **PC-PEGAWAI**, `10.96.224.11`. L2 berada pada `Serial0/0/1` R-PUSAT.
Jangan lupa untuk `ipconfig` saat baru memasuki Command Prompt.  

1. Di **PC-SENSOR-LALIN → Desktop → Command Prompt**, rekam jalur normal:

   ```
   tracert 10.96.224.11
   ```

   Jalur normal melewati gateway sensor `10.96.236.1`, lalu R-PUSAT pada `10.96.241.37`.

2. Di Command Prompt PC yang sama, mulai ping terus-menerus dan biarkan berjalan:

   ```
   ping -t 10.96.224.11
   ```

3. Di **CLI R-PUSAT**, matikan interface L2:

   ```
   enable
   configure terminal
   interface Serial0/0/1
   shutdown
   end
   ```

4. Tunggu OSPF menghitung ulang jalur, bisa sekitar **40 detik** atau bisa kurang. Nah, setelah ping kembali membalas, tekan **Ctrl+C** untuk menghentikan. Lalu, jalankan traceroute lagi untuk melihat jalur cadangan (setelah di `shutdown`):

   ```
   tracert 10.96.224.11
   ```

   Jalur seharusnya melewati L7 dari R-UTARA ke R-DC (`10.96.241.57`), lalu L1 menuju R-PUSAT (`10.96.241.33`).  
   Saat L2 masih mati, tetangga pada router R-UTARA juga dapat diperiksa dengan menggunakan:

   ```
   enable
   show ip ospf neighbor
   show ip route
   ```

   R-UTARA seharusnya masih bertetangga FULL dengan R-DC dan R-PELABUHAN, tetapi tidak bertetangga dengan R-PUSAT karena L2 mati.

5. Pulihkan L2 dari CLI R-PUSAT:

   ```
   enable
   configure terminal
   interface Serial0/0/1
   no shutdown
   end
   ```

   Pastikan interface kembali `up/up`, lima tetangga OSPF di R-PUSAT dan tiga tetangga di R-UTARA kembali `FULL`:

   ```
   show ip interface brief
   show ip ospf neighbor
   ```

| Kondisi | Jalur yang diinginkan |
|---|---|
| L2 aktif | R-UTARA (`10.96.236.1`) → R-PUSAT lewat L2 (`10.96.241.37`) → `10.96.224.11` |
| L2 mati | `10.96.236.1` → R-DC lewat L7 (`10.96.241.57`) → R-PUSAT lewat L1 (`10.96.241.33`) → `10.96.224.11` |

#### Bukti Uji B:

![Ping kontinu yang pulih setelah L2 diputus](Screenshot/soal-7/tahap-4/L2-sensor-ke-pegawai/4_L2-ping-continuous.png)

![Traceroute sebelum L2 diputus](Screenshot/soal-7/tahap-4/L2-sensor-ke-pegawai/4_L2-trace-sebelum.png)

![Traceroute melalui L7 setelah L2 diputus](Screenshot/soal-7/tahap-4/L2-sensor-ke-pegawai/4_L2-trace-setelah-shutdown.png)

![Perintah shutdown dan no shutdown pada L2](Screenshot/soal-7/tahap-4/L2-sensor-ke-pegawai/4_L2-Serial001-shutdown-and-noshutdown.png)

![Pemeriksaan router saat L2 mati](Screenshot/soal-7/tahap-4/L2-sensor-ke-pegawai/4_L2-cek-dc.png)

![Koneksi internet tetap berjalan saat L2 mati](Screenshot/soal-7/tahap-4/L2-sensor-ke-pegawai/4_L2-ping-internet-while-shutdown.png)

#### Setelah kedua pengujian tersebut

1. Pastikan L1 dan L2 sudah `no shutdown`, semua interface serial yang digunakan kembali `up/up`, dan tetangga OSPF kembali `FULL`.
2. Simpan konfigurasi R-PUSAT setelah semua link pulih:

   ```
   enable
   write memory
   ```

3. JANGAN PERNAH menyimpan keadaan saat link masih `shutdown`. Tidak lupa pastikan file Packet Tracer disimpan setelah topologi kembali normal.

---

## Soal 8: Layanan Aplikasi Smart City

>Dikerjakan Oleh I Made Tobby Anantha Adiwijaya

### Soal
> Gunakan domain internal `k12.smartcity.lab`. Sediakan DNS di LAN Server dan Storage DC, web Portal Smart City Surabaya di LAN Server Balai Kota, FTP dengan akun login agar perangkat CCTV dapat mengunggah rekaman, serta email dua arah antara pegawai Balai Kota dan Pelabuhan.

### 8.1 Rancangan

Domain internal yang digunakan adalah **`k12.smartcity.lab`**. Lalu, semua layanan memakai IP statis pada server, dan klien mendapatkan alamat server DNS dari DHCP (`10.96.239.2`).

| Layanan | Server | IP server | Nama |
|---|---|---|---|
| DNS | SRV-DC (LAN Server dan Storage DC) | `10.96.239.2` | n/a |
| FTP | SRV-DC | `10.96.239.2` | `ftp.k12.smartcity.lab` |
| Web | SRV-BALAIKOTA (LAN Server Balai Kota) | `10.96.241.2` | `www.k12.smartcity.lab` |
| Email (SMTP + POP3) | SRV-BALAIKOTA | `10.96.241.2` | `mail.k12.smartcity.lab` |

Di sini saya menggunakan `SRV-BALAIKOTA` sebagai tempat penempatan email karena Balai Kota adalah pusat jaringan, dan Pelabuhan menjangkaunya melalui tunnel GRE. Pengguna Balai Kota dan Pelabuhan sama-sama memakai email address seperti `nama@k12.smartcity.lab`.

Prinsipnya di sini adalah, DNS hanya menerjemahkan nama menjadi IP. Agar layanan dapat dipakai, harus ada tiga hal sekaligus:  
1. Record DNS yang menunjuk ke IP yang benar.
2. Layanan di server aktif
3. Rute yang menghubungkan klien dengan server.

Layanannya di sini tidak bergantung pada IPsec yang sebelumnya ditambahkan sebagai bonus pada soal 6. IPsecnya hanya mengenkripsi isi tunnel GRE, sedangkan layanan hanya memerlukan jalur yang tersedia.

### 8.2 Tahap 1: Pemeriksaan awal

| Pemeriksaan | Hasil yang diharapkan |
|---|---|
| IP statis server | SRV-DC `10.96.239.2` (gateway `10.96.239.1`), SRV-BALAIKOTA `10.96.241.2` (gateway `10.96.241.1`), DNS keduanya `10.96.239.2` |
| DNS di klien | `ipconfig /all` menampilkan DNS `10.96.239.2` |
| Jaringan dasar | PC di Balai Kota dan Pelabuhan dapat melakukan ping ke kedua server |
| Link dan OSPF | Semua serial `up/up`, tetangga OSPF FULL, IPsec `QM_IDLE` |

Hal yang perlu diperhatikan, adalah lalu lintas dari Pelabuhan ke SRV-BALAIKOTA melewati tunnel GRE+IPsec, sehingga beberapa paket pertama dapat hilang sebelum jalur terbentuk. Pengujian dari Pelabuhan juga harus dilakukan setelah jalur "*dipanaskan*" dengan beberapa ping dari `PC-Pegawai` ke `PC-Kantor-Pelabuhan`, dan status IPsec `QM_IDLE` terlihat di R-PUSAT.

Bukti:

![IP SRV-DC](Screenshot/soal-8/tahap-1/1_srv-dc-cek-ip.png)

![IP SRV-BALAIKOTA](Screenshot/soal-8/tahap-1/1_srv-balaikota-cek-ip.png)

![DNS di klien](Screenshot/soal-8/tahap-1/1_cek-dns-klien.png)

![Tes jaringan dasar dari PC-PEGAWAI](Screenshot/soal-8/tahap-1/1_pc-pegawai-tes-jaringan-dasar.png)

![Tes jaringan dasar dari PC-KANTOR-PELABUHAN](Screenshot/soal-8/tahap-1/1_pc-pelabuhan-tes-jaringan-dasar.png)

![Pemanasan jalur dari PC-PEGAWAI ke Pelabuhan](Screenshot/soal-8/tahap-1/1_ping-dari-pc-pegawai-ke-pc-pelabuhan-jika-QM_idle-tidak-ada.png)

![Pemeriksaan keseluruhan dan status QM_IDLE](Screenshot/soal-8/tahap-1/1_cek-semua-dan-pastikan-QM_idle-ada.png)

### 8.3 Tahap 2: DNS (SRV-DC)

Langkah: SRV-DC → Services → DNS → **DNS Service: On**, lalu tambahkan tiga **A Record**:

| Name | Type | Address |
|---|---|---|
| `www.k12.smartcity.lab` | A | `10.96.241.2` |
| `ftp.k12.smartcity.lab` | A | `10.96.239.2` |
| `mail.k12.smartcity.lab` | A | `10.96.241.2` |

![Pengaturan DNS dan tiga record di SRV-DC](Screenshot/soal-8/tahap-2/2_server-dc-setup-dns-dan-tambah-tiga-service.png)

Satu contoh di sini agar tidak penuh semua klien, saya uji DNS dari **PC-PEGAWAI → Desktop → Command Prompt**. Jalankan perintah berikut satu per satu:

```
nslookup www.k12.smartcity.lab
```

Nama `www.k12.smartcity.lab` harus diterjemahkan ke IP web server SRV-BALAIKOTA, yaitu `10.96.241.2`.

```
nslookup ftp.k12.smartcity.lab
```

Nama `ftp.k12.smartcity.lab` harus diterjemahkan ke IP FTP server SRV-DC, yaitu `10.96.239.2`.

```
nslookup mail.k12.smartcity.lab
```

Nama `mail.k12.smartcity.lab` harus diterjemahkan ke IP mail server SRV-BALAIKOTA, yaitu `10.96.241.2`.  
Tambahan keterangan, server DNS yang menjawab ketiga permintaan adalah SRV-DC (`10.96.239.2`).

![nslookup tiga layanan dari PC-PEGAWAI](Screenshot/soal-8/tahap-2/2_pc-pegawai-tes-nslookup-tiga-service.png)

Ping juga apabila diperlukan  

![ping tiga layanan dari PC-PEGAWAI](Screenshot/soal-8/tahap-2/2_pc-pegawai-tes-ping-tiga-service.png)

### 8.4 Tahap 3: Web (SRV-BALAIKOTA)

Langkah: SRV-BALAIKOTA → Services → HTTP → **On**.  

![Pengaturan HTTP dan index.html di SRV-BALAIKOTA](Screenshot/soal-8/tahap-3/3_server-balaikota-setup-http-dan-index.png)

Lalu `index.html` diubah menjadi halaman "Portal Smart City Surabaya" yang mencantumkan nomor kelompok 12:

```html
<html>
<body>
<h1>Portal Smart City Surabaya</h1>
<p>Kelompok 12</p>
</body>
</html>
```

![Perubahan isi index.html](Screenshot/soal-8/tahap-3/3_server-balaikota-ubah-isi-index.png)

Contoh uji dari salah satu klien, contohnya dari `PC-PEGAWAI`, buka Web Browser pada Devices.  

![Membuka Web Browser di PC-PEGAWAI](Screenshot/soal-8/tahap-3/3_pc-pegawai-buka-web-browser.png)

Pada URL, buka `http://www.k12.smartcity.lab` (**memakai nama domain, bukan IP**).  

![Halaman terbuka memakai nama domain](Screenshot/soal-8/tahap-3/3_pc-pegawai-browser-pakai-domain.png)

### 8.5 Tahap 4: FTP (SRV-DC)

1. Pertama, aktifkan FTP dan siapkan akun di SRV-DC. Kemudian, buka **Services → FTP**, ubah **Service** menjadi **On**, lalu isi User Setup dengan username `cctv`. Centang semua izin: **Write, Read, Delete, Rename**, dan **List**, kemudian klik **Add**. Hapus akun bawaan `cisco` agar akses FTP menggunakan akun yang kita siapkan.

   ![Mengaktifkan FTP dan menambahkan akun cctv di SRV-DC](Screenshot/soal-8/tahap-4/4_server-dc-setup-ftp-dan-tambah-user.png)

   Pastikan akun `cctv` muncul pada daftar user FTP.

   ![Daftar akun FTP setelah akun cctv ditambahkan](Screenshot/soal-8/tahap-4/4_server-dc-ftp-user.png)

2. Siapkan file bebas di contoh klien PC-CCTV-KOTA. Buka **Desktop → Text Editor**.

   ![Membuka Text Editor di PC-CCTV-KOTA](Screenshot/soal-8/tahap-4/4_pc-cctv-buka-text-editor.png)

   Tulis isi file `Rekaman CCTV Kota - Kelompok 12`.

   ![Menulis isi file rekaman di Text Editor](Screenshot/soal-8/tahap-4/4_pc-cctv-text-editor.png)

   Lalu pilih **File → Save**.

   ![File rekaman tersimpan di PC-CCTV-KOTA](Screenshot/soal-8/tahap-4/4_pc-cctv-save-text.png)

   Dan simpan dengan nama `rekaman.txt`.

   ![Menyimpan file dengan nama rekaman.txt](Screenshot/soal-8/tahap-4/4_pc-cctv-file-save-name.png)

   

3. Selanjutnya coba unggah file dari LAN CCTV Kota. Buka **Desktop → Command Prompt** pada PC-CCTV-KOTA dan sambungkan ke server FTP:

   ```
   ftp ftp.k12.smartcity.lab
   ```

   Saat diminta login, masukkan username `cctv` dan password akun FTP. Setelah prompt `ftp>` muncul.

   ![PC-CCTV-KOTA tersambung ke server FTP](Screenshot/soal-8/tahap-4/4_pc-cctv-connect-ftp-server.png)
   
   Unggah file dan periksa daftar file server:

   ```
   put rekaman.txt
   dir
   quit
   ```

   ![PC-CCTV-KOTA mengunggah rekaman.txt dan memeriksa file di server](Screenshot/soal-8/tahap-4/4_pc-cctv-tes-put-di-ftp-server.png)

   Perintah `put` mengirim file ke server; `dir` memastikan `rekaman.txt` sudah tersimpan.   

4. Kita bisa juga uji akses FTP dari wilayah lain. Contohnya pada `PC-SENSOR-LALIN` di wilayah R-UTARA, buka **Desktop → Command Prompt** lalu sambungkan ke nama server yang sama:

   ```
   ftp ftp.k12.smartcity.lab
   ```

   Login dengan akun FTP, lalu unduh dan periksa file:

   ```
   get rekaman.txt
   dir
   quit
   ```

   File `rekaman.txt` berhasil diunduh di PC-SENSOR-LALIN, membuktikan server FTP SRV-DC dapat diakses dari wilayah lain.

   ![PC-SENSOR-LALIN mengunduh rekaman.txt dari server FTP](Screenshot/soal-8/tahap-4/4_pc-sensor-lalin-tes-get-di-ftp-server.png)

### 8.6 Tahap 5: Email (SRV-BALAIKOTA)

1. Pertama, aktifkan layanan email di SRV-BALAIKOTA. Buka **Services → EMAIL**, nyalakan **SMTP Service** dan **POP3 Service**, isi **Domain Name** dengan `k12.smartcity.lab`, lalu klik **Set**.

   ![Mengaktifkan SMTP dan POP3 serta menetapkan domain email di SRV-BALAIKOTA](Screenshot/soal-8/tahap-5/5_server-balaikota-setup-konfigurasi-server.png)

2. Kemudian, siapkan akun pengguna. Pada bagian **User Setup**, tambahkan contoh akun `pegawai` untuk PC-PEGAWAI dan `pelabuhan` untuk PC-KANTOR-PELABUHAN. Kedua akun memakai domain `k12.smartcity.lab`.

   ![Daftar akun email pegawai dan pelabuhan di SRV-BALAIKOTA](Screenshot/soal-8/tahap-5/5_server-balaikota-user-setup-pegawai-dan-pelabuhan.png)

3. Lalu, atur aplikasi email pada kedua PC. Buka **Desktop → Email → Configure Mail**. Gunakan nilai berikut:

   | Kolom | PC-PEGAWAI | PC-KANTOR-PELABUHAN |
   |---|---|---|
   | Your Name | Pegawai | Pelabuhan |
   | Email Address | `pegawai@k12.smartcity.lab` | `pelabuhan@k12.smartcity.lab` |
   | Incoming Mail Server | `mail.k12.smartcity.lab` | `mail.k12.smartcity.lab` |
   | Outgoing Mail Server | `mail.k12.smartcity.lab` | `mail.k12.smartcity.lab` |
   | User Name | `pegawai` | `pelabuhan` |

   Isi password yang sesuai dengan akun di SRV-BALAIKOTA, kemudian simpan konfigurasi pada masing-masing PC. Nama server `mail.k12.smartcity.lab` dipakai untuk menguji DNS sekaligus layanan email.

   ![Pengaturan aplikasi email di PC-PEGAWAI](Screenshot/soal-8/tahap-5/5_pc-pegawai-configure-mail.png)

   ![Pengaturan aplikasi email di PC-KANTOR-PELABUHAN](Screenshot/soal-8/tahap-5/5_pc-pelabuhan-configure-mail.png)

4. Sekarang coba kirim email dari Pelabuhan ke Balai Kota. Buka aplikasi Email di kedua PC.

   ![Aplikasi email dibuka pada kedua PC](Screenshot/soal-8/tahap-5/5_buka-email-di-kedua-pc-klien-yang-dipakai.png)

   Pada PC-KANTOR-PELABUHAN, tulis email dengan penerima `pegawai@k12.smartcity.lab`, lalu klik **Send**.
   
   ![PC-KANTOR-PELABUHAN mengirim email](Screenshot/soal-8/tahap-5/5_pc-pelabuhan-tes-kirim.png)

   ![PC-KANTOR-PELABUHAN menulis email untuk PC-PEGAWAI](Screenshot/soal-8/tahap-5/5_pc-pelabuhan-isi-tes-kirim-ke-pc-pegawai.png)

   Pada PC-PEGAWAI, klik **Receive**. Email dari Pelabuhan muncul di inbox. Di sini kita bisa buka pesannya untuk memastikan isi dan alamat pengirim.

   ![Email dari Pelabuhan diterima pada PC-PEGAWAI](Screenshot/soal-8/tahap-5/5_pc-pegawai-terima-dari-pc-pelabuhan.png)

   ![PC-PEGAWAI membuka email dari Pelabuhan](Screenshot/soal-8/tahap-5/5_pc-pegawai-buka-mail-dari-pc-pelabuhan.png)

5. Kita juga bisa membalas email dari Balai Kota ke Pelabuhan. Dari pesan yang diterima, Pada PC-PEGAWAI coba kirim sebuah balasan. 

   ![PC-PEGAWAI menulis balasan untuk Pelabuhan](Screenshot/soal-8/tahap-5/5_pc-pegawai-reply-mail-ke-pc-pelabuhan.png)

   Pada PC-KANTOR-PELABUHAN, klik **Receive** dan pastikan balasan masuk.

   ![PC-KANTOR-PELABUHAN menerima balasan dari PC-PEGAWAI](Screenshot/soal-8/tahap-5/5_pc-pelabuhan-terima-reply-dari-pc-pegawai.png)

### 8.7 Pengujian dari berbagai wilayah

Pengujian dilakukan dari semua wilayah. Untuk setiap wilayah, kelima pengujian berikut berhasil dilakukan menggunakan PC atau smartphone sesuai perangkat yang tersedia dan layanan yang diuji:  
1. DNS klien dari DHCP (`ipconfig /all`).
2. `nslookup`.
3. Web memakai nama domain.
4. FTP login lalu `get`/`put`.
5. Email.

| Wilayah | Perangkat uji | 1. IP/DNS | 2. `nslookup` | 3. Web | 4. FTP | 5. Email |
|---|---|---|---|---|---|---|
| Balai Kota | PC-PEGAWAI | ✓ | ✓ | ✓ | ✓ | ✓ |
| Command Center / DC | PC-CCTV-KOTA | ✓ | ✓ | ✓ | ✓ | ✓ |
| Utara | PC-SENSOR-LALIN, HP-WIFI-UTARA | ✓ | ✓ | ✓ | ✓ | ✓ |
| Barat | PC-PUSKESMAS, HP-WIFI-BARAT | ✓ | ✓ | ✓ | ✓ | ✓ |
| Timur | PC-SENSOR-BANJIR, HP-WIFI-TIMUR | ✓ | ✓ | ✓ | ✓ | ✓ |
| Selatan | PC-PJU, HP-WIFI-SELATAN | ✓ | ✓ | ✓ | ✓ | ✓ |
| Pelabuhan | PC-KANTOR-PELABUHAN, PC-SCADA | ✓ | ✓ | ✓ | ✓ | ✓ |

Kelima pengujian berhasil di seluruh wilayah, baik menggunakan PC maupun smartphone sesuai pengujiannya. Email juga berhasil dikirim dan diterima dua arah antara semua PC dan smartphone.

---

## Kendala & Solusi

| Kendala | Penyebab | Solusi |
| :--- | :--- | :--- |
| Router-PT tidak bisa membuat GRE/IPsec | IOS 12.2 PT1000 tidak punya `interface Tunnel`; `crypto ?` hanya berisi `key` | Semua router diganti **Cisco 2911** + HWIC-2T sesuai modul |
| HP WiFi tersambung ke AP wilayah lain | Semua AP memakai SSID bawaan `Default` | SSID unik per wilayah di AP dan smartphone |
| Setelah router diganti, PC tidak bisa ping gateway | Cache ARP PC masih menyimpan MAC router lama | Tunggu ARP kedaluwarsa, atau ping PC dari router sekali |
| Perintah `show` ditolak di CLI (`% Invalid input`) | Mengklik interface di tab Config membuat console masuk mode `(config-if)#` | Ketik `end` dulu |
| `enable` / `configure terminal` ditolak | Console masih di mode `(config-if)#` atau `(config)#` | Ketik `end`, lalu ulangi |
| Ping pertama `Request timed out` | Proses ARP pada paket pertama | Ulangi ping |
| Salah ketik perintah membuat CLI "macet" | Router mencoba DNS lookup nama perintah | `no ip domain-lookup` di semua router |
| Serial `up/down` | Clock rate belum diset di sisi DCE | `clock rate 64000` di router atas; cek dengan `show controllers` |
| `SMTP authentication error` saat mengirim email | Username yang disetel di SRV-BALAIKOTA memakai tanda hubung (`-`), sedangkan username dengan tanda hubung tidak diterima | Gunakan username tanpa tanda hubung di SRV-BALAIKOTA, lalu samakan username dan password pada klien email |
