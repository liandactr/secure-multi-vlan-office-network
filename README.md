# Secure Multi-VLAN Office Network

Simulasi jaringan kantor menggunakan Cisco Packet Tracer dengan penerapan VLAN, 802.1Q trunking, Router-on-a-Stick, inter-VLAN routing, dan Extended ACL untuk mengatur akses antar jaringan.

## Deskripsi Project

Project ini merupakan simulasi jaringan kantor sederhana yang terdiri dari tiga departemen:

- HR
- Finance
- IT

Setiap departemen ditempatkan pada VLAN dan network yang berbeda untuk melakukan segmentasi jaringan.

Router digunakan untuk memungkinkan komunikasi antar-VLAN menggunakan metode Router-on-a-Stick. Selain itu, Extended ACL diterapkan untuk membatasi komunikasi antara jaringan HR dan Finance.

## Tujuan

Project ini dibuat untuk mempraktikkan:

1. Membuat segmentasi jaringan menggunakan VLAN.
2. Mengatur access port pada switch.
3. Mengonfigurasi 802.1Q trunk antara switch dan router.
4. Mengimplementasikan inter-VLAN routing menggunakan Router-on-a-Stick.
5. Mengonfigurasi default gateway untuk setiap VLAN.
6. Menerapkan Extended ACL untuk membatasi akses dari HR ke Finance.
7. Melakukan pengujian konektivitas dan ACL menggunakan perintah Cisco IOS.

## Topologi Jaringan

![Topologi Jaringan](topology/network-topology.png)

Topologi terdiri dari:

- 1 Cisco ISR4300 Series Router
- 1 Cisco Switch
- 3 PC yang mewakili departemen HR, Finance, dan IT

Router dan switch terhubung melalui satu link yang dikonfigurasi sebagai trunk 802.1Q untuk membawa traffic dari VLAN 10, VLAN 20, dan VLAN 30.

## Pembagian VLAN dan IP Address

| Departemen | VLAN | Network | Gateway | Contoh Host |
|------------|------|---------|---------|-------------|
| HR | 10 | 192.168.10.0/24 | 192.168.10.1 | 192.168.10.10 |
| Finance | 20 | 192.168.20.0/24 | 192.168.20.1 | 192.168.20.10 |
| IT | 30 | 192.168.30.0/24 | 192.168.30.1 | 192.168.30.10 |

## Konfigurasi Switch

Port yang digunakan untuk masing-masing departemen:

| Port Switch | Mode | VLAN | Departemen |
|-------------|------|------|------------|
| Fa0/1 | Access | 10 | HR |
| Fa0/2 | Access | 20 | Finance |
| Fa0/3 | Access | 30 | IT |
| Gi0/1 | Trunk | 802.1Q | Router |

Port `Gi0/1` digunakan sebagai trunk untuk membawa traffic dari VLAN 10, VLAN 20, dan VLAN 30 menuju router.

## Router-on-a-Stick

Router menggunakan beberapa subinterface pada `GigabitEthernet0/0/0`:

| Subinterface | VLAN | IP Address |
|--------------|------|------------|
| G0/0/0.10 | 10 | 192.168.10.1/24 |
| G0/0/0.20 | 20 | 192.168.20.1/24 |
| G0/0/0.30 | 30 | 192.168.30.1/24 |

Setiap IP address pada subinterface digunakan sebagai default gateway untuk VLAN masing-masing.

## Kebijakan Keamanan

Kebijakan keamanan yang diterapkan:

| Source | Destination | Aksi |
|--------|-------------|------|
| HR | Finance | ❌ Ditolak |
| HR | IT | ✅ Diizinkan |

Extended ACL dengan nama `BLOCK-HR-FINANCE` dibuat untuk menerapkan kebijakan tersebut.

ACL diterapkan secara inbound pada:

```text
GigabitEthernet0/0/0.10
```

Dengan konfigurasi tersebut, traffic yang berasal dari network HR akan diperiksa oleh ACL ketika memasuki router melalui subinterface tersebut.

## Konfigurasi ACL

ACL yang digunakan:

```text
ip access-list extended BLOCK-HR-FINANCE
 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
 permit ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
```

ACL kemudian diterapkan pada subinterface HR:

