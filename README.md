# TANGKAS V1
**Tata Kelola Aktivitas Kerja Selaras**

Versi V1 cloud-ready untuk operasional klinik. Aplikasi dirancang agar pengguna cukup memakai browser dari PC, laptop, Android, iPhone, atau tablet. Komputer kantor bukan server dan tidak perlu menginstal Python, PostgreSQL, Git, atau Node.js.

## Teknologi
- Cloudflare Workers — backend/API
- Cloudflare D1 — database
- Cloudflare R2 private bucket — foto perkembangan pasien
- Workers Static Assets — frontend/PWA
- GitHub Private Repository — source code
- Hono — routing API
- ZXing Browser — QR scanner kamera browser

## Fitur V1

### Multi-user dan Permission
Role template:
- Super Admin (Kepala Kantor)
- Admin
- Front Office
- Petugas / Terapis
- Stock
- Finance
- Viewer / Owner

Super Admin dapat mencentang permission aktual masing-masing user.

### Pasien / Member
- nomor member otomatis `TGS-YYYYMM-00001`
- token QR unik
- cari nomor member / nama / token QR
- identitas pasien hanya diisi sekali
- metode pembayaran default dapat berubah

### Kunjungan
- setiap kunjungan V1 = terapi infus
- keluhan
- catatan/riwayat
- foto perkembangan opsional
- satu paket aktif: otomatis potong saldo 1 sesi
- beberapa paket aktif: user memilih paket
- tidak ada paket aktif: pembayaran per kedatangan
- idempotency key membantu mencegah double submit
- transaksi paket memakai D1 batch transaction

### Paket
Default awal:
- Paket Infus 5x — Rp5.000.000
- Paket Infus 10x — Rp10.000.000
- Paket Sosial

Paket tidak memiliki masa berlaku. Status utama: AKTIF, HABIS, DIBATALKAN. Konsep KEDALUWARSA tetap dapat ditambahkan di masa depan tetapi V1 tidak menjalankan expiry otomatis.

### Pembayaran
- pembelian paket prepaid
- pembayaran per kedatangan
- tunai / QRIS / transfer / kartu / sosial
- lunas / belum lunas / gratis
- referensi pembayaran

### Pembatalan
Tidak ada hard delete.
- wajib alasan
- kunjungan menjadi DIBATALKAN
- paket dikembalikan +1 melalui ledger REVERSAL
- pembayaran per kedatangan yang sudah LUNAS menjadi PERLU_REFUND agar Finance dapat menindaklanjuti

### Foto Perkembangan
- opsional
- terkait ke kunjungan tertentu
- dikompres di browser sebelum upload
- disimpan di R2 private bucket
- hanya dilayani setelah cek login + permission

### Stok
- PCS
- ML
- stok masuk
- stok keluar
- adjustment (+/-)
- stok minus tetap tercatat dan diberi peringatan
- minimum stok, HPP, harga jual
- `batch_no` dan `expiry_date` sudah tersedia di database, tetapi UI ditunda ke V2

### Setting
- nama usaha
- tagline
- PPN default
- prefix faktur
- harga terapi per kedatangan
- paket, promo, HPP, harga jual

### Audit Log
Mencatat login, pasien, kunjungan, pembatalan, paket, foto, stok, setting, user, dan permission.

### Backup / Export
- Pasien CSV
- Kunjungan CSV
- Stok CSV
- Backup JSON data utama

### Operasional
Placeholder V1:
- Laporan Keuangan
- Laporan Pajak
- Kas Kecil
- Kas Utama

---

# Deployment Tanpa Instalasi di Komputer Kantor

Semua proses deployment dapat dilakukan melalui browser GitHub dan Cloudflare.

## 1. GitHub
1. Buat repository **PRIVATE**, contoh `tangkas-v1`.
2. Upload seluruh isi folder ini ke root repository.
3. Jangan menyimpan password asli di GitHub.

## 2. Buat Cloudflare D1
1. Login Cloudflare Dashboard.
2. Buat D1 database bernama `tangkas-db`.
3. Catat Database ID.

V1 membuat tabel otomatis ketika pertama kali diakses setelah binding DB tersedia.

## 3. Buat Cloudflare R2
1. Buat bucket `tangkas-private-photos`.
2. Jangan aktifkan public access.

Jika R2 belum dikonfigurasi, fungsi utama aplikasi tetap dapat berjalan, tetapi upload foto akan ditolak sampai binding PHOTOS tersedia.

## 4. Edit `wrangler.jsonc`
Aktifkan blok binding dan isi D1 ID:

```jsonc
"d1_databases": [
  {
    "binding": "DB",
    "database_name": "tangkas-db",
    "database_id": "ID-D1-ANDA"
  }
],
"r2_buckets": [
  {
    "binding": "PHOTOS",
    "bucket_name": "tangkas-private-photos"
  }
]
```

## 5. Environment Variable / Secret
Tambahkan di Cloudflare Worker:
- `ADMIN_USERNAME` — username Super Admin awal
- `ADMIN_PASSWORD` — password kuat Super Admin awal
- `SESSION_HOURS` — contoh `12`

Password jangan ditulis di source code GitHub.

Saat database masih kosong, request pertama akan membuat Super Admin berdasarkan dua environment variable tersebut.

## 6. Build dari GitHub
Gunakan:

Build command:
```bash
npm run build
```

Deploy command:
```bash
npx wrangler deploy
```

Cloudflare menjalankan npm di server build mereka. Komputer kantor tidak perlu menginstal npm/Node.js.

---

# Catatan sebelum data pasien asli
V1 adalah fondasi fungsional. Sebelum go-live produksi dengan data pasien nyata, lakukan:
1. uji menyeluruh memakai data dummy
2. review permission setiap role/user
3. tetapkan kebijakan foto pasien
4. uji backup dan prosedur restore
5. tambahkan reset password terkontrol
6. tambahkan rate limiting login
7. siapkan backup terjadwal
8. review kebutuhan kepatuhan dan retensi data

Lihat `docs/KEPUTUSAN_V1.md` untuk baseline keputusan bisnis yang telah disepakati.
