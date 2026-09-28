# Secure Multi-VLAN Office Network

Simulasi jaringan kantor menggunakan **Cisco Packet Tracer** dengan implementasi VLAN, 802.1Q trunking, Router-on-a-Stick, inter-VLAN routing, dan Extended ACL untuk mengatur komunikasi antar-departemen.

## Deskripsi Project

Project ini merupakan simulasi jaringan kantor sederhana yang membagi jaringan menjadi tiga departemen:

- HR
- Finance
- IT

Setiap departemen ditempatkan pada VLAN dan subnet yang berbeda untuk memberikan segmentasi jaringan.

Komunikasi antar-VLAN dilakukan menggunakan metode **Router-on-a-Stick**, sedangkan **Extended ACL** digunakan untuk membatasi komunikasi tertentu antar-departemen.

Project ini dibuat sebagai bagian dari portofolio untuk mendemonstrasikan pemahaman dasar mengenai konfigurasi dan troubleshooting jaringan Cisco.

---

## Tujuan

Project ini bertujuan untuk mempraktikkan:

1. Segmentasi jaringan menggunakan VLAN.
2. Konfigurasi access port pada switch.
3. Konfigurasi 802.1Q trunking.
4. Implementasi Router-on-a-Stick.
5. Konfigurasi inter-VLAN routing.
6. Konfigurasi default gateway untuk setiap VLAN.
7. Implementasi Extended ACL.
8. Pengujian konektivitas menggunakan ICMP/Ping.
9. Verifikasi konfigurasi menggunakan Cisco IOS.
10. Troubleshooting konektivitas jaringan.

---

## Topologi Jaringan

![Network Topology](topology/network-topology.png)

Topologi terdiri dari:

- 1 Cisco ISR4300 Series Router
- 1 Cisco Switch
- 3 PC sebagai representasi departemen:
  - HR
  - Finance
  - IT

Switch terhubung ke router menggunakan satu link trunk 802.1Q.

Link trunk membawa traffic dari VLAN 10, VLAN 20, dan VLAN 30.

---

## VLAN & IP Addressing

| Departemen | VLAN | Network | Gateway | Host |
|---|---:|---|---|---|
| HR | 10 | 192.168.10.0/24 | 192.168.10.1 | 192.168.10.10 |
| Finance | 20 | 192.168.20.0/24 | 192.168.20.1 | 192.168.20.10 |
| IT | 30 | 192.168.30.0/24 | 192.168.30.1 | 192.168.30.10 |

Setiap VLAN memiliki subnet sendiri sehingga broadcast domain antar-departemen terpisah.

---

## Switch Configuration

| Port | Mode | VLAN | Departemen |
|---|---|---:|---|
| Fa0/1 | Access | 10 | HR |
| Fa0/2 | Access | 20 | Finance |
| Fa0/3 | Access | 30 | IT |
| Gi0/1 | Trunk | 10, 20, 30 | Router |

### VLAN Configuration

```text
vlan 10
 name HR

vlan 20
 name FINANCE

vlan 30
 name IT
```

### Access Port Configuration

```text
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 30
```

### Trunk Configuration

```text
interface GigabitEthernet0/1
 switchport mode trunk
```

---

## Router-on-a-Stick

Router menggunakan satu physical interface dengan beberapa subinterface untuk menangani traffic dari VLAN yang berbeda.

| Subinterface | VLAN | IP Address |
|---|---:|---|
| G0/0/0.10 | 10 | 192.168.10.1/24 |
| G0/0/0.20 | 20 | 192.168.20.1/24 |
| G0/0/0.30 | 30 | 192.168.30.1/24 |

Konfigurasi:

```text
interface GigabitEthernet0/0/0
 no shutdown

interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface GigabitEthernet0/0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
```

Setiap subinterface digunakan sebagai **default gateway** untuk host pada VLAN masing-masing.

---

## Inter-VLAN Routing

Router digunakan untuk meneruskan traffic antar-VLAN.

```text
HR VLAN 10
192.168.10.0/24
        |
        v
192.168.10.1
        |
      Router
        |
        v
192.168.30.1
        |
IT VLAN 30
192.168.30.0/24
```

Dengan Router-on-a-Stick, perangkat pada VLAN yang berbeda dapat berkomunikasi melalui router.

---

## Security Policy

Extended ACL digunakan untuk mengatur komunikasi antar-departemen.

| Source | Destination | Aksi |
|---|---|---|
| HR | Finance | ❌ Deny |
| HR | IT | ✅ Permit |

