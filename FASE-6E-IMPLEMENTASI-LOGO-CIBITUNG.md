# FASE 6E — IMPLEMENTASI LOGO CIBITUNG PADA WORDPRESS STAGING

**Tanggal:** 16 Oktober 2026 · **Status:** selesai — logo terpasang & terverifikasi visual
**Lingkup:** **hanya** staging Cibitung. Tidak ada situs, database, konfigurasi, atau berkas lain yang diubah.

---

## 1. Tujuan dan ruang lingkup
Memasang **logo Cibitung yang disiapkan pengguna** pada WordPress staging Cibitung, tanpa membuat/mengedit logo, tanpa memakai logo Tambun lama, dan tanpa perubahan di luar pemasangan logo. Cakupan dikerjakan bertahap: pemeriksaan read-only → checkpoint → pemasangan → verifikasi → laporan.

## 2. Lokasi sumber logo yang diperiksa
- Path yang disebut pengguna: `C:\Users\sakha\Downloads\logo-SMK-10-November-Cibitung` → **ternyata berkas**, bukan folder: **`C:\Users\sakha\Downloads\logo-SMK-10-November-Cibitung.png`** (folder dengan nama itu **tidak ada**).
- Pencarian nama serupa di `Downloads` (maks 2–4 level) menemukan **tepat satu** kandidat tersebut (tidak ada alternatif lain, tidak ada subfolder).
- Sumber tidak diambil dari internet maupun dari situs produksi.

## 3. Berkas yang dipilih dan metadata terverifikasi

| Properti | Nilai |
|---|---|
| Berkas | `logo-SMK-10-November-Cibitung.png` |
| Ukuran | **261.615 B** (255,5 KB) |
| Format | **PNG 8-bit, color_type 6 (RGBA — transparan)** |
| Dimensi | **740 × 740 px** (rasio 1,000; 0,55 MP) |
| SHA-256 | `d55fcb09c3b74503c25633a324260babd4f4a548773d4ffc075fa7963aee17cb` |
| Isi (diverifikasi visual) | Lambang perisai merah; teks kuning **"SMK 10 NOVEMBER"** (atas) dan **"CIBITUNG – BEKASI"** (bawah) → **identitas Cibitung, bukan Tambun** |

Verifikasi silang: berkas di `Downloads` **identik byte** dengan berkas yang tersalin ke staging (`cmp` → identik; hash sama).

## 4. Konfirmasi sumber logo TIDAK diubah
- SHA-256 sumber **sebelum** = **sesudah** pemasangan (`d55fcb09c3b74503…`).
- Berkas sumber **masih ada** di lokasi semula (tidak dipindah/diganti nama/ditimpa), tidak diedit.

## 5. WordPress staging yang diubah
`C:\xampp\htdocs\staging\cibitung` · URL `http://localhost/staging/cibitung/`
- Direktori bukan repositori Git (`fatal: not a git repository`) → tidak ada perubahan Git yang berisiko tertimpa.
- Tema tetap **Astra**; tidak ada perubahan tema/plugin/Elementor.

## 6. Konfirmasi database staging yang digunakan (tanpa kredensial)
Dibaca dari `C:\xampp\htdocs\staging\cibitung\wp-config.php`:
**`DB_NAME = 'smk10_staging_cibitung'`** · `$table_prefix = 'wp_'` · `siteurl = http://localhost/staging/cibitung`.
Seluruh operasi database pada fase ini ditargetkan eksplisit ke database itu; **tidak ada koneksi/kueri ke database lain**. Kredensial tidak dicetak di laporan ini.

## 7. Checkpoint sebelum perubahan
| Aset | Nilai |
|---|---|
| Checkpoint DB | `recovery-work\staging\checkpoints\checkpoint-6e-pre-logo-cibitung.sql` — **948.106 B** (SHA-256 `c26952c6f2db7c54…`), 24 tabel = 13 tabel inti + 11 tabel WPForms Lite (dari 6C) |
| Snapshot opsi | `…\staging\cibitung\backup-6e\keadaan-awal-opsi.txt` → `site_logo = TIDAK ADA (fallback teks)`, `site_icon = 0` |
| Snapshot theme mods | `backup-6e\theme_mods_astra-awal.txt` (138 B; hanya `nav_menu_locations` + `custom_css_post_id`) |
| Snapshot header | `backup-6e\header-awal.html` (188.908 B; 0 `<img class="custom-logo">`) |
| Lain-lain | `backup-6e\logo-attachment-id.txt` (ID lampiran baru) |

