# FASE 6A — PEMULIHAN STAGING TAMBUN

**Tanggal:** 15 Oktober 2026 · **Metode:** pemulihan selektif (konverter offline → importer), sama seperti pilot Kebumen
**Status:** pemulihan dasar selesai; seluruh pengujian yang dijalankan LULUS; **impor Tambun bersih tanpa insiden unduhan** (berbeda dari Cibitung)

---

## 1. Kondisi awal dan path staging

| Hal | Nilai |
|---|---|
| Staging | `C:\xampp\htdocs\staging\tambun` |
| URL | `http://localhost/staging/tambun` |
| Database | `smk10_staging_tambun` (13 tabel, prefix `wp_`) |
| WordPress | 7.0.4 id_ID (paket resmi `wordpress-id_ID.zip`) |
| Tema | Astra 4.14.0 (repo resmi) |
| Plugin | Elementor (aktif, repo resmi); Akismet & Hello Dolly (nonaktif); `wordpress-importer` (dinonaktifkan setelah dipakai) |
| Administrator | `tambun_admin` (1 akun baru; sandi acak 32 karakter di berkas lokal, tidak pernah dicetak) |
| Sebelum fase | instalasi ini **belum ada**; dibuat di fase ini |

Catatan topologi: direktori bukti akar `D:\smk10november\` (docroot) adalah **Tambun** (domain `smk10novembertambun.sch.id`), bukan situs utama.

## 2. Database sumber dan target (tanpa kredensial)

| | Sumber | Target |
|---|---|---|
| Berkas | `D:\backup-database-smk10november\smkq8953_utama1.sql` (80.550.250 B, SHA-256 `04a7b303ac978e4c…`) | `smk10_staging_tambun` |
| Prefix tabel | `smk_` | `wp_` |
| Domain lama | `smk10novembertambun.sch.id` | diganti ke URL lokal |
| Jumlah tabel | 132 (39 berisi, 93 kosong) | 13 tabel inti |
| `posts` / `postmeta` / `options` / `users` / `usermeta` | 378 / 8.213 / 346 / **41** / 573 | 141 / 509 / 171 / **1** / 3 |

## 3. Temuan keamanan dan artefak yang TIDAK dipulihkan

### 3.1 Indikator kompromi di sumber (23 temuan)
- `options` — **`sso_token`**, dan **`tema_webshell`** pada 3 baris opsi: injeksi kode pada tema.
- `usermeta` id 217 dan `users` id 6–14 dan seterusnya — penanda **`wp2shell`** (akun penyerang).
- Opsi konfigurasi auth/kredensial lain (`mailserver_pass`, dsb.) — tidak dimigrasikan.

### 3.2 Artefak dari laporan terdahulu yang DIPASTIKAN tidak ada di staging
Defacement **`index.html`**, **`wp-slgnup.gz`**, **`info.php`**, **`sso-loader.php`**, file manager palsu, tema yang dimodifikasi, plugin **`wp-file-manager`** — semuanya **tidak ditemukan** di staging. Verifikasi: pencarian nama pada seluruh direktori staging (0 hasil selain berkas inti WordPress kita sendiri), daftar plugin = `akismet`, `elementor`, `hello.php`, `index.php`, `wordpress-importer`; tema = `astra` + tema bawaan paket resmi; **0 berkas PHP/JS/HTML di `uploads`**.

### 3.3 Yang dinyatakan TIDAK dipulihkan
Seluruh **41 akun lama** dan 573 `usermeta` · seluruh tabel plugin (WooCommerce, MailPoet, WPForms/Formidable, Jetpack, Action Scheduler, dsb.) · opsi kredensial/`sso_token`/`tema_webshell` · semua cache/transien · `elementor_unfiltered_files_upload` · tema & plugin lama · `wp-config.php` lama. Staging memuat **hanya** WordPress, Astra & Elementor resmi, dan konten/media yang sudah diperiksa.

### 3.4 Pencegahan unduhan produksi (diterapkan sejak awal)
WXR Tambun **langsung disanitasi** (`tambun-staging-pilot.noattach.wxr`: 120 `<wp:attachment_url>` dikosongkan) dan impor dijalankan dengan `WP_HTTP_BLOCK_EXTERNAL` aktif sementara. Hasil: setelah impor, `uploads` berisi **tepat 120 berkas = manifest**, **0 berkas tak terduga**. Insiden unduhan produksi yang terjadi pada Cibitung (laporan Cibitung §3.3) **tidak terulang** di sini.

## 4. Inventarisasi tabel dan alasan data dipilih/ditolak

| Kelompok | Sumber | Dipilih | Alasan |
|---|---|---|---|
| `posts` | 378 | **141** | Hanya `page` (11 publish + 1 draft), `attachment` (120), `elementor_library` (9), `nav_menu_item` (19), `wp_navigation` (1). Ditolak: 101 revisi, 45 oembed, 34 customize_changeset, 34 `request`, 1 `envato_tk_import`, 1 **Auto Draft** |
| `postmeta` | 8.213 | **509** | Hanya milik post yang ikut; `_wp_attachment_metadata` diregenerasi lokal |
| `terms`/`term_taxonomy`/`term_relationships` | …/…/27 | 9/9/27 | Hanya term yang direferensikan item ikut |
| `options` | 346 | **171** | Opsi UI/identitas terpilih (§5); 282 opsi non-bawaan lainnya ditolak |
| `users`/`usermeta` | 41/573 | **1/3** | Semua akun lama mencurigakan; 1 admin baru |
| Tabel plugin berisi data | 31 | **0** | Tidak diperlukan untuk pemulihan dasar |

**Catatan penting:** **sumber Tambun tidak punya konten berita** — satu-satunya baris `post_type=post` adalah **Auto Draft** (id 1493, judul "Auto Draft") yang benar-benar dikecualikan. Post bawaan WordPress ("Hello world!") juga dihapus. Jadi jumlah post publikasi staging = **0 adalah benar**, bukan kegagalan impor.

## 5. Konten dan media yang berhasil dipulihkan

- **Halaman:** **11 publish** (+1 draft) = jumlah sumber; **semuanya HTTP 200**.
- **Lampiran/media:** **120/120** berkas hasil salinan terverifikasi (SHA-256); **119 gambar** thumbnail diregenerasi lokal.
- **Templat Elementor:** 9 item library termasuk **kit aktif 433** (`global-kit-styles`, kit "Skoola"), `header`, `footer`, `home`, `single-news`, `archive-news`, `coming-soon`, `default-kit` (dari sumber) + `default-kit-2` (dibuat otomatis oleh Elementor saat aktivasi — lihat §8).
- **Menu:** **19 item** dalam 2 menu, dibuat langsung oleh importer (berkat blok `<wp:term>`).
- **Data Elementor:** `_elementor_data` **18 nilai, 18 valid JSON, 0 invalid**; 14 `_elementor_page_settings` + 8 `_menu_item_classes` diterapkan (K-8); 4 nilai cache dibuang.
- **Opsi UI/publikasi** dari dump sumber: `date_format`, `time_format`, `site_icon`, `blogdescription`, `show_on_front`, `page_on_front`, `page_for_posts`, `elementor_cpt_support`, `elementor_google_font`, `nav_menu_options`, `site_logo`, `elementor_active_kit`, + `WPLANG=id_ID`, `timezone_string=Asia/Jakarta`.
- **URL produksi:** 25 penggantian diterapkan → **0 sisa** (`://` maupun bentuk ter-escape).

