# FASE — PERSIAPAN DATABASE PRODUCTION

**23 Oktober 2026** · Export dibuat **hari ini** dari staging final · tanpa import/DDL ke DB mana pun · tanpa menyentuh hosting

## 1. Status database final

| Sekolah | Database staging | Prefix | Tabel | Tabel inti | Posts | Postmeta | Users | Status |
|---|---|---|---:|---|---:|---:|---:|---|
| Tambun | `smk10_staging_tambun` | `wp_` | 24 | lengkap ✓ | 166 | 547 | **1** | **VALID** ✓ |
| Cibitung | `smk10_staging_cibitung` | `wp_` | 24 | lengkap ✓ | 205 | 640 | **1** | **VALID** ✓ |
| Kebumen | `smk10_staging_kebumen` | `wp_` | 13 | lengkap ✓ | 187 | 590 | **1** | **VALID** ✓ |
| Sragen | `smk10_staging_sragen` | `wp_` | 13 | lengkap ✓ | 152 | 506 | **1** | **VALID** ✓ |

Semua DB diambil dari `wp-config.php` staging (bukan drill, bukan DB lain) ✓ · prefix seragam `wp_` ✓ · **1 akun** per situs ✓

## 2. File SQL final
| Sekolah | File | Ukuran | CREATE TABLE | `USE`/`CREATE DATABASE` | Dump selesai | SHA-256 |
|---|---|---:|---:|---:|---|---|
| Tambun | `production-ready-tambun-v2.sql` | 1.00 MB | 24 | **0** ✓ | True ✓ | `94184ca4935cfdd854a8a9b76441633cc3d07f9eab2758c77c65ce93bbf41625` |
| Cibitung | `production-ready-cibitung-v2.sql` | 1.09 MB | 24 | **0** ✓ | True ✓ | `15f6fcb948648bff3ea1ff89fd0b75da33569c995479f54b348aec54fb5260ae` |
| Kebumen | `production-ready-kebumen-v2.sql` | 2.04 MB | 13 | **0** ✓ | True ✓ | `95cf4e8368c503f78184010e1b9066301b652c29456da56ba83f8bd6366c2802` |
| Sragen | `production-ready-sragen.sql` | 0.78 MB | 13 | **0** ✓ | True ✓ | `a859391ccae561ec798a6818bd8dc794644d9e66b3157258814259c1d1994e0b` |

Folder: `recovery-work\staging\exports\`

> ⚠️ Berkas sisa percobaan pertama (`production-ready-cibitung.sql`, `production-ready-kebumen.sql`, `production-ready-tambun.sql`) **tidak dihapus** ✓ — gunakan versi **`-v2`** (dan `production-ready-sragen.sql`) sebagai acuan final.

## 3. Data produksi yang perlu dipertahankan
| Item | Status | Tindakan |
|---|---|---|
| `mailpoet_subscribers` (1 baris/sekolah) | **belum diverifikasi** ❓ | **jangan** disalin sebelum dipastikan ini subscriber nyata; 1 baris umumnya baris sistem |
| Tabel formulir (`e_submissions*`, `frmt_*`, `wpforms_entries`) | **0 baris** di produksi ✓ | tidak ada yang perlu diselamatkan |
| `users`/`usermeta`/`options` produksi | **DILARANG** dimigrasikan | 163 akun palsu dari serangan |
| `actionscheduler_*` | 3–6 baris | diabaikan (dapat dibuat ulang) |

## 4. Rencana import production (untuk dijalankan **Anda**, bukan saya)
**Prasyarat:** backup database produksi yang ada **sebelum** perubahan; pastikan akses phpMyAdmin & DB kosong tersedia.

| Sekolah | DB tujuan (usulan) | Prefix | File |
|---|---|---|---|
| Tambun | `smk10n_tambun` | `wp_` | `production-ready-tambun-v2.sql` |
| Cibitung | `smk10n_cibitung` | `wp_` | `production-ready-cibitung-v2.sql` |
| Kebumen | `smk10n_kebumen` | `wp_` | `production-ready-kebumen-v2.sql` |
| Sragen | `smk10n_sragen` | `wp_` | `production-ready-sragen.sql` |

**Langkah per situs:**
1. cPanel → **Backup** → salin database lama (jika ada) sebelum apa pun.
2. Buat database baru + user khusus, beri **hanya** hak pada DB itu.
3. phpMyAdmin → DB kosong → **Import** → pilih file SQL → charset `utf8mb4` → **Go**.
4. Periksa: `wp_options` ada, `siteurl`/`home` masih **URL staging** ⇒ wajib diganti di langkah 5.
5. Ubah 2 baris `wp_options` (`siteurl`, `home`) menjadi `https://<domain>` — **hanya 2 baris ini**, jangan massal; lalu `wp search-replace` untuk isi konten **setelah** site dapat diakses (dry-run dulu).
6. Sesuaikan `wp-config.php`: `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_HOST`, `$table_prefix='wp_'`; tambahkan `define('WP_HOME',...)`, `define('WP_SITEURL',...)`, `DISALLOW_FILE_EDIT`, dan **8 salt baru** (jangan pakai salt staging).
7. Buat **akun admin baru** dengan email sekolah; **hapus** akun staging `*_admin` setelah login baru berhasil.
8. Verifikasi: homepage 200, 11–12 halaman utama 200, menu tampil, logo tampil, ikon Kontak tampil, locale `id_ID`.
9. Aktifkan SSL + paksa HTTPS.

> **Jangan** impor `users`/`usermeta`/`options` produksi lama. **Jangan** gunakan `wp-config.php` dari sumber terkompromi.

## 5. Blocker yang masih ada
| # | Blocker | Pemilik |
|---|---|---|
| B-1 | Repo `vael-skaya/wp` **masih publik** | Saka (`gh auth login` → privatkan) |
| B-2 | Checksum inti **3/4 belum terverifikasi** | Saka (izin jalankan ulang) |
| B-3 | Prasyarat hosting: PHP ≥8.1, `memory_limit ≥256M`, SSL, DNS, disk | Pak Eko |
| B-4 | Email resmi + teks kebijakan privasi | Pak Eko |

**Tidak ada blocker database** ✓ — keempat ekspor valid dan siap diimpor.

## 6. Langkah terpendek menuju go-live
1. **Saka:** `gh auth login` → privatkan repo (1 menit).
2. **Saka:** izinkan saya jalankan ulang checksum 3 situs (5 menit).
3. **Pak Eko:** siapkan DB kosong + user per situs, konfirmasi email resmi.
4. **Anda:** import file di atas → ganti 2 baris `siteurl`/`home` → admin baru → verifikasi homepage.
5. Situs kedua dan seterusnya setelah situs pertama terbukti bersih.

## 7. Konfirmasi
Tanpa import ke production ✓ · tanpa perubahan hosting/DNS/SSL ✓ · tanpa perubahan staging ✓ · tanpa commit/push ✓ · `goly1264_ams` & `Downloads\admin.php` tidak disentuh ✓ · backup asli & checkpoint tidak ditimpa ✓ · tanpa kredensial/isi pribadi di laporan ✓
