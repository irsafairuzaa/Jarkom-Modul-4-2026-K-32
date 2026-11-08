## Cisco VLSM
### Topologi
Tahap pertama kita buat topologi sesuai dengan ketentuan pada soal, lalu memberi nama di tiap subnetnya (A1,A2, dst) dengan total 25 subnet.

<img width="944" height="617" alt="topologi cisco" src="https://github.com/user-attachments/assets/16938a3b-320e-4562-8598-da60c3b2d9a1" />

Berikut daftar rute subnet dari A1 - A25 dengan jumlah IP dan Netmask tiap subnetnya

<img width="517" height="284" alt="image" src="https://github.com/user-attachments/assets/45d9e0c0-c120-4f8f-8229-aa61397f2316" />

Dari perhitungan tiap subnet, total IP yang dapat digunakan sebanyak 2814, maka prefixnya `/19` dengan total 8192 IP.

### Pohon VLSM
Berikut pohon yang di buat untuk persebaran IP-nya yang dimulai base network yaitu `192.227.0.0/19`.

<img width="3287" height="715" alt="VLSM_Pohon" src="https://github.com/user-attachments/assets/61efb1b6-770b-4a4f-94e9-9c2c608e6b57" />

Subnet diurutkan dari blok terbesar ke terkecil. Root selalu dibelah dua, setiap belahannya netmask dinaikan sebanyak 1 bit.
- belahan kiri = networkID sama dengan induknya
- belahan kanan = networkID induk + ukuran blok belahan tersebut
Belahan kiri yang sudah sesuai kebutuhan, dipakai untuk subnet. Belahan kanan disimpan untuk subnet berikutnya.

### Subnetting
Berikut merupakan hasil subnetting menggunakan metode VLSM

<img width="455" height="275" alt="image" src="https://github.com/user-attachments/assets/c46af901-2b7d-4f97-a8bc-1b12cf612b43" />

#### Perhitungan
Contoh pada subnet A4 
1. Prefix
Prefix sudah ditentukan dari tabel rute subnet sebelumnya, yang dihitung dengan cara
- Melihat kebutuhan host yaitu 881 + 2 (1 untuk networkID, 1 untuk IP broadcast)
- Cari prefix yang sesuai dengan kebutuhan host dengan melihat tabel VLSM.

Pada subnet A4 dibutuhkan host sebanyak 883, prefix yang mencukupi adalah /22 dengan jumlah host 1024.

2. Network ID
Network ID adalah IP address pertama dalam subnet, sebagai identitas dari subnet tersebut. Maka networkID pada A4 adalah `192.227.0.0`.

3. Broadcast Address
Broadcast address adalah IP terakhir di dalam subnet tersebut. Ditentukan dengan menambah 1 dari IP range terakhir. Range IP berada pada `192.227.0.1 – 192.227.3.254`, maka broadcast address nya adalah `192.227.3.255`.

4. Range IP
Range IP ini dimulai dari alamat setelah Network ID sampai alamat sebelum broadcast, yaitu `192.227.0.1 – 192.227.3.254`

Setelah itu untuk subnet berikutnya mengambil IP dari IP setelah Broadcast address.

## Konfigurasi
1. Pada router (contoh Cyberia), Config > CLI 
```
enable
configure terminal
interface fa0/1
ip address 192.227.14.1 255.255.255.128
no shutdown
interface fa1/0
ip address 192.227.16.57 255.255.255.252
no shutdown
end
write
```

2. Pada Node (contoh Myumyu, laptop), Desktop > IP Configuration > Static
- IPv4 Address : 10.81.12.3
- Subnetmask : 255.255.255.128
- Default gateway : 192.227.14.1

## Routing
Routing ini dilakukan untuk menentukan jalur agar antar router/ node dapat berkomunikasi
```
ip route <network tujuan> <netmask> <next hop>
```

- Cyberia 
```
ip route 192.227.0.0 255.255.224.0 192.227.16.58
```

- Wired
```
ip route 0.0.0.0 0.0.0.0 192.227.16.57
ip route 192.227.12.0 255.255.254.0 192.227.16.66
ip route 192.227.15.64 255.255.255.192 192.227.16.66
ip route 192.227.10.0 255.255.254.0 192.227.16.70
ip route 192.227.14.128 255.255.255.128 192.227.16.70
ip route 192.227.15.0 255.255.255.0 192.227.16.70
ip route 192.227.16.0 255.255.255.128 192.227.16.70
ip route 192.227.0.0 255.255.252.0 192.227.16.34
ip route 192.227.8.0 255.255.254.0 192.227.16.35
ip route 192.227.15.192 255.255.255.192 192.227.16.35
ip route 192.227.4.0 255.255.252.0 192.227.16.36
ip route 192.227.16.40 255.255.255.248 192.227.16.36
ip route 192.227.16.60 255.255.255.252 192.227.16.36
```

