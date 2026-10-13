# Laporan Fase 3A — Investigasi Forensik Lanjutan WordPress

- **Sifat kerja:** analisis statis **read-only**. Tidak ada file di `D:\smk10november` yang dibuat, diubah, dihapus, dipindah, atau diganti nama.
- **Tidak dilakukan:** eksekusi file/payload dari direktori bukti, akses jaringan/DNS/URL penyerang/hosting/DB/kredensial, perubahan konfigurasi (Windows/Defender/PHP/WordPress), pembacaan rahasia `wp-config.php`/token/cookie/private key.
- **Tidak ada ekstraksi baru ke disk.** Isi arsip dibaca **di memori** (Python `zipfile`/`gzip`) lalu dianalisis statis. Berkas hasil ekstraksi Fase 2 di `extract/` tidak ditambah.
- **Seluruh laporan & alat disimpan di** `C:\Users\sakha\forensik-smk10november\` (direktori sudah ada; tidak membuat direktori baru).
- **Uji integritas:** `find D:\smk10november -newermt '2026-10-13 00:00'` → **0 entri**; jumlah file tetap **38.906**.

---

## 1. Ringkasan eksekutif

> ⚠️ **BACA JUGA:** `LAPORAN-FASE-3A-ADDENDUM.md` — addendum ini memuat temuan **lebih berat** yang tidak tercakup di bawah: **berkas inti WordPress ditimpa** (`wp-admin/includes/admin.php` → "PHP File manager ver 1.5" 84 KB), **injeksi webshell ke berkas inti** (`wp-signup.php`, `class-wp-query.php` di Cibitung) dan ke **tema aktif `hello-elementor`**, serta 622 berkas `error_log` (212,56 MB). Bagian §3.3 di laporan ini **tidak lengkap** — daftar di addendum adalah superset.

Fase 3A menemukan **satu kelas webshell yang belum teridentifikasi pada Fase 2** dan berhasil mengidentifikasinya secara pasti, termasuk **kondisi autentikasinya**:

> Dua salinan identik **`phpFileManager 1.7.9`** (file manager PHP pihak ketiga yang sah) ditanam sebagai `archive.php` di dua lokasi web-served pada instalasi **Sragen**, dengan `auth_pass` = **MD5 dari string kosong** (`d41d8cd98f00b204e9800998ecf8427e`) — artinya **tanpa password**.

Selain itu, analisis dilanjutkan dengan **normalisasi string PHP** yang membuktikan bahwa hampir semua artefak kritis memakai **evasi berbasis konkatenasi literal** (`"p"."r"."o"."c"."_"."o"."p"."e"."n"`), sehingga banyak regex naif akan menghasilkan *false negative*. Temuan Fase 2 ditegakkan ulang dengan bukti yang lebih kuat, dan **seluruh hash dicocokkan ulang terhadap manifest** — semuanya **COCOK**, artinya bukti belum berubah.

**Distribusi IoC sangat terpisah per instalasi** (bukan satu jalur seragam): Sragen memuat toolkit file-manager + dropper + upload shell; Cibitung memuat webshell RCE bergaya plugin; Kebumen memuat cluster SEO doorway; root memuat defacement + SEO-cloaking + `sso-loader.php`.

---

## 2. Tugas A — Validasi temuan lama

Semua artefak yang disebut Fase 2 **benar-benar ada** di salinan lokal, dan **semua SHA-256 COCOK dengan manifest** (bukti tidak berubah sejak hashing).

### 2.1 Bukti perilaku (diverifikasi ulang setelah normalisasi string)

| # | Path | SHA-256 | Ukuran | Perilaku yang terbukti dari kode | Risiko | Keyakinan |
|---|---|---|---|---|---|---|
| A1 | `smk10novembercibitung.sch.id/wp-content/plugins/wp-wordfence-security/proc.php` | `a8a2bb81720f50258555598c8af80f478dd698cbb9c6127a2d798f4ed360eb48` | 562 B | Eksekusi perintah via `proc_open()` dengan nama fungsi **dipecah**: `$prxs = "p"."r"."o"."c"."_"."o"."p"."e"."n"; $process = $prxs($_GET['data-admin'], …)`; output dikembalikan lewat `stream_get_contents` | **Kritis** | Tinggi |
| A2 | `…/wp-wordfence-security/pcntl.php` | `4f9ff8be285193ae0f66b10b3ff7892284b6f288941fb6e5f799e8b9902b61ef` | 892 B | `$_GET['data-admin']` → `pcntl_fork()` + `pcntl_exec('/bin/sh', ['-c', $cmd])`; output dibaca dari berkas temp | **Kritis** | Tinggi |
| A3 | `…/wp-wordfence-security/wp-wordfence-security.php` | `be7e3a1905b9439b9fe526962f2eb51ceeee9dd84050f587dc8f692232917881` | 358 B | Hanya header plugin: `Plugin Name: Wordfence Security`, `Author: exe`. Tidak ada kode fungsional → **decoy** | Tinggi (kamuflase) | Tinggi |
| A4 | `…/wp-wordfence-security/info.php` | `9c187470da5b287bf03552bcc36ef74d3a636125aa7e8492f64c3ef89c046ee3` | 20 B | `<?php phpinfo(); ?>` | Sedang | Tinggi |
| A5 | `wp-content/themes/wp_edvl/404.php` | `2c37260b3eb8eee8d156d1af1d088d78cbd29ddc183753c6ca2ed883dc05b258` | 1.892 B | Gate `?g=` bernilai `admin0995` (dipecah `"ad"."min"."0995"`); `$_POST['c']` → `base64_decode` → **XOR** key `TTTDowpf` → eksekusi `system`/`passthru`/`shell_exec`/`exec` (semua dipecah literalnya) | **Kritis** | Tinggi |
| A6 | `smk10novembersragen.sch.id/wp-content/themes/wp_zqyq/404.php` | `a1165eec7ef1ee43202e9617c0d6e5a297450858e67a143b06b718373b09388d` | 1.892 B | Sama seperti A5, gate `admin0978`, XOR key `VKl1tNpC` | **Kritis** | Tinggi |
| A7 | `smk10novembersragen.sch.id/wp-content/themes/uvisyfyj/index.php` | `7adfdaa88c62766e5343813d90dba0f2087d034ffb76402d8ae82e14aa757bf9` | 368 B | Gate `?g=admin888` (dipecah `'ad'.'mi'.'n8'.'88'`) → `move_uploaded_file($_FILES['f']['tmp_name'], $_FILES['f']['name'])` → **upload file arbitrer**; jika gate salah tampilkan form upload | **Kritis** | Tinggi |
| A8 | `smk10novembersragen.sch.id/goodsdetails.php` | `3d6b373e6489b163a9d7d40c8819517bc720f1c4973f497c81e69be7ec57fc0f` | 177.474 B | Rantai terkonfirmasi via dekode statis: `eval(base64_decode(…))` → `eval(base64_decode(…))` → `eval(gzinflate(base64_decode(…)))` | **Kritis** | Tinggi (rantai), sedang (payload terdalam) |
| A9 | `smk10novembersragen.sch.id/shopsdetails.php` | `ea1e3e379045730c422db5f065808dcf43f13c8febb2d44450f1d453cc3bcb57` | 15.089 B | Struktur `goto`-spaghetti (36×) + `base64_decode` + `gzinflate` + transformasi `chr(ord(…)-4)` + key literal | **Kritis** | Tinggi |
| A10 | `wp-content/mu-plugins/sso-loader.php` **dan** `smk10novembersragen.sch.id/wp-content/mu-plugins/sso-loader.php` | `337fee21376d07b27f6bcf7ef7c85e74c3273aadff491137f524ab1cb1aa111d` (keduanya **identik**) | 2.861 B | Header `Plugin Name: SSO` (kredit "Garth Mortensen, Mike Hansen"); hook `wp_ajax_nopriv_sso-check`; bandingkan `base64(sha256(nonce.salt))` dengan opsi `sso_token`; jika cocok → `wp_set_auth_cookie()` + `do_action('wp_login')` untuk user (default: **administrator pertama**) → **login admin tanpa password** | Tinggi (potensial) | **Sedang — perlu verifikasi nilai token** |
| A11 | `wp-slgnup.gz` | `9dc33afe68232584b0c00fe5e4feb1211da77b7aff5183bb24c3441e0b74e98d` | 2.640 B (8.120 B setelah dekompresi) | `curl_init` + `str_rot13(urldecode(...))` + `file_put_contents` (tulis ulang `robots.txt`, string dibentuk via escape `\57\162\157…`), `disbot()` (deteksi Googlebot/Bing/Yahoo), `handle_content()` (suntik konten per-tipe MIME) | Tinggi | Tinggi |
| A12 | `info.php` (root) | `075368f947e7b5399efee4f002b99c8cdc704276e63b46adae9a46256ddf1eb8` | 19 B | `<?php phpinfo(); ?>` | Sedang | Tinggi |
| A13 | `index.html` (root) | `537f3e5665dd556650541c9128a0edd09e30ddcf9199cb80c78a622a68c82306` | 9.598 B | Defacement: `REHAN23`, `hacked by rehan23`, iframe YouTube tersembunyi | Tinggi | Tinggi |

### 2.2 Teknik evasi yang terbukti (bukti terukur)

| Artefak | Pola di teks **mentah** | Setelah normalisasi |
|---|---|---|
| `proc.php` | `"p"."r"."o"."c"."_"."o"."p"."e"."n"` | `proc_open` (**0× → 1×**) |
| `wp_edvl/404.php` | `"ad"."min"."0995"`; `"s"."y"."s"."t"."e"."m"` dst. | `admin0995`, `system`, `passthru`, `shell_exec`, `exec` (**semua 0× → muncul**) |
| `wp_zqyq/404.php` | idem dengan `admin0978` | idem |
| `uvisyfyj/index.php` | `'ad'.'mi'.'n8'.'88'` | `admin888` (**0× → muncul**) |
| `wp-slgnup.gz` | `"\57\162\157\142\157\164\163\56\164\170\164"` | `robots.txt`, `Sitemap`, `User-agent` |

**Konsekuensi metodologis:** analisis string naif akan **melewatkan** fungsi berbahaya. Seluruh verifikasi di laporan ini dilakukan pada teks yang sudah dinormalkan (`tools/phputil.py`).

### 2.3 File yang disebut log/laporan tetapi TIDAK ditemukan (atau hanya di arsip)

| Nama | Status pada salinan lokal |
|---|---|
| `anjv2.php` | **TIDAK ADA** — hanya dirujuk `error_log` (4×, `/home/smkq8953/public_html/anjv2.php:98`, 09-Agu-2026) |
| `wp-content/plugins/wp-compat/wp-compat.php` | **TIDAK ADA** — direktori `wp-compat/` **kosong** (mtime 11-Sep-2026 20:18) |
| `hiroshi.php` | **TIDAK ADA di disk** — hanya di dalam `hiroshi.zip` |
| `item.php` (malware) | **TIDAK ADA di disk** — hanya di dalam `item.zip`. Yang ada di disk hanyalah `Item.php` milik library SimplePie (sah) |
| `admin.php` (file manager) | **TIDAK ADA di disk** — hanya di dalam `admin.zip` |
| `sso-loader.php` | **ADA** (root + sragen) |

---

## 3. Tugas B — Inventarisasi persistensi

### 3.1 PHP di direktori upload
- Total `.php` di `wp-content/uploads` (4 instalasi): **85 file** — **18 file berisi >0 byte**, **67 file 0 byte**.
- Ke-18 yang non-kosong **semuanya bernama `index.php`** dan isinya placeholder proteksi standar (`// Silence is golden`, atau header 404), bukan kode berbahaya:
  `elementor/forms/index.php` (27 B), `mailpoet/index.php` (27 B), `wpforms/cache/index.php` (99 B), `wpforms/cache/templates/index.php` (99 B), `download-manager-files/index.php` (11 B), `wpdm-custom-tags/index.php` (11 B).
