# TANGKAS V1 — Keputusan Fungsional

## Sudah disepakati
1. TANGKAS adalah Web App / PWA.
2. Bisa diakses dari komputer, HP, atau tablet melalui internet tanpa PC kantor sebagai server.
3. Fondasi awal: Cloudflare Workers + D1 + R2, GitHub repository private.
4. Data pasien master dipisahkan dari data kunjungan.
5. Paket 5x dan 10x tidak memiliki expiry; aktif sampai saldo habis.
6. Saldo paket menggunakan ledger.
7. Setiap kunjungan terapi otomatis memotong 1 saldo paket bila ada paket aktif.
8. Jika ada lebih dari satu paket aktif, user memilih paket yang digunakan.
9. Tidak ada hard delete untuk transaksi operasional.
10. Pembatalan kunjungan memakai reversal saldo.
11. Pembayaran mendukung paket prepaid dan per kedatangan.
12. Foto perkembangan pasien opsional dan terhubung ke kunjungan.
13. QR dapat discan dari browser HP; pencarian nomor member dan nama tetap tersedia.
14. Desain kartu member / QR visual menyusul memakai desain lama user.
15. Role adalah template, permission individual dicentang oleh Super Admin.
16. Audit Log wajib.
17. Stock mendukung PCS dan ML.
18. Batch/lot/expired stock ditunda ke V2; kolom database sudah disiapkan.
19. Setting dibuat configuration-driven.
20. Dashboard sederhana.
21. Backup/export tersedia sejak V1.
22. Operasional/keuangan/pajak/kas kecil/kas utama tampil sebagai placeholder.
23. V1 hanya mengenal kunjungan terapi infus; jenis kunjungan lain disimpan untuk masa depan.
24. Opsi “gunakan paket atau tidak” tidak ditampilkan; alur klinik otomatis menggunakan paket bila ada saldo.

## Role
- SUPER_ADMIN — Kepala Kantor
- ADMIN
- FRONT_OFFICE
- TERAPIS — Nakes / Dokter
- STOCK
- FINANCE
- VIEWER_OWNER

## Aturan transaksi kunjungan
Penyimpanan kunjungan dengan paket harus merupakan satu transaksi database:
- insert kunjungan
- saldo paket -1
- insert ledger
- insert audit log

Jika salah satu gagal, semua dibatalkan.

Pembatalan tidak menghapus data. Kunjungan berubah status menjadi DIBATALKAN dan saldo dikembalikan dengan ledger REVERSAL.