- Psyche
```
ip route 0.0.0.0 0.0.0.0 192.227.16.33
```

- Copland
```
ip route 0.0.0.0 0.0.0.0 192.227.16.33
```

- Schumann
```
ip route 0.0.0.0 0.0.0.0 192.227.16.33
ip route 192.227.4.0 255.255.252.0 192.227.16.62
```

- Accela
```
ip route 0.0.0.0 0.0.0.0 192.227.16.61
```

- Navi
```
ip route 0.0.0.0 0.0.0.0 192.227.16.65
```

- Tachibana
```
ip route 0.0.0.0 0.0.0.0 192.227.16.69
ip route 192.227.14.128 255.255.255.128 192.227.16.74
ip route 192.227.16.76 255.255.255.252 192.227.16.74
ip route 192.227.15.128 255.255.255.192 192.227.16.74
ip route 192.227.16.80 255.255.255.240 192.227.16.74
ip route 192.227.16.48 255.255.255.248 192.227.16.74
ip route 192.227.16.0 255.255.255.224 192.227.16.74
ip route 192.227.15.0 255.255.255.192 192.227.16.74
```

- Deus
```
ip route 0.0.0.0 0.0.0.0 192.227.16.73
ip route 192.227.15.128 255.255.255.192 192.227.16.78
ip route 192.227.16.80 255.255.255.240 192.227.16.78
ip route 192.227.16.48 255.255.255.248 192.227.16.78
ip route 192.227.16.0 255.255.255.224 192.227.16.78
ip route 192.227.15.0 255.255.255.192 192.227.16.78
```

- Protocol
```
ip route 0.0.0.0 0.0.0.0 192.227.16.77
ip route 192.227.16.48 255.255.255.248 192.227.16.82
ip route 192.227.16.80 255.255.255.240 192.227.16.82
ip route 192.227.16.0 255.255.255.224 192.227.16.82
ip route 192.227.15.0 255.255.255.192 192.227.16.82
```

- Knights
```
ip route 0.0.0.0 0.0.0.0 192.227.16.81
ip route 192.227.16.0 255.255.255.224 192.227.16.86
ip route 192.227.16.88 255.255.255.252 192.227.16.86
ip route 192.227.15.0 255.255.255.192 192.227.16.86
```

- Phantoma
```
ip route 0.0.0.0 0.0.0.0 192.227.16.85
ip route 192.227.15.0 255.255.255.192 192.227.16.90
```

- Shibuya
```
ip route 0.0.0.0 0.0.0.0 192.227.16.89
```

## Testing
Berikut test ping dari beberapa node
- Karl ke Lain
<img width="574" height="256" alt="image" src="https://github.com/user-attachments/assets/c0bfaaa2-d95e-4a55-b1aa-caaeb784d83d" />

- Alice ke Akashic
<img width="588" height="259" alt="image" src="https://github.com/user-attachments/assets/0127c4d7-da68-4e02-980b-b7578c9eb328" />

- Eiri ke Lin
<img width="556" height="265" alt="image" src="https://github.com/user-attachments/assets/a168fa0b-9f72-4571-9cef-f3cb53f3b38d" />

## Revisi
Belum dilakukan konfigurasi agar tiap node dapat terhubung/ ping google/ 8.8.8.8
1. Menambahkan router sebagai ISP, dan Server sebagai google
2. Tambah port di Cyberia karena semua port di Cyberia sudah digunakan
3. Konfigurasi Cyberia (Config > CLI)
```
enable
configure terminal
interface fa1/1
ip address 203.0.113.2 255.255.255.252
no shutdown
ip route 0.0.0.0 0.0.0.0 203.0.113.1
end
write
```
4. Konfigurasi ISP (Config > CLI)
```
enable
configure terminal
interface fa0/0
ip address 203.0.113.1 255.255.255.252
no shutdown
interface fa0/1
ip address 8.8.8.1 255.255.255.0
no shutdown
ip route 192.227.0.0 255.255.224.0 203.0.113.2
end
write
```
5. Konfigurasi server google
- IP Address: 8.8.8.8
- Subnet Mask: 255.255.255.0
- Default Gateway: 8.8.8.1

### Testing
ping 8.8.8.8 dari taro
<img width="582" height="268" alt="image" src="https://github.com/user-attachments/assets/41499f64-a71d-4ff9-9922-ec8e8a533776" />

