# FASE 6A — PEMULIHAN STAGING CIBITUNG

**Tanggal:** 15 Oktober 2026 · **Metode:** pemulihan selektif (konverter offline → importer), sama seperti pilot Kebumen
**Status:** pemulihan dasar selesai; seluruh pengujian yang dijalankan LULUS; **ada 1 insiden yang saya laporkan terbuka** (lihat §3.3)

---

## 1. Kondisi awal dan path staging

| Hal | Nilai |
|---|---|
| Staging | `C:\xampp\htdocs\staging\cibitung` |
| URL | `http://localhost/staging/cibitung` |
| Database | `smk10_staging_cibitung` (13 tabel, prefix `wp_`) |
| WordPress | 7.0.4 id_ID (paket resmi `wordpress-id_ID.zip`) |
| Tema | Astra 4.14.0 (repo resmi) |
| Plugin | Elementor (aktif, repo resmi); Akismet & Hello Dolly (nonaktif); `wp-cli`, `wordpress-importer` (dinonaktifkan setelah dipakai) |
| Administrator | `cibitung_admin` (1 akun baru; sandi acak 32 karakter disimpan di berkas lokal, tidak pernah dicetak) |
| Sebelum fase | instalasi ini **belum ada**; direktori, DB, dan berkas dibuat di fase ini |

## 2. Database sumber dan target (tanpa kredensial)

| | Sumber | Target |
|---|---|---|
| Berkas | `D:\backup-database-smk10november\smkq8953_cbitung.sql` (85.372.896 B, SHA-256 `85d9820370f9044b…`) | `smk10_staging_cibitung` |
| Prefix tabel | `cbt_` | `wp_` |
| Domain lama | `smk10novembercibitung.sch.id` | diganti ke URL lokal |
| Jumlah tabel | 131 (38 berisi, 93 kosong) | 13 tabel inti |
| `posts` / `postmeta` / `options` / `users` / `usermeta` | 681 / 9.361 / 307 / **90** / 1.202 | 195 / 613 / 178 / **1** / 3 |

Kredensial DB staging memakai user minimum-privilege yang sudah ada (`smk10_staging`, hak hanya pada `smk10_staging_%`), tanpa membuat user baru.

## 3. Temuan keamanan dan artefak yang TIDAK dipulihkan

### 3.1 Indikator kompromi di sumber (54 temuan)
- `options` id 41052 — **plugin palsu** (`wp-wordfence-security`), tidak dimigrasikan.
- `usermeta` id 347 dan `users` id 4–18 dan seterusnya — penanda **`wp2shell`** (akun penyerang).
- `options` non-bawaan lain yang tergolong sensitif/konfigurasi auth (`mailserver_pass`, dsb.) — tidak dimigrasikan.
- 2 nilai serialisasi sumber berlabel INVALID pada inventaris lama (akibat bug `unescape` yang sudah diperbaiki; kini tidak ada nilai invalid di staging).

### 3.2 Yang dinyatakan TIDAK dipulihkan (dan terbukti tidak ada di staging)
Seluruh **90 akun lama** dan 1.202 `usermeta` · seluruh tabel plugin (WooCommerce, MailPoet, WPForms/Formidable, Action Scheduler, GLA, dsb.) · 11 opsi sensitif/kredensial · semua cache/transien · `elementor_unfiltered_files_upload` · tema & plugin lama · plugin palsu `wp-wordfence-security`, `ioyntkq`, `wp-file-manager` · berkas PHP/JS/HTML di `uploads` (**0 berkas**, terverifikasi) · `wp-config.php` lama.

Staging memuat **hanya** WordPress resmi, Astra & Elementor resmi, dan konten/media yang sudah diperiksa.

### 3.3 ⚠️ INSIDEN — importer menarik media dari domain produksi (dilaporkan terbuka)
**Apa yang terjadi.** Pada impor **pertama**, plugin `wordpress-importer` (perilaku bawaan) mencoba mengunduh media dari URL produksi. Ia **berhasil mengunduh 47 berkas** ke `uploads`: 14 berkas asli (nama diawali `kebumen…`, mis. `2025/12/kebumen.webp`, `kebumen-1.webp`, `kebumen-2.webp`) + 33 turunan ukuran. Ini melanggar batas "hanya media terverifikasi" dan berisiko memuat berkas dari situs produksi yang belum diperiksa.

**Deteksi.** Perbandingan daftar berkas `uploads` vs manifest media: 119 berkas tanpa padanan manifest, semuanya bermtime saat impor.

