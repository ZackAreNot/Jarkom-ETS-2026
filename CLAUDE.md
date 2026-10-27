# Mid-Project Jarkom — Smart City Surabaya (Kelompok 12)

Konteks kerja untuk dilanjutkan di Claude Code. File ini merangkum apa yang
sudah dikerjakan dan apa langkah berikutnya.

## Identitas proyek

- Mata kuliah: Komunikasi Data dan Jaringan Komputer, Semester Gasal 2026/2027
- Tugas: Mid-Project "Implementasi Jaringan Smart City Kota Surabaya"
- Nomor kelompok: **K = 12** (karena base IP 10.96.0.0, oktet ke-2 = 8 x 12)
- Deadline: **Minggu, 1 November 2026, 23.59 WIB**
- Demo: minggu ke-10 (TBA)
- Skema yang dipakai di file .pkt: **VLSM**
- Domain internal: `k12.smartcity.lab`
- Tool: Cisco Packet Tracer 9.0.1, semua router **Cisco 2911** + modul **HWIC-2T**
  (mengikuti modul praktikum "06-Tutorial CPT": backbone serial, R-PUSAT sisi DCE)
- Backup versi lama (Router-PT): `backup/K12_VLSM_master_routerPT.pkt`

## Pembagian tugas (3 orang)

| Peran | Soal | Poin | Tugas |
|---|---|---|---|
| **A (saya)** | 1, 3, 4, 5 | 40 | Topologi, IP statis, static routing, DHCP (+relay CCTV), NAT/PAT. Pegang file master. |
| **B** | 6, 7, 8 | 30 (+5 bonus) | Tunnel GRE (+IPsec bonus), file OSPF, DNS/Web/FTP/Email |
| **C** | 2 | 5 | Spreadsheet VLSM+CIDR+revisi, repo GitHub, ZIP (perlu dikonfirmasi ulang) |

Pembagian diubah 27 Okt 2026 sore: A ambil soal 4–5, B ambil 6–8 (OSPF pindah
dari A ke B, GRE pindah dari C ke B).

Soal 9 (demo, 25 poin) dikerjakan bertiga. Uji 2 bersifat individu, jadi
tiap anggota harus bisa menemukan dan memperbaiki kesalahan di bagian mana pun.

## Status pekerjaan saya (peran A)

### SELESAI

1. **Topologi lengkap di Packet Tracer**
   - 8 router Cisco 2911, 14 switch 2960, 3 Server-PT, 8 PC-PT, 4 AccessPoint-PT,
     4 Smartphone
   - Semua perangkat sudah di-rename; 3 "Power Distribution Device" bawaan PT 9
     (off-canvas) di-rename jadi PDU-1..PDU-3
   - Label LAN dan link sudah dipasang (nama subnet, Network ID/prefix, jumlah host)
2. **Router diganti Router-PT → 2911 + HWIC-2T** (27 Okt 2026, mengikuti modul
   "06-Tutorial CPT"): backbone L1–L6 serial, `clock rate 64000` di sisi DCE,
   `no ip domain-lookup` di semua router. Alasan: lihat Catatan teknis.
3. **IP statis semua interface router** terpasang, semua up/up, CDP cocok
4. **Static routing selesai** (soal 3) — semua route sudah
   `copy running-config startup-config` (running = startup di 8 router)
5. **SRV-INTERNET** sudah 8.8.8.8/24 gw 8.8.8.1; `router rip` nyasar di R-ISP
   hilang bersama Router-PT lama
6. File master **`K12_VLSM_master.pkt`** (Packet Tracer 9.0.1) sudah disimpan

Hasil uji setelah ganti ke 2911 (27 Okt 2026): ping PC-CCTV-KOTA dan
PC-KANTOR-PELABUHAN ke ke-14 gateway LAN **28/28 OK**; R-ISP → 8.8.8.8 OK;
`pt_health_check` healthy. `tracert 10.96.240.2` dari PC-CCTV-KOTA:
10.96.96.1 → 10.96.241.33 → 10.96.241.38 → 10.96.241.54 → 10.96.240.2.

Catatan: setelah router diganti, PC masih menyimpan MAC router lama di cache ARP
sehingga ping gateway gagal sampai router mem-ping PC itu sekali (atau tunggu
ARP kedaluwarsa). Bukan masalah konfigurasi.

