# FASE — AUDIT BACKUP SQL ASLI vs STAGING

**23 Oktober 2026** · READ-ONLY · tanpa import/migrasi ke staging atau produksi · tanpa menyentuh hosting

## 1. Jumlah tabel
| Sekolah | Backup asli | Staging | Hanya di backup | (yang **berisi data**) | Hanya di staging |
|---|---:|---:|---:|---:|---:|
| Tambun | **132** | **24** | **111** | **26** | 3 |
| Cibitung | **131** | **24** | **110** | **25** | 3 |
| Kebumen | **138** | **13** | **125** | **29** | 0 |
| Sragen | **132** | **13** | **119** | **30** | 0 |

Metode: nama tabel dari `CREATE TABLE` di dump; jumlah baris dihitung dengan parser **quote-aware** atas setiap statement `INSERT … VALUES (…),(…)` (menghormati escape & tanda kutip dalam nilai). Jumlah baris staging dari `COUNT(*)` langsung. Daftar lengkap 500+ baris ada di `FASE-DATABASE-ORIGINAL-BACKUP-TABLES.csv`.

## 2. Tabel hanya di backup — yang berisi data (gabungan 4 sekolah)

| Keluarga plugin | Tabel | Total baris | Sifat | Perlu dipertahankan? |
|---|---|---:|---|---|
| gla (Google Listings & Ads) | 4 | 16.920 | **boilerplate** — rekomendasi anggaran bawaan plugin | **TIDAK** — dibuat ulang plugin |
| mailpoet | 32 | 704 | template newsletter bawaan + migrasi + settings | **TIDAK** — kecuali `mailpoet_subscribers` |
| wc_/woocommerce | 36 | 1.148 | catatan admin, lookup, sesi, **8 order** | **PERIKSA** — lihat §3 |
| wpf_ (WP Fusion) | 12 | 184 | konfigurasi meta/filter CRM | **TIDAK** — konfigurasi, bukan data siswa |
| actionscheduler | 6 | 20 | antrean tugas sementara | **TIDAK** |
| jetpack / jet_ | 8 | 20 | antrean sinkronisasi & taxonomy | **TIDAK** |
| snippets | 2 | 8 | **cuplikan PHP kustom** | **WAJIB TINJAU** — lihat §3 |
| cartflows / frmt / wpforms | 10 | 20 | 1 abandoned cart, report cache, task meta | **TIDAK** |

**Tidak ada satu pun** tabel konten sekolah (posts/pages/menu/media) yang hanya ada di backup ✓ — konten sudah utuh di staging.

## 3. Data yang perlu diperiksa / dipulihkan selektif
| # | Data | Bukti | Risiko bila diabaikan | Rekomendasi |
|---|---|---|---|---|
| **R-1** | `snippets` (4 baris di Tambun & Sragen) | tabel Code Snippets berisi **kode PHP kustom** | bisa jadi fungsi situs (redirect, form, dsb.) — **atau** sisa injeksi penyerang | **TINJAU ISI sebelum memutuskan**; jangan salin apa adanya |
| **R-2** | `woocommerce_order_items` (8) + `order_itemmeta` (80) | 8 pesanan di keempat situs | bila transaksi nyata → kehilangan riwayat | **PERIKSA** apakah order uji/bot — jangan salin buta |
| **R-3** | `cartflows_ca_cart_abandonment` (1) | 1 keranjang terbengkalai | hampir pasti bot/uji | abaikan |
| **R-4** | `mailpoet_subscribers` (1/situs) | 1 baris | mungkin baris sistem | cek; salin hanya bila subscriber nyata |
| **R-5** | `wpf_meta_values` / `wpf_filters` (28/6) | konfigurasi WP Fusion | hanya relevan bila WP Fusion dipakai | abaikan bila plugin tidak dipakai |

**Sudah terbukti TIDAK hilang** ✓: `e_submissions*` (0 baris), `frmt_form_entry*` (0), `wpforms_entries` (tidak ada), `ahm_*` (0) ⇒ **tidak ada kiriman formulir/pendaftaran** di backup ✓

## 4. Asal perbedaan
1. **Plugin tidak dipakai lagi** — WooCommerce, WP Fusion, Jetpack, CartFlows, GLA: tabel ada di backup, tidak dipasang di staging ✓
2. **Versi/instalasi baru** — staging memasang **WPForms Lite** (tabel `wpforms_*` baru) yang tidak ada di backup ✓
3. **Proses recovery** — staging hanya mengimpor konten inti + tabel plugin yang dipakai ✓
4. **Bukan** data yang belum dipulihkan ✓ — semua tabel inti & konten ada di kedua sisi ✓

## 5. Apakah `production-ready-*.sql` masih layak?
**YA, layak dipakai** ✓ — dengan dua catatan:
- Keempat ekspor memuat **seluruh tabel inti + konten + tabel plugin aktif** ✓
- **Tidak** memuat tabel legacy (WooCommerce/GLA/WP Fusion) — **tidak diperlukan** untuk situs informasi sekolah ✓
- ⚠️ **Sebelum import**, selesaikan **R-1 (`snippets`)** dan **R-2 (orders)** — bila ternyata dibutuhkan, tambahkan **hanya tabel itu** ke ekspor (bukan seluruh backup)

## 6. Langkah terpendek menuju production
1. **Tinjau `snippets`** (4 baris) dan **8 order WooCommerce** — perlu keputusan Saka: pakai atau abaikan.
2. **Privatkan repo** (`gh auth login` → `gh repo edit vael-skaya/wp --visibility private`).
3. **Selesaikan checksum** 3 situs (izin menjalankan ulang).
4. **Pak Eko:** DB kosong + user per situs, email resmi, PHP ≥8.1, `memory_limit ≥256M`, SSL/DNS.
5. **Import** `production-ready-*.sql` → ganti 2 baris `siteurl`/`home` → admin baru → verifikasi halaman.

## 7. Ketidakpastian
- Isi **`snippets`** dan **`woocommerce_order_*`** belum saya baca (tidak boleh menampilkan kode/data) ⇒ **BELUM TERVERIFIKASI** ❓
- Apakah plugin WooCommerce/WP Fusion **pernah** dipakai untuk data nyata — tidak dapat dipastikan dari nama tabel ❓
- Kolom/index antarversi plugin tidak dibandingkan ✗

## 8. Konfirmasi
Tanpa import/migrasi ✓ · tanpa perubahan staging/DB asli ✓ · backup tidak dihapus/diubah ✓ · tanpa commit/push ✓ · `goly1264_ams` & `Downloads\admin.php` tidak disentuh ✓ · tanpa isi pribadi/kredensial ✓