Tidak ada backup/dump database lain yang dibuat (lihat §13–§14).

## 8. Pengaturan logo yang diubah
| Objek | Sebelum | Sesudah |
|---|---|---|
| Media Library | 153 lampiran | **+1 → `1948` "Logo SMK 10 November Cibitung"** (`image/png`, `_wp_attached_file = 2026/10/logo-SMK-10-November-Cibitung.png`, metadata 740×740) |
| Opsi `site_logo` | **tidak ada** (fallback teks) | **`1948`** |
| Opsi `site_icon` | `0` | **tetap `0`** (favicon **di luar lingkup** fase ini) |
| `theme_mods_astra` | tanpa `custom_logo` | **tidak diubah** — mekanisme yang sudah dipakai situs ini adalah opsi `site_logo` (dibuktikan: sebelum 6C logo gambar muncul dari opsi itu; setelah dihapus di 6C, header berubah ke teks) sehingga `custom_logo` tidak perlu diset |

**Tidak diubah:** ukuran/lebar logo di pengaturan Astra (tidak diperlukan — lihat §9), menu, konten, Elementor, CSS/JS global, permalink, email, tema, plugin.
**Tidak disentuh:** lampiran lama **1818** (logo Tambun) — diverifikasi **tidak dirujuk** oleh `wp_options` (0) maupun `wp_postmeta` (0); tidak dihapus.

## 9. Hasil verifikasi desktop dan mobile
**Screenshot (sesudah pemasangan, preview pane Hermes):** header menampilkan **lambang perisai Cibitung** ("SMK 10 NOVEMBER" / "CIBITUNG – BEKASI") **berdampingan dengan judul situs**, **berukuran proporsional (≈150 px, bukan 740 px → Astra membatasi sendiri)**, **tidak terpotong, tidak terdistorsi, tidak menutupi menu**; togol menu (☰) tampil; hero "Unggul, Berkarakter, Kompeten, Kreatif, dan Inovatif" + tombol Pendaftaran + WELCOME normal.

| Uji | Hasil |
|---|---|
| Logo tampil di header (visual, viewport sempit/mobile breakpoint) | **LULUS** |
| Proporsi & integritas gambar (tidak overflow/terpotong) | **LULUS** |
| Menu tetap berfungsi (togol tampil; tautan menu di HTML: 24) | **LULUS** |
| Beranda & halaman Kontak | **200 / 200** |
| URL logo | **lokal**: `http://localhost/staging/cibitung/wp-content/uploads/2026/10/logo-SMK-10-November-Cibitung.png` (+ `-300x300`, `-150x150`) |
| Aset logo dimuat | **200** untuk ketiga ukuran; byte disajikan: **261.615 B** (asli, identik sumber) · 62.041 B (300²) · 20.703 B (150²) |
| Permintaan ke domain produksi | **0** (HTML beranda: 0; log akses: 0 pada 200 baris terakhir) |
| JSON/Elementor & warning kit | tidak berubah (warning kit tetap 13 total, **0 tambahan**) |
| Galat PHP terkait logo | **0** |
| **Viewport desktop lebar penuh** | **BELUM DIUJI** — alat preview tidak menyediakan pengaturan lebar viewport; yang teruji adalah breakpoint sempit/mobile |

Ukuran logo **tidak** disesuaikan karena hasil render sudah proporsional — sesuai batasan "sesuaikan hanya bila memang diperlukan".

## 10. Pemeriksaan console / log
- **Console JavaScript: tidak dapat diperiksa** (preview pane tidak menyediakan console; tidak ada browser MCP). Ini dicatat sebagai keterbatasan, bukan kelulusan.
- **Log Apache/PHP:** 0 entri terkait logo; tidak ada galat/warning baru akibat pemasangan ini.