**Remediasi yang sudah dijalankan.**
1. Seluruh tabel konten dikosongkan dan **119 berkas non-manifest dihapus** (`wipe`).
2. WXR disanitasi: **153 `<wp:attachment_url>` dikosongkan** → importer tidak punya URL untuk diunduh; ditambah `define('WP_HTTP_BLOCK_EXTERNAL', true)` **hanya selama impor** (dicabut sesudahnya).
3. Impor ulang dari WXR tersanitasi; lampiran dibangun **deterministik** dari dump+manifest (153/153, 0 gagal).
4. Verifikasi ulang: berkas di `uploads` = 153 berkas manifest + 494 turunan yang **dibuat lokal** (`wp media regenerate`, 150/151 gambar) + 14 CSS Elementor lokal. **Tidak ada lagi berkas tak terjelaskan.**

**Yang jujur harus dicatat (keterbatasan):** daftar lengkap 14 berkas itu **tidak seluruhnya tersimpan** — perintah diagnosa yang mencoba merekam hash-nya timeout, sehingga hanya 3 nama yang terkonfirmasi dari output (dicatat di atas) dan 1 berkas jejak yang tersisa kosong. Turunan hasil unduhan sudah terhapus; **tidak ada berkas unduhan yang tersisa di staging**. Jika Anda ingin bukti forensik lengkap atas 47 berkas itu, langkah itu harus diulang pada salinan staging lain (tidak saya lakukan tanpa persetujuan).

**Pencegahan untuk sisa fase:** setiap impor kini memakai WXR `*.noattach.wxr` (URL lampiran kosong) + blokir HTTP eksternal sementara.

## 4. Inventarisasi tabel dan alasan data dipilih/ditolak

| Kelompok | Sumber | Dipilih | Alasan |
|---|---|---|---|
| `posts` | 681 | **195** | Hanya `page` (15), `attachment` (153), `elementor_library` (7), `nav_menu_item` (19), `wp_navigation` (1). Ditolak: 153 revisi, 149 oembed, 89 customize_changeset, 89 `request`, 1 `envato_tk_import`, 2 post sampah berjudul `x` |
| `postmeta` | 9.361 | **613** | Hanya milik post yang ikut; `_wp_attachment_metadata` diregenerasi lokal; cache Elementor dibuang (36) |
| `terms`/`term_taxonomy`/`term_relationships` | 62/62/31 | 9/9/26 | Hanya term yang direferensikan item ikut (menu + `elementor_library_type`) |
| `options` | 307 | **178** | Hanya opsi UI/identitas terpilih (§5). 243 opsi non-bawaan lainnya **ditolak** |
| `users`/`usermeta` | 90/1.202 | **1/3** | Semua akun lama mencurigakan; dibuat 1 admin baru |
| 29 tabel plugin berisi data | 31 | **0** | Data plugin lama (termasuk data pribadi) tidak diperlukan untuk pemulihan dasar |
| `termmeta`, `comments`, `commentmeta` | 0/… | **0** | Tidak ada di sumber / tidak diperlukan |

## 5. Konten dan media yang berhasil dipulihkan

- **Halaman:** 12 publish + 3 draft (total 15 = jumlah sumber), semuanya HTTP 200.
- **Lampiran/media:** **153/153** berkas hasil salinan terverifikasi (SHA-256 sebelum–sesudah, 69.387.312 B); **150 gambar** thumbnail diregenerasi lokal; 0 berkas gagal.
- **Templat Elementor:** 7 item library termasuk **kit aktif 433** (`global-kit-styles`, kit pihak ketiga "Skoola") dan 6 templat Theme Builder (header, footer, single-news, archive-news, coming-soon, default-kit).
- **Menu:** **19 item** dalam 2 menu (Main + Footer), dibuat langsung oleh importer (blok `<wp:term>` pada WXR buatan sendiri — perbaikan atas masalah Kebumen).
- **Data Elementor:** `_elementor_data` **18 nilai, 18 valid JSON, 0 invalid**; 13 `_elementor_page_settings` + 2 `_menu_item_classes` diterapkan dari metadata tertunda (K-8); 36 nilai cache dibuang.
- **Opsi UI/publikasi** (dari dump sumber): `date_format`, `time_format`, `site_icon`, `blogdescription`, `show_on_front`, `page_on_front`, `page_for_posts`, `elementor_cpt_support`, `elementor_google_font`, `nav_menu_options`, `site_logo`, `elementor_active_kit`, ditambah `WPLANG=id_ID` dan `timezone_string=Asia/Jakarta`.
- **URL produksi:** 27 penggantian diterapkan (12 polos + 15 ter-escape) → **0 sisa** di `post_content`, `postmeta`, maupun HTML.

## 6. Checkpoint dan bukti validasi