- **Tidak ada** `.phtml`, `.phar`, `.inc`, `.php5/7` di uploads.
- **Penilaian:** uploads **bersih** dari webshell. File 0 byte = artefak plugin/tool.

### 3.2 Plugin / tema / mu-plugins
Audit dilakukan dengan membandingkan **nama direktori vs header di dalam file** (`Plugin Name`/`Text Domain`/`Theme Name`) — bukan berdasarkan nama saja.

**Kandidat kuat (berbasis ketidaksesuaian, bukan nama):**

| Lokasi | Temuan | Bukti |
|---|---|---|
| `cibitung/…/plugins/ioyntkq` | Direktori bernama acak, isinya plugin **"Protect Uploads"** (`text domain: protect-uploads`) | Header vs nama direktori tidak cocok → indikasi direktori di-rename |
| `cibitung/…/plugins/lvcibfqxpseo` | direktori **berisi hanya `.htaccess` 32 B** | Sisa plugin SEO spam yang isinya sudah dihapus |
| `sragen/…/plugins/wp-compat` | direktori **benar-benar kosong** (0 file) | Dirujuk `error_log` root sebagai `wp-compat.php` |
| `cibitung/…/plugins/wp-db-optimizer` | **Stub**: hanya `readme.txt` (257 B, klaim "WP Database Optimizer" oleh `wpcoreteam`) + 2 class PHP di `assets/includes` + **3 file JS 0 byte**; **tidak ada bootstrap plugin / header `Plugin Name`** | Tidak ada fungsionalitas; kandidat bekas wadah muatan |

