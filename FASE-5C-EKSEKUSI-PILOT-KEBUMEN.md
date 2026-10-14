# FASE 5C — EKSEKUSI PEMULIHAN PILOT KEBUMEN (laporan akhir)

**Tanggal:** 14 Oktober 2026 · **Situs:** Kebumen · **Status:** pemulihan konten ke staging **SELESAI**, seluruh V LULUS.
**Cakupan:** hanya pilot Kebumen. Tambun/Cibitung/Sragen **tidak disentuh**. Produksi **tidak disentuh**. `goly1264_ams` **tidak disentuh**.

---

## 1. Path staging & database yang diverifikasi

| Item | Nilai (terverifikasi) |
|---|---|
| Direktori staging | `C:\xampp\htdocs\staging\kebumen` (satu-satunya di `htdocs\staging\`) |
| Berkas konfigurasi | `wp-config.php` ada; prefix tabel **`wp_`** |
| Database | **`smk10_staging_kebumen`** (`DB_HOST=127.0.0.1`, user DB `smk10_staging`) |
| URL situs | `http://localhost/staging/kebumen` (siteurl = home = sama) |
| Nama situs | `SMK 10 November Kebumen (STAGING pemulihan)` |
| WordPress / PHP / MariaDB / WP-CLI | 7.0.4 / 8.2.12 / 10.4.32 / 2.12.0 |
| Bukti bukan produksi | `http://localhost/...`; daftar DB hanya `goly1264_ams`, `smk10_staging_kebumen`, sistem |
| Tema aktif | **astra** 4.14.0 · Plugin aktif: **elementor** 4.3.5 |

**Cadangan (checkpoint) yang dibuat sebelum operasi yang mengubah data:**

| Checkpoint | Ukuran | SHA-256 (16 awal) |
|---|---:|---|
| `checkpoint-01-pre-import.sql` | 137.914 B | `90f95e9d…` |
| `checkpoint-02-post-import.sql` | 1.318.237 B | `674e8695…` |
| `checkpoint-03-post-recovery.sql` | 2.561.769 B | `1a01263b…` |

Empat dump sumber `D:\backup-database-smk10november\` **tetap utuh** (0 berkas berubah).

---

## 2. Tahapan yang benar-benar dieksekusi

1. Verifikasi lingkungan + checkpoint-01.
2. **Penyalinan 122 media** dari salinan bukti ke `uploads` staging (struktur `tahun/bulan` dipertahankan).
3. Pemasangan plugin resmi `wordpress-importer` (wajib untuk `wp import`), lalu **impor WXR utama** (`--authors=skip`).
4. Perbaikan metadata lampiran yang diduplikasi importer (`-1`) + pembuatan record lampiran **SVG**.
5. **Pembangunan ulang menu** via WP-CLI (importer melewati item bertipe halaman).
6. **Pemulihan mutu data (K-8)**: kit style, menu classes, screenshot.
7. **Penggantian URL** domain produksi (dry-run → terapkan).
8. **Regenerasi metadata media**, setelan permalink + `.htaccess`, halaman depan statis.
9. **V-01…V-11** + uji tampilan browser.

---

## 3. Media: 122 berkas disalin + verifikasi hash

| Ukuran | Hasil |
|---|---|
| Entri manifes | **122** |
| Tersalin & **COCOK (ukuran + SHA-256)** | **122 (100 %)** |
| Tidak cocok / gagal / ditolak jenis | 0 / 0 / 0 |
| Total byte tersalin | **28.086.567 B** |
| Berkas kode (`.php/.phtml/.js`) disalin | **0** |
| Log per berkas | `recovery-work/kebumen/export/_5c-media-copy-log.tsv`, `_5b-media-verify.tsv` |
| Cek akhir `_wp_attached_file` | 122 nilai, **122 berkas ada di disk** |

**Catatan duplikasi importer:** WordPress Importer membuat salinan `-1` untuk 120 lampiran (karena berkas sudah ada) dan menunjuk `_wp_attached_file` ke salinan itu. Saya **mengarahkan ulang seluruh 121 nilai ke berkas asli yang sudah diverifikasi** (§6) sehingga berkas yang dirujuk adalah berkas ber-hash terverifikasi. Salinan `-1` **masih ada di disk** (tidak dihapus — penghapusan berkas di luar cakupan izin); jumlah berkas di `uploads` = 702.

**27 referensi media yang tidak ditemukan:** dicatat sebagai masalah sementara, **tidak** dibuat penggantinya dan lokasinya tidak ditebak. Fakta uji: dari **217 rujukan media yang benar-benar dirender** pada 18 halaman/berita, **0 gagal dimuat** (V-11) — artinya berkas hilang itu tidak muncul pada halaman yang diuji.

---

## 4. Hasil impor WXR

| Ukuran | Hasil |
|---|---|
| Item diproses | **174 dari 174** |
| **Berhasil dibuat** | **173** (122 lampiran, 19 item menu, 11 halaman, 10 berita, 8 templat Elementor, 2 global styles, 1 navigasi) |
| **Dilewati** | **1** — `wpdmpro` ("Kalender Akademik" paket Download Manager): *Invalid post type wpdmpro* karena plugin tidak dipasang (keputusan yang disetujui) |
| **Gagal** | **1** — lampiran **SVG** `Kalender-Pendidikan-Minggu-Efektif-Senov_00001.svg`: WordPress menolak impor SVG (fitur keamanan). **Ditangani terpisah**: record lampiran dibuat (id **1949**) + `_wp_attached_file` + `_wp_attachment_metadata` + `_elementor_inline_svg` (456.918 karakter = sesuai ukuran berkas) |
| Akun pengguna setelah impor | **1** (tidak ada akun baru; penulis dipetakan ke admin staging) |

**Temuan kendala importer (dokumentasi):** `nav_menu_item` (19) **tidak dibuat** pada impor pertama karena WXR tidak memuat blok `<wp:term>` sehingga term `nav_menu` belum ada — importer hanya membuat 3 item bertipe `custom` dan melewatkan 16 item bertipe halaman **tanpa pesan kesalahan**. Diselesaikan dengan WXR khusus menu (lihat §5). Untuk tiga situs berikutnya, `convert_dump_to_wxr.py` harus ditambah emisi `<wp:term>`.

---

## 5. Status pemetaan ID lama → ID baru

Metode: kunci **(post_type, post_name)** + verifikasi **post_date**, **judul**, **MD5(post_content)**, dan untuk lampiran **`_wp_attached_file` cocok manifes**. Ambiguitas **tidak** ditebak.

| Ukuran | Hasil |
|---|---|
| Item lama | 174 |
| Dipetakan otomatis (verifikasi berlapis) | **153** (151 dengan tanggal+judul+MD5; 0 ambigu; 0 verifikasi lemah) |
| Tidak ditemukan saat otomatis | 21 = 19 item menu + `wpdmpro` + SVG |
| Diselesaikan manual terverifikasi | **19 item menu** (dipetakan dari keluaran `build_menu.py`: 20→1957, 22→1958, 23→1962, 749→1950, 754→1951, 780→1955, 781→1967, 834→1968, 835→1956, 908→1959, 909→1960, 910→1961, 967→1963, 968→1964, 973→1965, 974→1966, 975→1953, 976→1954, 977→1952) dan **SVG** 1920→**1949** |
| Tidak dipetakan (sengaja) | `wpdmpro` (tidak diimpor) |
| Berkas peta | `recovery-work/staging/id-mapping.tsv` + `id-mapping-menu.tsv` |

**Menu:** 19 item dibangun ulang secara deterministik (Main Menu 12 + Footer Menu 7, hierarki induk-anak benar, tautan kustom `#`/`#akademik`), relasi taksonomi 19 ✓.

---

## 6. Metadata K-8 yang dipulihkan / dilewati / gagal

| Kelompok | Jml | Tindakan | Hasil |
|---|---:|---|---|
| `_elementor_page_settings` (**kit style: warna & tipografi**) | 14 | **dipulihkan** | 14 diterapkan; kit aktif id 5 dengan **18 elemen** array → terbaca benar |
| `_menu_item_classes` | 19 | dipulihkan | 19 diterapkan (array kelas) |
| `_elementor_screenshot` | 1 | dipulihkan | 1 diterapkan |
| `_elementor_css`, `_elementor_page_assets`, `_elementor_conditions` | 41 | **dibuang** (cache/turunan) | tidak diterapkan (diregenerasi Elementor) |
| `_wp_attachment_metadata` | 121 | **diregenerasi** via `wp media regenerate` | **120 dari 121** berhasil; 1 "gagal" = berkas **video/mp4** (bukan gambar, tidak punya turunan) — bukan kerusakan |
| `__wpdm_files`, `__wpdm_access` | 2 | **ditunda** | plugin Download Manager tidak dipasang |
| 57 rujukan ID kit "Skoola" tanpa target | — | **dibiarkan tidak terpetakan** | didokumentasikan, tidak ditebak |

Validasi: seluruh nilai PHP-serialized **divalidasi parser aman (tanpa `unserialize` objek)** sebelum diterapkan; **0 objek** ditemukan; diterapkan sebagai **array PHP** melalui `update_post_meta()` sehingga **tidak ada serialisasi ganda**.

---

## 7. Penggantian URL produksi — dry-run lalu terapkan

**Dry-run** (`wp search-replace --all-tables-with-prefix --precise --dry-run`):

| Pola | Hasil dry-run |
|---|---|
| `https://smk10novemberkebumen.sch.id` | **22** penggantian (postmeta 3 + post_content 19) |
| `http://smk10novemberkebumen.sch.id` | 0 |
| `//smk10novemberkebumen.sch.id` | 22 (himpunan yang sama dengan pola pertama — tumpang tindih) |
| `https:\/\/smk10novemberkebumen.sch.id` (bentuk ter-escape JSON Elementor) | **18** penggantian (postmeta) |
| `http:\/\/smk10novemberkebumen.sch.id` | 0 |

**Penerapan:** 22 + 18 = **40 penggantian**; pola lain 0.
**Verifikasi:** sisa kemunculan **0** untuk kelima pola. `JSON_VALID` `_elementor_data` tetap 19 valid (3 invalid = revisi sisa, lihat §10).
**Alamat email** `info@smk10novemberkebumen.sch.id` (3 baris) **sengaja dipertahankan** — itu alamat kontak sekolah, bukan aset.

---

## 8. Hasil V-01 … V-11

| ID | Pemeriksaan | Status | Keterangan |
|---|---|---|---|
| V-01 | Jumlah objek per tipe | **LULUS** | WXR {page 11, post 10, attachment 122, nav_menu_item 19, elementor_library 8, wp_global_styles 2, wp_navigation 1} — semua terpenuhi |
| V-02 | Judul + slug halaman/berita | **LULUS** | 23 halaman/berita publish; tidak ada yang hilang |
| V-03 | Lampiran & berkasnya | **LULUS** | `_wp_attached_file` = 122; berkas ada = 122 |
| V-04 | `_thumbnail_id` → lampiran sah | **LULUS** | 10 thumbnail, 0 tidak sah |
| V-05 | Menu (19 item, 2 menu, 19 relasi) | **LULUS** | item 19, menu 2, relasi 19 |
| V-06 | URL domain produksi (berskema) | **LULUS** | sisa URL = 0; email dipertahankan = 3 baris |
| V-07 | Data Elementor + kit style | **LULUS** | JSON_VALID: 19 valid / 3 invalid (revisi); kit 18 elemen |
| V-08 | Halaman inti HTTP 200 | **LULUS** | `/`, `/home`, `/berita-artikel`, `/kontak`, `/sejarah`, `/visi-misi`, `/prestasi`, `/fasilitas`, `/pendaftaran`, `/jurusan-konsentrasi-keahlian`, `/struktur-organisasi`, `/kalender-akademik` — semua 200 |
| V-09 | Bebas IoC (opsi & meta) | **LULUS** | opsi 0, meta 0 |
| V-10 | HTML tampil bebas aset produksi | **LULUS** | URL produksi = 0 (alamat email dikecualikan) |
| V-11 (tambahan) | Media tampil dapat dimuat | **LULUS** | **217 rujukan diperiksa, 0 gagal** |

---

## 9. Halaman tampil, media rusak, fitur bermasalah

**Berhasil ditampilkan (uji HTTP + browser):** seluruh 12 halaman + berita index + berita individual (200), termasuk halaman **Home** yang merender **konten Elementor** (hero + semboyan "Cerdas Dalam Berpikir / Jujur Dalam Bertindak") dan **menu navigasi lengkap 12 tautan** hasil pemulihan. Bukti visual: `…\cookies\tmp\shot.png` (tangkapan halaman depan) dan tangkapan berita index.

**Media rusak:** **0** dari 217 rujukan media yang dirender (V-11). Yang masih belum tersedia: **27 berkas** yang dirujuk di suatu tempat dalam data (19 asli + 8 turunan) tetapi tidak ada di salinan bukti dan tidak punya record lampiran — **tidak muncul** pada halaman yang diuji.

**Fitur yang masih bermasalah / berbeda:**
1. **Elementor Pro belum dipasang** → 4 templat Theme Builder (Header, Footer, Single News, Archive News) tidak aktif dan 15 widget Pro di 3 halaman tidak dirender. Halaman & berita tetap utuh; tata letak memakai bawaan **Astra**. (Rincian: Fase 5B §G.7.)
2. **Halaman "Kalender Akademik" versi paket Download Manager** tidak ikut (plugin tidak dipasang). Halaman "Kalender Akademik" biasa **ada** ✓.
3. **Warna/typografi kit "Skoola"** sudah diterapkan sebagai metadata, tetapi **CSS Elementor dibangkitkan ulang** saat halaman dibuka — tampilan akhir bisa berbeda halus dari situs lama.
4. 3 revisi lama menyimpan `_elementor_data` yang belum diperbaiki (JSON invalid) — **tidak berpengaruh** pada tampilan; bisa dibersihkan kapan saja.

---

## 10. Kesalahan, perubahan database, checkpoint, langkah lanjutan

### 10.1 Kesalahan yang ditemukan & diperbaiki (transparansi)
1. **BUG BESAR MILIK SAYA — `unescape` menghapus kutip ganda.** Fungsi `unescape` di `tools/fase4c6.py` memakai aturan gaya MySQL "doubling" (`""` → `"`), padahal dump ini meng-escape kutip dengan backslash. Akibatnya nilai JSON yang sah rusak: `"size":"","alt":""` menjadi `"size":","alt":""`. **Bukti:** pola utuh muncul **8.820×** di dump, pola rusak **0×**; `JSON_VALID` di staging **0 untuk 15 dari 18** nilai. **Perbaikan:** aturan itu dihapus + perbaikan diterapkan ke data (22 nilai: 3 `post_content`, 1 `post_title`, 18 `_elementor_data`) → `JSON_VALID` menjadi **19 valid**.
   **Dampak pada fase sebelumnya (perlu diketahui):** artefak/laporan Fase 4C.2/4C.3/4D yang memuat nilai dari tabel dump dapat terpengaruh pada **string yang mengandung `""`** — khususnya kesimpulan "9 nilai serialized INVALID = artefak escape ganda" di Fase 4C.3: **penyebab sebenarnya adalah bug ini**, bukan escape ganda, dan perlu direvisi. Kesimpulan struktural lain (jumlah tabel, akun, IoC) tidak bergantung pada karakter `""`.
2. **`update_post_meta()` menghapus backslash JSON** (karena `wp_unslash`) → nilai harus ditulis dengan `wp_slash()`. Diterapkan.
3. Beberapa bug skrip tambahan yang saya temukan dan perbaiki: `split("_")[-1]` salah memotong nama tabel (`term_taxonomy`), guard `pid in items` membuang metadata menu (dump menaruh `postmeta` sebelum `posts`), `--skip-column-names` bukan flag WP-CLI (menyebabkan perbandingan "semua berbeda" palsu), variabel `man` dipakai sebelum diisi, dan regex path menyerap `]]` (melaporkan 126 berkas hilang padahal 27).

### 10.2 Perubahan yang dilakukan (semua hanya di staging)
- DB `smk10_staging_kebumen`: impor 174 item, 121 `_wp_attached_file`, 1 record SVG + metanya, K-8 (34 meta), perbaikan mutu (22 nilai), 40 penggantian URL, opsi (`permalink_structure`, `show_on_front`, `page_on_front`, `page_for_front`), 19 item menu.
- Berkas: 122 media disalin (28.086.567 B), `.htaccess` dibuat, 4 revisi baru (efek `wp_update_post`), plugin `wordpress-importer` dipasang lalu **dinonaktifkan**.
- **Tidak ada** perubahan pada dump sumber, salinan bukti, `goly1264_ams`, maupun produksi.

### 10.3 Langkah lanjutan yang disarankan
1. **Keputusan K-10** (27 media hilang): terima sebagai gambar kosong, cari di arsip backup hosting `cgi-bin.zip` (perlu persetujuan + prosedur ekstraksi terpandu), atau ganti.
2. **Keputusan Elementor Pro**: pasang bila sekolah punya lisensi, atau ganti widget Pro (posts/testimonial-carousel/form) dengan padanan gratis.
3. **Bersihkan salinan `-1`** di `uploads` (702 berkas menjadi ±580) — perlu izin penghapusan.
4. Bersihkan 3 revisi ber-JSON invalid.
5. **Sebelum replikasi ke Tambun/Cibitung/Sragen:** (a) tambahkan emisi `<wp:term>` pada `convert_dump_to_wxr.py`, (b) perbaiki `unescape` sudah dilakukan — regenerasi WXR wajib, (c) perbaiki dokumentasi Fase 4C.3 sesuai §10.1.
6. Bila tampilan sudah disetujui: ganti basis URL staging ke domain asli (atau lakukan replikasi ke hosting) — **butuh keputusan dan jalur aman terpisah**.

### 10.4 Tempat pemulihan (rollback)
- `checkpoint-01-pre-import.sql` → keadaan sebelum impor
- `checkpoint-02-post-import.sql` → sesudah impor (sebelum perbaikan menu/lampiran)
- `checkpoint-03-post-recovery.sql` → keadaan akhir saat ini
- Cara memulihkan: `mysql -u root smk10_staging_kebumen < checkpoint-XX-*.sql` (tidak dijalankan sekarang).

---

## 11. Lampiran berkas

| Berkas | Isi |
|---|---|
| `recovery-work/FASE-5C-EKSEKUSI-PILOT-KEBUMEN.md` | laporan ini |
| `recovery-work/staging/checkpoints/*.sql` | 3 checkpoint rollback |
| `recovery-work/staging/logs/{import,import-menu,verify_v}.log|txt` | log impor & verifikasi |
| `recovery-work/staging/id-mapping.tsv`, `id-mapping-menu.tsv` | peta ID lama→baru |
| `recovery-work/kebumen/export/_5c-media-copy-log.tsv`, `_5b-media-verify.tsv` | verifikasi hash media |
| `recovery-work/kebumen/export/kebumen-menu.wxr` | WXR khusus menu (dengan `<wp:term>`) |
| `recovery-work/tools/{copy_media,fix_attachments,build_menu,apply_k8,repair_escape,url_replace,verify_v,menu_wxr,map_ids}.py` | alat eksekusi & verifikasi |
