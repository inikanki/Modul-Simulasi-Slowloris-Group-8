# TECHNICAL SPECIFICATION

**Modul:** Simulasi Slowloris (DDoS & DoS) — Kelompok 8, Keamanan Data

**Versi dokumen:** 1.0
**Tanggal:** October 5, 2025
**Penulis:** Kelompok 8


## 1. Ringkasan Proyek

Dokumen ini menjelaskan spesifikasi teknis untuk menyiapkan, menjalankan, mengukur, dan memitigasi simulasi **Slowloris (application-layer low-and-slow)** terhadap server web (Nginx) dalam lingkungan VM terisolasi. Termasuk: arsitektur, konfigurasi VM, parameter serangan, konfigurasi mitigasi Nginx, metrik yang dikumpulkan, prosedur keamanan, dan skenario uji.


## 2. Tujuan Teknis

1. Menyediakan konfigurasi reproducible untuk lab simulasi Slowloris.
2. Mendefinisikan parameter serangan yang dapat direplikasi (socket counts, durations).
3. Menstandarisasi konfigurasi mitigasi Nginx untuk pengujian efektivitas.
4. Menetapkan metrik, format pengumpulan data, dan laporan hasil.
5. Menjamin eksperimen berjalan aman dalam lingkungan terisolasi dan dapat di-rollback.


## 3. Scope

* In-scope: pembuatan 2 VM (Victim + Attacker), instalasi Nginx, cloning Slowloris, pengukuran, mitigasi, re-test.


## 4. Arsitektur Sistem & Topologi Jaringan

Topologi minimal:

```
[Host Physical] --(VirtualBox)--> [VM Victim: Ubuntu 22.04 (Nginx)]
                                  [VM Attacker: Kali Linux (Slowloris)]
Network: NAT/Bridged (lab isolated)
```

* Semua VM berada pada network internal atau host-only sehingga tidak menyentuh jaringan publik.
* IP statis atau DHCP dengan alamat yang diketahui (contoh: Victim 192.168.56.10, Attacker 192.168.56.11).


## 5. Spesifikasi VM (Rekomendasi)

**Victim (Ubuntu 22.04)**

* OS: Ubuntu Server 22.04.5 LTS (minimal)
* CPU: 3 vCPU
* RAM: 2048 MB
* Disk: 25 GB
* Network Adapter: NAT / Bridged
* Software: nginx-full (versi terbaru di repo distro), curl, htop, ss

**Attacker (Kali Linux 2025.3 atau setara)**

* OS: Kali Linux (terupdate)
* CPU: 4 vCPU
* RAM: 2048 MB
* Disk: 20+ GB
* Software: git, python3, pip (jika diperlukan)

**Host**

* VirtualBox / VMware terbaru yang mendukung nested paging.
* Cukup resource untuk menjalankan 2 VM bersamaan (minimal 8 GB RAM fisik direkomendasikan).


## 6. Software & Versi yang Direkomendasikan

* Ubuntu 22.04.5 LTS — Victim.
* Nginx (package `nginx-full`) — versi yang tersedia di repo Ubuntu 22.04.
* Kali Linux 2025.3 — Attacker.
* Python 3.10+ — untuk menjalankan skrip Slowloris.
* Slowloris repo: `https://github.com/gkbrk/slowloris` (clone aktu al repo).

> Catatan: semua versi harus dicatat dalam laporan eksperimen.


## 7. Keamanan & Isolasi

* Gunakan **Host-only** atau **Internal Network** pada VirtualBox agar tidak memengaruhi jaringan publik.
* Ambil **snapshot** VM Victim & Attacker sebelum setiap set skenario.
* Nonaktifkan akses port-forwarding dari host ke VM yang dapat mengekspos VM ke jaringan publik.