**Tema "cangkang" (shell theme):**

| Tema | `style.css` | Bukti |
|---|---|---|
| root `themes/wp_edvl` | 25 B: `/* Theme Name: wp_edvl */` | `index.php` (21 B) + **`404.php` = webshell RCE** (A5) |
| sragen `themes/wp_zqyq` | 25 B: `/* Theme Name: wp_zqyq */` | `index.php` (21 B) + **`404.php` = webshell RCE** (A6) |
| sragen `themes/uvisyfyj` | 39 B (malformed: `/*Theme Name: uvisyfyjVersion: 1.0*/`) | **`index.php` = upload shell** (A7) |

**mu-plugins** — `sso-loader.php` (root & sragen, identik), `hostinger-auto-updates.php`, `hostinger-preview-domain.php` (Cibitung/Kebumen; header Hostinger resmi), plus berkas `error_log` yang ditanam di dalam `mu-plugins` (root & sragen).

**Plugin lain** (dengan header valid, bukan indikasi kompromi): `super-easy-maintenance-mode`, `wp-file-manager`, `all-in-one-wp-migration-unlimited-6.77.1`, `classic-editor`, `disable-wordpress-updates`, `elementor`, `elementor-pro`, `elementskit-lite`, `skyboot-custom-icons-for-elementor`, `custom-codes` (CodeKit), `download-manager`.
> **Catatan (dugaan):** `all-in-one-wp-migration-unlimited-6.77.1` adalah varian "unlimited" (premium) — indikasi plugin *nulled*. Ini **vektor infeksi klasik**, tetapi **belum terbukti** sebagai jalur masuk di kasus ini.

### 3.3 `.htaccess` dan PHP di lokasi tidak lazim
- Total `.htaccess`: **355**. Mayoritas normal (wp-admin, uploads per-tahun, plugin bawaan, `jetpack-waf`, `wpforms`, dll.).
- **Lokasi tidak lazim:**