```text
interface gigabitEthernet 0/0/0.10
ip access-group BLOCK-HR-FINANCE in
```

## Pengujian

### 1. Pengujian HR → Finance

Pengujian dilakukan dari PC HR menggunakan:

```text
ping 192.168.20.10
```

Hasil pengujian:

```text
Reply from 192.168.10.1: Destination host unreachable.

Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

Hasil:

**Traffic dari HR menuju Finance berhasil diblokir sesuai dengan konfigurasi ACL.**

### 2. Pengujian HR → IT

Pengujian dilakukan menggunakan:

```text
ping 192.168.30.10
```

Hasil pengujian:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

Hasil:

**Traffic dari HR menuju IT berhasil diteruskan.**

## Verifikasi ACL

ACL diverifikasi menggunakan perintah:

```text
show access-lists
```

Hasil pengujian:

```text
Extended IP access list BLOCK-HR-FINANCE

10 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255 (4 match(es))

20 permit ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255 (8 match(es))
```

Nilai `match(es)` menunjukkan bahwa rule ACL telah memproses traffic selama pengujian.

## Troubleshooting

Selama proses konfigurasi, terdapat beberapa kondisi yang perlu diperiksa dan diperbaiki.

### Interface Router Administratively Down

Pada awal konfigurasi, `GigabitEthernet0/0/0` berada dalam kondisi:

```text
administratively down
```

Interface kemudian diaktifkan menggunakan:

```text
interface gigabitEthernet 0/0/0
no shutdown
```

Setelah itu, status interface berubah menjadi `up/up`.

### CDP Neighbor Tidak Terlihat

Pada awalnya, `show cdp neighbors` tidak menampilkan perangkat tetangga.

Setelah interface router diaktifkan, koneksi antara switch dan router dapat terdeteksi.

Koneksi yang terdeteksi:

```text
Switch Gi0/1 ↔ Router Gi0/0/0
```

### Konfigurasi Trunk

Port `Gi0/1` pada switch dikonfigurasi sebagai trunk menggunakan:

```text
interface gigabitEthernet 0/1
switchport mode trunk
```

Konfigurasi kemudian diverifikasi menggunakan:

```text
show interfaces trunk
```

Interface menunjukkan status `trunking` dengan encapsulation `802.1Q`.

### Inter-VLAN Routing

Sebelum Router-on-a-Stick dikonfigurasi, PC yang berada pada VLAN berbeda belum dapat berkomunikasi.

Untuk memungkinkan komunikasi antar-VLAN, dibuat tiga subinterface pada router:

```text
G0/0/0.10
G0/0/0.20
G0/0/0.30
```

Setiap subinterface kemudian diberikan VLAN ID menggunakan `encapsulation dot1Q` dan IP address sebagai default gateway.

### Pengujian Ping

Pada beberapa pengujian, ping pertama mengalami timeout sementara ping berikutnya berhasil.

Pengujian kemudian diulangi untuk memastikan konektivitas.

## Perintah Verifikasi

Beberapa perintah Cisco IOS yang digunakan dalam project ini:

```text
show vlan brief
show cdp neighbors
show interfaces trunk
show ip interface brief
show access-lists
```

Perintah tersebut digunakan untuk memeriksa:

- VLAN dan port
- Koneksi antarperangkat
- Status trunk
- Status interface dan IP address
- Aktivitas ACL

## Teknologi yang Digunakan

- Cisco Packet Tracer
- Cisco IOS
- IPv4
- VLAN
- 802.1Q Trunking
- Router-on-a-Stick
- Inter-VLAN Routing
- Extended ACL
- ICMP / Ping
- CDP

## Hal yang Dipelajari

Melalui project ini, saya mempraktikkan:

- Segmentasi jaringan menggunakan VLAN
- Perbedaan access port dan trunk port
- IPv4 addressing
- Default gateway
- Router-on-a-Stick
- Inter-VLAN routing
- Extended ACL
- Source dan destination pada traffic
- Network troubleshooting
- Verifikasi konfigurasi menggunakan Cisco IOS

## Status Project

**Selesai**

Konfigurasi jaringan, inter-VLAN routing, ACL, dan pengujian konektivitas telah berhasil dilakukan menggunakan Cisco Packet Tracer.
