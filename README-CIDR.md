# Laporan dan Panduan Demo GNS3 CIDR Kelompok K-32

**Proyek:** K-32-MODUL-4  
**Mata praktikum:** Modul 4 Komunikasi Data dan Jaringan Komputer  
**Metode:** CIDR pada GNS3; VLSM dikerjakan terpisah pada Cisco Packet Tracer  
**Prefix kelompok:** 192.227.x.x  
**Blok induk hasil penggabungan:** 192.227.0.0/19  
**Acuan:** script terbaru yang memiliki opsi `--apply-config` dan `--check-mac`, tabel alokasi, serta pohon CIDR yang diberikan.

> Dokumen ini menjelaskan rancangan dan konfigurasi pada script. Keberhasilan koneksi di server harus dibuktikan dengan pengujian. Jangan menyebut seluruh node sudah terhubung sebelum hasil pengujiannya tersedia.

## 1. Daftar isi

1. Daftar isi
2. [Tujuan dan gambaran topologi](#2-tujuan-dan-gambaran-topologi)
3. [Konsep subnet dan CIDR](#3-konsep-subnet-dan-cidr)
4. [Menghitung kebutuhan dan prefix subnet](#4-menghitung-kebutuhan-dan-prefix-subnet)
5. [Penggabungan CIDR sampai /19](#5-penggabungan-cidr-sampai-19)
6. [Membaca pohon dan membagi alamat](#6-membaca-pohon-dan-membagi-alamat)
7. [Tabel pembagian alamat](#7-tabel-pembagian-alamat)
8. [Penetapan IP node dan interface](#8-penetapan-ip-node-dan-interface)
9. [Routing dan perjalanan paket](#9-routing-dan-perjalanan-paket)
10. [Implementasi dan konfigurasi permanen](#10-implementasi-dan-konfigurasi-permanen)
11. [Pengujian dan bukti demo](#11-pengujian-dan-bukti-demo)
12. [Naskah demo 5–7 menit](#12-naskah-demo-57-menit)
13. [Pertanyaan asisten dan jawaban](#13-pertanyaan-asisten-dan-jawaban)
14. [Troubleshooting dan lembar hasil](#14-troubleshooting-dan-lembar-hasil)



## 2. Tujuan dan gambaran topologi

Tujuannya adalah menyediakan alamat sesuai kebutuhan host, menghubungkan semua node, dan membuat routing yang ringkas. Cyberia menjadi satu-satunya router yang terhubung ke NAT. Wired menghubungkan cabang utama.

Diagram ini menyederhanakan switch LAN agar hubungan antarrouter terlihat jelas. Switch2 ditampilkan karena menghubungkan empat router dalam subnet yang sama.

```text
NAT1
└── Cyberia
    ├── A1: JJ, MyuMyu
    └── A2: Wired
        ├── A3: Switch2 (Wired, Psyche, Copland, Schumann)
        │   ├── Psyche
        │   │   └── A4: Karl, Lin
        │   ├── Copland
        │   │   ├── A5: Juri
        │   │   └── A6: Masayuki
        │   └── Schumann
        │       ├── A7: Akashic
        │       └── A8: Accela
        │           └── A9: Omnipresence, Denpa
        ├── A10: Navi
        │   ├── A11: Alice, Reika
        │   ├── A12: Taro
        │   └── A13: Yasuo, Miho, Kuma
        └── A14: Tachibana
            ├── A15: Collective, Ghost
            └── A16: Deus
                ├── A17: Lilith, Specter
                └── A18: Protocol7
                    ├── A19: Chisa, Hodgeson
                    └── A20: Knights
                        ├── A21: Eiri
                        └── A22: Phantoma
                            ├── A23: Semi
                            └── A24: Shibuya
                                └── A25: Lain, Mika
```

Node Karl mewakili kebutuhan 700 host, tetapi script hanya membuat satu node bernama Karl. Jumlah host pada soal digunakan untuk menyediakan kapasitas subnet, bukan untuk membuat 700 container.

## 3. Konsep subnet dan CIDR

### Menentukan batas subnet

Satu subnet menghubungkan interface yang berada dalam jaringan layer 2 yang sama. Router memisahkan jaringan IP. Switch layer 2 tidak membentuk subnet baru pada setiap portnya.

Contoh A3 berisi interface Wired, Psyche, Copland, dan Schumann yang tersambung melalui Switch2. Jadi A3 membutuhkan **4 alamat interface router**. Switch bawaan GNS3 tidak membutuhkan IP untuk meneruskan trafik pada rancangan ini.

### Arti prefix

IPv4 berukuran 32 bit. Pada `/26`, sebanyak 26 bit adalah bagian network dan 6 bit adalah bagian host.

```text
Jumlah alamat dalam blok = 2^(32 − prefix)
Kapasitas host biasa     = 2^(32 − prefix) − 2
```

Dua alamat dicadangkan untuk network ID dan broadcast. Perhitungan ini memakai subnet biasa /30 sampai /22 sesuai rancangan praktikum; bukan skema point-to-point /31.

| Prefix | Netmask | Jumlah alamat blok | Kapasitas host |
|---|---|---:|---:|
| /30 | 255.255.255.252 | 4 | 2 |
| /29 | 255.255.255.248 | 8 | 6 |
| /28 | 255.255.255.240 | 16 | 14 |
| /27 | 255.255.255.224 | 32 | 30 |
| /26 | 255.255.255.192 | 64 | 62 |
| /25 | 255.255.255.128 | 128 | 126 |
| /24 | 255.255.255.0 | 256 | 254 |
| /23 | 255.255.254.0 | 512 | 510 |
| /22 | 255.255.252.0 | 1.024 | 1.022 |

### CIDR dan VLSM dalam konteks praktikum

CIDR berarti Classless Inter-Domain Routing: jaringan ditentukan dengan prefix dan dapat diagregasi tanpa terikat kelas A/B/C. Pada pengerjaan CIDR praktikum ini, subnet digabung bertahap berdasarkan struktur topologi, lalu alamat dibagi kembali menggunakan pohon.

VLSM menggunakan ukuran subnet yang berbeda sesuai kebutuhan, biasanya dengan mengalokasikan subnet terbesar terlebih dahulu. Secara konsep CIDR dan VLSM saling berkaitan; perbedaan alur pengerjaan di praktikum tidak berarti CIDR harus memakai satu ukuran subnet saja.

**Kesesuaian yang harus dijaga:** tabel CIDR cocok dengan GNS3, dan tabel VLSM cocok dengan CPT. Penempatan alamat hasil kedua metode dapat berbeda.

## 4. Menghitung kebutuhan dan prefix subnet

```text
Kebutuhan IP = jumlah host pada subnet + jumlah interface router pada subnet
```

Pilih prefix dengan kapasitas host paling kecil yang tetap mencukupi kebutuhan. Jangan menambahkan network dan broadcast ke kolom kebutuhan lalu menguranginya lagi secara keliru: keduanya sudah diperhitungkan saat membandingkan kebutuhan dengan kapasitas `2^h − 2`.

| Subnet | Anggota/rute | Perhitungan kebutuhan | Kebutuhan | Prefix | Kapasitas host |
|---|---|---|---:|---:|---:|
| A1 | Cyberia–Switch1–JJ, MyuMyu | 95 + 30 + 1 | 126 | /25 | 126 |
| A2 | Cyberia–Wired | 2 interface router | 2 | /30 | 2 |
| A3 | Wired–Switch2–Psyche, Copland, Schumann | 4 interface router | 4 | /29 | 6 |
| A4 | Psyche–Switch3–Karl, Lin | 700 + 180 + 1 | 881 | /22 | 1.022 |
| A5 | Copland–Juri | 31 + 1 | 32 | /26 | 62 |
| A6 | Copland–Switch4–Masayuki | 300 + 1 | 301 | /23 | 510 |
| A7 | Schumann–Akashic | 3 + 1 | 4 | /29 | 6 |
| A8 | Schumann–Accela | 2 interface router | 2 | /30 | 2 |
| A9 | Accela–Switch5–Omnipresence, Denpa | 510 + 3 + 1 | 514 | /22 | 1.022 |
| A10 | Wired–Navi | 2 interface router | 2 | /30 | 2 |
| A11 | Navi–Switch6–Alice, Reika | 208 + 45 + 1 | 254 | /24 | 254 |
| A12 | Navi–Switch7–Taro | 57 + 1 | 58 | /26 | 62 |
| A13 | Navi–Switch8–Yasuo, Miho, Kuma | 120 + 18 + 4 + 1 | 143 | /24 | 254 |
| A14 | Wired–Tachibana | 2 interface router | 2 | /30 | 2 |
| A15 | Tachibana–Switch9–Collective, Ghost | 260 + 6 + 1 | 267 | /23 | 510 |
| A16 | Tachibana–Deus | 2 interface router | 2 | /30 | 2 |
| A17 | Deus–Switch10–Lilith, Specter | 77 + 21 + 1 | 99 | /25 | 126 |
| A18 | Deus–Protocol7 | 2 interface router | 2 | /30 | 2 |
| A19 | Protocol7–Switch11–Chisa, Hodgeson | 23 + 9 + 1 | 33 | /26 | 62 |
| A20 | Protocol7–Knights | 2 interface router | 2 | /30 | 2 |
| A21 | Knights–Eiri | 2 + 1 | 3 | /29 | 6 |
| A22 | Knights–Phantoma | 2 interface router | 2 | /30 | 2 |
| A23 | Phantoma–Switch12–Semi | 14 + 1 | 15 | /27 | 30 |
| A24 | Phantoma–Shibuya | 2 interface router | 2 | /30 | 2 |
| A25 | Shibuya–Switch13–Lain, Mika | 57 + 4 + 1 | 62 | /26 | 62 |
| **Total** | | | **2.814** | | |

### Contoh hitungan yang sebaiknya dijelaskan

**A4:** kebutuhan 881. Sembilan bit host menghasilkan `2^9 − 2 = 510`, tidak cukup. Sepuluh bit menghasilkan `2^10 − 2 = 1.022`, cukup. Prefixnya `32 − 10 = /22`.

**A9:** kebutuhan 514. Walaupun Omnipresence sendiri membutuhkan 510, masih ada 3 host Denpa dan 1 interface Accela. /23 hanya menampung 510 alamat host, sehingga harus /22.

**A25:** kebutuhan 62. Enam bit host menghasilkan `2^6 − 2 = 62`, tepat. Prefixnya /26.

**A23:** kebutuhan 15. /28 mempunyai 16 alamat total tetapi hanya 14 alamat host. Karena itu digunakan /27 dengan 30 alamat host.

## 5. Penggabungan CIDR sampai /19

Penggabungan dilakukan dari bagian akhir cabang menuju blok induk. Pada pohon alokasi biner yang dipakai di laporan ini:

```text
Prefix induk = min(prefix anak pertama, prefix anak kedua) − 1
```

Contoh `/30 + /30 → /29` dan `/26 + /30 → /25`. Untuk dua anak berbeda ukuran, pohon ini menyediakan dua bagian sebesar anak yang lebih besar; anak yang kecil hanya memakai sebagian bagiannya.

> Rumus tersebut adalah aturan membangun pohon alokasi ini. Untuk merangkum dua jaringan yang alamatnya sudah ditentukan, periksa juga network ID, alignment, dan prefix bit bersama. Dua jaringan /26 yang berjauhan tidak otomatis dapat diringkas menjadi /25.

| Tahap | Hasil | Anak pertama | Anak kedua | Prefix hasil |
|---|---|---|---|---|
| I | B1 | A24 /30 | A22 /30 | /29 |
| I | B2 | A19 /26 | A18 /30 | /25 |
| I | B3 | A16 /30 | A14 /30 | /29 |
| I | B4 | A11 /24 | A13 /24 | /23 |
| I | B5 | A12 /26 | A10 /30 | /25 |
| I | B6 | A2 /30 | A3 /29 | /28 |
| I | B7 | A7 /29 | A8 /30 | /28 |
| I | B8 | A6 /23 | A5 /26 | /22 |
| II | C1 | A21 /29 | B1 /29 | /28 |
| II | C2 | A17 /25 | B3 /29 | /24 |
| II | C3 | B4 /23 | B5 /25 | /22 |
| II | C4 | A1 /25 | B6 /28 | /24 |
| II | C5 | A9 /22 | B7 /28 | /21 |
| II | C6 | A4 /22 | B8 /22 | /21 |
| III | D1 | C1 /28 | A20 /30 | /27 |
| III | D2 | C5 /21 | C6 /21 | /20 |
| IV | E1 | A23 /27 | D1 /27 | /26 |
| V | F1 | A25 /26 | E1 /26 | /25 |
| VI | G1 | F1 /25 | B2 /25 | /24 |
| VII | H1 | G1 /24 | C2 /24 | /23 |
| VIII | I1 | A15 /23 | H1 /23 | /22 |
| IX | J1 | C3 /22 | I1 /22 | /21 |
| X | K1 | J1 /21 | C4 /24 | /20 |
| XI | L1 | D2 /20 | K1 /20 | /19 |

### Mengapa blok induk /19?

Tiga angka ini memiliki arti berbeda:

```text
Jumlah kebutuhan host dan interface router      = 2.814
Jumlah kapasitas penuh blok A1 sampai A25        = 4.188
Kapasitas blok induk 192.227.0.0/19              = 8.192
```

Perincian total ukuran blok:

```text
9 subnet /30 ×     4 =    36
3 subnet /29 ×     8 =    24
1 subnet /27 ×    32 =    32
4 subnet /26 ×    64 =   256
2 subnet /25 ×   128 =   256
2 subnet /24 ×   256 =   512
2 subnet /23 ×   512 = 1.024
2 subnet /22 × 1.024 = 2.048
TOTAL               = 4.188 alamat
```

/20 hanya memiliki 4.096 alamat, sehingga tidak dapat menampung seluruh blok subnet akhir tersebut tanpa overlap. /19 memiliki 8.192 alamat. Pohon juga menghasilkan D2 /20 dan K1 /20 yang digabung menjadi L1 /19.

 “Bukan karena 2.814 langsung membutuhkan /19. Setelah masing-masing subnet dibulatkan ke blok pangkat dua, totalnya 4.188 alamat, melebihi kapasitas /20. Hasil penggabungan pohon juga mencapai /19.”

Efisiensi pada rancangan ini berarti setiap subnet akhir memakai prefix terkecil yang cukup dan cabang dapat diringkas untuk routing. Blok induk masih memiliki ruang kosong karena pembulatan serta penggabungan hierarkis; jangan menyebut seluruh 8.192 alamat terpakai atau mengklaim susunan pohon ini satu-satunya susunan yang mungkin.

## 6. Membaca pohon dan membagi alamat

Pada gambar pohon:

- Hijau: blok induk L1.
- Kuning: blok hasil penggabungan.
- Biru: subnet akhir A1–A25 yang benar-benar dipasang pada jaringan.

**Penggabungan dibaca dari kanan ke kiri. Pembagian alamat dibaca dari kiri ke kanan.** Pohon adalah hubungan blok alamat, bukan gambar kabel. Dua subnet yang digabung tidak selalu merupakan dua perangkat yang terhubung langsung.

### Langkah menentukan network ID anak

1. Hitung ukuran blok induk.
2. Bagi induk menjadi dua bagian sama besar.
3. Anak pertama dimulai pada awal bagian pertama.
4. Anak kedua dimulai pada awal bagian kedua.
5. Gunakan prefix aktual tiap anak; ruang tersisa dalam bagiannya menjadi cadangan.

```text
Network anak pertama = network induk
Network anak kedua   = network induk + setengah kapasitas induk
```

**Contoh tingkat awal:**

```text
L1 192.227.0.0/19 — 8.192 alamat
├── D2 192.227.0.0/20  — bagian pertama
└── K1 192.227.16.0/20 — bagian kedua, maju 4.096 alamat

K1 192.227.16.0/20 — 4.096 alamat
├── J1 192.227.16.0/21
└── C4 192.227.24.0/24
```

C4 hanya /24, tetapi dimulai pada `.24.0` karena bagian kedua K1 dimulai setelah 2.048 alamat atau 8 nilai oktet ketiga. Jangan menghitung awal C4 hanya dari ukuran /24-nya.

**Contoh tingkat kecil:**

```text
C4 192.227.24.0/24
├── A1 192.227.24.0/25
└── B6 192.227.24.128/28

B6 192.227.24.128/28 — 16 alamat
├── bagian 1: .128–.135 → A2 memakai .128/30, yaitu .128–.131
└── bagian 2: .136–.143 → A3 memakai .136/29, yaitu .136–.143
```

Alamat `.132–.135` tidak dipakai oleh subnet akhir pada rancangan ini. Ruang kosong tersebut bukan subnet tambahan.

### Menghitung broadcast dan range host

```text
Broadcast  = network ID + ukuran blok − 1
Host awal  = network ID + 1
Host akhir = broadcast − 1
```

Untuk A25 `192.227.22.0/26`, ukuran blok 64: broadcast `.63`, host `.1–.62`. Untuk A2 `192.227.24.128/30`, ukuran blok 4: broadcast `.131`, host `.129–.130`.

## 7. Tabel pembagian alamat

Prefix menentukan netmask sesuai tabel pada bagian 3.

| Subnet | Network/prefix | Broadcast | Range host yang dapat dipakai |
|---|---|---|---|
| A1 | 192.227.24.0/25 | 192.227.24.127 | 192.227.24.1–192.227.24.126 |
| A2 | 192.227.24.128/30 | 192.227.24.131 | 192.227.24.129–192.227.24.130 |
| A3 | 192.227.24.136/29 | 192.227.24.143 | 192.227.24.137–192.227.24.142 |
| A4 | 192.227.8.0/22 | 192.227.11.255 | 192.227.8.1–192.227.11.254 |
| A5 | 192.227.14.0/26 | 192.227.14.63 | 192.227.14.1–192.227.14.62 |
| A6 | 192.227.12.0/23 | 192.227.13.255 | 192.227.12.1–192.227.13.254 |
| A7 | 192.227.4.0/29 | 192.227.4.7 | 192.227.4.1–192.227.4.6 |
| A8 | 192.227.4.8/30 | 192.227.4.11 | 192.227.4.9–192.227.4.10 |
| A9 | 192.227.0.0/22 | 192.227.3.255 | 192.227.0.1–192.227.3.254 |
| A10 | 192.227.18.64/30 | 192.227.18.67 | 192.227.18.65–192.227.18.66 |
| A11 | 192.227.16.0/24 | 192.227.16.255 | 192.227.16.1–192.227.16.254 |
| A12 | 192.227.18.0/26 | 192.227.18.63 | 192.227.18.1–192.227.18.62 |
| A13 | 192.227.17.0/24 | 192.227.17.255 | 192.227.17.1–192.227.17.254 |
| A14 | 192.227.23.132/30 | 192.227.23.135 | 192.227.23.133–192.227.23.134 |
| A15 | 192.227.20.0/23 | 192.227.21.255 | 192.227.20.1–192.227.21.254 |
| A16 | 192.227.23.128/30 | 192.227.23.131 | 192.227.23.129–192.227.23.130 |
| A17 | 192.227.23.0/25 | 192.227.23.127 | 192.227.23.1–192.227.23.126 |
| A18 | 192.227.22.192/30 | 192.227.22.195 | 192.227.22.193–192.227.22.194 |
| A19 | 192.227.22.128/26 | 192.227.22.191 | 192.227.22.129–192.227.22.190 |
| A20 | 192.227.22.112/30 | 192.227.22.115 | 192.227.22.113–192.227.22.114 |
| A21 | 192.227.22.96/29 | 192.227.22.103 | 192.227.22.97–192.227.22.102 |
| A22 | 192.227.22.108/30 | 192.227.22.111 | 192.227.22.109–192.227.22.110 |
| A23 | 192.227.22.64/27 | 192.227.22.95 | 192.227.22.65–192.227.22.94 |
| A24 | 192.227.22.104/30 | 192.227.22.107 | 192.227.22.105–192.227.22.106 |
| A25 | 192.227.22.0/26 | 192.227.22.63 | 192.227.22.1–192.227.22.62 |

## 8. Penetapan IP node dan interface

### Aturan penetapan

Script memberikan alamat host pertama pada router pertama dalam daftar subnet. Jika ada beberapa router, alamat diberikan berurutan. Alamat awal kelompok host diberikan setelah alamat router, kemudian kelompok berikutnya dimulai setelah jumlah alamat yang dicadangkan untuk kelompok sebelumnya.

Contoh A1:

```text
Cyberia : .24.1
JJ      : .24.2   → dicadangkan 95 alamat, .24.2–.24.96
MyuMyu  : .24.97  → dicadangkan 30 alamat, .24.97–.24.126
```

Contoh A4: Karl dimulai di `192.227.8.2`. Setelah 700 alamat, Lin dimulai di `192.227.10.190`. Kedua kelompok tetap berada dalam subnet A4 /22 yang sama.

Contoh A9: Omnipresence dimulai `.0.2`; setelah 510 alamat, Denpa dimulai `192.227.2.0`. Alamat berakhiran `.0` ini **valid sebagai host** karena network A9 adalah `192.227.0.0/22`, bukan `192.227.2.0/24`.

### Interface router

Urutan berikut sama dengan `ROUTER_IFACES` pada script. Urutan subnet menentukan eth0, eth1, dan seterusnya.

| Router | eth0 | eth1 | eth2 | eth3 |
|---|---|---|---|---|
| Cyberia | NAT: DHCP | A2: 192.227.24.129 | A1: 192.227.24.1 | — |
| Wired | A2: 192.227.24.130 | A3: 192.227.24.137 | A10: 192.227.18.65 | A14: 192.227.23.133 |
| Psyche | A3: 192.227.24.138 | A4: 192.227.8.1 | — | — |
| Copland | A3: 192.227.24.139 | A5: 192.227.14.1 | A6: 192.227.12.1 | — |
| Schumann | A3: 192.227.24.140 | A7: 192.227.4.1 | A8: 192.227.4.9 | — |
| Accela | A8: 192.227.4.10 | A9: 192.227.0.1 | — | — |
| Navi | A10: 192.227.18.66 | A11: 192.227.16.1 | A12: 192.227.18.1 | A13: 192.227.17.1 |
| Tachibana | A14: 192.227.23.134 | A15: 192.227.20.1 | A16: 192.227.23.129 | — |
| Deus | A16: 192.227.23.130 | A17: 192.227.23.1 | A18: 192.227.22.193 | — |
| Protocol7 | A18: 192.227.22.194 | A19: 192.227.22.129 | A20: 192.227.22.113 | — |
| Knights | A20: 192.227.22.114 | A21: 192.227.22.97 | A22: 192.227.22.109 | — |
| Phantoma | A22: 192.227.22.110 | A23: 192.227.22.65 | A24: 192.227.22.105 | — |
| Shibuya | A24: 192.227.22.106 | A25: 192.227.22.1 | — | — |

### IP node akhir

Semua node akhir memakai eth0, prefix subnet terkait, serta gateway router pada LAN-nya.

| Node | Subnet | IP | Gateway |
|---|---|---|---|
| JJ | A1 | 192.227.24.2 | 192.227.24.1 |
| MyuMyu | A1 | 192.227.24.97 | 192.227.24.1 |
| Karl | A4 | 192.227.8.2 | 192.227.8.1 |
| Lin | A4 | 192.227.10.190 | 192.227.8.1 |
| Juri | A5 | 192.227.14.2 | 192.227.14.1 |
| Masayuki | A6 | 192.227.12.2 | 192.227.12.1 |
| Akashic | A7 | 192.227.4.2 | 192.227.4.1 |
| Omnipresence | A9 | 192.227.0.2 | 192.227.0.1 |
| Denpa | A9 | 192.227.2.0 | 192.227.0.1 |
| Alice | A11 | 192.227.16.2 | 192.227.16.1 |
| Reika | A11 | 192.227.16.210 | 192.227.16.1 |
| Taro | A12 | 192.227.18.2 | 192.227.18.1 |
| Yasuo | A13 | 192.227.17.2 | 192.227.17.1 |
| Miho | A13 | 192.227.17.122 | 192.227.17.1 |
| Kuma | A13 | 192.227.17.140 | 192.227.17.1 |
| Collective | A15 | 192.227.20.2 | 192.227.20.1 |
| Ghost | A15 | 192.227.21.6 | 192.227.20.1 |
| Lilith | A17 | 192.227.23.2 | 192.227.23.1 |
| Specter | A17 | 192.227.23.79 | 192.227.23.1 |
| Chisa | A19 | 192.227.22.130 | 192.227.22.129 |
| Hodgeson | A19 | 192.227.22.153 | 192.227.22.129 |
| Eiri | A21 | 192.227.22.98 | 192.227.22.97 |
| Semi | A23 | 192.227.22.66 | 192.227.22.65 |
| Lain | A25 | 192.227.22.2 | 192.227.22.1 |
| Mika | A25 | 192.227.22.59 | 192.227.22.1 |

## 9. Routing dan perjalanan paket

### Prinsip routing

- **Connected route:** jaringan yang langsung terpasang pada interface router. Terbentuk ketika IP/interface aktif; tidak perlu ditambahkan sebagai static route lagi.
- **Default route:** tujuan `0.0.0.0/0`, dipakai jika tidak ada route yang lebih spesifik. Mengarah ke hulu.
- **Route agregat:** satu route mencakup beberapa subnet di satu cabang.
- **Longest prefix match:** router memilih route dengan prefix terpanjang yang cocok dengan IP tujuan.
- **Next hop:** IP router tetangga yang dapat dicapai melalui jaringan penghubung.

### Tabel static route dan default route

Tabel ini mengikuti script. Ada **24 route yang ditulis script**, ditambah default route Cyberia dari DHCP NAT sehingga tabel rancangan berikut memiliki 25 baris. Connected route tidak dihitung di sini.

| Router | Network tujuan | Next hop | Penjelasan |
|---|---|---|---|
| Cyberia | 0.0.0.0/0 | Dari DHCP NAT | Akses keluar |
| Cyberia | 192.227.0.0/19 | 192.227.24.130 | Jaringan internal melalui Wired |
| Wired | 0.0.0.0/0 | 192.227.24.129 | Ke Cyberia |
| Wired | 192.227.0.0/21 | 192.227.24.140 | Cabang Schumann |
| Wired | 192.227.8.0/22 | 192.227.24.138 | Cabang Psyche |
| Wired | 192.227.12.0/22 | 192.227.24.139 | Cabang Copland |
| Wired | 192.227.16.0/22 | 192.227.18.66 | Cabang Navi |
| Wired | 192.227.20.0/22 | 192.227.23.134 | Cabang Tachibana |
| Psyche | 0.0.0.0/0 | 192.227.24.137 | Ke Wired |
| Copland | 0.0.0.0/0 | 192.227.24.137 | Ke Wired |
| Schumann | 0.0.0.0/0 | 192.227.24.137 | Ke Wired |
| Schumann | 192.227.0.0/22 | 192.227.4.10 | Ke Accela |
| Accela | 0.0.0.0/0 | 192.227.4.9 | Ke Schumann |
| Navi | 0.0.0.0/0 | 192.227.18.65 | Ke Wired |
| Tachibana | 0.0.0.0/0 | 192.227.23.133 | Ke Wired |
| Tachibana | 192.227.22.0/23 | 192.227.23.130 | Cabang Deus |
| Deus | 0.0.0.0/0 | 192.227.23.129 | Ke Tachibana |
| Deus | 192.227.22.0/24 | 192.227.22.194 | Cabang Protocol7 |
| Protocol7 | 0.0.0.0/0 | 192.227.22.193 | Ke Deus |
| Protocol7 | 192.227.22.0/25 | 192.227.22.114 | Cabang Knights |
| Knights | 0.0.0.0/0 | 192.227.22.113 | Ke Protocol7 |
| Knights | 192.227.22.0/25 | 192.227.22.110 | Cabang Phantoma |
| Phantoma | 0.0.0.0/0 | 192.227.22.109 | Ke Knights |
| Phantoma | 192.227.22.0/26 | 192.227.22.106 | LAN Shibuya |
| Shibuya | 0.0.0.0/0 | 192.227.22.105 | Ke Phantoma |

**Contoh efisiensi:** Wired cukup menyimpan `192.227.20.0/22` melalui Tachibana untuk menjangkau subnet A15–A25. Wired tidak memerlukan sebelas route terpisah untuk subnet-subnet tersebut.

**Mengapa Knights punya route /25 ke Phantoma padahal A21 juga ada di dalam /25 itu?** A21 adalah connected route /29 pada Knights. Untuk tujuan Eiri, /29 lebih spesifik sehingga paket tetap dikirim ke LAN Eiri. Route /25 dipakai untuk tujuan lain yang cocok dan berada di cabang bawah. Prinsip serupa berlaku pada jaringan penghubung yang sudah connected.

Route agregat juga mencakup ruang alamat yang belum dialokasikan. Pengujian demo diarahkan ke alamat node yang benar-benar dipasang. Konfigurasi saat ini tidak memasang discard route untuk seluruh ruang kosong; jangan mengklaim semua alamat dalam /19 merupakan tujuan aktif atau pasti bebas loop untuk alamat yang tidak dialokasikan.

### Contoh Mika menuju Karl

```text
Mika 192.227.22.59
  → Shibuya → Phantoma → Knights → Protocol7
  → Deus → Tachibana → Wired → Psyche
  → Karl 192.227.8.2
```

1. Mika melihat Karl berada di luar subnet A25, lalu mengirim paket ke gateway `192.227.22.1`.
2. Shibuya sampai Tachibana menggunakan default route menuju hulu untuk tujuan Karl.
3. Wired menemukan route `192.227.8.0/22` melalui Psyche `192.227.24.138`.
4. Psyche memiliki A4 sebagai connected network dan meneruskan ke Karl.
5. Balasan Karl melewati gateway Psyche. Wired memilih `192.227.20.0/22` ke Tachibana; route cabang berikutnya membawa balasan sampai Shibuya dan Mika.

Ping membutuhkan jalur pergi **dan** jalur balik. Default route saja pada semua router tidak cukup jika router hulu belum tahu arah cabang tujuan.

### NAT pada Cyberia

Cyberia eth0 memperoleh IP dan default route dari DHCP NAT. Masquerade menerjemahkan alamat sumber internal menjadi alamat sisi luar Cyberia untuk koneksi keluar.

```bash
iptables -t nat -A POSTROUTING -o eth0 -s 192.227.0.0/19 -j MASQUERADE
```

NAT bukan syarat komunikasi antarsubnet internal. Koneksi internal bergantung pada IP, gateway, forwarding, route, dan jalur balik yang benar.

## 10. Implementasi dan konfigurasi permanen

Script menggunakan API GNS3 untuk membuat node dan link, kemudian menulis konfigurasi Linux ke `/etc/network/interfaces`. Konfigurasi otomatis ditujukan untuk template Docker/Linux yang sesuai.

Contoh konfigurasi Mika yang sesuai rancangan:

```text
auto eth0
iface eth0 inet static
    address 192.227.22.59
    netmask 255.255.255.192
    gateway 192.227.22.1
    up ip link set dev eth0 address 02:32:00:26:00:00 || true
    up echo nameserver 192.168.122.1 > /etc/resolv.conf
```

Contoh bagian interface Wired menuju Navi:

```text
auto eth2
iface eth2 inet static
    address 192.227.18.65
    netmask 255.255.255.252
    up ip link set dev eth2 address 02:32:00:02:00:02 || true
    up route add -net 192.227.16.0 netmask 255.255.252.0 gw 192.227.18.66
```

Pada router, script juga mengaktifkan forwarding:

```text
up echo 1 > /proc/sys/net/ipv4/ip_forward
```

Perintah langsung seperti `ip addr add` dan `route add` mengubah kondisi aktif, tetapi tidak otomatis menjadi konfigurasi permanen. Baris dalam file interfaces dijalankan saat interface diaktifkan oleh mekanisme jaringan template tersebut.

### Fungsi opsi script

Jalankan dari komputer yang dapat menjangkau server, dengan nama file script yang sesuai.

| Perintah | Fungsi dan dampak |
|---|---|
| `py buat_topologi_gns3.py --dry-run` | Menampilkan rencana node/link tanpa koneksi server |
| `py buat_topologi_gns3.py --export-config konfigurasi.txt` | Menulis konfigurasi ke file lokal |
| `py buat_topologi_gns3.py --list-templates` | Membaca daftar template server |
| `py buat_topologi_gns3.py --check-mac` | Membaca MAC melalui console dan memeriksa duplikasi |
| `py buat_topologi_gns3.py --apply-config` | Menimpa file konfigurasi node yang sudah ada, lalu stop/start semua node |
| `py buat_topologi_gns3.py --start` | Mengisi project yang masih kosong lalu menyalakan node |

Gunakan `python` sebagai pengganti `py` jika instalasi Python di komputermu memakai perintah tersebut. Mode pembuatan biasa menolak project yang sudah berisi node. Jangan menjalankan `--apply-config` hanya untuk menunjukkan isi konfigurasi saat demo karena opsi itu mengubah konfigurasi dan mengganggu koneksi selama restart.

MAC tetap diberikan per interface untuk menghindari duplikasi. Hasil `--check-mac` baru lengkap jika **38/38 node terbaca** dan tidak ada MAC ganda. Pemeriksaan MAC bukan pengujian ping.

## 11. Pengujian dan bukti demo

### A. Periksa IP dan route aktif

Jalankan pada router atau node yang sedang dijelaskan:

```bash
ip -br link
ip -br addr
ip route
cat /etc/network/interfaces
```

Yang diperiksa: interface aktif, IP/prefix cocok tabel, gateway benar, dan route cabang tersedia. Isi file membuktikan rancangan tersimpan, sedangkan `ip addr` dan `ip route` membuktikan keadaan aktif. Keduanya perlu sesuai.

Pada router:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Nilainya harus `1` agar router meneruskan paket IPv4.

### B. Uji bertahap dari Mika

```bash
# Gateway sendiri: membuktikan koneksi LAN A25
ping -c 4 192.227.22.1

# Lain: membuktikan komunikasi dalam subnet yang sama
ping -c 4 192.227.22.2

# Karl: menguji perjalanan panjang melintasi cabang
ping -c 4 192.227.8.2

# Alice: menguji cabang Navi
ping -c 4 192.227.16.2

# JJ: menguji LAN Cyberia
ping -c 4 192.227.24.2

# Omnipresence: menguji cabang Schumann–Accela
ping -c 4 192.227.0.2
```

Target bukti yang baik: seluruh paket diterima dan `0% packet loss`. Jika paket awal hilang saat pembentukan ARP, ulangi dan periksa kestabilannya; jangan langsung mengabaikan kehilangan paket berulang.

Jika `traceroute` tersedia:

```bash
traceroute -n 192.227.8.2
```

Lintasan router diperkirakan sesuai contoh Mika–Karl pada bagian 9. IP yang ditampilkan traceroute dapat bergantung pada interface sumber balasan ICMP. Tanda `*` saja tidak selalu berarti forwarding gagal; cocokkan dengan hasil ping tujuan.

### C. Uji arah balik

Dari Karl:

```bash
ping -c 4 192.227.22.59
```

Ini menunjukkan Karl juga dapat memulai komunikasi menuju Mika. Ping sebelumnya sudah memerlukan balasan, tetapi uji dua arah memberi bukti tambahan bahwa kedua node dapat menjadi sumber komunikasi.

### D. Uji internet sebagai tambahan

Dari Mika atau host lain:

```bash
ping -c 4 8.8.8.8
```

Pada Cyberia, periksa:

```bash
ip -br addr
ip route
iptables -t nat -L POSTROUTING -n -v
```

Jika ping IP internet berhasil tetapi nama domain gagal, periksa `/etc/resolv.conf` dan ketersediaan DNS. Nilai `192.168.122.1` pada script adalah asumsi DNS NAT, sehingga harus cocok dengan lingkungan aktual. Jangan menyamakan keberhasilan internet dengan keberhasilan seluruh koneksi internal.

### E. Pembuktian seluruh node

Contoh ping di atas adalah demonstrasi perwakilan, bukan pengujian menyeluruh. Sebelum pengumpulan, uji semua 38 node ber-IP sebagai sumber dan seluruh IP internal yang dikonfigurasi sebagai tujuan; keluarkan interface NAT yang dinamis dari daftar internal. Catat sumber, tujuan, paket terkirim/diterima, dan hasil.

Jika pengujian hanya dilakukan pada 25 node akhir, tulis lingkup tersebut dengan jujur dan uji interface router secara terpisah. Switch layer 2 bawaan GNS3 tidak memiliki IP untuk diping; fungsinya dibuktikan melalui komunikasi node yang terhubung padanya.

Setelah penyimpanan dan restart melalui GNS3, ulangi pengujian penting untuk membuktikan konfigurasi tetap aktif. Demo menggunakan project yang sama dengan yang dikumpulkan.

## 12. Naskah demo 5–7 menit

### Menit 0–1: gambaran topologi

“Kelompok kami menggunakan CIDR pada GNS3 dengan prefix 192.227. Topologi dibagi menjadi 25 subnet. Cyberia merupakan satu-satunya router menuju NAT, dan Wired menghubungkan cabang utama. Total ada 13 router dan 25 node akhir. Node akhir mewakili jumlah host yang tercantum pada soal.”

**Tunjukkan:** Cyberia, Wired, cabang Psyche/Copland/Schumann, Navi, dan Tachibana.

### Menit 1–2: kebutuhan IP

“Kebutuhan setiap subnet adalah jumlah host ditambah interface router. Contohnya A4 berisi 700 host Karl, 180 host Lin, dan satu interface Psyche, sehingga membutuhkan 881 alamat host. /23 hanya menyediakan 510, sehingga dipilih /22 yang menyediakan 1.022 alamat host. Untuk A25, 57 ditambah 4 ditambah 1 sama dengan 62, sehingga cukup /26.”

**Tunjukkan:** baris A4 dan A25 pada tabel kebutuhan.

### Menit 2–3: penggabungan CIDR

“Setelah prefix subnet akhir ditentukan, kami menggabungkannya dari ujung cabang sesuai pohon. Contohnya A24 /30 dan A22 /30 menjadi B1 /29. B1 digabung A21 /29 menjadi C1 /28. Proses dilanjutkan sampai D2 /20 dan K1 /20 bergabung menjadi L1 /19.”

“Jumlah kebutuhan host/router adalah 2.814, tetapi jumlah ukuran seluruh blok subnet mencapai 4.188 alamat. /20 hanya memiliki 4.096 alamat, sehingga tidak cukup. Karena itu blok induk yang digunakan adalah 192.227.0.0/19.”

**Tunjukkan:** contoh cabang kanan pohon dan akar L1.

### Menit 3–4: pembagian alamat

“Setelah blok induk diketahui, alamat dibagi dari kiri ke kanan. L1 /19 dibagi menjadi D2 di 192.227.0.0 dan K1 di 192.227.16.0. Proses diteruskan sampai subnet akhir. Pada A2, network ID-nya 192.227.24.128/30, sehingga Cyberia menggunakan .129, Wired .130, dan broadcast-nya .131.”

**Tunjukkan:** tabel A2 dan IP aktif Cyberia/Wired jika diperlukan.

### Menit 4–5: routing

“Routing menggunakan static route, default route ke hulu, dan route agregat ke cabang bawah. Wired cukup menggunakan 192.227.20.0/22 melalui Tachibana untuk menjangkau A15 sampai A25. Connected network dikenali langsung. Jika beberapa route cocok, router memilih prefix paling panjang.”

**Tunjukkan:** `ip route` pada Wired dan route cabang `192.227.20.0/22`.

### Menit 5–7: pembuktian

“Sekarang kami menguji Mika ke gateway Shibuya, lalu ke Karl pada cabang Psyche dan ke JJ pada LAN Cyberia. Hasil berikut menunjukkan koneksi untuk pasangan yang diuji. Konfigurasi permanen tersimpan pada /etc/network/interfaces.”

**Jalankan:** ping pada bagian 11. Sebutkan hasil aktual, bukan hasil yang diharapkan. Jika tersedia, tunjukkan log pengujian menyeluruh dan pengujian setelah restart.

## 13. Pertanyaan asisten dan jawaban

| Pertanyaan | Jawaban yang mudah disampaikan |
|---|---|
| Mengapa A3 hanya satu subnet? | Empat interface router berada pada satu jaringan layer 2 melalui Switch2. |
| Mengapa interface router ikut dihitung? | Interface router memerlukan alamat host dalam subnet yang sama agar menjadi tetangga/gateway. |
| Mengapa switch tidak dihitung? | Switch layer 2 bawaan pada rancangan ini tidak memakai IP untuk forwarding. Jika ada IP manajemen, kebutuhan itu harus ditambahkan. |
| Mengapa A9 /22, bukan /23? | Kebutuhannya 514, sedangkan /23 hanya menyediakan 510 alamat host. |
| Mengapa A23 /27, bukan /28? | Kebutuhannya 15; /28 hanya memiliki 14 alamat host yang dapat digunakan. |
| Mengapa A13 /24? | Kebutuhannya 143, melebihi kapasitas /25 yaitu 126. |
| Mengapa /19 padahal total kebutuhan 2.814? | Jumlah kapasitas blok subnet akhir adalah 4.188, melebihi /20 yang hanya 4.096. Pohon juga menghasilkan /19. |
| Mengapa ada alamat kosong? | Ukuran blok pangkat dua dan pembagian hierarkis menyisakan ruang pada blok gabungan. |
| Apakah semua blok kuning dipasang di interface? | Tidak. Blok kuning adalah agregasi; subnet biru A1–A25 yang digunakan pada interface. Agregasi tertentu dipakai sebagai tujuan route. |
| Mengapa C4 dimulai .24.0? | C4 ditempatkan pada awal bagian kedua K1 /20, yaitu setelah 2.048 alamat dari .16.0. |
| Mengapa alamat Denpa berakhiran .0 boleh? | Status network/host ditentukan oleh prefix. 192.227.2.0 bukan network ID atau broadcast A9 /22. |
| Mengapa IP MyuMyu melompat ke .97? | Rentang .2–.96 dicadangkan untuk 95 host JJ. |
| Mengapa tidak semua subnet ditulis pada setiap router? | Connected route sudah tersedia, dan satu route agregat dapat mewakili banyak subnet cabang. |
| Mengapa Cyberia tidak mengirim paket JJ ke Wired? | Connected route A1 /25 lebih spesifik daripada route internal /19. |
| Apakah default route saja cukup? | Router hulu tetap membutuhkan route menuju cabang bawah; balasan juga harus memiliki jalur yang benar. |
| Mengapa IP router tidak diberi gateway di tabel interface? | Next hop router dikelola melalui tabel routing; default route router dicantumkan di tabel route. |
| Apa fungsi forwarding? | Memungkinkan Linux meneruskan paket antarinterface. |
| Apa fungsi NAT? | Menerjemahkan alamat sumber saat koneksi internal keluar melalui Cyberia. |
| Apakah ping internet membuktikan semua node terhubung? | Tidak. Konektivitas internal harus diuji antar-node/cabang. |
| Apakah MAC unik membuktikan routing benar? | Tidak. Itu pemeriksaan layer 2; IP, route, forwarding, dan ping tetap perlu diperiksa. |
| Haruskah IP CIDR GNS3 sama dengan VLSM CPT? | Alokasi kedua metode dapat berbeda. Masing-masing hasil hitungan harus cocok dengan implementasi pada platformnya. |
| Apakah semua 8.192 alamat digunakan? | Tidak. /19 adalah blok induk yang mencakup subnet akhir dan ruang kosong. |

## 14. Troubleshooting dan lembar hasil

### Urutan pemeriksaan jika ping gagal

| Gejala | Pemeriksaan awal |
|---|---|
| Gateway sendiri tidak dapat diping | Status node/link, interface, IP/prefix, kesesuaian eth dengan kabel, dan ARP/MAC |
| Gateway berhasil, tujuan cabang lain gagal | Default route sumber, route cabang pada router hulu, forwarding, dan route balik |
| Koneksi sesekali berhasil/gagal | Duplikasi IP/MAC, cache ARP, stabilitas link/node |
| Internal berhasil, internet gagal | DHCP/default route Cyberia, masquerade, koneksi NAT/upstream |
| Ping IP internet berhasil, domain gagal | Resolver DNS dan akses ke server DNS |
| Berhasil sebelum restart, gagal sesudahnya | Isi interfaces, hook route/forwarding, dan apakah file dipakai template |

Perintah baca untuk pemeriksaan:

```bash
ip -br link
ip -br addr
ip route
ip neigh show
cat /etc/network/interfaces
cat /proc/sys/net/ipv4/ip_forward
```

Perintah tersebut tidak mengubah konfigurasi. Hindari memperbaiki dengan perintah sementara tanpa memperbarui konfigurasi permanen yang sesuai.

### Batas acuan laporan

Angka Lain 57 dan Mika 4 mengikuti tabel serta script yang diberikan. Jika soal/gambar resmi menyatakan angka berbeda, hitung ulang subnet terkait dan penggabungannya. Dokumen ini tidak menyatakan bahwa script atau server sudah berhasil diuji secara langsung.

