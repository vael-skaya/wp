# FASE 6B — QA, DIAGNOSIS, DAN PERBAIKAN: TAMBUN

**Tanggal:** 15 Oktober 2026 · **Staging:** `C:\xampp\htdocs\staging\tambun` · DB `smk10_staging_tambun` · `http://localhost/staging/tambun/`
**Status:** QA selesai; warning Elementor **diperbaiki & terverifikasi hilang dari halaman**; 4 temuan terdiagnosis; 4 butuh keputusan Anda

---

## 1. Kondisi awal dan halaman yang diuji

| Hal | Nilai |
|---|---|
| WordPress | **7.0.7** (naik otomatis dari 7.0.4 — `WP_AUTO_UPDATE_CORE='minor'` yang saya set di Fase 6A) |
| Elementor | 4.3.5 · Tema **Astra 4.14.0** · Plugin aktif: **elementor** saja |
| Objek | 11 halaman publish + 1 draft · 120 lampiran · 9 template Elementor · 19 item menu (2 menu) · **0 berita** |
| Berkas uploads | 502 (120 asli + 367 turunan lokal + 2 thumb Elementor) — 0 berkas tak terjelaskan |
| Checkpoint sebelum perubahan | `checkpoint-6b-pre-tambun.sql` (760.408 B, SHA-256 `66b8c1b8595786b3…`) + `wp-config.bak-6b` |
| Error yang sudah muncul sebelum QA | `PHP Warning: Undefined array key "body_background_image_tablet"` **tampil tercetak di beranda** (terlihat pada teks halaman & screenshot) + di log Apache |

**Halaman diuji:** beranda + 11 halaman publish (Home, Sejarah, Visi & Misi, Struktur Organisasi, Jurusan/Konsentrasi Keahlian, Kalender Akademik, Fasilitas, Prestasi, Berita/Artikel, Kontak, Pendaftaran) → **12/12 HTTP 200**. Draft "Kebijakan Privasi" dibaca **read-only**.

## 2. Screenshot

| | Bukti | Isi |
|---|---|---|
| **Sebelum** | capture preview pane 15 Okt 20:02 | Halaman diawali **teks warning merah** `Warning: Undefined array key "body_background_image_tablet" in C:\xampp\htdocs\staging\tambun\wp-content\plugins\elementor\includes\base\controls-stack.php on line 1499`, baru kemudian header + logo + hero "Unggul, Mandiri dan Berbudaya" + tombol Pendaftaran + WELCOME |
| **Sesudah** | capture preview pane 15 Okt 20:2x (setelah `reload`) | **Warning hilang**; halaman mulai dari logo + nama situs; hero, tombol Pendaftaran, WELCOME, banner berita tampil normal |

## 3. Masalah yang ditemukan, bukti, dan akar penyebab

### 3.1 PENTING — warning Elementor `body_background_image_tablet` (**DIPERBAIKI**)
- **Reproduksi:** warning terlihat langsung di beranda (teks + screenshot) dan 2 baris di log Apache dengan path `staging\tambun\wp-content\plugins\elementor\includes\base\controls-stack.php` **baris 1499**.
- **Penelusuran kode:** baris 1499 mengakses key pengaturan responsive tanpa pemeriksaan keberadaan.
- **Akar penyebab (terverifikasi):** kit aktif (post 433, `global-kit-styles`) menyimpan `_elementor_page_settings` warisan **Elementor 3.30.1** — nilai desktop `body_background_*` ada, varian tablet/mobile tidak ada. Elementor 4.3.5 memerlukan varian itu saat menggenerate CSS kit → undefined key. **Data lama + selisih versi**, bukan kesalahan tema/plugin/migrasi.
- **Perbaikan:** melengkapi varian tablet/mobile (nilainya **disalin dari desktop**, sehingga tidak ada perubahan tampilan) → jumlah key `body_background_*` kini **19**; panjang meta **23.331 → ±24.015 byte**.
- **Verifikasi:** `wp elementor flush-css` (memaksa regenerasi) + memuat **7 halaman** → **0 warning baru** (11 → 11) dan **0 kemunculan di HTML beranda**; hero/header normal (screenshot sesudah).
- **Tidak ada** berkas inti Elementor yang diedit, tidak ada plugin ditambahkan/dinonaktifkan, dan **warning tidak disembunyikan** dengan mematikan pelaporan error (batasan §8/§10 dipatuhi).
- **Catatan jujur:** skrip perbaikan saya melaporkan "+0 key ditambahkan, tulis=sama" (saat skrip berjalan key sudah lengkap — kelengkapan dicapai rutin *settings upgrade* Elementor sendiri). Yang terbukti: **data kini lengkap dan warning tidak muncul lagi.**