7. **SSID per AP** (27 Okt 2026, lewat GUI): AP & HP pakai SSID `WIFI-BARAT`,
   `WIFI-UTARA`, `WIFI-TIMUR`, `WIFI-SELATAN`. Sebelumnya semua "Default" dan
   HP-WIFI-UTARA/SELATAN nyasar ke AP-WIFI-TIMUR. Tanpa password (Authentication
   Disabled).
8. **Soal 4 DHCP selesai** — lihat bagian "DHCP (soal 4)". 12 host (8 PC + 4 HP)
   dapat IP, gateway, dan DNS 10.96.239.2 dari subnet yang benar; PC-CCTV-KOTA
   dilayani SRV-DC lewat relay.
9. **Soal 5 NAT selesai** — lihat bagian "NAT (soal 5)".

### BELUM

1. **Kirim master ke B (dan C)** + kabari nama port berubah (Tabel port router),
   IP server, dan pembagian tugas baru.
2. ~~Dokumentasi bagian saya~~ SELESAI 27 Okt 2026: `README.md` berisi ringkasan
   (VLSM, port, alamat statis), "Panduan Dasar Packet Tracer" (buka perangkat,
   tab CLI, mode CLI, simpan), lalu soal 1, 3, 4, 5 per langkah (buka router →
   ketik perintah ini → tabel fungsi) dengan 53 screenshot ber-highlight di
   `Screenshot/{umum,soal-N}/`, termasuk screenshot tiap uji ping. Soal 2, 6, 7, 8
   masih placeholder untuk B dan C. Nama/NRP anggota B dan C masih placeholder.
   Skrip: `~/mcp-servers/pt-client/{shot.py,typeshots.py,pingshots.py}`.
3. Opsional: `write memory` di 14 switch (startup-config switch masih kosong).

## Langkah terakhir yang dikerjakan

27 Okt 2026 sore: router diganti ke 2911, lalu SSID, DHCP (soal 4), dan NAT
(soal 5) dipasang lewat MCP packet-tracer, diuji, dan master disimpan. GRE dan
OSPF belum ada di file (bagian B). Config CLI yang dipakai:
`~/mcp-servers/pt-client/configs_2911.json` (IP + static route) dan
`~/mcp-servers/pt-client/dhcp_nat.py` (DHCP + NAT). Backup tiap tahap di
`backup/`.

## Alamat statis (jangan dipakai DHCP)

Di setiap LAN, host .1–.10 di-exclude dari pool DHCP: .1 gateway router,
.2 server (kalau ada), sisanya cadangan.

| Perangkat | IP | Gateway | DNS |
|---|---|---|---|
| SRV-DC (DNS, DHCP CCTV, FTP — soal 8) | 10.96.239.2/24 | 10.96.239.1 | 10.96.239.2 |
| SRV-BALAIKOTA (Web — soal 8) | 10.96.241.2/27 | 10.96.241.1 | 10.96.239.2 |
| SRV-INTERNET | 8.8.8.8/24 | 8.8.8.1 | — |

## DHCP (soal 4)

Pool di router pemilik LAN, pola sama untuk tiap LAN pengguna (11 pool):

```
ip dhcp excluded-address 10.96.224.1 10.96.224.10
ip dhcp pool DHCP-PEGAWAI
 network 10.96.224.0 255.255.248.0
 default-router 10.96.224.1
 dns-server 10.96.239.2
```

| Router | Pool |
|---|---|
| R-PUSAT | DHCP-PEGAWAI |
| R-BARAT | DHCP-WIFI-BARAT, DHCP-PUSKESMAS |
| R-UTARA | DHCP-WIFI-UTARA, DHCP-SENSOR-LALIN |
| R-TIMUR | DHCP-WIFI-TIMUR, DHCP-SENSOR-BANJIR |
| R-SELATAN | DHCP-WIFI-SELATAN, DHCP-PJU |
| R-PELABUHAN | DHCP-SCADA, DHCP-KANTOR-PELABUHAN |

LAN Server Balai Kota dan Server DC tidak punya pool (isinya server statis).

**CCTV Kota** dilayani server, bukan router:
- SRV-DC → Services → DHCP: pool `DHCP-CCTV` 10.96.96.11–10.96.127.254, gw
  10.96.96.1, DNS 10.96.239.2 (pool bawaan `serverPool` dihapus).
