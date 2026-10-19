# FASE 6K-B — AUDIT OFFLINE & PERSIAPAN PEMULIHAN SRAGEN

**Tanggal:** 19 Oktober 2026 · **Mode:** read-only, **tanpa** membuat staging/database, **tanpa** impor, **tanpa** ekstraksi arsip, **tanpa** menjalankan berkas apa pun
**Ringkas:** dokumentasi & inventaris Sragen **lengkap dan terverifikasi** ✓ · **manifest media khusus belum ada** (harus dibangun offline) · media (2.421 berkas, 196,7 MB) **bersih dari webshell** ✓ · **docroot Sragen penuh artefak penyerang** (7 arsip dropper, 2 file-manager, defacement/SEO-cloak, shell tema) ✗ · 30 akun admin & 2 menu `x`/`proof` ✗

---

## 1. Dokumen & sumber yang benar-benar diperiksa
| Berkas | Lokasi terverifikasi | Ukuran | Relevansi Sragen |
|---|---|---|---|
| `LAPORAN-FASE-2.md` | `C:\Users\sakha\forensik-smk10november\` | 19.526 B | A8/A9/A11/A13 + hash artefak |
| `LAPORAN-FASE-3A.md` | idem | 32.067 B | toolkit per-instalasi, phpFileManager ×2, tema shell |
| `LAPORAN-FASE-3A-ADDENDUM.md` | idem | 14.777 B | hash 6 artefak kunci, `sso-loader.php` |
| `FASE-4A-INVENTARIS.md` | `…\audit-backup\` | 16.922 B | 16 arsip, 7 dropper di root Sragen |
| `FASE-4C-1/4C-2/4C-3` | `…\audit-backup\` | 15,7/48,1/23,6 KB | tabel & akun per situs, `wp2shell`, `wp-compat` |
| `FASE-4D-RENCANA-MIGRASI.md` | `…\recovery-work\` | 24.031 B | rencana migrasi awal |
| `FASE-6H-SRAGEN.md`, `FASE-6I-AUDIT.md`, `FASE-6J-AUDIT.md`, `FASE-6J-AKUN-WORDPRESS.md` | `…\recovery-work\` | 4,7–15 KB | staging tak ditemukan, rencana, akun |
| **Inventaris berkas Sragen** | `…\manifest\inventory-smk10novembersragen.sch.id_.txt` | 1.082.251 B | **7.445 baris** berformat `mtime\|waktu\|ukuran\|mode\|path` |
| **Checksum Sragen** | `…\manifest\sha256-smk10novembersragen.sch.id_.txt` | 1.110.274 B | **7.445 baris SHA-256** ✓ |
| `inventory-ROOTFILES.txt` / `sha256-ROOTFILES.txt` | `…\manifest\` | 1.804 / 1.896 B | docroot instalasi root |
| `inventory-cgi-bin_.txt` / `sha256-cgi-bin_.txt` | `…\manifest\` | **0 B** | **belum pernah diinventaris** ✗ |

**Catatan:** clone lokal repository `vael-skaya/wp` **tidak ditemukan** di host ini ⇒ laporan disimpan di direktori kerja terverifikasi `recovery-work\`.

## 2. Identitas sumber Sragen (read-only)
- Dump: `D:\backup-database-smk10november\smkq8953_sragen10.sql` — **80.897.366 B**, SHA-256 `0448259bb0a11763…`, prefix **`srg_`** (tidak diubah, tidak diimpor).
- Versi WordPress sumber: **7.1.3** · tema: **hello-elementor** · 119 tabel plugin · 30 akun (semua administrator) · 107 lampiran · 0 `_thumbnail_id`.
- Inventaris berkas: **7.445 berkas / 412,1 MB**; di antaranya **2.421 berkas / 196,7 MB** di `wp-content/uploads`.

## 3. Audit manifest media (offline, dari inventaris + checksum)
**Manifest khusus media tidak ada.** Yang tersedia = inventaris + checksum, yang **cukup** untuk membangun manifest (path + ukuran + SHA-256) — **belum dibangun** (butuh persetujuan).

### 3.1 Komposisi `wp-content/uploads` (2.421 berkas)
| Ekstensi | Jumlah | Penilaian |
|---|---:|---|
| `.jpg` / `.jpeg` | 1.046 / 49 | media sah |
| `.webp` | 833 | media sah |
| `.png` | 170 | media sah |
| `.woff2` | 152 | font tema/plugin |
| `.json` | 94 | data Forminator/Elementor |
| `.log` | 26 | **log WooCommerce/Midtrans** — bukan media |
| `.html` | 13 | stub `index.html` + cache WPForms (4,4 MB ×2) |
| `.svg` | 12 | ikon |
| `.css` | 5 | aset |
| `.php` | **8** | **semuanya stub plugin sah** (0–99 B, "silence is golden") ✓ |
| tanpa ekstensi | 11 | **semuanya `.htaccess`** plugin ✓ |

### 3.2 Pemeriksaan pola berbahaya
| Pemeriksaan | Hasil |
|---|---|
| **Ekstensi ganda** (`.jpg.php` dsb.) di seluruh situs | **0** ✓ |
| PHP di uploads | 8 berkas, **semuanya stub plugin** (Elementor forms, Forminator, MailPoet, Template Kits, WPForms cache) ✓ |
| `.htaccess` di uploads | 11, semuanya direktif plugin (`Deny from all` / hardening) ✓ |
| Nama berpola malware (`c99`, `wso`, `b374k`, `r57`, `shell`, `backdoor`, `priv8`) | 4 hit, **semua false-positive** (CodeMirror `shell.js`, `uploader-icons.png`, PEAR `Text/Diff/Engine/shell.php`) ✓ |
| `..` / path traversal | **0** ✓ |
| Ekstensi executable di luar stub | **0** ✓ |

⇒ **Folder media Sragen tidak memuat webshell.** Ini bertentangan dengan dugaan umum "uploads penuh shell" — buktinya: seluruh 2.421 path diperiksa, 8 PHP semuanya stub ✓.

### 3.3 Berkas non-media di uploads yang perlu **dikarantina** (jangan dimigrasikan)
| Item | Alasan |
|---|---|
| `uploads/wc-logs/*.log` (24) + `cartflows-logs` (1) | log transaksi pembayaran (**Midtrans**) — berisi data pelanggan/transaksi |
| `uploads/astra-sites/import-*.log` (**16,5 MB**) | log impor kit, tidak diperlukan |
| `uploads/wpforms/cache/templates-*.html` (4,4 MB ×2) | cache, bukan konten |
| **`uploads/wp-file-manager-pro/fm_backup/`** ✗ | cadangan situs oleh plugin file-manager — berpotensi memuat **konfigurasi/kredensial** |
| `uploads/ai-builder/`, `uploads/template-kits/`, `uploads/forminator/<hash>/` | sisa sesi pembuatan situs |

## 4. Temuan kompromi (bukti dokumen forensik, tingkat keyakinan terpisah)
### 4.1 **Terkonfirmasi malware** (hash + dekode statis terdokumentasi)
| Artefak | Lokasi | Ukuran | Bukti | Keyakinan |
|---|---|---|---|---|
| **`goodsdetails.php`** | docroot Sragen | 177.474 B (`3d6b373e…`) | rantai `eval(base64_decode(...))` (A8, Fase 3A) | **Tinggi** |
| **`shopsdetails.php`** | docroot Sragen | 15.089 B (`ea1e3e37…`) | `goto`-spaghetti 36× + `base64_decode` + `gzinflate` (A9) | **Tinggi** |
| **`wp-slgnup.gz`** | docroot | 2.640 B (`9dc33afe…`; 8.120 B setelah dekompresi) | `curl_init` + `str_rot13(urldecode())` + `file_put_contents`; iframe YouTube tersembunyi + SEO-cloaking (A11/F-06) | **Tinggi** |
| **7 arsip dropper** (`item.zip`, `hiroshi.zip`, `wp-blog-header.zip`, `google006f33b90c2ea6f4.zip`, `shopsdetails.zip`, `goodsdetails.zip`, `.zip`) | docroot | 241 B – 33.496 B | `hiroshi.php` 106.425 B (dropper `eval(base64_decode)→gzinflate`), `item.php`; `goodsdetails.zip` 31.646 B **lebih kecil** dari berkas on-disk ⇒ berkas tumbuh setelah pengarsipan | **Tinggi** |
| **phpFileManager 1.7.9 ×2** sebagai `archive.php` | dua lokasi web-served | — | perkakas file-manager sah ditanam dengan **`auth_pass` kosong** (Fase 3A) | **Tinggi** |
| **`themes/wp_zqyq/404.php`** | tema rogue | 1.892 B (`a1165eec…`) | RCE XOR + gate `admin0978` (A6) | **Tinggi** |
| **`themes/uvisyfyj/index.php`** | tema rogue | — | upload shell (Fase 3A) | **Tinggi** |
| **injeksi `hello-elementor/footer.php`** | tema aktif | 1.458 B → **961 B** (`9c6f372f…`) | footer terinjeksi (Fase 3A-ADDENDUM) | **Tinggi** |

### 4.2 **Indikasi kuat, bukti tidak mandiri**
| Artefak | Temuan | Keyakinan |
|---|---|---|
| `mu-plugins/sso-loader.php` (root & sragen, identik) | diakses langsung via HTTP 28-Sep-2026; opsi `sso_token` di DB | **Sedang–tinggi** |
| `wp-compat/` | direktori plugin **kosong (0 berkas)**, mtime **11-Sep-2026 20:18** (dalam jendela serangan), **tidak** ada di `active_plugins`, pernah dirujuk `error_log` sebagai `wp-compat.php` | **Sedang** — tidak ada yang bisa dijalankan, tetapi asal-usulnya mencurigakan |
| `info.php` (19 B, `075368f9…`) | `<?php phpinfo(); ?>` — probe informasi | **Rendah bila sendiri**, menguat bersama `sso-loader` |
| `performance-tweaks.php` | **0 B** (kosong) | **Rendah** — tidak berbahaya, tapi sampah |
| `error_log` di `mu-plugins/` | 5 entri akses langsung ke `sso-loader.php` (28-Sep, 29-Sep ×2, 03-Oct, 09-Oct) | **Sedang** |

### 4.3 **Bukan bukti malware (dinyatakan agar tidak salah tangkap)**
- `googlefb3f6c75552e5d2d.html` (53 B) = verifikasi Google Search Console.
- Log WooCommerce (`fatal-errors`, `place-order-debug`, `midtrans-*`) = log aplikasi, **bukan** payload.
- `index.html` 9.598 B **defacement "hacked by rehan23"** — terletak di **tree root** (`D:\smk10november\index.html`), mtime **2026-10-12 21:21** (setelah jendela penyalinan) ⇒ **bukan** milik docroot Sragen, tetapi bukti defacement situs root.
- `index.php` di docroot Sragen = **0 byte** ✗ ⇒ instalasi sumber **tidak dapat di-bootstrap** (tidak berfungsi); ini juga berarti **tidak ada gunanya** menyalin berkas inti dari sumber.
- `wp2shell` di DB Sragen: **18 kemunculan** (menu item `https://wp2shell.research`, `wp2shell.com`) + akun `wp2shell_aa673039837f` ✗ (lihat §5).

## 5. Akun & privasi (ringkas; rincian hanya di berkas lokal)
- Sragen: **30 akun, semuanya `administrator`**; **1 akun asli** (`admin@…`, dibuat 2025-10-30), **29 akun** dibuat 2026-07…09 dengan pola penyerang ✗.
- Status akun: **belum terverifikasi** — tidak ada yang dapat dibuktikan sebagai staf sekolah dari dump.
- **Tidak ada** hash/potongan hash/email yang dimuat di laporan ini; daftar rinci ada di `recovery-work\sensitive\6j-akun\` (lokal, **jangan** diunggah) ✓.

## 6. Rencana migrasi (belum dijalankan)
### 6.1 Media
| Kategori | Isi | Tindakan |
|---|---|---|
| **Layak ditinjau untuk migrasi** | Media yang **dirujuk** 107 `_wp_attached_file` + varian ukurannya (jpg/webp/png/svg), diverifikasi dengan SHA-256 dari inventaris | impor setelah manifest dibuat & hash cocok |
| **Dikarantina / tinjau manual** | Semua berkas uploads **yang tidak dirujuk** DB; log WooCommerce/Midtrans; cache WPForms; `fm_backup/`; sisa `ai-builder`/`template-kits` | **tidak** diimpor; disimpan sebagai bukti |
| **Dikecualikan (IoC)** | Semua artefak §4.1–4.2 (PHP/zips/tema rogue/mu-plugin/footer terinjeksi) | **tidak** disalin sama sekali |
| **Belum dapat ditentukan** | Isi `cgi-bin.zip` (belum diinventaris, butuh persetujuan ekstraksi); payload termediah `goodsdetails.php` (rantai >14 lapis belum didekode) | ditunda |

### 6.2 Platform
Inti WordPress + **hello-elementor** + Elementor dari paket resmi; **tanpa** Elementor Pro, **tanpa** `wp-file-manager`, **tanpa** `wp-compat`, **tanpa** WooCommerce/Midtrans pada tahap awal (keputusan sekolah bila toko memang diperlukan).

### 6.3 Urutan kerja (setelah persetujuan)
1. **Bangun manifest media offline** (107 lampiran ⇄ 2.421 berkas) + verifikasi SHA-256; laporkan berkas hilang.
2. **Pemeriksaan integritas & keamanan**: 0 ekstensi ganda (sudah ✓), 0 PHP non-stub (sudah ✓), 0 IoC §4.1 di kandidat, semua hash cocok.
3. **Checkpoint wajib**: snapshot manifest + hash kandidat; setelah staging dibuat → checkpoint DB `checkpoint-6l-01-pre-impor-sragen.sql` dan seterusnya **sebelum** setiap tahap.
4. Buat direktori `C:\xampp\htdocs\staging\sragen` + DB `smk10_staging_sragen` (**hanya setelah disetujui**).
5. Instal inti/tema/plugin resmi, `WP_HTTP_BLOCK_EXTERNAL true`.
6. Konversi dump→WXR **dengan pengecualian**: menu `x`/`proof` & URL `wp2shell.*`, 30 akun lama, tabel plugin, opsi sensitif, `sso_token`.
7. Impor → media dari manifest → `attach-make`/`fixmime`/`regen` → ID map → URL remap → perms.
8. Validasi V-01..V-12 + suite QA 6G/6H + **verifikasi 0 IoC**.
9. Rollback: drop DB + hapus direktori (atas persetujuan), checkpoint SQL tersimpan.

## 7. Kekurangan bukti & risiko yang tersisa
1. **Manifest media belum ada** — harus dibangun (bisa, karena inventaris memuat ukuran + SHA-256).
2. **Payload `goodsdetails.php` belum didekode penuh** (>14 lapis) — sifat persisnya belum diketahui; **tidak akan** dieksekusi.
3. **`cgi-bin.zip` (1,33 GB)** belum diinventaris (0-byte inventory) — kemungkinan memuat salinan keempat situs; ekstraksi butuh persetujuan terpisah.
4. **`index.php` sumber 0 byte** ⇒ tidak ada berkas inti yang bisa disalin; inti harus dari paket resmi.
5. Plugin/Commerce: WooCommerce + Midtrans + WPForms + Forminator + MailPoet + Cartflows + wp-file-manager-pro ada di sumber ⇒ keputusan cakupan.
6. 30 akun admin & `sso_token` ⇒ model akun harus dibuat ulang.
7. Versi sumber 7.1.3 vs paket staging; K-1 masih terbuka.

## 8. Tindakan yang **belum** dilakukan & butuh persetujuan
- Membangun manifest media Sragen; membuat direktori staging & database; impor dump/WXR; mengekstrak `cgi-bin.zip`; menghapus/mengubah akun; perbaikan pemetaan lampiran (6K-A §8); apa pun terhadap produksi.

## 9. Konfirmasi
- **Tidak ada** staging/database Sragen dibuat; **tidak ada** impor; **tidak ada** arsip diekstrak; **tidak ada** berkas dijalankan/dipindai ulang secara destruktif; dump **tidak diubah** ✓
- **Produksi tidak disentuh**; tidak ada deployment/DNS/hosting ✓
- **`goly1264_ams` tidak diakses, tidak di-query, tidak di-backup, tidak di-restore, tidak diubah** ✓
- **`C:\Users\sakha\Downloads\admin.php` tidak disentuh** (tidak dibaca/dijalankan/dipindai/diedit/dipindah/diganti nama/dihapus) ✓
- Tidak ada `git push`/deployment ✓ · laporan tanpa password/hash kredensial/token ✓