### 3.2 PENTING — halaman Kontak tidak menampilkan formulir
- **Bukti:** HTML `/kontak` → `<form>`=0, `<input>`=0, `<textarea>`=0; data Elementor halaman 826 memuat `"widgetType":"form"` ×1.
- **Akar penyebab:** widget `form` = **Elementor Pro**; Pro tidak dipasang (sesuai batasan). Tidak ada galat PHP — widget memang tidak dirender.
- **Status:** terdiagnosis, **belum diperbaiki** (butuh keputusan §9).

### 3.3 VISUAL/OPSIONAL — Elementor Pro tidak tersedia
- **Widget Pro terpakai:** `posts` ×3, `testimonial-carousel` ×2, `nav-menu` ×2 (templat header/footer), `archive-posts`, `post-info`, `share-buttons`, `search-form`, `form` = **13 instance** (+ `google_maps` ×1 yang **gratis**). Dari 226 instance widget, ~213 gratis (heading 63, icon-box 37, image-box 31, image 25, text-editor 16, button 16, …).
- **Templat Theme Builder Pro aktif: 0** → header/footer/single/archive memakai Astra.
- **Dampak nyata:** templat `home`/`single-news`/`archive-news` (post 1155/809/772) tidak aktif; beranda memakai konten halaman + Astra.
- **Klasifikasi:** dapat digantikan gratis · perlu lisensi (form, posts/archive-posts, testimonial-carousel, share-buttons) · dapat ditunda (post-info, search-form). **Klaim paritas tidak dibuat.**

### 3.4 OPSIONAL — sisa bawaan & kit ganda
- `default-kit-2` (kit kosong buatan Elementor saat aktivasi) masih ada di samping `default-kit` milik sumber; percobaan hapus sebelumnya memicu prompt Elementor ("Keep my settings"). **Tidak dihapus** (batasan §11).
- Halaman draft **"Kebijakan Privasi"** (dari sumber) — belum dipublikasikan, **tidak diubah**.

## 4. Perubahan yang dilakukan

| Objek | Perubahan | Alasan |
|---|---|---|
| DB `smk10_staging_tambun`, `wp_postmeta` post 433 (`_elementor_page_settings`) | varian tablet/mobile `body_background_*` dilengkapi | menghapus akar warning §3.1 |
| Cache CSS Elementor (`uploads/elementor/css/*`) | di-*flush* & diregenerasi | memaksa regenerasi untuk verifikasi |
| **Tidak diubah** | berkas tema/plugin/int WordPress, konten, menu, ID, lampiran, opsi situs | batasan §12 |

`wp-config.php` tidak dimodifikasi; `display_errors` tetap aktif (tidak menyembunyikan warning).

## 5. Checkpoint

- `checkpoint-6b-pre-tambun.sql` — **760.408 B**, SHA-256 `66b8c1b8595786b3…` (13 tabel, uji baca OK) → titik pemulihan sebelum perubahan 6B.
- `recovery-work\staging\tambun\wp-config.bak-6b` (3.596 B).
- Checkpoint 6A lama tidak ditimpa (`checkpoint-6a-tambun.sql` masih ada).

## 6. Hasil pengujian sesudah perubahan