- R-DC: `interface g0/0` → `ip helper-address 10.96.239.2` (relay broadcast DHCP
  dari LAN CCTV ke SRV-DC).
- Bukti: `ipconfig /all` di PC-CCTV-KOTA → `DHCP Servers: 10.96.239.2`.

## NAT (soal 5)

PAT di R-PUSAT, semua LAN keluar memakai 203.0.113.50 (pola modul slide 43):

```
access-list 1 permit 10.96.0.0 0.0.255.255
interface g0/1 .. g0/2, s0/0/0 .. s0/2/0   → ip nat inside   (7 interface)
interface g0/0                              → ip nat outside
ip nat inside source list 1 interface g0/0 overload
```

R-ISP tetap tanpa rute ke 10.96.0.0 (syarat soal). Catatan untuk B: Tunnel0 nanti
jangan dijadikan `ip nat inside/outside` — trafik Balai Kota ↔ Pelabuhan lewat
tunnel tidak keluar G0/0, jadi tidak di-NAT (sama seperti modul slide 55).

## Topologi

```
                 R-ISP ---- SRV-INTERNET (8.8.8.8)
                   |
                R-PUSAT   (2 LAN: Pegawai, Server Balai Kota)
   +-------+-------+-------+-------+
 R-DC   R-BARAT  R-UTARA  R-TIMUR  R-SELATAN
                   |
              R-PELABUHAN
```

Tunnel GRE R-PUSAT <-> R-PELABUHAN (bukan kabel fisik, dibuat lewat konfigurasi
oleh C).

## Tabel subnetting VLSM (base 10.96.0.0/16)

Diurutkan dari kebutuhan host terbesar. Gateway = host pertama.

| Subnet | Host | Network ID | Prefix | Mask | Gateway | Broadcast |
|---|---|---|---|---|---|---|
| WiFi Publik Utara | 11.000 | 10.96.0.0 | /18 | 255.255.192.0 | 10.96.0.1 | 10.96.63.255 |
| WiFi Publik Selatan | 6.500 | 10.96.64.0 | /19 | 255.255.224.0 | 10.96.64.1 | 10.96.95.255 |
| CCTV Kota | 6.000 | 10.96.96.0 | /19 | 255.255.224.0 | 10.96.96.1 | 10.96.127.255 |
| WiFi Publik Timur | 5.500 | 10.96.128.0 | /19 | 255.255.224.0 | 10.96.128.1 | 10.96.159.255 |
| Sensor & SCADA Pelabuhan | 5.000 | 10.96.160.0 | /19 | 255.255.224.0 | 10.96.160.1 | 10.96.191.255 |
| WiFi Publik Barat | 4.500 | 10.96.192.0 | /19 | 255.255.224.0 | 10.96.192.1 | 10.96.223.255 |
| Pegawai Balai Kota | 1.800 | 10.96.224.0 | /21 | 255.255.248.0 | 10.96.224.1 | 10.96.231.255 |
| Sensor Banjir Timur | 700 | 10.96.232.0 | /22 | 255.255.252.0 | 10.96.232.1 | 10.96.235.255 |
| Sensor Lalu Lintas Utara | 450 | 10.96.236.0 | /23 | 255.255.254.0 | 10.96.236.1 | 10.96.237.255 |
| Puskesmas & Sekolah Barat | 220 | 10.96.238.0 | /24 | 255.255.255.0 | 10.96.238.1 | 10.96.238.255 |
| Server & Storage DC | 180 | 10.96.239.0 | /24 | 255.255.255.0 | 10.96.239.1 | 10.96.239.255 |
| Kantor Pelabuhan | 120 | 10.96.240.0 | /25 | 255.255.255.128 | 10.96.240.1 | 10.96.240.127 |
| PJU Pintar Selatan | 90 | 10.96.240.128 | /25 | 255.255.255.128 | 10.96.240.129 | 10.96.240.255 |
| Server Balai Kota | 20 | 10.96.241.0 | /27 | 255.255.255.224 | 10.96.241.1 | 10.96.241.31 |

### Link backbone (/30, mask 255.255.255.252)

Konvensi: **router atas = host 1, router bawah = host 2**.

