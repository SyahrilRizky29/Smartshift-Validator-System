# 🚀 SmartShift: Automated Validation & Telegram Reporting

**SmartShift** adalah solusi otomasi *backend* berbasis n8n yang dirancang untuk mengelola pendaftaran shift pegawai secara mandiri, akurat, dan terverifikasi. Sistem ini menghilangkan proses manual dalam pengecekan jadwal dan memastikan setiap pengajuan mematuhi aturan perusahaan serta standar keselamatan kerja (K3).

---

## 📑 Daftar Isi
1. [Tentang Project](#-tentang-project)
2. [Fitur Utama](#-fitur-utama)
3. [Panduan Implementasi](#-panduan-implementasi-lengkap)
4. [Dynamic Policy Control (Fitur Unggulan)](#-dynamic-policy-control)
5. [Panduan Dokumentasi](#-panduan-dokumentasi)

---

## 🧐 Tentang Project
Project ini dibangun menggunakan n8n untuk menghubungkan form pendaftaran dengan database Google Sheets, melakukan validasi logika menggunakan JavaScript, dan memberikan respon instan melalui Gmail serta laporan berkala ke Telegram.

## ✨ Fitur Utama
*   **Web Portal Input**: Form input mandiri untuk pegawai.
*   **Smart Rule Engine**: Validasi otomatis untuk mencegah *Double Shift* dan *Pelanggaran K3*.
*   **Multi-Channel Notification**: Konfirmasi sukses/gagal via Gmail HTML.
*   **Weekly Recap**: Laporan otomatis setiap Senin jam 08:00 WIB ke Telegram.

---

## 🚀 Panduan Implementasi Lengkap

### Langkah 1: Persiapan Google Sheets
1. Buat Spreadsheet bernama **"Jadwal Shift Pegawai"**.
2. Buat dua sheet:
   *   **Master_Jadwal**: Kolom `ID`, `Nama`, `Email`, `Tanggal`, `Shift`, `Status`.
   *   **Aturan_Shift**: Kolom `Nama Aturan`, `Nilai`. Isi dengan:
     *   `Cek_Double_Shift` | `1` (Artinya Aktif)
     *   `Cek_K3_Malam_Pagi` | `1` (Artinya Aktif)

### Langkah 2: Konfigurasi n8n
1. Import file JSON workflow (**Main Workflow** & **Mini-Server**) ke n8n.
2. Hubungkan kredensial Google Sheets, Gmail, dan Telegram Bot Anda.
3. Pada node `Weekly Telegram Recap`, pastikan opsi **Execute Once** di tab Settings sudah aktif.

---

## ⚙️ Dynamic Policy Control
Salah satu keunggulan utama SmartShift adalah **fleksibilitas kebijakan**. Admin dapat mengubah aturan validasi secara "Live" tanpa harus menyentuh kode program.

| Skenario | Tindakan di Sheet `Aturan_Shift` | Efek pada Sistem |
|---|---|---|
| **Mode Ketat** | Set Nilai ke `1` | Sistem akan menolak pengajuan yang melanggar K3 atau Double Shift. |
| **Mode Darurat** | Set Nilai ke `0` | Validasi dimatikan. Sistem akan menerima semua pengajuan (berguna saat kekurangan personil mendadak). |

---

## 📸 Panduan Dokumentasi (Screenshot)
1.  **Dashboard Kontrol (Spreadsheet)**.
    <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/cdd9d3a1-8add-4548-9046-52dd78ab6acb" />
2.  **Arsitektur Workflow (n8n)**.
    <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/fff311c3-fe8d-480c-ae67-a6c6bc1e9f8d" />
3.  **Halaman Konfirmasi (Web)**.
    <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/ac81a96d-e0f8-453c-94af-8f309eba5848" />
4.  **Bukti Output (Email & Telegram)**: Screenshot hasil konfirmasi Gmail yang estetik dan pesan rekap di grup Telegram.
    <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/879880c4-505c-4282-82a8-498b2f1edcf6" />
    <img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/f3bf2b45-d914-4617-b956-3a4c1727f30c" />
---

**Author:** Syahril 
**System Date:** 2026  
**Tools:** n8n, Google Workspace, Telegram API.
