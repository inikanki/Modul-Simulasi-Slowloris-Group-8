# USER MANUAL

**Modul Simulasi Slowloris (DDoS & DoS)**
Kelompok 8 — Keamanan Data

> ⚠️ **Peringatan penting**
> Semua instruksi di manual ini hanya untuk lingkungan lab terisolasi yang kamu miliki atau yang mendapat izin eksplisit. DILARANG menjalankan serangan terhadap sistem publik atau pihak lain tanpa izin.

## Daftar Isi

1. Tujuan
2. Ruang Lingkup & Etika
3. Prasyarat & Perangkat
4. Topologi & Spesifikasi VM
5. Persiapan Lingkungan
6. Menjalankan Simulasi Slowloris
7. Mitigasi di Nginx (Contoh Konfigurasi)
8. Evaluasi & Pengukuran Pasca-Mitigation


## 1. Tujuan

Memberikan panduan praktis untuk mensimulasikan serangan *low-and-slow* (Slowloris) terhadap server web (Nginx) di lingkungan virtual terisolasi, memantau dampak, dan menguji langkah mitigasi konfigurasi Nginx.


## 2. Ruang Lingkup & Etika

* Simulasi hanya dilakukan di VM yang dikelola peserta atau lab tertutup.
* Dokumentasikan semua tindakan dan ambil snapshot sebelum eksperimen.
* Hentikan experiment bila terjadi kondisi tidak aman.
* Hasil penelitian tidak boleh dipublikasikan tanpa izin.


## 3. Prasyarat & Perangkat

* Host: PC/Laptop dengan VirtualBox atau VMware.
* Dua VM: Victim (Ubuntu 22.04) & Attacker (Kali Linux atau distro pentest).
* Git & Python (pada Attacker) untuk menjalankan slowloris.
* Akses `sudo`/root pada kedua VM.
* Jaringan VM: NAT / Bridged (pastikan terisolasi).


## 4. Topologi & Spesifikasi VM (Rekomendasi)

* **Victim (Ubuntu 22.04)**

  * RAM: 2 GB
  * vCPU: 3
  * Disk: 25 GB
  * Layanan: Nginx (package `nginx-full`)
* **Attacker (Kali Linux)**

  * RAM: 2 GB
  * vCPU: 4
  * Disk: sesuai kebutuhan
* Jaringan: Pastikan kedua VM dapat saling ping (`ping <IP_VICTIM>`).


## 5. Persiapan Lingkungan

### 5.1. Victim — Install & verifikasi Nginx

```bash
sudo apt update
sudo apt install -y nginx-full
sudo systemctl enable --now nginx
systemctl status nginx
```

Buka browser pada host/VM client: `http://<IP_VICTIM>` → harus tampil `Welcome to nginx!`.

### 5.2. Attacker — Clone Slowloris

```bash
# pada VM attacker
sudo apt update
sudo apt install -y git python3
git clone https://github.com/gkbrk/slowloris
cd slowloris
```


## 6. Menjalankan Simulasi Slowloris

> Sesuaikan parameter (jumlah sockets) dengan kapasitas lab. Jangan langsung gunakan angka ekstrem.

Contoh menjalankan:

```bash
# dari direktori slowloris
python3 slowloris.py <IP_VICTIM> 10000
```

Amati pada Victim:
* Log nginx berisi error client timeouts / connection limits

**Catatan keselamatan:** jika layanan host lain terganggu, hentikan serangan segera (Ctrl+C).


## 7. Mitigasi di Nginx (Contoh)

Buka dan edit konfigurasi utama (`/etc/nginx/nginx.conf`) atau file di `sites-available` sesuai arsitektur.

**Contoh blok konfigurasi (letakkan di dalam konteks `http { ... }`):**

```nginx
# TIMEOUTS
client_header_timeout 10s;
client_body_timeout 10s;
keepalive_timeout 10s;

# LIMIT CONNECTIONS PER IP
limit_conn_zone $binary_remote_addr zone=addr:10m;
limit_conn addr 10;

# LIMIT REQUEST RATE PER IP
limit_req_zone $binary_remote_addr zone=req_per_ip:10m rate=10r/s;
```

Setelah edit:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

## 8. Evaluasi & Pengukuran Pasca-Mitigation

Ulangi langkah serangan setelah menerapkan mitigasi dan bandingkan:
* Log Nginx

Catat apakah mitigasi mengurangi dampak (mis. halaman kembali responsif, koneksi berkurang).


## Kontak & Kontributor

**Kelompok 8 — Keamanan Data**
(Tambahkan nama anggota, email, atau kontak institusi di sini jika perlu.)