| Link | Pasangan | Network ID | Host 1 (atas) | Host 2 (bawah) |
|---|---|---|---|---|
| L1 | R-PUSAT <-> R-DC | 10.96.241.32 | 10.96.241.33 | 10.96.241.34 |
| L2 | R-PUSAT <-> R-UTARA | 10.96.241.36 | 10.96.241.37 | 10.96.241.38 |
| L3 | R-PUSAT <-> R-SELATAN | 10.96.241.40 | 10.96.241.41 | 10.96.241.42 |
| L4 | R-PUSAT <-> R-TIMUR | 10.96.241.44 | 10.96.241.45 | 10.96.241.46 |
| L5 | R-PUSAT <-> R-BARAT | 10.96.241.48 | 10.96.241.49 | 10.96.241.50 |
| L6 | R-UTARA <-> R-PELABUHAN | 10.96.241.52 | 10.96.241.53 | 10.96.241.54 |

### Alamat tambahan (diberikan soal, bukan hasil UTS)

| Keperluan | Alamat |
|---|---|
| Link R-PUSAT ke R-ISP | 203.0.113.48/30 — R-ISP .49, R-PUSAT .50 |
| Server internet | 8.8.8.0/24 — server 8.8.8.8, gateway 8.8.8.1 di R-ISP |
| Tunnel GRE | 192.168.12.0/30 — R-PUSAT .1, R-PELABUHAN .2 |

### Sisa alamat

Alamat terakhir yang terpakai: **10.96.241.55**.
Sisa 10.96.241.56 ke atas masih kosong. **L7 untuk file OSPF pakai
10.96.241.56/30** (host .57 dan .58).

Total terpakai 61.752 alamat dari /16 (65.536), sekitar 94,2%, terbuang 3.784.

## Tabel port router

Semua router Cisco 2911 (G0/0–G0/2 bawaan). Backbone L1–L6 memakai kabel
**Serial DCE** dengan ujung DCE di router atas (`clock rate 64000`): R-PUSAT untuk
L1–L5, R-UTARA untuk L6. R-PUSAT pakai 3× HWIC-2T (slot 0/0, 0/1, 0/2), router
lain 1× HWIC-2T (slot 0/0), R-ISP tanpa modul. Router → switch pakai kabel
straight ke port **G0/1** switch; PC/server tetap di Fa0/2 switch.

| Router | Port | Fungsi | IP |
|---|---|---|---|
| R-ISP | G0/0 | ke R-PUSAT (cross) | 203.0.113.49/30 |
| | G0/1 | ke SRV-INTERNET (cross) | 8.8.8.1/24 |
| R-PUSAT | G0/0 | ke R-ISP (cross) | 203.0.113.50/30 |
| | G0/1 | LAN Pegawai Balai Kota | 10.96.224.1/21 |
| | G0/2 | LAN Server Balai Kota | 10.96.241.1/27 |
| | S0/0/0 (DCE) | L1 ke R-DC | 10.96.241.33/30 |
| | S0/0/1 (DCE) | L2 ke R-UTARA | 10.96.241.37/30 |
| | S0/1/0 (DCE) | L3 ke R-SELATAN | 10.96.241.41/30 |
| | S0/1/1 (DCE) | L4 ke R-TIMUR | 10.96.241.45/30 |
| | S0/2/0 (DCE) | L5 ke R-BARAT | 10.96.241.49/30 |
| | S0/2/1 | cadangan (kosong, shutdown) | — |
| R-DC | S0/0/0 | L1 ke R-PUSAT | 10.96.241.34/30 |
| | G0/0 | LAN CCTV Kota | 10.96.96.1/19 |
| | G0/1 | LAN Server & Storage DC | 10.96.239.1/24 |
| R-BARAT | S0/0/0 | L5 ke R-PUSAT | 10.96.241.50/30 |
| | G0/0 | LAN WiFi Publik Barat | 10.96.192.1/19 |
| | G0/1 | LAN Puskesmas & Sekolah | 10.96.238.1/24 |
| R-UTARA | S0/0/0 | L2 ke R-PUSAT | 10.96.241.38/30 |
| | S0/0/1 (DCE) | L6 ke R-PELABUHAN | 10.96.241.53/30 |
| | G0/0 | LAN WiFi Publik Utara | 10.96.0.1/18 |
| | G0/1 | LAN Sensor Lalu Lintas | 10.96.236.1/23 |
| R-TIMUR | S0/0/0 | L4 ke R-PUSAT | 10.96.241.46/30 |
| | G0/0 | LAN WiFi Publik Timur | 10.96.128.1/19 |
| | G0/1 | LAN Sensor Banjir Timur | 10.96.232.1/22 |
| R-SELATAN | S0/0/0 | L3 ke R-PUSAT | 10.96.241.42/30 |
| | G0/0 | LAN WiFi Publik Selatan | 10.96.64.1/19 |
| | G0/1 | LAN PJU Pintar Selatan | 10.96.240.129/25 |
| R-PELABUHAN | S0/0/0 | L6 ke R-UTARA | 10.96.241.54/30 |
| | G0/0 | LAN Sensor & SCADA | 10.96.160.1/19 |
| | G0/1 | LAN Kantor Pelabuhan | 10.96.240.1/25 |

