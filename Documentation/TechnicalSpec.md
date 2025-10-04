Oke 🔥 — berikut ini adalah **versi lengkap dan siap-paste ke GitHub** untuk file bernama
`TECHNICAL_SPACE.md`

Semua sudah disusun rapi dalam format Markdown GitHub, lengkap dengan heading, struktur, dan isi yang diambil dari dokumenmu.
Kamu tinggal **salin semuanya ke file baru di repo GitHub-mu** 👇

---

````markdown
# 🧠 TECHNICAL SPACE — Modul Simulasi DDoS (Slowloris)

**Kelompok 8 — Keamanan Data**  
_Politeknik / Universitas Anda_

---

## 📘 1. Overview

Ketersediaan layanan jaringan merupakan aspek penting dalam sistem informasi modern. Salah satu ancaman serius terhadap aspek ini adalah **DDoS (Distributed Denial of Service)** — sebuah serangan yang bertujuan membuat layanan, server, atau website menjadi tidak dapat diakses dengan cara membanjiri bandwidth, sumber daya protokol, atau aplikasi target.

Modul ini dirancang sebagai **panduan teknis dan eksperimental** untuk memahami cara kerja serangan DDoS (terutama jenis *application-layer* seperti **Slowloris**) dalam **lingkungan laboratorium terisolasi** menggunakan Virtual Machine (VM).  
Tujuannya agar peserta dapat:
- Memahami konsep dasar dan jenis-jenis serangan DDoS.
- Menjalankan simulasi serangan secara aman dan etis.
- Menerapkan konfigurasi mitigasi di server web (Nginx).
- Menganalisis dampak dan efektivitas mitigasi.

---

## ⚙️ 2. Objective & Scope

**Tujuan utama:**  
Menyimulasikan serangan DDoS tipe *low-and-slow* (Slowloris) terhadap server web Nginx dalam lab virtual, serta menerapkan langkah mitigasi untuk mengembalikan ketersediaan layanan.

**Ruang lingkup eksperimen:**
- Serangan hanya dilakukan pada **lingkungan lab virtual (VM terisolasi)**.
- Menggunakan dua mesin virtual: *Attacker* dan *Victim*.
- Fokus pada serangan **Slowloris (HTTP partial request)**.
- Menerapkan **mitigasi konfigurasi Nginx** seperti `timeout` dan `limit_conn`.

---

## 🧩 3. Lab Architecture

**Topologi Virtual Machine:**

| Role | OS | Tools | Fungsi |
|------|----|--------|--------|
| Victim | Ubuntu 22.04.5 | Nginx | Server web target |
| Attacker | Kali Linux 2025.3 | Slowloris (Python3) | Mesin penyerang |

**Spesifikasi VM (contoh konfigurasi VirtualBox):**
- **RAM:** 2 GB  
- **CPU:** 3–4 vCPU  
- **Disk:** 25 GB (VDI)  
- **Network:** Bridged Adapter / Host-Only (isolasi penuh)  
- **Virtualization:** Nested Paging, KVM Paravirtualization  

> Pastikan kedua VM dapat saling berkomunikasi (ping antar IP) dan memiliki akses `sudo/root`.

---

## 🧰 4. Tools & Environment

