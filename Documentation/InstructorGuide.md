# INSTRUCTOR GUIDE — Modul Simulasi Slowloris (DDoS & DoS)

**Kelompok 8 — Keamanan Data**
Dokumen ini adalah panduan lengkap untuk pengajar yang akan menyampaikan modul praktikum simulasi Slowloris pada lingkungan lab terisolasi. Salin-tempel langsung ke `INSTRUCTOR_GUIDE.md`.


## Ringkasan singkat

Modul ini memperkenalkan peserta pada kategori serangan DoS/DDoS, fokus pada *low-and-slow* attack (Slowloris), melakukan simulasi terkontrol dalam VM, mengumpulkan metrik performa, dan menerapkan mitigasi di Nginx. Eksperimen hanya dijalankan di lingkungan yang dimiliki/instruksikan dengan izin eksplisit.


## Tujuan Pembelajaran (Learning Objectives)

Setelah mengikuti sesi ini, peserta diharapkan mampu:

1. Menjelaskan perbedaan kelas serangan DoS/DDoS (volumetric, protocol, application-layer).
2. Menjabarkan mekanisme kerja Slowloris dan mengapa ia efektif terhadap server HTTP berbasis koneksi.
3. Menyiapkan topologi VM terisolasi (Victim + Attacker) dan mengambil snapshot sebelum eksperimen.
4. Menjalankan skrip Slowloris secara terkontrol dan mengamati metrik sistem dan aplikasi.
5. Menganalisis hasil pengukuran (CPU, memory, jumlah koneksi, response time).
6. Menerapkan dan menguji konfigurasi mitigasi Nginx (timeout, limit_conn, limit_req).
7. Menyusun laporan laboratorium yang memuat metode, data, analisis, dan rekomendasi keamanan.


## Target Peserta & Prasyarat

* Target: mahasiswa/trainee keamanan jaringan tingkat dasar-menengah.
* Prasyarat peserta:

  * Pengalaman dasar Linux (terminal, editing file).
  * Pemahaman TCP/IP & HTTP dasar.
  * Izin akses ke VirtualBox/VM host.
* Prasyarat instruktur:

  * Mengetahui Nginx config dasar dan perintah monitoring Linux.
  * Menyiapkan snapshot VM dan kontrol jaringan lab.


## Durasi & Format Sesi

Total durasi rekomendasi: **3 — 4 jam** (dapat dipecah ke 2 pertemuan).
Pembagian waktu (contoh 4 jam):

1. **Pendahuluan & teori** — 30 menit

   * Konsep DoS/DDoS, klasifikasi, dan Slowloris.
2. **Persiapan lingkungan & demo singkat** — 30 menit

   * Menyiapkan VM, install Nginx, clone Slowloris.
3. **Praktikum: Baseline & Serangan** — 60 menit

   * Baseline metrics, jalankan Slowloris bertahap.
4. **Mitigasi & Re-test** — 45 menit

   * Terapkan konfigurasi Nginx, uji ulang.
5. **Analisa & Diskusi** — 30 menit

   * Bandingkan metrik, diskusikan trade-offs.
6. **Penilaian & Penutup** — 15 menit

   * Pengumpulan laporan, kuis singkat.

Catatan: sisihkan waktu tambahan untuk troubleshooting.


## Perlengkapan & Bahan Ajar

* Host dengan VirtualBox/VMware + ISO Ubuntu 22.04 + Kali Linux.
* File modul PDF (sumber bahan bagi peserta).
* Repository Slowloris: `https://github.com/gkbrk/slowloris`.


## Persiapan Instruktur (sebelum kelas)

1. Siapkan dua VM template (Victim + Attacker). Konfigurasikan IP statis atau pastikan diketahui.
2. Ambil snapshot VM template sebelum eksperimen.
3. Pastikan Nginx terinstall pada Victim dan Slowloris diclone pada Attacker.
4. Taruh file contoh `nginx-mitigation-example.conf` di repositori lab.
5. Siapkan salinan *lab report template* dan rubrik penilaian untuk distribusi.
6. Cek firewall host agar lab terisolasi dari jaringan publik (host-only / internal network).


## Rencana Pelaksanaan — Langkah demi langkah (Detail Praktikum)

### A. Verifikasi awal (10 menit)

* Instruktur menunjukkan topologi: Host → VirtualBox → VM Victim (Ubuntu) & VM Attacker (Kali).
* Perintahkan peserta mengecek konektivitas: `ping <IP_VICTIM>`.

### B. Baseline — Victim (15 menit)

Instruksikan peserta untuk mencatat kondisi awal:

