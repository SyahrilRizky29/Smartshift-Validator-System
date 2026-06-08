# 🚀 SmartShift: Automated Validation & Telegram Reporting

**SmartShift** adalah solusi otomasi *backend* berbasis n8n yang dirancang untuk mengelola pendaftaran paket shift pegawai mingguan secara mandiri, akurat, dan terverifikasi secara *real-time*. Sistem ini mengeliminasi proses birokrasi manual dalam manajemen jadwal kerja dan memastikan setiap pengajuan mematuhi aturan internal serta standar ketat keselamatan kerja (K3 Fatigue Rule).

---

## 📑 Daftar Isi
1. [Tentang Project](#-tentang-project)
2. [Fitur Utama](#-fitur-utama)
3. [Arsitektur Sistem & Alur Kerja](#-arsitektur-sistem--alur-kerja)
4. [Panduan Implementasi Lengkap](#-panduan-implementasi-lengkap)
5. [Mekanisme Validasi (Fitur Unggulan)](#-mekanisme-validasi-fitur-unggulan)
6. [Dynamic Policy Control & Sistem Tatakelola](#-dynamic-policy-control--sistem-tatakelola)
7. [Panduan Dokumentasi (Screenshot)](#-panduan-dokumentasi-screenshot)

---

## 🧐 Tentang Project
Project ini dibangun memanfaatkan ekosistem n8n untuk menghubungkan form pendaftaran horizontal (paket mingguan) dengan database Google Sheets. Logika pemrosesan dan mesin aturan (*Rule Engine*) sepenuhnya dikendalikan menggunakan skrip JavaScript modern (Luxon API) untuk menghasilkan keputusan instan berupa notifikasi Gmail HTML interaktif serta pelaporan otomatis ke Telegram.

Target utama sistem ini difokuskan pada manajemen personil dengan rotasi kerja dinamis 24/7, seperti petugas keamanan (security), operator manufaktur pabrik, perawat rumah sakit, maupun tim teknis operasional lapangan.

## ✨ Fitur Utama
* 📱 **Web Portal Input**: Portal pengisian mandiri satu pintu berbasis paket mingguan (Senin–Jumat).
* 🛡️ **Dual-Gate Validation Checker**: Proteksi berlapis yang mendeteksi kecurangan pengajuan ganda (*Anti-Double Submit*) sekaligus mengalkulasi batas kelelahan fisik karyawan (*K3 Fatigue Rule*).
* 📨 **Rich HTML Multi-Channel Notification**: Pengiriman respon instan dengan desain transparan (Hijau untuk *Approved*, Merah untuk *Rejected* beserta detail matematis pelanggarannya).
* 📊 **Automated Weekly Report**: Penarikan data otomatis setiap hari Senin jam 08:00 WIB yang di-rekap dan ditembak langsung ke grup Telegram koordinasi.

---

## 🏗️ Arsitektur Sistem & Alur Kerja

Sistem berjalan menggunakan dua jenis arsitektur workflow di n8n:
1. **Main Workflow (Core Validation & Processing)**: Mengelola penangkapan data form, penarikan data pembanding ke spreadsheet, eksekusi JavaScript engine, percabangan kondisi (IF), hingga distribusi output (Gmail/Database).
2. **Mini-Server Workflow (Webhook Web Responder)**: Berperan sebagai server mini independen yang menangkap sinyal *redirect* pasca-submit form untuk menyajikan halaman sukses HTML *"Data Terkirim!"* secara instan.

---

## 🚀 Panduan Implementasi Lengkap

### Langkah 1: Persiapan Database Google Sheets
1. Buat Spreadsheet baru di Google Drive dengan nama **"Jadwal Shift Pegawai"**.
2. Konfigurasikan dua sheet internal sebagai berikut:
   * 📑 **Master_Jadwal**: Kolom wajib di baris pertama: `ID`, `Nama`, `Email`, `Tanggal`, `Shift - Senin`, `Shift - Selasa`, `Shift - Rabu`, `Shift - Kamis`, `Shift - Jumat`, `Status`.
   * ⚙️ **Aturan_Shift**: Kolom `Nama Aturan` dan `Nilai`. Isi baris awal dengan:
     * `Cek_Double_Shift` | `1` (Artinya Aktif)
     * `Cek_K3_Malam_Pagi` | `1` (Artinya Aktif)

### Langkah 2: Konfigurasi n8n & API Credentials
1. Import file JSON workflow (**Main Workflow** & **Mini-Server**) ke dalam kanvas n8n.
2. Hubungkan kredensial OAuth2/API Token untuk Google Sheets, Gmail API, dan Telegram Bot API.
3. Aktifkan fitur **Execute Once** pada tab settings node `Weekly Telegram Recap` agar data rekap tidak dikirim berulang-ulang.
4. Pastikan toggle switch mini-server dan main workflow di pojok kanan atas diatur ke posisi **Active (Warna Hijau)** sebelum melakukan demo.

---

## 🧠 Mekanisme Validasi (Fitur Unggulan)

SmartShift v2.0 menerapkan **Dual-Gate Validation Engine** berbasis JavaScript pada Code Node sebelum data diizinkan masuk ke database:

### 🛑 Gate 1: Anti-Double Submit (Anti-Double Shift)
Sistem memanggil data historis melalui database untuk mencocokkan kombinasi `Email` + `Tanggal` pengajuan. Sistem menetapkan parameter **Email Karyawan sebagai Kunci Unik (Primary Key)** untuk mengantisipasi ambiguitas identitas jika terdapat kesamaan nama pada karyawan yang berbeda. Jika email yang sama terdeteksi mencoba mengirim ulang form pada minggu kalender yang sama, sistem langsung memblokir data tersebut demi menjaga integritas database.

### ⚠️ Gate 2: K3 Fatigue Rule Engine (Batas Istirahat 12 Jam)
Sistem memetakan jam operasional bruto industri (mencakup jam kerja efektif dan alokasi waktu istirahat/ISOMA harian):
* **Shift Pagi**: 07:00 – 15:00 WIB
* **Shift Siang**: 14:00 – 22:00 WIB
* **Shift Malam**: 22:00 – 06:00 WIB (Pulang di hari berikutnya)

Menggunakan kalkulasi **Luxon API**, sistem akan memberikan status **Rejected** jika karyawan mengambil **Shift Malam**, lalu di hari berikutnya langsung mengambil **Shift Siang** (Jeda istirahat riil dari pukul 06:00 ke 14:00 hanya berjarak 8 jam). Hal ini melanggar regulasi K3 Fatigue Management yang mewajibkan waktu pemulihan fisik minimal **12 jam istirahat**. Sebaliknya, perpindahan maju dari Shift Pagi ke Shift Malam di hari berikutnya diizinkan karena memberikan durasi istirahat melimpah (31 jam bruto).

---

## ⚙️ Dynamic Policy Control & Sistem Tatakelola
Sistem ini menyediakan dua tingkat kontrol kebijakan operasional yang fleksibel bagi manajemen tanpa perlu menyentuh baris kode program n8n:

### 1. Kontrol Kebijakan Global (Tabel `Aturan_Shift`)
Digunakan oleh Top Management untuk merespon kondisi krisis operasional perusahaan skala besar (misal: kondisi kekurangan personil massal / *understaffed* di musim mudik).

| Skenario | Nilai di Sheet `Aturan_Shift` | Dampak Nyata pada Sistem |
|---|---|---|
| **Mode Ketat (Default)** | Set Nilai ke `1` | Aturan K3 dan Anti-Double Submit aktif penuh. Pelanggaran input langsung memicu email penolakan otomatis. |
| **Mode Darurat** | Set Nilai ke `0` | Validasi di-bypass total secara global. Sistem melonggarkan aturan portal agar personil bisa mendaftar shift lembur ekstra mandiri secara bebas demi menyelamatkan operasional bisnis. |

### 2. Kontrol Kasus Darurat Perorangan (`Master_Jadwal`)
Digunakan oleh Atasan atau Manajer Operasional langsung di lapangan jika terjadi urgensi harian perorangan (misal: satu personil mendadak izin sakit di hari Kamis). Atasan dapat melakukan *override* dengan langsung menyunting kotak jadwal personil pengganti di sheet `Master_Jadwal`. Pegawai pengganti tidak perlu mengisi form mandiri 2x lewat web portal untuk menghindari benturan validasi sistem.

---

## 📸 Panduan Dokumentasi (Screenshot)

1. **Dashboard Kontrol (Spreadsheet)**: Menampilkan data rekap jadwal mingguan karyawan dan kontrol tabel aktivasi aturan fitur (1/0).
   <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/07663de4-193c-44c3-8886-2e86961bc061" />
   <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/c0cc4ad2-ac60-45cb-9379-6158e535b59b" />

2. **Arsitektur Workflow (n8n Canvas)**: Tampilan visual seluruh node Core Logic (Gate Validation) dan Mini-Server Webhook Responder yang terhubung rapi dengan dokumentasi Sticky Notes.
   <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/a9193b9b-a53a-4e81-9b50-1930de9cd014" />
   <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/22d7c8b3-be33-4cf5-98f1-6399f3021038" />

3. **Halaman Konfirmasi (Web Landingpage)**: Respon render browser HTML profesional berupa tampilan status interaktif "Data Terkirim!" sesaat setelah karyawan melakukan klik submit form.
   <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/5d39f097-81e3-4a12-bf10-6c0509bfe07e" />

4. **Bukti Output Validasi (Email & Telegram)**: Output asli hasil penyaringan sistem berupa notifikasi email Gmail HTML (Success/Rejection Alert) serta otomasi rangkuman laporan grup koordinasi Telegram.
   <img width="1600" height="900" alt="Image" src="https://github.com/user-attachments/assets/3c2f3917-d787-4a6c-b513-dcf66aa9e42b" />
   <img width="1600" height="900" alt="Image" src="https://github.com/user-attachments/assets/39d5548d-2ced-4139-8da2-0bc764ea849c" />
   <img width="1600" height="900" alt="Image" src="https://github.com/user-attachments/assets/21023b77-eddf-4f2b-8469-8fb166771b5d" />
   <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/72dd9d4d-379c-4fd5-a901-b0bed7946989" />

---
**Author:** Syahril & Kelompok 1 (Grup 4)  
**Development Year:** 2026  
**Tech Stack:** n8n Workflow Automation, Google Workspace API, Telegram Bot API.