| Path | Ukuran | Isi / catatan |
|---|---|---|
| `cibitung/…/plugins/wp-wordfence-security/.htaccess` | 32 B | `Options -Indexes` + `Allow from all` → **membuka akses direktori webshell** |
| `cibitung/…/plugins/lvcibfqxpseo/.htaccess` | 32 B | `Order allow,deny` + `Allow from all` (identik polanya) |
| `sragen/.htaccess` (root) | 204 B | Ditulis tangan, **tanpa blok WordPress/LiteSpeed** — berbeda dari root install lain (833 B); hanya rewrite dasar |
| `*/wp-content/ai1wm-backups/.htaccess` (4×) | 188 B | Milik plugin migrasi (wajar) |

- **PHP di lokasi tidak lazim (non-core, ukuran >0):** selain yang di atas → `smk10novembercibitung.sch.id/wp-loadss.php` (3.937 B), `smk10novembersragen.sch.id/{goodsdetails,shopsdetails}.php`, `smk10novembersragen.sch.id/wp-content/themes/archive.php` (**998.842 B**), `smk10novembersragen.sch.id/wp-includes/block-supports/archive.php` (**998.842 B**), `smk10novemberkebumen.sch.id/prestasi/{index,bb,home}.php`.

**`wp-loadss.php` (Cibitung)** — byte-identik dengan `wp-load.php` kecuali newline akhir (tidak ada baris unik):
```
sha256 wp-loadss.php : bfdc84087782be7aa13f64deb6fe6762f4c80058d9fa3d27705859716dc5dd11
sha256 wp-load.php   : 353db635f0f4021936a9c744ffe3a76eaf0e7feef951463e130bd445897d0685
baris hanya di loadss: []      baris hanya di load  : []
```
→ **Duplikat nama mirip, tanpa kode tambahan.** Tidak ada bukti perilaku jahat. **Status: belum dapat diverifikasi sebagai artefak serangan** (bisa jadi artefak penyalinan).

### 3.4 Nama acak & ekstensi ganda
- Nama acak: `ioyntkq` (plugin, ter-rename), `lvcibfqxpseo` (sisa plugin `*seo`), tema `wp_edvl`/`wp_zqyq`/`uvisyfyj`, plugin `wp-db-optimizer`, direktori `wp-compat`.
- Ekstensi ganda: hanya 2 pola — `product-hoodie4.jpg.jpg` (penamaan gambar, wajar) dan `scss.inc.php` (library `scssphp`, sah). **Tidak ada** pola `*.jpg.php` / `*.php.jpg` (teknik upload-bypass) di tree ini.

### 3.5 Referensi ke artefak yang disebut sebelumnya
- `anjv2.php` → **tidak ada** (hanya di `error_log`, 4×, 09-Agu-2026, dalih `shell_exec` yang dinonaktifkan).
- `wp-compat.php` → **tidak ada**; direktori `wp-compat/` kosong.
- `/tmp/phpXXXXXX` (18 rujukan di `error_log`) → jejak berkas upload PHP sementara pada periode log.

---

## 4. Tugas C — Analisis arsip (read-only, di memori)

**Tidak ada ekstraksi ke disk.** Semua pembacaan lewat `zipfile`/`gzip` di memori.

| Arsip | Member | Ukuran member | Tanggal internal zip | Hasil |
|---|---|---|---|---|
| **`sragen/wp-content/themes/admin.zip`** | `admin.php` | 39.353 B | **2026-08-12 09:40** | File-manager webshell bergaya Cina (teks UTF-8), tanpa header terlihat; tanda perilaku: `move_uploaded_file`, `shell_exec`×2, `system`×3, `passthru`×2, `proc_open`×2, `popen`×2, `pcntl_exec`×2, `assert`×2, `create_function`×2, `chmod`×19, `scandir`×3. **Tidak ada di disk.** |
| **`sragen/wp-content/themes/archive.zip`** + **`sragen/wp-includes/block-supports/archive.zip`** (identik, `cf7bf33a…`) | `archive.php` | **998.842 B** | **2025-12-24 10:46** | **phpFileManager 1.7.9** (lihat §5) |
| `sragen/item.zip` | `item.php` | 26.227 B | 2026-08-31 15:36 | Dropper `eval(gzinflate(base64_decode(…)))` |
| `sragen/hiroshi.zip` | `hiroshi.php` | 106.425 B | 2026-09-07 11:45 | Dropper `eval(base64_decode(…))`→`eval(gzinflate(…))` |
| `sragen/wp-blog-header.zip` | `wp-blog-header.php` | 12.207 B | 2026-09-07 11:40 | `PHP Encode v1.0 by zeura.com` + `file(__FILE__)`. **Dikonfirmasi AV: ThreatID 2147761377** |
| `sragen/google006f33b90c2ea6f4.zip` | html verifikasi | 53 B | 2026-09-15 17:28 | Salinan berkas verifikasi Google |
| `sragen/shopsdetails.zip` | `shopsdetails.php` | 15.089 B | 2026-09-22 14:44 | Sama dengan A9 |
| `sragen/goodsdetails.zip` | `goodsdetails.php` | 31.646 B | 2026-09-22 15:00 | Versi **lebih kecil** dari berkas on-disk (177.474 B) |
| `sragen/index.zip` | `index.php` | 115.859 B | 2026-09-22 15:05 | Dropper `eval(base64_decode(…))`→`eval(gzinflate(…))` |
| `sragen/.zip` (nama file tersembunyi) | `.htaccess` | 204 B | 2026-01-29 09:05 | Salinan `.htaccess` root sragen |
| `*/uploads/template-kits/<hash>/source-424.zip` (4×) | template kit (json/jpg) | ~3 MB | 2024-03-24 | Template kit Elementor — **normal** |