## 6. Checkpoint dan bukti validasi

| Aset | Nilai |
|---|---|
| Checkpoint pasca-pemulihan | `recovery-work\staging\checkpoints\checkpoint-6a-tambun.sql` — 789.727 B, SHA-256 `0db0e6006c0f28e9…` |
| WXR dipakai | `tambun-staging-pilot.noattach.wxr` (sumber: `tambun-staging-pilot.wxr`, SHA-256 `a1a3c141b54dcceb…`) |
| Manifest media | `tambun-media-manifest.tsv` (SHA-256 `c6e3e27d474ab69f…`) |
| Peta ID | `staging\tambun\id-mapping.tsv` (147 pasangan), `id-mapping-att.tsv` (120 lampiran deterministik) |
| Log/eveidence | `staging\logs\import-tambun.log`, `staging\tambun\logs\verify-6a.txt`, `staging\tambun\logs\http-6a.txt` |

**⚠️ Penyimpangan yang saya akui:** **tidak ada checkpoint pra-perubahan** untuk situs ini (wajib menurut §3.7 fase Anda). Kondisi pra-perubahan adalah instalasi bersih yang dapat direproduksi persis; jalur rollback = hapus direktori + DB staging lalu jalankan ulang `pipeline6a.py` (tanpa menyentuh dump/bukti).

## 7. Pengujian: lulus / gagal / belum dilakukan

**LULUS:**
- **12 halaman + beranda = 13/13 HTTP 200** (`/laman-contoh` sudah dihapus sebagai sisa bawaan WordPress).
- Lampiran **120/120** ada di disk; mime lengkap; 119 thumbnail lokal.
- JSON Elementor **18 valid / 0 invalid**.
- IoC **0/0** pada `options` dan `postmeta`; tidak ada `sso_token`/`tema_webshell`/`wp2shell`; 1 pengguna.
- URL produksi **0 sisa** pada konten, metadata, HTML.
- Galat PHP pada halaman yang dirender: **0 kemunculan**.
- Tidak ada berkas kompromi (§3.2); 0 berkas PHP/JS/HTML di `uploads`.
- Berkas `uploads` = **tepat manifest** + turunan lokal (tidak ada unduhan eksternal).

**BELUM dilakukan (jangan dianggap lulus):**
- Perbandingan visual/tata letak terhadap situs asli (desktop & mobile) — belum diperiksa.
- Uji kesalahan JavaScript di konsol browser — belum diukur.
- Uji interaksi formulir (templat Pro).
- Konsistensi 3 item menu yang tidak termigrasi (22 → 19) — dugaan item yatim, **belum diverifikasi**.
- 1 halaman draft: belum ditinjau.

## 8. Kekurangan visual dan fungsi yang tersisa

1. **Widget/templat Elementor Pro tidak tersedia** → header/footer/single/archive/home memakai padanan gratis; paritas tampilan belum dijamin.
2. **`default-kit-2`** (kit kosong buatan Elementor saat aktivasi) masih ada; percobaan hapus memicu prompt Elementor ("Keep my settings") sehingga tidak tuntas — inert, perlu keputusan Anda.
3. Media yang dirujuk konten tetapi tidak ada di dump dibiarkan kosong (tidak ditebak).
4. Tidak ada konten berita di sumber, jadi halaman berita tetap kosong secara wajar.

## 9. Langkah lanjutan dan keputusan yang membutuhkan persetujuan

**Perlu keputusan Anda:**
1. Prioritas berikutnya setelah Cibitung & Tambun: **paritas visual**, atau lanjut ke **Sragen** (satu-satunya situs yang belum dipulihkan).
2. Apakah boleh menindak `default-kit-2` dan meninjau 1 halaman draft.
3. Apakah Elementor Pro berlisensi sekolah akan diadakan (mempengaruhi paritas 4 templat Theme Builder di Tambun dan 6 di Cibitung).
4. Rotasi kredensial lama di sisi produksi — di luar cakupan staging.