| Aset | Nilai |
|---|---|
| Checkpoint pasca-pemulihan | `recovery-work\staging\checkpoints\checkpoint-6a-cibitung.sql` — 989.457 B, SHA-256 `e75035bc7f99a06dfa3d…` |
| WXR dipakai | `cibitung-staging-pilot.noattach.wxr` (sumber: `cibitung-staging-pilot.wxr`, SHA-256 `4764d2b2adfed9ec…`) |
| Manifest media | `cibitung-media-manifest.tsv` (SHA-256 `30085c4e4a8ab19e…`) |
| Peta ID | `staging\cibitung\id-mapping.tsv` (179 pasangan), `id-mapping-att.tsv` (153 lampiran deterministik) |
| Log | `staging\logs\import-cibitung.log`, `staging\cibitung\logs\http-6a.txt`, `staging\cibitung\purgatt-6a.tsv`, `staging\cibitung\fixmime.sql` |

**⚠️ Penyimpangan yang saya akui:** **tidak ada checkpoint pra-perubahan** untuk situs ini (wajib menurut §3.7 fase Anda). Sebabnya: kondisi pra-perubahan adalah instalasi WordPress bersih yang **dapat direproduksi persis** (paket resmi + skrip `pipeline6a.py install`). Jalur rollback efektif = hapus direktori + DB staging ini lalu jalankan ulang pipeline (tanpa menyentuh dump/bukti). Checkpoint pasca-pemulihan di atas adalah titik pemulihan bila langkah berikutnya bermasalah.

## 7. Pengujian: lulus / gagal / belum dilakukan

**LULUS (dijalankan, dengan bukti):**
- Lampiran: 153/153 berkas ada di disk; mime types lengkap (68 jpeg, 56 webp, 26 png, 2 mp4, 1 pdf).
- Contoh media tampil: 12/12 URL gambar pada beranda → HTTP 200, 0 gagal.
- HTTP: beranda + 12 halaman = **13/13 → 200** (`staging\cibitung\logs\http-6a.txt`).
- JSON Elementor: 18 valid / **0 invalid**.
- Indikator kompromi: **0** di `options` dan `postmeta`; tidak ada opsi sensitif baru; 1 pengguna saja.
- URL produksi: **0 sisa** pada konten, metadata, dan HTML beranda.
- Galat PHP pada halaman yang dirender: **0 kemunculan** `Fatal/Parse/Warning/Notice`.
- Integritas berkas: 0 berkas PHP/JS/HTML di `uploads`; tidak ada plugin/tema dari instalasi lama.

**BELUM dilakukan (jangan dianggap lulus):**
- Perbandingan visual/tata letak terhadap situs asli (desktop & mobile) — **belum diperiksa**.
- Uji kesalahan JavaScript di konsol browser — **belum diukur** (hanya pemindaian teks HTML untuk galat PHP).
- Uji formulir/kontak interaktif (Kontak memakai widget **Pro** yang tidak tersedia).
- Konsistensi 3 item menu yang tidak termigrasi (**22 → 19**): dugaan item yatim (menu induknya tidak ada), **belum diverifikasi**.
- 3 halaman berstatus draft: belum ditinjau isinya.

## 8. Kekurangan visual dan fungsi yang tersisa

1. **Widget/templat Elementor Pro tidak tersedia** (komersial) → halaman Kontak dan 6 templat Theme Builder memakai padanan gratis; tampilan bisa berbeda dari aslinya.
2. Logo/tema lama (`site_logo`) mengarah ke lampiran yang ada, namun beberapa elemen desain Pro belum punya padanan.
3. `default-kit` bawaan Elementor ikut ada (bawaan dari sumber) — inert, tidak mengganggu.
4. Tautan yang dulu memakai berkas media yang tidak ada di dump tetap kosong (dibiarkan, tidak ditebak).

## 9. Langkah lanjutan dan keputusan yang membutuhkan persetujuan

**Perlu keputusan Anda:**
1. Apakah pemulihan **paritas visual** perlu dikejar untuk Cibitung sebelum berpindah ke Tambun/Sragen (termasuk keputusan membeli/memasang Elementor Pro berlisensi sekolah).
2. Apakah saya boleh melakukan **pemeriksaan lanjutan 3 item menu yatim** dan meninjau 3 halaman draft.
3. Apakah staging Kebumen perlu diperiksa ulang untuk kemungkinan **berkas turunan hasil unduhan produksi** (temuan §3.3; saya tidak menyentuh staging Kebumen sesuai batas fase ini).
4. Rotasi kredensial lama di sisi sekolah (produksi) — di luar cakupan staging.