```bash
# pada Victim
sudo apt update
# (jika belum terinstall) sudo apt install -y nginx-full htop
systemctl status nginx
# monitoring singkat
top -b -n1 | head -n15
free -h
ss -s
ss -antp | grep nginx || true
tail -n 20 /var/log/nginx/access.log
tail -n 20 /var/log/nginx/error.log
```

* Bimbing peserta untuk mengakses `http://<IP_VICTIM>` secara manual.

### C. Setup Attacker (10 menit)

```bash
# pada Attacker
sudo apt update
sudo apt install -y git python3
git clone https://github.com/gkbrk/slowloris
cd slowloris
ls -la
```

* Perlihatkan isi skrip Slowloris dan opsi penggunaannya.

### D. Pengukuran Baseline Response Time (10 menit)

Contoh: gunakan `curl` untuk mengukur response time:

```bash
# optional: buat file curl-format.txt atau gunakan inline
curl -o /dev/null -s -w "time_total: %{time_total}s\n" "http://<IP_VICTIM>/"
```

Instruksikan peserta menyimpan hasil baseline ke laporan.

### E. Menjalankan Simulasi (Serangan Bertahap) (30–40 menit)

**Penting**: ingatkan etika dan batasi target ke VM lab saja.

* Mulai dengan parameter kecil, amati, lalu tingkatkan:

```bash
# contoh awal (sesuaikan)
python3 slowloris.py <IP_VICTIM> 500
# lalu tingkatkan
python3 slowloris.py <IP_VICTIM> 2000
python3 slowloris.py <IP_VICTIM> 10000
```

* Instruksikan peserta memonitor Victim selama serangan:

  * `ss -antp | grep nginx` — periksa ESTABLISHED sockets.
  * `top` / `htop` — CPU & memory.
  * `tail -f /var/log/nginx/error.log` — error terkait koneksi.
  * Akses `http://<IP_VICTIM>` dari browser untuk menguji availability.

**Expected instructor actions**: demonstrasikan peningkatan dampak saat sockets bertambah; diskusikan trade-offs (kenapa server kehabisan slot).

### F. Mitigasi (45 menit)

1. Tunjukkan file contoh `nginx-mitigation-example.conf`.
2. Pandu peserta untuk mengedit konfigurasi Nginx:

```bash
sudo nano /etc/nginx/nginx.conf
# tambahkan di dalam blok http { ... }:
client_header_timeout 10s;
client_body_timeout 10s;
keepalive_timeout 10s;

limit_conn_zone $binary_remote_addr zone=addr:10m;
limit_conn addr 10;

limit_req_zone $binary_remote_addr zone=req_per_ip:10m rate=10r/s;
```

3. Uji konfigurasi dan reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

4. Jalankan ulang Slowloris (parameter sama) dan minta peserta mengamati apakah halaman kembali responsif.

### G. Analisis & Laporan (30 menit)

Instruksikan peserta menulis laporan singkat berisi:

* Tujuan & desain eksperimen.
* Metrik baseline vs saat serangan vs pasca-mitigasi (tabel).
* Screenshot/hasil `ss`, `top`, `curl` output.
* Diskusi: efektivitas mitigasi, batasan, rekomendasi lanjutan (WAF, reverse proxy, auto-scaling).
* Kesimpulan & refleksi etika.


## Soal Latihan / Kuis (untuk penilaian singkat)

Berikan kuis singkat 5 soal (pilihan ganda / singkat) — dapat dipakai sebagai exit quiz.

1. **Apa tujuan utama serangan Slowloris? (jawaban singkat)**
   *Menjaga banyak koneksi HTTP terbuka dengan sangat lambat sehingga menghabiskan slot koneksi server.*

2. **Sebutkan 2 parameter Nginx yang membantu mitigasi Slowloris.**
   *client_header_timeout, keepalive_timeout (atau client_body_timeout, limit_conn, limit_req).*

3. **Apa perbedaan utama antara DoS & DDoS? (pilihan ganda)**
   A. DoS menggunakan banyak sumber, DDoS satu sumber
   B. DoS satu sumber, DDoS banyak sumber ← **Benar**
   C. Tidak ada perbedaan
   D. DDoS hanya menyerang protocol layer

4. **Sebutkan satu alasan kenapa limit_conn per-IP tidak selalu solusi sempurna pada lingkungan produksi. (jawaban singkat)**
   *Karena serangan dapat datang dari banyak IP (botnet) atau ada NAT/shared IP sehingga pengguna sah bisa dibatasi.*

5. **Berapa tindakan pertama yang harus dilakukan jika eksperimen menyebabkan gangguan jaringan di lab?**
   *Hentikan serangan (Ctrl+C), revert ke snapshot, dan dokumentasikan insiden.*

Berikan kunci jawaban saat menilai.


## Rubrik Penilaian Laporan (total 100 poin)

