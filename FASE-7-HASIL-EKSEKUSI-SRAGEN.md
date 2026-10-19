# FASE 7 — HASIL EKSEKUSI PEMULIHAN SRAGEN (status: staging hidup; konten & media SEBAGIAN)

**Tanggal:** 19 Oktober 2026 · **Persetujuan:** pembuatan staging + database diberikan pemilik proyek
**Ringkas:** staging **dibuat & dapat dibuka (HTTP 200)** ✓ · 13 tabel, 1 admin baru `sragen_admin` ✓ · media **418/418 tersalin & SHA-256 cocok** ✓ · uploads **bebas PHP/JS** ✓ · **0 IoC** ✓ · **0 URL produksi** ✓ · **TETAPI** konten yang terimpor hanya **12 halaman + 1 artikel**, lampiran menjadi post **61/107**, dan perbaikan UI serta QA responsif **belum** dikerjakan ✗

## 1. Lingkungan
| Item | Nilai |
|---|---|
| Direktori | `C:\xampp\htdocs\staging\sragen` |
| URL | `http://localhost/staging/sragen/` → **200** ✓ |
| Database | `smk10_staging_sragen` (**13 tabel**) |
| Prefix | `wp_` |
| WordPress | **7.0.4** (paket resmi `wordpress-id_ID.zip`, 38.837.302 B) |
| Tema | Astra **4.14.0** (aktif) |
| Plugin | Elementor (aktif) · wordpress-importer (aktif, sekali pakai) |
| PHP / MariaDB | 8.2.12 / 10.4.32 |
| User DB | `smk10_staging` — hak hanya pada `smk10_staging_%` (minimum privilege) ✓ |
| wp-config | 8 SALT baru · `DISALLOW_FILE_EDIT` · `WP_AUTO_UPDATE_CORE='minor'` · `WP_HTTP_BLOCK_EXTERNAL` selama impor ✓ |

## 2. Sumber & konversi (read-only terhadap sumber)
- Dump `smkq8953_sragen10.sql` tidak diubah; SHA-256 = `0448259bb0a11763…0b46f3af` ✓
- Konverter offline `convert_dump_to_wxr.py` → `sragen-staging-pilot.wxr` (576.342 B, 4.819 baris, **149 item**)
- Laporan konverter: **149 post dipilih · 135 dikeluarkan · 352 postmeta · 0 diblokir IoC · 147 ditunda (serialized)**
- Sanitasi WXR: **107 `<wp:attachment_url>` dikosongkan** ⇒ importer tidak mengunduh apa pun dari internet/produksi ✓
- Impor: `--authors=skip` (akun lama tidak diimpor) ✓ · exit 0 · **"Failed to import" 99 — semuanya item Media** (memang disengaja; lampiran dibangun deterministik) ✓

## 3. Yang berhasil (terbukti)
| Aspek | Hasil |
|---|---|
| Staging dibuka | **4/4** URL uji → 200 (`/`, `/sejarah/`, `/kontak/`, `/struktur-organisasi/`) ✓ |
| Media | **418/418** berkas (107 asli + 310 derivatif + 1 `original_image`), **21,1 MB**, SHA-256 per berkas cocok ✓ |
| Uploads | **0** berkas `.php`/`.js`/`.html` ✓ (449 berkas total, termasuk thumbnail hasil regenerasi) |
| IoC | **0** pada opsi/meta ✓ |
| URL produksi | **0** sisa (termasuk bentuk ter-escape) ✓ |
| Menu | 19 `nav_menu_item`, 2 menu ✓ |
| Akun | hanya `sragen_admin` (1 admin) ✓ — 0 akun lama |
| Elementor JSON | 18 valid / **1 tidak valid** ✗ |

## 4. Yang belum berhasil (dinyatakan jujur)
1. **Artikel sangat kurang:** hanya **1 `post`** (sumber ~96 publish) — konverter mengeluarkan 135 item ⇒ perlu keputusan kriteria pemulihan artikel.
2. **Lampiran menjadi post 61/107** — 46 lampiran belum punya record meski berkasnya ada di uploads (manifest 418 baris dipakai apa adanya; perlu manifest per-lampiran-asli).
3. Thumbnail baru diregenerasi **8 dari 61**.
4. **Label "(STAGING pemulihan)" pada judul situs masih ada** ✗.
5. **Perbaikan UI/fungsi (header, logo, banner, menu mobile) belum dikerjakan** ✗.
6. **QA responsif 360–1366 & konsol JS belum dijalankan** ✗.
7. 1 JSON Elementor tidak valid.

## 5. Checkpoint
| Berkas | Ukuran | SHA-256 |
|---|---:|---|
| `recovery-work\staging\checkpoints\checkpoint-7-01-pasca-pemulihan-konten-sragen.sql` | 677987 B | `ad7e92e883a17b876fa2ccbf2805a34c19837c6dbc5d76c7d137cac4703ef37b` |

## 6. Kepatuhan
Tidak mengubah dump/tree bukti/inventaris/checksum/manifest ✓ · tidak menyalin root/tema/plugin/wp-config lama ✓ · tidak menjalankan payload ✓ · tidak mengekstrak `cgi-bin.zip` ✓ · tidak mengambil media dari produksi ✓ · tidak memakai akun/password lama ✓ · staging lain hanya read-only ✓ · tidak menyentuh produksi, `goly1264_ams`, proyek AMS, `C:\Users\sakha\Downloads\admin.php` ✓ · tanpa `git push` ✓.

## 7. Kredensial
Password `sragen_admin` (32 karakter acak) disimpan lokal di luar repository: `C:\Users\sakha\.smk10\sragen-admin.txt`; **tidak** ditulis di laporan/CSV/Git ✓.
⚠️ `recovery-work\staging\` **masih memuat berkas sandi lama** (`.dbpass`, `.adminpass`, `.cibitungadminpass`, `.tambunadminpass`, `.my_staging.cnf`) — sebaiknya dipindahkan keluar repository sebelum sinkronisasi berikutnya (tidak diubah pada fase ini).