> Belum dianalisis lebih dalam: rantai payload terdalam `index.php`/`hiroshi.php`/`item.php` (bersarang >3 lapis). Untuk analisis lebih lanjut tidak diperlukan ekstraksi — semua dapat dibaca di memori.

---

## 5. Temuan kritis baru: phpFileManager dengan password kosong

**Dua salinan identik, keduanya di instalasi Sragen, keduanya berada di lokasi yang dapat dilayani web:**

| Path | Ukuran | SHA-256 |
|---|---|---|
| `smk10novembersragen.sch.id/wp-content/themes/archive.php` | 998.842 B | `cae2c22123b91641d6d4df2552cce2bc81d7659bad638976abff63a8910dbbb4` |
| `smk10novembersragen.sch.id/wp-includes/block-supports/archive.php` | 998.842 B | `cae2c22123b91641d6d4df2552cce2bc81d7659bad638976abff63a8910dbbb4` |

**Identifikasi (dari header berkas, dibaca di memori):**
```
| PHP FILE MANAGER
| phpFileManager 1.7.9
| By Fabricio Seger Kolling
| Copyright (c) 2004-2021 Fabrício Seger Kolling
| URL: http://phpfm.sf.net
| Last Changed: 2021-09-18
```

**Konfigurasi tertanam (baris ke-2 berkas):**
```json
{"lang":"","fm_root":"","timezone":"","date_format":"Y\/m\/d H:i",
 "auth_pass":"d41d8cd98f00b204e9800998ecf8427e","error_reporting":1}
```

**Verifikasi `auth_pass` (dihitung, bukan diasumsikan):**
```
$ printf '' | md5sum
d41d8cd98f00b204e9800998ecf8427e *-
```
→ `auth_pass` = **MD5 dari string kosong** ⇒ **file manager tanpa autentikasi efektif**.

**Sifat temuan yang penting (agar tidak salah label):** `phpFileManager` adalah **perangkat lunak sah pihak ketiga**, bukan malware berdasarkan namanya. Yang menjadikannya artefak serangan adalah **konteks**: (a) ditanam di direktori web-served dengan nama menyamar sebagai template tema WordPress (`archive.php`) dan sebagai berkas di dalam `wp-includes/block-supports/`, (b) **autentikasi kosong**, (c) disertai salinan arsip `archive.zip` di kedua lokasi, dan (d) satu instalasi yang sama juga memuat upload shell + dropper terobfuskasi.

**Kemampuan efektif bila diakses:** manajemen berkas penuh (unggah/edit/hapus/chmod), eksekusi perintah (`system`×50, `shell_exec`×6, `passthru`×9, `proc_open`×6, `popen`×8, `pcntl_exec`×7 di dalam berkas), `phpinfo`×27, `chmod`×146, `curl_init`×3. **Risiko: Kritis. Keyakinan: Tinggi.**

---

## 6. Tugas D — Timeline & hubungan antarsitus

### 6.1 Timeline (dengan peringatan metodologis)

> **Peringatan:** `mtime` pada salinan ini merekam **kapan penyalinan dilakukan**, bukan waktu kompromi. Bukti waktu yang lebih bermakna adalah **timestamp internal arsip** (saat dropper dibuat/di-zip) dan **timestamp di log**. Semua tanggal di bawah adalah **indikatif**, bukan penetapan waktu.

| Waktu | Peristiwa | Sumber |
|---|---|---|
| 2025-12-24 10:46 | Tanggal internal `archive.php` (build tool phpFileManager) | timestamp zip |
| 2026-01-29 09:05 | `.htaccess` di dalam `.zip` (sragen) | timestamp zip |
| 2026-03-27 14:10 | Entri `error_log` paling awal | `error_log` |
| 2026-08-09 23:31–23:32 | **4× percobaan eksekusi perintah** di `anjv2.php:98` (gagal: `shell_exec` dinonaktifkan) | `error_log` |
| 2026-08-12 04:31 | `wp-slgnup.gz` (mtime internal gzip) | metadata gzip |
| 2026-08-12 09:40 | `admin.php` masuk `admin.zip` | timestamp zip |
| 2026-08-31 15:36 | `item.php` (dropper) di-zip | timestamp zip |
| 2026-09-07 11:40 / 11:45 | `wp-blog-header.php` (`PHP Encode by zeura.com`) dan `hiroshi.php` di-zip | timestamp zip |
| 2026-09-15 17:28 | html verifikasi Google di-zip | timestamp zip |
| 2026-09-22 14:44 / 15:00 / 15:05 | `shopsdetails.php`, `goodsdetails.php`, `index.php` di-zip | timestamp zip |
| 2026-09-28 13:39 & 13:43 | **Akses langsung** ke `mu-plugins/sso-loader.php` (root **dan** sragen) | `mu-plugins/error_log` |
| 2026-09-29 02:41 / 02:42 | Akses langsung `sso-loader.php` (sragen) | idem |
| 2026-10-03 01:55 | Akses langsung `sso-loader.php` (sragen) | idem |
| 2026-10-08 06:54 | Fatal `plugin_basename()` — akses langsung `wp-file-manager/file_folder_manager.php` (root) | `wp-file-manager/error_log` |
| 2026-10-09 22:07 | Akses langsung `sso-loader.php` (root & sragen) | idem |
| 2026-10-12 00:14 | Entri `error_log` terakhir | `error_log` |
| **2026-10-12 11:34–12:21** | **Jendela penyalinan massal** (mtime seluruh file berada di sini) | mtime |
| **2026-10-12 21:21** | `index.html` defacement (≈9 jam **setelah** jendela penyalinan) | mtime |
| 2026-10-12 23:47 | `wp-content/wp-content.txt` (dump `tree` volume Data) | mtime |

