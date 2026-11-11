# Panduan Demo Chiko — Soal 6: GRE dan Bonus IPsec

**File demo:** `K12_VLSM_master_GRE.pkt` · **Kelompok:** 12 · **Tunnel:** `192.168.12.0/30`

Target soal: tunnel GRE R-PUSAT ↔ R-PELABUHAN; trafik antara LAN Balai Kota dan LAN Pelabuhan melewati tunnel; jelaskan pemilihan rute dan mengapa GRE saja tidak mengenkripsi. IPsec adalah bonus +5. Alokasi waktu seluruh demo kelompok soal 1–8 adalah **15 menit**.

## Persiapan

1. Buka file GRE tersebut di Packet Tracer dan tunggu interface stabil. Pada PC, buka **Desktop → Command Prompt**; pada router, buka **CLI**, tekan Enter lalu `enable`.
2. Periksa IP yang sedang didapat perangkat dengan `ipconfig /all`. Saat pengecekan: PC-PEGAWAI `10.96.224.11`, PC-KANTOR-PELABUHAN `10.96.240.11`, gateway masing-masing `10.96.224.1` dan `10.96.240.1`. IP DHCP bisa berubah.
3. Setelah idle atau membuka file, counter IPsec bisa 0. Kirim trafik antara kedua LAN dahulu untuk memicu negosiasi, lalu cek lagi. Pengujian awal sempat 2/4 reply, pengulangan 4/4; jika terus gagal, lakukan diagnosis di bawah.

## Checklist ping dan tracert saat demo

Pastikan dulu IP PC tujuan melalui **Desktop → Command Prompt → `ipconfig /all`**, karena alamat DHCP dapat berubah. Pada Command Prompt perangkat asal, jalankan:

| Urutan | Perangkat asal | Perintah | Hasil yang ditunjukkan |
|---|---|---|---|
| 1 | PC-PEGAWAI | `ping 10.96.240.11` | Reply dari PC-KANTOR-PELABUHAN. |
| 2 | PC-PEGAWAI | `tracert 10.96.240.11` | Ada hop `192.168.12.2`, ujung tunnel Pelabuhan. |
| 3 | PC-KANTOR-PELABUHAN | `ping 10.96.224.11` | Reply kembali dari PC-PEGAWAI. |
| 4 | PC-KANTOR-PELABUHAN | `tracert 10.96.224.11` | Cari hop `192.168.12.1`, ujung tunnel Pusat. |

Target ping **4/4 reply**. Jika percobaan pertama sebagian timeout, ulangi sekali. Ping membuktikan konektivitas; tracert dan tabel routing menunjukkan jalur tunnel. Untuk bonus IPsec, lakukan ping lagi sambil membandingkan counter `encrypt` dan `decrypt` pada `show crypto ipsec sa` di R-PUSAT.

## Pembuka: apa yang dipresentasikan

> “Tunnel GRE membuat hubungan virtual antara R-PUSAT dan R-PELABUHAN. Walaupun secara fisik paket melewati R-UTARA, trafik dari Balai Kota ke Pelabuhan dirutekan melalui interface Tunnel0. IPsec ditambahkan untuk mengenkripsi dan memverifikasi integritas trafik GRE.”

| Sisi | IP Tunnel0 | Interface sumber fisik | Tujuan fisik |
|---|---|---|---|
| R-PUSAT | `192.168.12.1/30` | `Serial0/0/1`, `10.96.241.37` | `10.96.241.54` |
| R-PELABUHAN | `192.168.12.2/30` | `Serial0/0/0`, `10.96.241.54` | `10.96.241.37` |

Jelaskan **underlay** adalah jalur fisik PUSAT → UTARA → PELABUHAN; **overlay** adalah Tunnel0 di atas jalur itu. GRE tidak menggantikan kabel fisik.

## Test 6.1 — Cek interface tunnel dan konfigurasi kedua ujung

Pada **R-PUSAT → CLI**:

```text
show ip interface brief
show interface tunnel0
show running-config | section interface Tunnel
```

Cari Tunnel0 `192.168.12.1`, status `up/up`, source `Serial0/0/1`, destination `10.96.241.54`. Pada **R-PELABUHAN** jalankan perintah yang sama: Tunnel0 `192.168.12.2`, source `Serial0/0/0`, destination `10.96.241.37`, juga `up/up`.

**Lulus:** alamat kedua ujung benar dan tunnel up/up. Status up/up saja belum membuktikan trafik LAN melewati tunnel atau terenkripsi; lanjutkan test berikutnya.

## Test 6.2 — Buktikan rute dipilih

Pada **R-PUSAT → CLI**:

```text
show ip route static
show running-config | include ip route
```

Cari dua rute aktif:

```text
10.96.160.0/19 via 192.168.12.2
10.96.240.0/25 via 192.168.12.2
```

Di running-config, kedua rute juga mempunyai cadangan melalui `10.96.241.38` dengan **administrative distance 200**. Untuk prefix yang sama, static route tunnel dengan AD bawaan **1** dipilih dibandingkan AD 200. Rute cadangan yang tidak terpilih biasanya tidak muncul di `show ip route`.

Pada **R-PELABUHAN → CLI**:

```text
show ip route static
```

