# FASE — KEPUTUSAN FINAL DATABASE

**23 Oktober 2026** · READ-ONLY · tanpa import/migrasi · tanpa mengeksekusi kode dari backup · tanpa PII di laporan

## 1. Code Snippets — **RISIKO RENDAH · TIDAK PERLU DIPULIHKAN** ✓
| Properti | Temuan (Tambun = Sragen, 4 baris identik) |
|---|---|
| Status | **semua `active = 0`** (nonaktif) ✓ |
| Ukuran | 29–207 byte (sangat kecil) |
| Nama | "Make upload filenames lowercase", "Disable admin bar", "Allow smilies", "Current year" |
| Indikator berbahaya | **0** — tidak ada `eval`, `base64_decode`, `gzinflate`, `shell_exec`/`system`/`proc_open`, `curl`/`wp_remote_*`, `wp_redirect`/`header(Location)`, `wp_create_user`/`add_role` ✓ |
| Hook | tidak ada `add_action`/`add_filter` yang terdeteksi ✓ |
| Akses input | tidak membaca `$_POST/$_GET/$_COOKIE` ✓ |
| Satu flag "hidden_admin" | dari `is_admin()` pada snippet "Disable admin bar" — **fungsi normal**, bukan backdoor ✓ |

**Keputusan:** **tidak dipulihkan** ✓ — semuanya nonaktif dan bersifat kosmetik; tidak ada fungsi yang dibutuhkan situs hasil recovery.

## 2. WooCommerce 8 "pesanan" — **KEMUNGKINAN BESAR DATA UJI · BELUM PASTI** ⚠️
| Bukti | Temuan |
|---|---|
| `woocommerce_order_items` | **8 baris identik di keempat sekolah** ✓ |
| `_product_id`, `_qty`, `_line_subtotal` | masing-masing **8** ⇒ **1 item per order**, satu jenis produk |
| Pola seragam antar-situs | sangat khas **data contoh** (bukan riwayat transaksi nyata) |
| Jumlah produk | katalog kemungkinan kosong (situs sekolah tidak berjualan) |
| **`posts` bertipe `shop_order`** | **0** dalam blok INSERT yang diperiksa (71 baris Tambun, 78 baris Sragen) ⇒ item order **tanpa induk order** = **orphan** ✓ |

**Keputusan:** **tidak dimigrasikan** ✗ untuk sekarang. **Bukti minimum yang masih diperlukan** bila ingin memastikan: status order (`wc-*`), tanggal order, dan total — saya **tidak** mencetak detail transaksi karena bisa memuat data pribadi ✓.
**Status: BELUM TERVERIFIKASI PENUH** ❓ — tidak saya nyatakan "pasti palsu". Namun bukti **orphan** (0 `shop_order` di `posts`) sangat kuat mengarah ke sisa instalasi plugin, bukan transaksi nyata.
⚠️ Batas: parser hanya membaca blok `INSERT` pertama tabel `posts` ⇒ jumlah posts yang diperiksa **parsial**, bukan total.

## 3. MailPoet (1 subscriber/sekolah) — **RECORD SISTEM** ✓
| Field | Nilai (tanpa PII) |
|---|---|
| `status` | **unconfirmed** |
| `source` | **wordpress_user** |
| `wp_user_id` | terisi ⇒ **tertaut ke akun WordPress**, bukan pendaftar mandiri |
| `created_at` | 2025-07-2x (sebelum jendela serangan 2025-10-30) |
| `confirmed_at` | kosong (belum pernah konfirmasi) |

⇒ Ini **sinkronisasi otomatis akun admin ke MailPoet**, bukan langganan newsletter nyata ✓
**Keputusan:** **tidak dipulihkan** ✓ — tidak ada daftar pelanggan yang hilang.

## 4. Finalisasi export
**Export `production-ready-*.sql` TETAP LAYAK DIGUNAKAN** ✓ — tidak perlu diperbarui:
- Snippets tidak diperlukan ✓ · Orders tidak dibutuhkan untuk situs informasi ✓ · Subscriber MailPoet adalah record sistem ✓
- Seluruh tabel inti + konten + tabel plugin aktif sudah ada di ekspor ✓
- **Tidak ada** tabel/kolom yang perlu ditambahkan ✓

## 5. Status repository & checksum
| Item | Status |
|---|---|
| Visibilitas `vael-skaya/wp` | **masih PUBLIC** ✗ — tidak saya ubah ulang (tidak ada autentikasi) ✓ |
| Checksum inti | Tambun **PASS** ✓ · Cibitung/Kebumen/Sragen **NOT VERIFIED** ✗ (metode sama gagal; saya **tidak** mengulang) ✓ |

## 6. Blocker tersisa sebelum import
| # | Blocker | Pemilik | Aksi |
|---|---|---|---|
| B-1 | Repo publik | **Saka** | `gh auth login` → `gh repo edit vael-skaya/wp --visibility private` |
| B-2 | Checksum 3 situs | **Saka** | izin jalankan ulang saat jaringan stabil |
| B-3 | DB kosong + user per situs | **Pak Eko** | buat 4 DB + user terbatas |
| B-4 | PHP ≥8.1 · `memory_limit ≥256M` · SSL · DNS | **Pak Eko** | verifikasi di cPanel |
| B-5 | Email resmi + teks kebijakan privasi | **Pak Eko** | konfirmasi |

## 7. Langkah terpendek menuju go-live
1. Saka: privatkan repo + izin checksum (2 menit).
2. Pak Eko: 4 DB kosong + user, PHP/memory/SSL/DNS, email resmi.
3. Import `production-ready-*.sql` → ubah **2 baris** `siteurl`/`home` → admin baru → verifikasi 11–12 halaman.
4. Situs 1 lulus → lanjut situs 2–4.

## 8. Konfirmasi
Kode snippet **tidak dieksekusi** ✓ · tidak diaktifkan ✓ · tanpa import/migrasi ✓ · backup asli tidak diubah ✓ · tanpa PII/email/kredensial di laporan ✓ · tanpa commit/push ✓ · `goly1264_ams` & `Downloads\admin.php` tidak disentuh ✓