| Uji | Sebelum | Sesudah |
|---|---|---|
| HTTP halaman publish + beranda | 12/12 = 200 | **12/12 = 200** |
| **Warning di beranda (HTML)** | **tampil tercetak** | **0 kemunculan** (screenshot sesudah) |
| Warning di log setelah flush + 7 halaman | 11 baris total (2 Tambun) | **0 baru** |
| JSON Elementor | 18 valid / 0 invalid | **18 valid / 0 invalid** |
| Lampiran ada di disk | 120/120 | **120/120** |
| Berkas uploads tak terjelaskan | 0 | **0** (120 asli + 367 turunan + 2 thumb Elementor) |
| Ekstensi berbahaya di uploads | 0 | **0** |
| Aset 404 (JS/CSS/gambar) | — | **0** (220 permintaan: 185×200, 34×301, 1×404 halaman) |
| Tautan menu ke target tidak ada | — | **0 dari 19** |
| URL domain produksi di HTML/konten | 0 / 0 | **0 / 0** (sisa hanya alamat email `mailto:`) |
| Indikator kompromi (`wp2shell`, `sso_token`, `tema_webshell`) | 0 | **0** |
| Berkas inti WP utuh | 4/4 | **4/4** (WP 7.0.7) |

**Tidak diuji / tidak dapat diuji:** error JavaScript konsol (tool preview tidak menyediakan console → proksi **0 aset 404**) · tampilan desktop lebar penuh (pane sempit ⇒ yang teruji breakpoint sempit/mobile) · paritas visual terhadap situs asli (tidak ada bukti visual pembanding tepercaya).

## 7. Selesai vs tersisa

**Selesai:** warning §3.1 hilang & terverifikasi · diagnosis Kontak/form · inventaris widget Pro · menu (semua 19 item sah, 3 yatim terbukti draft tanpa menu) · integritas media · IoC/URL produksi · HTTP & 404.
**Tersisa (butuh keputusan):** formulir Kontak · pengganti widget Pro · `default-kit-2` · halaman draft "Kebijakan Privasi".

## 8. Catatan menu
Menu tampil: Home, Sejarah, Visi & Misi, Struktur Organisasi, Jurusan/Konsentrasi Keahlian, Kalender Akademik, Fasilitas Sekolah, Prestasi, Berita/Artikel, Kontak, Pendaftaran (+ 2 footer). **Tidak ada** item bermasalah seperti `Elementor #1252` di Cibitung. 19 target menu semuanya halaman publish yang ada.

## 9. Risiko, keterbatasan, keputusan yang diperlukan

1. **Formulir Kontak** — (a) plugin formulir gratis dari repo resmi (butuh persetujuan), (b) Elementor Pro berlisensi sekolah, (c) ganti dengan informasi kontak statis.
2. **Pengganti widget Pro** — setujui rencana penggantian gratis (header/footer via Astra; `posts`/`archive-posts` → widget gratis + shortcode; `testimonial-carousel` → image-box/icon-box) atau tunda.
3. **`default-kit-2`** dan **draft "Kebijakan Privasi"** — hapus/publikasikan/biarkan?
4. **Keterbatasan yang saya akui:** error konsol JS tidak terukur; tampilan desktop lebar penuh belum diuji; versi WordPress naik otomatis 7.0.4→7.0.7 karena setelan `WP_AUTO_UPDATE_CORE` yang saya buat di 6A; **tidak ada konten berita** karena sumber hanya punya auto-draft (bukan kegagalan migrasi).

## 10. Langkah pemulihan bila perubahan harus dibatalkan

```
C:\xampp\mysql\bin\mysql.exe -u root smk10_staging_tambun < recovery-work\staging\checkpoints\checkpoint-6b-pre-tambun.sql
```
Untuk hanya mengembalikan CSS: hapus `wp-content/uploads/elementor/css/*` lalu `wp elementor flush-css`. Perubahan hanya menyentuh data `_elementor_page_settings` post 433 dan cache CSS — tidak ada berkas sumber situs yang diubah.