Cari `10.96.224.0/21` dan `10.96.241.0/27` melalui `192.168.12.1`; default tetap melalui R-UTARA `10.96.241.53`. Dua rute Balai Kota menang karena **lebih spesifik** daripada default `/0` — longest prefix match diperiksa sebelum AD untuk prefix yang berbeda.

R-PUSAT juga memiliki rute fisik `10.96.241.52/30` via `10.96.241.38` untuk mencapai endpoint `10.96.241.54`. Tanpa keterjangkauan endpoint fisik, tunnel tidak dapat membawa trafik.

## Test 6.3 — Ping dan tracert dari Balai Kota

Pada **PC-PEGAWAI → Desktop → Command Prompt**:

```text
ping 10.96.240.11
```

Ulangi jika pertama timeout; **lulus** bila ada reply yang stabil. Lanjut:

```text
tracert 10.96.240.11
```

Hasil yang teramati saat pengecekan:

```text
1  10.96.224.1    R-PUSAT, gateway Pegawai
2  192.168.12.2   ujung Tunnel0 R-PELABUHAN
3  10.96.240.11   PC-KANTOR-PELABUHAN
```

**Yang dikatakan:** “Hop `192.168.12.2` menunjukkan paket IP internal menggunakan tunnel. R-UTARA tidak terlihat sebagai hop internal, tetapi tetap dilewati oleh paket pembungkus GRE di jaringan fisik.”

Uji balik dari **PC-KANTOR-PELABUHAN**:

```text
ping 10.96.224.11
tracert 10.96.224.11
```

Cari hop tunnel `192.168.12.1` pada arah balik. Bila IP PC berubah, gunakan hasil `ipconfig /all` terbaru. Ping antar-PC membuktikan konektivitas dua arah; gunakan routing dan tracert sebagai bukti pemilihan tunnel.

## Bonus +5 — Buktikan IPsec bekerja

**GRE saja tidak mengenkripsi:** GRE melakukan enkapsulasi; isi paket tidak otomatis menjadi rahasia. IPsec memberi enkripsi dan pemeriksaan integritas/autentikasi sesuai algoritma yang dikonfigurasi. Jangan menyamakan sekadar status tunnel up dengan bukti enkripsi.

1. Setelah mengirim ping antar-LAN di Test 6.3, buka **R-PUSAT → CLI**:

```text
show crypto isakmp sa
show crypto ipsec sa
```

2. Cari sesi ISAKMP dengan peer `10.96.241.54`, state `QM_IDLE` dan status aktif. Pada IPsec SA cari peer yang sama, **Status ACTIVE**, serta `#pkts encrypt` dan `#pkts decrypt`.
3. Catat angka counter. Kirim lagi `ping` dari PC-PEGAWAI ke PC-KANTOR-PELABUHAN. Jalankan lagi:

```text
show crypto ipsec sa
```

**Lulus:** kedua counter bertambah setelah trafik dua arah. Pada pengecekan, setelah ping muncul sesi `QM_IDLE` dan counter encrypt 13/decrypt 12; angka demo bisa berbeda atau mulai dari nol. Counter naik bersama sesi aktif adalah bukti kerja IPsec di simulasi.

4. Opsional, ulangi `show crypto isakmp sa` dan `show crypto ipsec sa` pada R-PELABUHAN untuk melihat sisi seberang. Tanpa membacakan atau membagikan pre-shared key, jelaskan kedua peer menggunakan key yang cocok dan konfigurasi kripto yang sesuai.

## Pertanyaan yang mungkin ditanyakan

- **Mengapa via tunnel?** R-PUSAT punya rute spesifik menuju dua LAN Pelabuhan via `192.168.12.2` AD 1; rute dengan prefix sama via Utara AD 200 menjadi cadangan. Baliknya, rute ke LAN Balai Kota via tunnel lebih spesifik daripada default.
- **Mengapa R-UTARA tidak muncul di tracert?** Tracert melihat hop paket internal; R-UTARA meneruskan paket luar hasil enkapsulasi. Jalur fisik tetap melewatinya.
- **GRE vs IPsec?** GRE membuat dan membungkus trafik tunnel; IPsec melindungi trafik dengan enkripsi dan integritas. GRE sendiri bukan enkripsi.
- **Tunnel down?** Cek `show ip interface brief`, source/destination kedua ujung, status serial dan rute ke endpoint fisik. Jangan membuat route menuju destination fisik melalui tunnel itu sendiri: berisiko recursive routing.
- **Tunnel up tetapi IPsec kosong?** Kirim trafik yang sesuai, cek kembali `show crypto isakmp sa` dan `show crypto ipsec sa`, peer, kebijakan kedua sisi, ACL trafik GRE, dan crypto map pada interface fisik.
- **Ping pertama timeout?** Bisa terjadi saat ARP atau negosiasi SA; ulangi. Jika tetap gagal, periksa rute kedua arah, ACL dan error counter, bukan langsung menganggap normal.

## Catatan demo

Cadangan AD 200 tidak perlu diuji dengan mematikan interface saat presentasi kecuali diminta asisten. Bila diminta, lakukan pada salinan dan pahami dampaknya: putusnya jalur fisik yang sama bisa ikut menjatuhkan tunnel maupun rute cadangan. Jangan menyatakan failover sudah berhasil tanpa mengujinya. Soal 7 OSPF adalah file dan pengujian terpisah.