| Komponen | Versi / Deskripsi |
|-----------|-------------------|
| Oracle VirtualBox | 7.x atau lebih baru |
| Ubuntu Server | 22.04.5 LTS |
| Kali Linux | 2025.3 |
| Nginx | `nginx-full` package |
| Slowloris | [https://github.com/gkbrk/slowloris](https://github.com/gkbrk/slowloris) |
| Git | untuk clone repository Slowloris |

**Prasyarat:**
- Semua simulasi dilakukan di jaringan **Host-Only / Internal**.
- Pastikan setiap VM memiliki snapshot sebelum pengujian.
- Jangan pernah menjalankan serangan pada jaringan publik atau server yang tidak memiliki izin.

---

## 🧪 5. Simulation Steps

### A. Setup VM
1. Buat dua VM di **VirtualBox**:  
   - `ATTACK` → Kali Linux  
   - `VICTIM` → Ubuntu 22.04.5  
2. Alokasikan RAM 2 GB, CPU 3 core, disk 25 GB.  
3. Jalankan masing-masing VM dan login dengan hak `sudo/root`.

---

### B. Install dan Siapkan Tools
**Pada VM Victim:**
```bash
sudo apt update
sudo apt install nginx-full -y
systemctl status nginx
````

Akses melalui browser host:
`http://<IP_VICTIM>` → tampil halaman *Welcome to Nginx!* menandakan server aktif.

**Pada VM Attacker:**

```bash
git clone https://github.com/gkbrk/slowloris
cd slowloris
```

---

### C. Baseline Test

Sebelum serangan, uji akses normal ke Nginx dan catat:

* Waktu respon halaman
* Penggunaan CPU / memori
* Status koneksi TCP (`ss -tuna` atau `netstat`)

---

### D. Attack Simulation

Jalankan Slowloris dari mesin Attacker:

```bash
python3 slowloris.py <IP_VICTIM> 10000
```

> Angka `10000` menunjukkan jumlah socket terbuka.
> Sesuaikan dengan spesifikasi lab agar efek serangan terlihat namun tetap terkontrol.

Selama serangan, halaman web Nginx akan menjadi **lambat atau tidak merespons**.

---

### E. Mitigation Configuration

Edit konfigurasi Nginx di mesin Victim:

```bash
sudo nano /etc/nginx/nginx.conf
```

Tambahkan / sesuaikan nilai berikut:

```nginx
client_header_timeout 10s;
client_body_timeout 10s;
keepalive_timeout 5s 5s;
limit_conn_zone $binary_remote_addr zone=addr:10m;
limit_conn addr 10;
```

Simpan dan restart Nginx:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

### F. Re-run Attack & Observe

Jalankan ulang Slowloris.
Perhatikan apakah:

* Layanan Nginx kembali stabil.
* Waktu respon dan koneksi TCP membaik.

Catat hasil pengamatan untuk laporan analisis.

---

## 📈 6. Observation & Analysis

Dari hasil simulasi:

* Serangan **Slowloris** berhasil membuat server Nginx kehabisan slot koneksi aktif.
* Setelah mitigasi dengan parameter timeout & limit, server dapat kembali melayani permintaan sah.
* Peserta memahami mekanisme kerja serangan *application-layer* dan pentingnya pengaturan parameter Nginx dalam mempertahankan **availability**.

---

## 🛡️ 7. Ethical & Safety Notice

> ⚠️ **Seluruh simulasi hanya boleh dilakukan di lingkungan lab terisolasi.**
>
> * Jangan pernah menyerang sistem publik atau pihak lain tanpa izin.
> * Gunakan jaringan internal (Host-Only / NAT).
> * Ambil snapshot VM sebelum simulasi.
> * Dokumentasikan setiap langkah untuk audit dan pembelajaran.

Eksperimen ini bertujuan **pendidikan dan riset keamanan**, bukan untuk eksploitasi.

---

## 👥 8. Contributors

**Kelompok 8 — Keamanan Data**
Anggota:

* Nama 1
* Nama 2
* Nama 3
* Nama 4
* Nama 5

Dosen Pembimbing:

> *(Isi sesuai nama pengampu praktikum)*

---

## 📚 9. References

* gkbrk/slowloris — [https://github.com/gkbrk/slowloris](https://github.com/gkbrk/slowloris)
* Nginx Documentation — [https://nginx.org/en/docs/](https://nginx.org/en/docs/)
* OWASP: Denial of Service Attacks — [https://owasp.org/www-community/attacks/Denial_of_Service](https://owasp.org/www-community/attacks/Denial_of_Service)
* Modul Praktikum Keamanan Data — Kelompok 8 (2025)

---

> 🧾 **Note:**
> File ini adalah dokumentasi teknis untuk repository GitHub modul simulasi DDoS/Slowloris.
> Silakan gunakan sebagai *README tambahan* di folder `/docs/` atau `/technical-space/` repository Anda.

```

---

Apakah kamu ingin aku buatkan **README utama (`README.md`)** juga biar cocok dipasang bareng file ini di repositori GitHub (misalnya untuk tampilan depan repo)?  
Biasanya README lebih ringkas, sedangkan `TECHNICAL_SPACE.md` ini jadi dokumen teknis lengkapnya.
```
