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
6. Pengukuran Baseline
7. Menjalankan Simulasi Slowloris
8. Mitigasi di Nginx (Contoh Konfigurasi)
9. Evaluasi & Pengukuran Pasca-Mitigation
10. Cleanup & Revert
11. Troubleshooting
12. Lampiran (Perintah & Contoh File)


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
* Jaringan VM: Host-only / Internal / Bridged (pastikan terisolasi).
* Tools opsional: `curl`, `ab`/`wrk`, `htop`, `ss`.


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


## 6. Pengukuran Baseline (Sebelum Serangan)

Rekam metrik baseline untuk perbandingan:

* CPU/Memory: `top` / `htop` / `free -h`
* Jumlah koneksi TCP: `ss -s` ; `ss -antp | grep nginx`
* Waktu respon HTTP:

```bash
# simpan format curl jika perlu
curl -w "@curl-format.txt" -o /dev/null -s "http://<IP_VICTIM>/"
```

(Opsional) Gunakan `ab` atau `wrk` untuk test beban terkontrol.


## 7. Menjalankan Simulasi Slowloris

> Sesuaikan parameter (jumlah sockets) dengan kapasitas lab. Jangan langsung gunakan angka ekstrem.

Contoh menjalankan:

```bash
# dari direktori slowloris
python3 slowloris.py <IP_VICTIM> 10000
```

Amati pada Victim:

* Banyak koneksi ESTABLISHED/WAIT
* Respons HTTP menjadi lambat atau timeout
* CPU/RAM meningkat (jika relevan)
* Log nginx berisi error client timeouts / connection limits

**Catatan keselamatan:** jika layanan host lain terganggu, hentikan serangan segera (Ctrl+C).


## 8. Mitigasi di Nginx (Contoh)

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

### 8.1. Opsi tambahan

* Gunakan modul `ngx_http_limit_req_module` & `ngx_http_limit_conn_module`.
* Pertimbangkan reverse proxy / WAF / load balancer pada lingkungan produksi.
* Atur `worker_connections` / `worker_processes` jika perlu.


## 9. Evaluasi & Pengukuran Pasca-Mitigation

Ulangi langkah serangan setelah menerapkan mitigasi dan bandingkan:

* Availability halaman (akses via browser)
* Jumlah koneksi TCP (`ss -s`)
* Waktu respon (`curl`, `wrk`)
* CPU / Memory
* Log Nginx

Catat apakah mitigasi mengurangi dampak (mis. halaman kembali responsif, koneksi berkurang).


## 10. Cleanup & Revert

* Hentikan serangan pada Attacker (`Ctrl+C` atau kill process).
* Rollback konfigurasi Nginx jika perlu (atau gunakan snapshot VM untuk revert cepat).
* Simpan log & hasil eksperimen di folder `reports/` pada repo.
* Jika menggunakan snapshot, revert ke snapshot sebelum eksperimen.


## 11. Troubleshooting

* **Nginx tidak mau start**

  * Jalankan `sudo nginx -t` untuk cek error; perbaiki konfigurasi; lihat `/var/log/nginx/error.log`.
* **Tidak bisa ping antar VM**

  * Periksa pengaturan adapter di VirtualBox (bridged/host-only), firewall di VM.
* **Slowloris tidak jalan (error Python)**

  * Pastikan Python3 terinstall, jalankan `python3 --version`, cek dependensi pada repo slowloris.
* **Tidak ada efek saat serangan**

  * Mungkin parameter sockets terlalu kecil; naikkan bertahap, tetap di lingkungan lab terisolasi.


## 12. Lampiran — Perintah & Contoh File

### 12.1. Perintah monitoring ringkas

```bash
# CPU/Memory
top
free -h

# TCP sockets (ringkasan)
ss -s

# Koneksi ke proses nginx (ganti <nginx_pid> bila perlu)
ss -antp | grep nginx

# Nginx logs
tail -f /var/log/nginx/access.log /var/log/nginx/error.log
```

### 12.2. Contoh file `nginx-mitigation-example.conf`

Simpan di `configs/nginx-mitigation-example.conf` (opsional):

```nginx
http {
    client_header_timeout 10s;
    client_body_timeout 10s;
    keepalive_timeout 10s;

    limit_conn_zone $binary_remote_addr zone=addr:10m;
    limit_conn addr 10;

    limit_req_zone $binary_remote_addr zone=req_per_ip:10m rate=10r/s;

    server {
        listen 80 default_server;
        server_name _;
        root /var/www/html;
        index index.html;
    }
}
```


## Kontak & Kontributor

**Kelompok 8 — Keamanan Data**
(Tambahkan nama anggota, email, atau kontak institusi di sini jika perlu.)