## Nama perangkat

- Router: R-ISP, R-PUSAT, R-DC, R-BARAT, R-UTARA, R-TIMUR, R-SELATAN, R-PELABUHAN
- Switch: SW-PEGAWAI, SW-SERVER-BALAIKOTA, SW-CCTV, SW-SERVER-DC, SW-WIFI-BARAT,
  SW-PUSKESMAS, SW-WIFI-UTARA, SW-SENSOR-LALIN, SW-WIFI-TIMUR, SW-SENSOR-BANJIR,
  SW-WIFI-SELATAN, SW-PJU, SW-SCADA, SW-KANTOR-PELABUHAN
- Server: SRV-INTERNET, SRV-BALAIKOTA, SRV-DC
- PC: PC-PEGAWAI, PC-CCTV-KOTA, PC-PUSKESMAS, PC-SENSOR-LALIN, PC-SENSOR-BANJIR,
  PC-SCADA, PC-PJU, PC-KANTOR-PELABUHAN
- AP: AP-WIFI-BARAT, AP-WIFI-UTARA, AP-WIFI-TIMUR, AP-WIFI-SELATAN
- Smartphone: HP-WIFI-BARAT, HP-WIFI-UTARA, HP-WIFI-TIMUR, HP-WIFI-SELATAN

## Static routing yang sudah dipasang

### Router cabang — default route ke atas saja

| Router | Perintah |
|---|---|
| R-DC | `ip route 0.0.0.0 0.0.0.0 10.96.241.33` |
| R-BARAT | `ip route 0.0.0.0 0.0.0.0 10.96.241.49` |
| R-TIMUR | `ip route 0.0.0.0 0.0.0.0 10.96.241.45` |
| R-SELATAN | `ip route 0.0.0.0 0.0.0.0 10.96.241.41` |
| R-PELABUHAN | `ip route 0.0.0.0 0.0.0.0 10.96.241.53` (ke R-UTARA) |

### R-UTARA — default + rute ke LAN Pelabuhan

```
ip route 0.0.0.0 0.0.0.0 10.96.241.37
ip route 10.96.160.0 255.255.224.0 10.96.241.54
ip route 10.96.240.0 255.255.255.128 10.96.241.54
```

### R-PUSAT — 13 rute ke bawah + default ke ISP

```
ip route 10.96.96.0 255.255.224.0 10.96.241.34
ip route 10.96.239.0 255.255.255.0 10.96.241.34
ip route 10.96.192.0 255.255.224.0 10.96.241.50
ip route 10.96.238.0 255.255.255.0 10.96.241.50
ip route 10.96.0.0 255.255.192.0 10.96.241.38
ip route 10.96.236.0 255.255.254.0 10.96.241.38
ip route 10.96.241.52 255.255.255.252 10.96.241.38
ip route 10.96.160.0 255.255.224.0 10.96.241.38
ip route 10.96.240.0 255.255.255.128 10.96.241.38
ip route 10.96.128.0 255.255.224.0 10.96.241.46
ip route 10.96.232.0 255.255.252.0 10.96.241.46
ip route 10.96.64.0 255.255.224.0 10.96.241.42
ip route 10.96.240.128 255.255.255.128 10.96.241.42
ip route 0.0.0.0 0.0.0.0 203.0.113.49
```

## Catatan teknis yang sudah dipelajari

- **GUI vs CLI**: tab Config bisa untuk IP dan static route, tapi TIDAK menyimpan
  ke startup-config. Setelah pakai GUI, wajib `enable` lalu `write memory` di CLI.
  NAT, DHCP relay, GRE, dan OSPF hanya bisa lewat CLI.