* **Kelengkapan eksperimen & dokumentasi** — 30 pts
  (snapshot, perintah yang dipakai, parameter, log)
* **Pengukuran & analisis** — 30 pts
  (tabel metrik baseline/attack/mitigation, interpretasi data)
* **Mitigasi & penjelasan konfigurasi** — 20 pts
  (mengaplikasikan perubahan Nginx, uji ulang, alasan pemilihan)
* **Etika & keamanan** — 10 pts
  (snapshot, isolasi, dokumentasi izin)
* **Kerapihan & presentasi laporan** — 10 pts


## Troubleshooting & Tips Instruktur

* **Nginx gagal reload**: jalankan `sudo nginx -t` untuk menemukan error. Periksa kurung kurawal yang hilang atau direktif di tempat yang salah.
* **Peserta tidak bisa ping VM**: cek pengaturan network adapter (host-only / bridged); matikan firewall sementara (`sudo ufw disable`) hanya untuk lab.
* **Slowloris script error (Python)**: pastikan `python3` terinstall; cek error trace untuk dependency; jalankan `python3 slowloris.py --help` bila tersedia.
* **Tidak terlihat efek serangan**: parameter sockets terlalu kecil atau worker_connections Nginx terlalu besar; tingkatkan bertahap tapi tetap aman.
* **Shared NAT IP**: jika semua peserta berbagi IP/public, pastikan lab terisolasi sehingga limit_conn tidak memblokir seluruh host.


## Variasi & Ekstensi Skenario (opsional)

1. **Scale-up**: gunakan beberapa VM attacker (clone Attacker) untuk mensimulasikan DDoS terdistribusi.
2. **Protocol attack demo**: singkat demo SYN flood (gunakan tools terkontrol) untuk perbandingan kelas serangan.
3. **Mitigasi lanjut**: konfigurasi reverse proxy (HAProxy), WAF sederhana, atau rate-limiting berbasis aplikasi.
4. **Monitoring lebih lanjut**: gunakan Prometheus + Grafana untuk metrik terstruktur.


## Checklist Keamanan & Etika (harus ditegaskan tiap sesi)

* [ ] Semua pengujian dijalankan hanya pada VM yang dimiliki/instruksikan.
* [ ] Snapshot VM diambil sebelum eksperimen.
* [ ] Jaringan lab diisolasi dari jaringan publik.
* [ ] Peserta menandatangani agreement bahwa tidak akan menjalankan serangan di luar lab.
* [ ] Semua log dan bukti disimpan untuk audit.


## Lampiran — Contoh File & Perintah (Siap Copy-Paste)

### Contoh `nginx-mitigation-example.conf`

Simpan di `configs/nginx-mitigation-example.conf`:

```nginx
http {
    # TIMEOUTS
    client_header_timeout 10s;
    client_body_timeout 10s;
    keepalive_timeout 10s;

    # LIMIT CONNECTIONS PER IP
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    limit_conn addr 10;

    # LIMIT REQUEST RATE PER IP
    limit_req_zone $binary_remote_addr zone=req_per_ip:10m rate=10r/s;

    server {
        listen 80 default_server;
        server_name _;
        root /var/www/html;
        index index.html;
    }
}
```

### Perintah monitoring ringkas (copyable)

```bash
# Cek nginx
systemctl status nginx
# Test config
sudo nginx -t
# Reload
sudo systemctl reload nginx

# Monitoring
top
htop
free -h
ss -s
ss -antp | grep nginx
tail -f /var/log/nginx/access.log /var/log/nginx/error.log

# Curl response time simple
curl -o /dev/null -s -w "time_total: %{time_total}s\n" "http://<IP_VICTIM>/"
```

### Contoh menjalankan Slowloris (copyable)

```bash
# di direktori slowloris pada Attacker
python3 slowloris.py <IP_VICTIM> 10000
# hentikan dengan Ctrl+C
```


## Template Slide Singkat (Outline)

1. Judul & tujuan praktikum
2. Apa itu DoS / DDoS (definisi + klasifikasi)
3. Slowloris — konsep & cara kerja
4. Topologi lab & prasyarat VM
5. Langkah eksperimen (baseline, attack, mitigasi)
6. Observasi metrik & hasil
7. Mitigasi Nginx (konfigurasi contoh)
8. Etika & kesimpulan


## Kontak Instruktur & Referensi

* **Kelompok 8 — Keamanan Data** (tambahkan email/nomor jika perlu)
* Referensi utama:

  * Repo Slowloris: `https://github.com/gkbrk/slowloris`
  * Dokumentasi Nginx (time-out, limit modules)
  * Modul Simulasi Slowloris DDoS & DoS (file PDF distribusi kursus)