**Distribusi mtime file:** **100%** file (38.906) bertanggal **2026-10-12**; **0** file dengan mtime di luar itu. Artinya: mtime **tidak dapat** dipakai untuk membedakan kapan tiap situs dikompromikan.
**Anomali yang perlu dijelaskan terpisah:** `index.html` bermtime 21:21 (di luar jendela salinan) dan `wp-content.txt` 23:47.
`cgi-bin/` (root) = direktori **kosong**, mtime 2026-01-05; `wp-compat/` (sragen) = kosong, mtime 11-Sep-2026 (keduanya adalah **direktori**, karena itu tidak muncul di distribusi mtime file).

### 6.2 Peta IoC per instalasi (berbasis hash — bukan dugaan)

| Instalasi | Versi WP | File | Ukuran | IoC yang terkonfirmasi |
|---|---|---|---|---|
| **root** | 7.1.3 | 7.276 | 786,4 MB | `index.html` defacement; `wp-slgnup.gz` (SEO cloaking + C2); `themes/wp_edvl/404.php` (RCE, key `TTTDowpf`); `info.php`; `mu-plugins/sso-loader.php`; `error_log` + `mu-plugins/error_log` |
| **cibitung** | 6.9.1 | 12.156 | 616,2 MB | `plugins/wp-wordfence-security/{proc,pcntl}.php` (RCE); `info.php`; `.htaccess` "Allow from all" ×2; `wp-loadss.php`; plugin ter-rename `ioyntkq`; stub `wp-db-optimizer`; `lvcibfqxpseo` (kosong) |
| **kebumen** | 6.9.9 | 12.029 | 551,5 MB | cluster SEO doorway `prestasi/` (`index.php`, `bb.php`, `home.php`, `lol.txt`, `urls.txt`, `site1–4.xml`, `robots.txt`, `sitemap.xml`). Tidak ada webshell RCE yang ditemukan di instalasi ini |
| **sragen** | 7.1.3 | 7.445 | 393,0 MB | **`archive.php` ×2 (phpFileManager, password kosong)**; `admin.zip`/`admin.php`; `themes/uvisyfyj/index.php` (upload shell); `themes/wp_zqyq/404.php` (RCE, key `VKl1tNpC`); `goodsdetails.php` + `shopsdetails.php` + 7 arsip dropper; `info.php`; `mu-plugins/sso-loader.php` |

**Kesamaan lintas instalasi (hash identik):**

| Bukti | Menyebar di | Interpretasi |
|---|---|---|
| `sso-loader.php` (`337fee21…`, 2.861 B) | root + sragen | **Deploy yang sama**, identik byte — indikasi penanganan oleh aktor/toolkit yang sama atau lingkungan yang di-clone |
| `info.php` (`075368f9…`, 19 B) | root + sragen | Idem (bukti lemah bila berdiri sendiri, menguat bersama `sso-loader.php`) |
| 1.984 file identik | root + sragen | Konsisten dengan **versi WP sama (7.1.3)** — bukan bukti kompromi |
| 5.648 file identik | cibitung + kebumen | Konsisten dengan **set plugin/tema yang sama** (elementor dkk.), bukan bukti kompromi |
| `wp_edvl/404.php` vs `wp_zqyq/404.php` | root **hanya** / sragen **hanya** | Ukuran & struktur sama, **gate dan kunci XOR berbeda** → dibangun dari generator yang sama, **ditanam terpisah per situs** |
| `uvisyfyj` (gate `admin888`) vs `404.php` (gate `admin0978`) | sragen | **Konvensi gate yang serupa** + evasi konkatenasi yang sama → **dugaan** aktor yang sama, tahap berbeda |

**Hal yang TIDAK dapat disimpulkan:** tidak ada bukti bahwa keempat instalasi dikompromikan melalui **satu jalur masuk yang sama**. Pola IoC justru **terpisah per situs** (toolkit berbeda di tiap instalasi), sehingga kemungkinan besar ada beberapa **vektor/tahap yang berbeda**. Penentuan jalur masuk awal (initial access) **belum dapat ditetapkan** dari artefak statis saja — memerlukan log akses web server.

---

## 7. Tugas E — Daftar query audit database (BELUM dijalankan)

**Tidak ada akses DB, dump, atau kredensial yang dibuka pada tahap ini.** Berikut daftar query **read-only** (SELECT saja) untuk dijalankan nanti pada **salinan/staging atau DB read-only**, dijalankan terpisah untuk **masing-masing dari 4 instalasi**.