- **Kesalahan paling sering**: lupa centang kotak **On** di Port Status (akibatnya
  port `administratively down`), dan mask ditebak otomatis oleh Packet Tracer
  (selalu cek ulang, terutama /18, /19, /22, /23, /25).
- **`show cdp neighbors`** dipakai untuk memastikan kabel menuju port yang benar.
- **Kenapa ganti dari Router-PT ke 2911** (27 Okt 2026): Router-PT (IOS 12.2 PT1000)
  tidak punya `interface Tunnel` (GRE soal 6 mustahil) dan `crypto` hanya ada `key`
  (IPsec bonus mustahil). 2911 (IOS 15.1) punya GRE bawaan (`ipbasek9`); IPsec
  butuh `license boot module c2900 technology-package securityk9` lalu `reload`.
  Alasan ini perlu ditulis di dokumentasi (soal 1, desain bebas tapi wajib alasan).
- **Serial**: `clock rate` hanya di sisi DCE. Cek dengan `show controllers s0/0/0`
  (harus `DCE V.35, clock rate 64000` di R-PUSAT / R-UTARA-L6). Serial up/down =
  clock rate belum diset.
- **`no ip domain-lookup`** sudah dipasang di semua router supaya CLI tidak macet
  saat salah ketik perintah.
- **Next hop selalu alamat router seberang**, bukan alamat sendiri.
- **Ping ke 203.0.113.49 dari router cabang gagal** itu normal sebelum NAT ada,
  karena R-ISP tidak punya rute balik ke alamat privat (memang diminta soal 5).
- Status `up/down` artinya port hidup tapi lawannya belum aktif.

## Soal 7 — OSPF (sekarang bagian B)

Dikerjakan di **file terpisah**, hasil copy dari master yang SUDAH lengkap
(DHCP, NAT, GRE, layanan sudah masuk), supaya semua ikut terbawa.

Ketentuan dari soal:
1. Tambah link cadangan **L7 antara R-DC dan R-UTARA**, pakai alamat yang belum
   terpakai: **10.96.241.56/30**. Port serial R-DC `S0/0/1` masih kosong; R-UTARA
   kedua port serialnya terpakai (L2, L6), jadi pasang 1 HWIC-2T lagi di R-UTARA
   slot 0/1 → `S0/1/0`. Simpan config dulu sebelum pasang modul (router mati).
   Pola OSPF di modul: `router-id`, `network ... area 0`, `passive-interface` ke
   LAN, `default-information originate` di R-PUSAT.
2. Ganti seluruh static route antar-router dengan OSPF.
3. Router tidak mengirim pesan OSPF ke arah LAN pengguna (`passive-interface`).
4. Default route ke internet hanya dikonfigurasi di R-PUSAT, router lain
   mendapatkannya otomatis lewat OSPF (`default-information originate`).
5. Kalau salah satu link backbone putus, konektivitas harus pulih otomatis.

## Output yang dikumpulkan (semua dalam 1 ZIP)

Nama ZIP: `MidProject Jarkom_K12_<3 angka NRP terakhir 1>_<2>_<3>.ZIP`

- `K12_VLSM.pkt` — file utama
- `K12_VLSM_OSPF.pkt` — file OSPF (soal 7)
- Spreadsheet .xlsx: tabel VLSM, tabel CIDR, supernet per router, tabel revisi
- Dokumentasi link GitHub Markdown: screenshot topologi, alasan memilih VLSM,
  penjelasan desain topologi, penjelasan semua konfigurasi, bukti pengujian
  tiap soal

## Hal yang masih perlu diputuskan kelompok

1. Server email ditaruh di Balai Kota atau DC (soal tidak menyebutkan)
2. Sheet CIDR di spreadsheet masih kosong — C perlu mengisinya, termasuk
   supernet per router
3. Apakah ada perbaikan dari jawaban UTS yang perlu masuk tabel revisi

## Cara kerja bareng

File .pkt tidak bisa digabung. Satu file master dipegang A. B dan C bekerja di
salinan lalu mengirim konfigurasi dalam bentuk **teks CLI**, yang ditempel ke
master. Simpan tiap config router di folder `config/` di repo, sekaligus jadi
bahan penjelasan konfigurasi di dokumentasi.
