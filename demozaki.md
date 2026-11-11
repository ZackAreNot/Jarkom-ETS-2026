# Panduan Demo Zaki — Soal 1, 3, 4, dan 5

**File demo:** `K12_VLSM_master_GRE.pkt` · **Kelompok:** 12 · **Skema:** VLSM `10.96.0.0/16`

Panduan ini untuk Uji 1 kelompok. Seluruh demo kelompok mencakup soal 1–8 dalam **15 menit**, jadi koordinasikan waktu dengan Chiko dan anggota lainnya. Angka IP DHCP dapat berubah; cek alamat terkini dengan `ipconfig /all` sebelum mengetik alamat tujuan.

## Persiapan

1. Buka Packet Tracer → **File → Open** → pilih `K12_VLSM_master_GRE.pkt`. Pastikan judul jendela sesuai. Jangan memakai file master lama atau file OSPF.
2. Tunggu lampu koneksi hijau. Siapkan `README.md` dan spreadsheet bila asisten meminta tabel.
3. Untuk membuka router: klik router → **CLI** → Enter → ketik `enable`. Prompt `R-NAMA#` berarti siap menerima perintah `show`.
4. Untuk membuka PC: klik PC → **Desktop → Command Prompt**. Ketik perintah setelah `C:\>`.
5. Ping pertama dapat kehilangan beberapa paket karena ARP atau negosiasi IPsec; ulangi dan laporkan hasil ulangnya. Jangan menganggap semua timeout pasti normal.

## Checklist ping saat demo

Sebelum mulai, buka setiap PC tujuan → **Desktop → Command Prompt** → `ipconfig /all` untuk memastikan alamat DHCP-nya masih sama. Lalu jalankan tes berikut dari Command Prompt perangkat asal:

| Soal | Perangkat asal | Perintah | Bukti yang ditunjukkan |
|---|---|---|---|
| 3 | PC-CCTV-KOTA | `ping 10.96.240.11` | Reply dari PC-KANTOR-PELABUHAN: koneksi antar-LAN. |
| 5 | PC-CCTV-KOTA | `ping 8.8.8.8` | Reply dari server internet melalui NAT. |
| 5 | PC-PEGAWAI | `ping 8.8.8.8` | Akses internet dari wilayah Balai Kota. |
| 5 | PC-KANTOR-PELABUHAN | `ping 8.8.8.8` | Akses internet dari wilayah Pelabuhan. |

Tunjukkan baris `Reply from ...` dan statistik `Sent`, `Received`, `Lost`. Targetnya **4/4 reply**. Jika percobaan pertama sebagian timeout, ulangi sekali; bila tetap gagal, lakukan pemeriksaan pada bagian terkait di bawah. Sesudah ping ke `8.8.8.8`, periksa `show ip nat translations` di R-PUSAT untuk membuktikan translasi PAT.

## Soal 1 — Topologi (10 poin)

**Yang ditunjukkan:** tampilkan keseluruhan topologi di tab **Logical**, lalu tunjuk R-ISP, R-PUSAT, lima cabang, R-PELABUHAN di bawah R-UTARA, 14 LAN, L1–L6, serta server internet.

**Yang disampaikan:**
> “Kami memakai VLSM dari blok `10.96.0.0/16`, karena kebutuhan host setiap LAN berbeda. Backbone mengikuti gambar soal: pusat terhubung ke DC, Utara, Selatan, Timur, dan Barat; Pelabuhan terhubung lewat Utara. Gateway tiap LAN adalah host pertama. Kami memakai 2911 dan modul HWIC-2T untuk link serial; port DCE diberi clock rate. Setiap LAN diwakili satu perangkat, tetapi subnet dihitung untuk kebutuhan host penuh.”

Tunjukkan label yang memuat nama LAN, network/prefix, jumlah host; nama perangkat; switch, PC, AP/smartphone, dan server. Contoh: CCTV `10.96.96.0/19`, gateway `10.96.96.1`, kebutuhan 6.000 host. AP dipakai agar smartphone terhubung nirkabel; switch menyambungkan perangkat dalam LAN; Server-PT mewakili server.