Sesuaikan prefiks tabel (default `wp_`).

**E1 — Akun administrator (deteksi akun tak dikenal)**
```sql
SELECT u.ID, u.user_login, u.user_email, u.user_registered, u.user_status, m.meta_value AS caps
FROM wp_users u
JOIN wp_usermeta m ON m.user_id = u.ID AND m.meta_key = 'wp_capabilities'
WHERE m.meta_value LIKE '%administrator%'
ORDER BY u.user_registered;
```
```sql
-- akun dengan level tinggi
SELECT u.user_login, u.user_registered, m.meta_value
FROM wp_users u JOIN wp_usermeta m ON m.user_id=u.ID
WHERE m.meta_key IN ('wp_user_level','wp_capabilities','session_tokens')
ORDER BY u.user_registered DESC LIMIT 100;
```
```sql
SELECT user_login, user_email, user_registered FROM wp_users WHERE user_registered > '2026-01-01' ORDER BY user_registered;
```

**E2 — Plugin & konfigurasi aktif**
```sql
SELECT option_name, option_value FROM wp_options WHERE option_name IN ('active_plugins','active_sitewide_plugins');
SELECT option_name, LENGTH(option_value) AS len, autoload FROM wp_options
WHERE option_name IN ('siteurl','home','template','stylesheet','users_can_register','default_role','admin_email','WPLANG');
SELECT option_name, LENGTH(option_value) AS len FROM wp_options WHERE option_name='cron';  -- tinjau hook tak dikenal
```

**E3 — Opsi `sso_token` (TANPA menampilkan nilainya)**
```sql
SELECT option_id, option_name, LENGTH(option_value) AS len, autoload
FROM wp_options WHERE option_name = 'sso_token';
-- Cek apakah token lemah, tanpa membocorkan nilainya:
SELECT option_name,
       (option_value = '')                                        AS kosong,
       (LENGTH(option_value) < 16)                                AS terlalu_pendek,
       (option_value IN ('admin','password','123456','sso','token')) AS pola_umum
FROM wp_options WHERE option_name = 'sso_token';
-- Jika token berupa base64(sha256(nonce.salt)) dengan nilai diketahui,
-- bandingkan hash kandidat (bukan nilai mentah):
SELECT option_name FROM wp_options
WHERE option_name='sso_token'
  AND option_value = TO_BASE64(UNHEX(SHA2(CONCAT('coba-salt','-e',UNIX_TIMESTAMP()),256)));
```

**E4 — Indikasi SEO spam / doorway**
```sql
SELECT ID, post_type, post_status, post_name, post_date, post_modified, post_author
FROM wp_posts
WHERE post_status = 'publish' AND post_type IN ('post','page')
  AND post_date > '2026-08-01' ORDER BY post_date DESC LIMIT 200;
```
```sql
SELECT ID, post_title, post_name FROM wp_posts
WHERE post_content LIKE '%<script%' OR post_content LIKE '%<iframe%'
   OR post_content LIKE '%d41d8cd98f00b204e9800998ecf8427e%'
   OR post_content LIKE '%phpFileManager%' OR post_content LIKE '%rehan23%';
```
```sql
SELECT ID, post_title, post_name FROM wp_posts
WHERE post_content LIKE '%[url=%' OR post_content LIKE '%オンライン%' OR post_content LIKE '%카지노%';
```
```sql
SELECT meta_id, post_id, meta_key, LENGTH(meta_value) FROM wp_postmeta
WHERE meta_key IN ('_wp_page_template','_elementor_data') AND LENGTH(meta_value) > 100000 LIMIT 50;
```

**E5 — Opsi mencurigakan / besar (kandidat injeksi)**
```sql
SELECT option_name, LENGTH(option_value) AS len, autoload FROM wp_options
WHERE autoload='yes' AND LENGTH(option_value) > 10000 ORDER BY len DESC LIMIT 50;
```
```sql
SELECT option_name, LENGTH(option_value) AS len FROM wp_options
WHERE option_name LIKE '%sso%' OR option_name LIKE '%seo%' OR option_name LIKE '%rank%'
   OR option_name LIKE '%spam%' OR option_name LIKE '%gateway%' OR option_name LIKE '%cloa%'
   OR option_name LIKE '%idx%' ORDER BY len DESC;
```

**E6 — Sisa artefak & pembersihan**
```sql
SELECT option_name FROM wp_options WHERE option_name LIKE '%ai1wm%' OR option_name LIKE '%_transient_%sso%';
SELECT COUNT(*) AS jumlah_admin FROM wp_users u JOIN wp_usermeta m ON m.user_id=u.ID
WHERE m.meta_key='wp_capabilities' AND m.meta_value LIKE '%administrator%';
```

**Catatan penting saat eksekusi nanti:** semua query di atas **SELECT-only**; jangan `UPDATE/DELETE` pada DB produksi; jangan menampilkan nilai `sso_token`, hash user, atau `session_tokens` ke dalam laporan (cukup properti turunan seperti panjang/pola/kecocokan boolean seperti contoh E3).

---

## 8. Tindakan berikutnya & prasyaratnya

