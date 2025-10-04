# MODUL SIMULASI SLOWLORIS DDoS & DoS


## ⚠️ Peringatan / Disclaimer

Semua eksperimen **harus** dijalankan **hanya** pada infrastruktur milik sendiri atau lingkungan lab yang mendapat izin eksplisit.
DILARANG menargetkan sistem publik atau pihak lain tanpa izin — tindakan tersebut melanggar hukum dan etika.


## Deskripsi singkat

Modul ini menjelaskan konsep DoS/DDoS, tipe serangan (volumetric, protocol, application-layer), dan memberikan skenario praktikum yang fokus pada **Slowloris**. Peserta men-setup dua VM (Victim + Attacker), menjalankan serangan terkontrol, mengumpulkan metrik, lalu menerapkan konfigurasi mitigasi pada **Nginx**.


## Alat & prasyarat

* Virtualization: **Oracle VirtualBox** (atau VMware)
* **Victim VM:** Ubuntu 22.04 (Nginx)
* **Attacker VM:** Kali Linux (atau distro pentest)
* Slowloris script: `https://github.com/gkbrk/slowloris` (clone pada Attacker)
* Akses `sudo`/root pada kedua VM
* Jaringan VM: host-only / internal / bridged sesuai isolasi lab
* Ambil snapshot sebelum pengujian

---

## Konfigurasi VM (rekomendasi)

* **Victim (Ubuntu 22.04)**

  * RAM: 2 GB
  * vCPU: 3
  * Disk: 25 GB
* **Attacker (Kali Linux)**

  * RAM: 2 GB
  * vCPU: 4
  * Disk: sesuai kebutuhan

## Hasil yang diharapkan

Peserta dapat:

1. Menjelaskan perbedaan kelas DDoS (volumetric, protocol, application-layer).
2. Mengamati perilaku Slowloris (konektivitas partial/slow) yang menghabiskan slot koneksi.
3. Menerapkan mitigasi dasar di Nginx (timeout, limit_conn, limit_req) dan mengevaluasi efektivitasnya.
4. Menyusun laporan eksperimen berisi metode, metrik, dan rekomendasi.


## Etika & Keamanan

* Uji hanya dalam lab terisolasi.
* Simpan log dan dokumentasi perubahan konfigurasi.
* Kembalikan VM ke kondisi semula setelah eksperimen (atau gunakan snapshot revert).
* Jangan mempublikasikan hasil serangan ke target yang tidak berizin.


## Referensi

* Slowloris GitHub: `https://github.com/gkbrk/slowloris`
* Dokumentasi Nginx (timeout, limit_conn, limit_req)
* Modul Simulasi Slowloris DDoS & DoS (dokumen internal Kelompok 8)


## Kontributor

**Kelompok 8 — Keamanan Data**



## Lisensi

Materi ini disusun untuk tujuan **pendidikan**. Kode Slowloris mengikuti lisensi dari repo asal — periksa lisensi repo tersebut sebelum penggunaan ulang. Untuk modul/dokumen internal, tambahkan lisensi sesuai kebijakan institusi (mis. CC BY-NC-SA).