Kebijakan utama project adalah membatasi akses dari jaringan **HR menuju Finance**, sementara komunikasi HR menuju IT tetap diperbolehkan.

---

## Extended ACL

ACL yang digunakan:

```text
ip access-list extended BLOCK-HR-FINANCE
 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
 permit ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
```

ACL diterapkan secara inbound pada subinterface VLAN HR:

```text
interface GigabitEthernet0/0/0.10
 ip access-group BLOCK-HR-FINANCE in
```

Dengan konfigurasi tersebut, traffic yang berasal dari network HR diperiksa sebelum diteruskan ke network tujuan.

---

## Connectivity Testing

### HR → Finance

```text
ping 192.168.20.10
```

Traffic dari HR menuju Finance diblokir oleh ACL.

```text
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

**Status: Blocked as intended.**

### HR → IT

```text
ping 192.168.30.10
```

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

**Status: Successful.**

### Finance → IT

```text
ping 192.168.30.10
```

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

**Status: Successful.**

### IT → HR

```text
ping 192.168.10.10
```

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

**Status: Successful.**

---

## ACL Verification

ACL diverifikasi menggunakan:

```text
show access-lists
```

Contoh rule:

```text
Extended IP access list BLOCK-HR-FINANCE

10 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
20 permit ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
```

Counter `match(es)` dapat digunakan untuk memastikan rule ACL telah memproses traffic selama pengujian.

---

## Troubleshooting

### Interface Router Administratively Down

Jika interface router berada dalam kondisi:

```text
administratively down
```

gunakan:

```text
interface GigabitEthernet0/0/0
 no shutdown
```

Verifikasi:

```text
show ip interface brief
```

### Memastikan VLAN Assignment

```text
show vlan brief
```

Digunakan untuk memastikan setiap PC berada pada VLAN yang sesuai.

### Memastikan Trunk

```text
show interfaces trunk
```

Digunakan untuk memastikan link antara switch dan router berfungsi sebagai trunk.

### Memastikan Subinterface

```text
show ip interface brief
```

Subinterface yang digunakan:

```text
G0/0/0.10
G0/0/0.20
G0/0/0.30
```

### Memastikan Default Gateway

| VLAN | Default Gateway |
|---:|---|
| 10 | 192.168.10.1 |
| 20 | 192.168.20.1 |
| 30 | 192.168.30.1 |

Kesalahan default gateway dapat menyebabkan host tidak dapat berkomunikasi dengan jaringan lain.

---

## Verification Commands

Perintah Cisco IOS yang digunakan:

```text
show vlan brief
show interfaces trunk
show ip interface brief
show cdp neighbors
show access-lists
```

Perintah tersebut digunakan untuk memverifikasi:

- VLAN dan assignment port
- Status trunk
- Status interface
- Koneksi antarperangkat
- Aktivitas ACL

---

## Network Design Overview

```text
                 +----------------+
                 |     Router     |
                 | Router-on-Stick|
                 +-------+--------+
                         |
                    802.1Q Trunk
                         |
                 +-------+--------+
                 |     Switch     |
                 +---+-----+---+--+
                     |     |   |
                   VLAN10 VLAN20 VLAN30
                     |     |   |
                    HR  Finance IT
```

Segmentasi VLAN digunakan untuk memisahkan broadcast domain, sedangkan router digunakan untuk menyediakan komunikasi antar-VLAN.

---

## Technologies Used

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
- Network Troubleshooting

---

## Skills Demonstrated

Project ini mendemonstrasikan kemampuan dalam:

- VLAN configuration
- Access port configuration
- Trunk configuration
- IPv4 addressing
- Default gateway configuration
- Router-on-a-Stick
- Inter-VLAN routing
- Extended ACL
- Traffic filtering
- Connectivity testing
- Cisco IOS verification
- Basic network troubleshooting

---

## Project Files

Struktur repository:

```text
project-1-secure-multi-vlan-office-network/
│
├── README.md
│
├── topology/
│   └── network-topology.png
│
└── packet-tracer/
    └── secure-multi-vlan-office-network.pkt
```

File `.pkt` dapat dibuka menggunakan **Cisco Packet Tracer** untuk melihat dan menguji konfigurasi jaringan secara langsung.

---

## Project Status

**Completed**

Project ini telah menyelesaikan implementasi:

- VLAN segmentation
- Access port
- 802.1Q trunking
- Router-on-a-Stick
- Inter-VLAN routing
- Extended ACL
- Connectivity testing
- Basic troubleshooting

Project berhasil disimulasikan menggunakan Cisco Packet Tracer.