| # | Tindakan | Prasyarat |
|---|---|---|
| 1 | **Putuskan mitigasi risiko integritas bukti** (Defender terbukti mengarantina berkas malware: ThreatID 2147761377). Opsi: *exclusion* sementara untuk `D:\smk10november`, atau gandakan folder bukti ke media terpisah | **Persetujuan Saka** (mengubah konfigurasi AV saat ini dilarang) |
| 2 | Verifikasi file inti vs paket resmi **WordPress 7.1.3** (`wp-admin/`, `wp-includes/`) — memakai manifest SHA-256 yang sudah siap | **Paket WP 7.1.3 resmi harus tersedia lokal** (saat ini **BLOCKED**; tidak ada unduhan yang dilakukan) |
| 3 | Tuntaskan dekode payload terdalam (`goodsdetails.php`, `hiroshi.php`, `item.php`, `index.php`) untuk identifikasi keluarga malware | Tidak ada (dapat dilakukan di memori) |
| 4 | **Log akses HTTP/web server** untuk 09-Agu-2026 23:31, 28-Sep & 09-Okt-2026 (akses `sso-loader.php`), dan periode penyebaran dropper (31-Agu s/d 22-Sep-2026). Cari `anjv2.php`, `sso-loader.php`, `archive.php`, `admin.php` | Data log dari penyedia hosting — **membutuhkan akses Rumahweb (OTP)** → **di luar izin saat ini** |
| 5 | Audit database read-only dengan query §7 | **Salinan/staging DB atau DB read-only** + persetujuan kredensial |
| 6 | Bandingkan hash plugin terhadap rilis resmi (`ioyntkq`/protect-uploads, `wp-db-optimizer`, `custom-codes`, `super-easy-maintenance-mode`, `all-in-one-wp-migration-unlimited-6.77.1`) | Bahan pembanding resmi lokal — **belum tersedia** |
| 7 | Klarifikasi artefak persiapan salinan: `wp-content/wp-content.txt` (dump `tree`), `wp-loadss.php`, `kalender-akademik/index.php`, `performance-tweaks.php` (0 B), `.tmb`/`.well-known` (0 B) | **Konfirmasi dari pihak yang menyalin** apakah ini artefak mereka |
| 8 | Verifikasi apakah `sso-loader.php` adalah plugin SSO pihak ketiga resmi yang memang dipasang, atau penanaman | Nilai/kemampuan `sso_token` (via E3) + riwayat instalasi |

---

## 9. Keterbatasan akses & berkas yang tidak tersedia

**Keterbatasan:**
- Tidak ada akses jaringan/DNS/hosting/DB/kredensial (sesuai batasan). Karena itu **tidak ada** pemeriksaan reputasi C2, tidak ada pemeriksaan versi plugin terhadap rilis resmi, tidak ada log akses web server.
- Tidak ada paket WordPress resmi lokal → **verifikasi file inti belum dapat dilakukan** (§8 item 2).
- Tree-walk (`find`) pada volume ini kini sangat lambat (indikasi kuat scanning Antivirus real-time); beberapa perintah full-tree karena itu dibatasi cakupannya dan dijalankan dengan strategi inkremental.
- Semua analisis **statis**; tidak ada payload yang dieksekusi, sehingga **perilaku runtime** (mis. koneksi C2 nyata, efek samping payload terdalam) tidak dapat diamati.

**Berkas yang dirujuk tetapi tidak tersedia pada salinan:**
`anjv2.php`; `wp-compat/wp-compat.php`; `hiroshi.php` (hanya dalam `hiroshi.zip`); `item.php` malware (hanya dalam `item.zip`); `admin.php` (hanya dalam `admin.zip`); berkas pada `/tmp/phpXXXXXX` (hanya jejak di log); paket WordPress 7.1.3 resmi; log akses web server.

---

## 10. Kepatuhan batasan keamanan

| Batasan | Status |
|---|---|
| 1. `D:\smk10november` read-only | **Dipatuhi** — 0 entri berubah hari ini; 38.906 file tetap |
| 2. Tidak membuat/mengubah/menghapus/memindah/mengganti nama di direktori bukti | **Dipatuhi** |
| 3. Tidak menjalankan berkas dari direktori bukti | **Dipatuhi** — semua analisis statis (Python), tidak ada PHP/EXE/shell yang dijalankan |
| 4. Tidak mengakses jaringan/DNS/URL penyerang/hosting/DB/kredensial | **Dipatuhi** — domain C2 hanya ditampilkan sebagai string hasil dekode, **tidak** dihubungi |
| 5. Tidak mengubah konfigurasi Windows/Defender/PHP/WordPress | **Dipatuhi** |
| 6. Tidak membaca/menampilkan rahasia | **Dipatuhi** — `wp-config.php` hanya hash+ukuran; pesan log di-mask; `sso_token` **tidak dibaca sama sekali**; `auth_pass` dilaporkan sebagai **properti** (`md5 string kosong`), bukan kredensial milik seseorang |
| 7. Laporan hanya di `C:\Users\sakha\forensik-smk10november\` | **Dipatuhi** (direktori sudah ada; tidak membuat direktori baru) |
| 8. Tidak mengunduh paket/dependensi | **Dipatuhi** — dinyatakan **belum dapat diverifikasi** |

**Berhenti setelah analisis. Tidak ada remediasi atau perubahan yang dilakukan.**