## 11. Daftar berkas dan pengaturan yang berubah
**Database `smk10_staging_cibitung`:**
1. `wp_posts` +1 baris (lampiran ID 1948) dan `wp_postmeta` baris terkait (`_wp_attached_file`, `_wp_attachment_metadata`).
2. `wp_options.site_logo` → `1948`.

**Sistem berkas staging Cibitung:**
3. `wp-content/uploads/2026/10/logo-SMK-10-November-Cibitung.png` (261.615 B, identik sumber)
4. `wp-content/uploads/2026/10/logo-SMK-10-November-Cibitung-300x300.png` (62.041 B, dibuat lokal)
5. `wp-content/uploads/2026/10/logo-SMK-10-November-Cibitung-150x150.png` (20.703 B, dibuat lokal)

**Di luar itu tidak ada yang berubah** (tidak ada berkas inti WordPress, tema Astra, plugin, `wp-config.php`, `.htaccess`, CSS/Elementor).

## 12. Kendala / pengujian yang belum dilakukan
1. **Desktop lebar penuh belum diuji** (§9).
2. **Console JavaScript tidak tersedia** (§10).
3. Preview pane kadang menampilkan kanvas lama saat berpindah tab; pada fase ini render terbaca normal sehingga screenshot dapat dipakai.
4. Favicon (`site_icon`) sengaja **tidak** diubah (di luar lingkup); bila diinginkan: pakai ID 1948 sebagai favicon sementara atau berkas 512×512 khusus.
5. Logo 740×740 cukup untuk header web; untuk kebutuhan cetak/banner besar sebaiknya sekolah menyediakan master vektor (SVG).

## 13. Konfirmasi: tidak ada perubahan pada situs produksi/staging lain
- Staging **Tambun**, **Kebumen**, dan situs/staging **Sragen**: tidak disentuh.
- **Produksi** (keempat situs): tidak disentuh; tidak ada pengunduhan media dari produksi (diverifikasi: 0 permintaan).
- **Proyek AMS**: tidak disentuh.
- **Database selain `smk10_staging_cibitung`**: tidak diakses maupun diubah.

## 14. Konfirmasi eksplisit — database `goly1264_ams`
**Database `goly1264_ams` TIDAK diakses, TIDAK dibuka, TIDAK di-query, TIDAK diubah, TIDAK diimpor/diekspor, TIDAK di-backup/di-dump/di-restore, dan TIDAK digunakan dalam bentuk apa pun** selama Fase 6E. Tidak ada perintah massal atau skrip yang dijalankan terhadapnya; tidak ada konfigurasi koneksi yang diarahkan kepadanya; hanya **satu** dump dibuat, yaitu `smk10_staging_cibitung` (§7).

## 15. Konfirmasi eksplisit — berkas `admin.php` di Downloads
Berkas `C:\Users\sakha\Downloads\admin.php` **dibiarkan persis seperti kondisinya sekarang**: tidak dihapus, tidak dikarantina, tidak dipindahkan, tidak diganti nama, tidak dibuka/dijalankan, tidak diedit, dan tidak dipindai/diremediasi. Kondisi tercatat: **84.070 B, mtime 2026-10-13 12:30:18**.

---

## Prosedur rollback (bila diperlukan)
1. **Kembali ke fallback teks (ringan, tanpa DB):** `wp option delete site_logo` → header kembali menampilkan nama sekolah sebagai teks (kondisi sebelum fase ini).
2. **Rollback penuh fase ini:** `mysql -u root smk10_staging_cibitung < recovery-work\staging\checkpoints\checkpoint-6e-pre-logo-cibitung.sql` (menghapus lampiran 1948 + opsi `site_logo`).
3. Berkas lampiran 1948 di `uploads/2026/10/` boleh dibiarkan (tidak mengganggu) atau dihapus manual — **tetap tidak menyentuh lampiran 1818** dan tidak menyentuh berkas sumber di Downloads.