### Test 1.1 — Status antarmuka R-PUSAT

Buka **R-PUSAT → CLI**, lalu:

```text
show ip interface brief
```

**Yang dicari:** G0/0 `203.0.113.50`, G0/1 `10.96.224.1`, G0/2 `10.96.241.1`, serial yang dipakai berstatus `up/up`. Tunnel0 `192.168.12.1` juga ada karena file ini sudah mengerjakan soal 6. Port cadangan `administratively down` boleh karena tidak dipakai.

### Test 1.2 — Tetangga dan DCE

Masih di R-PUSAT:

```text
show cdp neighbors
show controllers serial0/0/0
```

**Yang dicari:** R-DC pada S0/0/0 dan sisi serial menunjukkan `DCE` dengan clock rate `64000`. CDP menunjukkan tetangga, bukan membuktikan end-to-end ping. Jika output panjang, baca sampai bagian DCE.

## Soal 3 — Static routing (15 poin)

**Yang disampaikan:**
> “Router cabang menggunakan default route ke arah atas. R-PUSAT punya rute ke LAN di bawahnya; R-UTARA punya rute menuju LAN Pelabuhan. Pada file GRE, rute R-PUSAT untuk dua LAN Pelabuhan sudah diarahkan ke Tunnel0; ini tetap static route. Rute fisik melalui R-UTARA disimpan sebagai cadangan.”

### Test 3.1 — Cek rute

1. Pada **R-DC → CLI**:

```text
show ip route
```

Cari `S* 0.0.0.0/0` melalui `10.96.241.33`, yaitu R-PUSAT.

2. Pada **R-UTARA → CLI**:

```text
show ip route static
```

Cari dua LAN Pelabuhan `10.96.160.0/19` dan `10.96.240.0/25` melalui `10.96.241.54`, serta default ke `10.96.241.37`.

3. Pada **R-PUSAT → CLI**:

```text
show ip route static
```

Cari rute menuju cabang dan default ke `203.0.113.49`. Dua LAN Pelabuhan aktif melalui `192.168.12.2`; Chiko menjelaskan tunnel pada soal 6. `S` berarti static; `S*` menandai default route.

### Test 3.2 — Ping antarwilayah

Pada **PC-CCTV-KOTA → Desktop → Command Prompt**:

```text
ping 10.96.240.11
```

`10.96.240.11` adalah IP Kantor Pelabuhan yang teramati saat pengecekan; pastikan lagi pada PC tujuan. **Lulus:** `Reply from ...`, idealnya 4/4 setelah pengulangan. Bila gagal, cek IP/gateway kedua PC, interface `up/up`, dan rute kedua arah.

**Catatan penting:** untuk membuktikan jalur GRE, Chiko menggunakan trafik Balai Kota ↔ Pelabuhan. CCTV ↔ Pelabuhan bukan pasangan LAN yang diwajibkan melewati GRE.

## Soal 4 — DHCP (10 poin)

**Yang disampaikan:**
> “IP interface router dan server statis. PC dan smartphone memperoleh IP, subnet mask, gateway, dan DNS secara otomatis. Router membagikan DHCP untuk LAN pengguna, kecuali CCTV Kota yang mendapat alamat dari SRV-DC melalui DHCP relay.”

### Test 4.1 — PC biasa

Pada **PC-PEGAWAI → Desktop → IP Configuration**, tunjuk pilihan **DHCP**. Pada **Command Prompt**:

```text
ipconfig /all
```

Cari pada **FastEthernet0**: alamat `10.96.224.11` (dapat berubah), mask `255.255.248.0`, gateway/DHCP server `10.96.224.1`, DNS `10.96.239.2`. Jangan keliru membaca bagian Bluetooth.

Pada **R-PUSAT → CLI**:

```text
show ip dhcp pool
show ip dhcp binding
show running-config | include excluded
```

Jelaskan `excluded-address 10.96.224.1 10.96.224.10` mencadangkan gateway dan alamat statis; binding menunjukkan alamat yang telah dibagikan.

### Test 4.2 — CCTV melalui relay

Pada **PC-CCTV-KOTA → Command Prompt**:

```text
ipconfig /all
```

Cari alamat dalam `10.96.96.0/19`, gateway `10.96.96.1`, DNS `10.96.239.2`, dan **DHCP Servers `10.96.239.2`**. Nilai yang diamati: `10.96.96.11`.

Pada **R-DC → CLI**:

```text
show running-config | section interface GigabitEthernet0/0
```

Cari `ip helper-address 10.96.239.2`. Lalu buka **SRV-DC → Services → DHCP** dan tunjuk service **On**, pool `DHCP-CCTV`, start `10.96.96.11`, gateway `10.96.96.1`, dan DNS `10.96.239.2`.

**Jawaban singkat:** DHCP Discover adalah broadcast dan tidak diteruskan router secara biasa. `ip helper-address` meneruskannya ke server DHCP di subnet lain. DNS yang dibagikan adalah alamat server; keberhasilan layanan DNS merupakan pengujian soal 8.

## Soal 5 — NAT/PAT (5 poin)

**Yang disampaikan:**
> “Semua LAN internal memakai satu alamat publik R-PUSAT, `203.0.113.50`. PAT overload membedakan sesi berdasarkan informasi transport/identifikasi. R-ISP tidak memiliki route ke blok privat `10.96.0.0/16`.”

### Test 5.1 — Akses server internet

Pada **PC-CCTV-KOTA → Command Prompt**:

```text
ping 8.8.8.8
```

Ulangi jika pertama hanya sebagian reply. Sebagai perwakilan wilayah lain, ulangi dari **PC-PEGAWAI** dan **PC-KANTOR-PELABUHAN**. **Lulus:** reply dari `8.8.8.8`; ini simulasi server di Packet Tracer, bukan tes internet sebenarnya.

### Test 5.2 — Tunjukkan translasi

Segera setelah ping, buka **R-PUSAT → CLI**:

```text
show ip nat translations
show ip nat statistics
```

Cari **Inside local** `10.96.96.11` (IP CCTV saat itu), **Inside global** `203.0.113.50`, dan tujuan `8.8.8.8`. Entri NAT dapat kedaluwarsa, jadi jalankan ping lagi jika tabel kosong.

### Test 5.3 — Pastikan ISP tidak punya rute privat

Pada **R-ISP → CLI**:

```text
show ip route
```

Cari jaringan connected `8.8.8.0/24` dan `203.0.113.48/30`; pastikan tidak ada rute `10.96.0.0/16` atau subnet privat internal. Bisa juga ping IP privat dari SRV-INTERNET sebagai tes tambahan, tetapi ping gagal saja tidak membuktikan jaringan aman dari seluruh serangan.

## Tanya jawab dan penanganan cepat Uji 2

- **Kenapa /19 untuk CCTV 6.000 host?** `/19` punya 8.190 alamat host; `/20` hanya 4.094, tidak cukup.
- **Apa beda gateway dan DNS?** Gateway meneruskan paket ke jaringan lain; DNS menerjemahkan nama menjadi IP.
- **Kenapa IP server statis?** Agar alamat tujuan DHCP relay, DNS, dan layanan tetap diketahui.
- **Ping gagal?** `ipconfig /all` → `show ip interface brief` → ping gateway/tetangga → `show ip route` → cek return route; untuk 8.8.8.8 tambah `show ip nat translations` dan route ISP.
- **DHCP gagal?** Cek port dan VLAN, pool/excluded, lalu helper serta DHCP server untuk CCTV. Periksa apakah host memang mode DHCP.
- Jika asisten meminta perbaikan, jelaskan penyebabnya, perbaiki hanya yang bermasalah, lalu ulangi test. Jangan menghapus konfigurasi sebagai demonstrasi tanpa diminta.

**Hasil pengecekan saat penulisan:** PC CCTV dan Pegawai menerima IP/gateway/DNS yang sesuai; ping CCTV ke 8.8.8.8 sempat 2/4 pada percobaan pertama, sehingga ulangi saat demo. Entri PAT CCTV → `203.0.113.50` terlihat di R-PUSAT. Hasil percobaan bukan jaminan status saat demo.
