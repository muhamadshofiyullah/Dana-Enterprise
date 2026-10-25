# Panduan Penggunaan DANA Enterprise Gateway (B2B Disbursement)

Selamat datang di modul **DANA Enterprise Gateway**. Modul ini dirancang khusus untuk menjembatani software server pulsa / loket pembayaran Anda (OtomaX, IRS, Tiger, Voucha, Flash Machine, dsb.) dengan akun DANA Enterprise B2B Anda secara aman, cepat, dan tanpa ribet.

Aplikasi ini sudah dikemas dalam bentuk satu file program siap pakai (`DanaEnterprise.exe`), sehingga Anda **tidak perlu menginstal Node.js** atau pengaturan sistem yang rumit di komputer Anda.

---

## 🛡️ Jaminan Keamanan Saldo & Transaksi

Bagi pemilik server dan pengelola loket, keamanan saldo adalah prioritas tertinggi. Modul ini dilengkapi dengan sistem pengaman berlapis:

1. **Saldo Tidak Pernah Terpotong Jika Akun Tujuan Salah**:
   Sebelum uang dikirimkan, sistem otomatis memeriksa nomor DANA pelanggan ke server DANA. Jika nomor tidak terdaftar, belum aktif, atau dibekukan, transaksi langsung dibatalkan di awal. Saldo deposit merchant Anda tetap utuh 100%.

2. **Aman dari Potong Dobel**:
   Setiap transaksi dikunci dengan nomor referensi unik (`trxid`). Jika terjadi gangguan internet sesaat atau server DANA sempat lambat merespons (*timeout*), sistem tidak langsung menembak ulang secara membabi buta. Sistem akan memeriksa status kepastian transaksi terlebih dahulu, sehingga Anda terbebas dari resiko saldo terpotong dua kali.

3. **Pencegahan Saldo Minus di Depan**:
   Sistem selalu memastikan saldo deposit Anda di DANA mencukupi sebelum transaksi dilepas. Jika saldo deposit menipis, transaksi langsung ditolak dengan status yang jelas (`INSUFFICIENT_BALANCE`), bukan menggantung.

4. **Nomor Tujuan Bebas Format**:
   Anda atau pelanggan tidak perlu pusing mengetik nomor dengan awalan `08`, `628`, atau `+628`. Modul ini otomatis merapikan format nomor ke format resmi yang diminta DANA di belakang layar.

---

## 📋 Langkah Cepat Memulai

### 1. Persiapan Folder & Kunci
Pastikan file program diletakkan dalam satu folder bersama kelengkapannya:
```text
Folder-DanaEnterprise/
├── DanaEnterprise.exe       <-- Program utama
├── .env                     <-- File setelan akun DANA Anda
└── KEY/                     <-- Folder tempat file sertifikat (.pem)
    ├── pkcs8_rsa_private_key.pem
    └── rsa_public_key.pem
```

### 2. Pengisian Pengaturan (`.env`)
Buka file `.env` menggunakan Notepad, lalu sesuaikan data akun yang diberikan oleh pihak DANA:
```env
PORT=3300
DANA_CLIENT_ID=nomor_client_id_anda
DANA_CLIENT_SECRET=kode_rahasia_client_secret
DANA_BASE_URL=https://api.saas.dana.id
DANA_PRIVATE_KEY=KEY/pkcs8_rsa_private_key.pem
DANA_PUBLIC_KEY=KEY/rsa_public_key.pem
DANA_X_PARTNER_ID=nomor_partner_id_anda
DANA_MERCHANT_ID=nomor_merchant_id_anda
DANA_CHANNEL_ID=95221
```

> **Tips Kunci RSA**: Anda cukup menaruh file kunci di dalam folder `KEY/` dan menuliskan nama jalurnya seperti di atas (`KEY/pkcs8_rsa_private_key.pem`). Tidak perlu repot menyalin teks kunci yang panjang.

---

## 🔑 Aktivasi Lisensi Perangkat (SWID)

Aplikasi ini dikunci per perangkat komputer/server agar aman dari penyalahgunaan.

1. Buka Command Prompt (CMD) di folder aplikasi, lalu ketik:
   ```cmd
   DanaEnterprise.exe --swid
   ```
2. Layar akan menampilkan kode **MACHINE SWID** komputer Anda (contoh: `2e73abcf8cf2...`).
3. Salin kode tersebut dan kirimkan ke Admin / Penyedia Layanan untuk diaktifkan.
4. Setelah mendapat konfirmasi aktif, cukup klik dua kali file `DanaEnterprise.exe` untuk menjalankannya.

---

## 🚀 Menguji Transaksi dari Browser

Setelah program berjalan (muncul tulisan `Server berjalan pada http://localhost:3300`), Anda bisa langsung mencoba fitur-fiturnya lewat web browser:

### 1. Mengecek Sisa Saldo Deposit Merchant DANA Anda
Buka browser dan ketik alamat:
```text
http://localhost:3300/api/check-disbursement-account
```
*Layar akan menampilkan sisa saldo deposit (`MERCHANT_DEPOSIT_BALANCE`) yang siap digunakan.*

### 2. Mengecek Apakah Nomor Pelanggan Terdaftar
Ketik alamat:
```text
http://localhost:3300/api/account-inquiry?customerNumber=0811742234&trxid=TES001&amount=10000
```
*Sistem akan mengecek nomor tersebut ke DANA. Nomor `0811...` akan otomatis disesuaikan dan diperiksa tanpa memotong saldo Anda sedikit pun.*

### 3. Mengirim Saldo / Top-up Penuh (Jalur Utama Software Pulsa)
Gunakan jalur ini di pengaturan modul software pulsa Anda:
```text
http://localhost:3300/topup?customerNumber=0811742234&amount=10000&trxid=TRX20261023001
```
*Alur ini otomatis melakukan pengecekan saldo -> validasi nomor penerima -> kirim saldo -> verifikasi kepastian status.*

---

## ❓ Pertanyaan yang Sering Diajukan (FAQ)

* **T: Apakah komputer server harus selalu menyala dan terhubung internet?**
  * **J:** Ya. Aplikasi ini bertindak sebagai jembatan lokal ke server DANA. Pastikan koneksi internet server stabil agar transaksi tidak terhambat.
* **T: Apakah aplikasi ini boleh dipasang di Startup Windows?**
  * **J:** Sangat disarankan. Buat shortcut `DanaEnterprise.exe` ke folder Startup Windows agar aplikasi otomatis menyala setiap kali komputer server di-restart.
* **T: Bagaimana jika pelanggan memasukkan nomor dengan format berbeda?**
  * **J:** Tidak masalah. Format `08xxx`, `+628xxx`, maupun nomor yang terlanjur berspasi/bergaris strip otomatis dirapikan oleh sistem sebelum dikirim ke DANA. Saldo tetap aman dan transaksi berhasil.
