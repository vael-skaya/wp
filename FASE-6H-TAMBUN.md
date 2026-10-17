# FASE 6H — TAMBUN (verifikasi, tanpa perubahan)

**Tanggal:** 16 Oktober 2026 · **Staging:** `C:\xampp\htdocs\staging\tambun` · **Database (diverifikasi dari `wp-config.php`):** `smk10_staging_tambun` · bukan repositori Git
**Ringkas:** label bersih ✓ · logo Tambun 72/48 px ✓ · kontainer `6c931a7` tersembunyi ✓ · **overflow 0 px** ✓ · uji gambar/tabel/iframe ✓ · **console error = 0** ✓ · **tidak ada perubahan**

## 1. Checkpoint
Tidak ada perubahan ⇒ tidak ada checkpoint baru. Checkpoint rujukan: `checkpoint-6g-pre-tambun.sql` (930.777 B), `checkpoint-6f-pre-tambun.sql`, `checkpoint-6a-tambun.sql` (tidak ditimpa).

## 2. Status label staging
`blogname` = **SMK 10 November Tambun Selatan** ✓ · `'STAGING'`/`'pemulihan'` pada HTML = **0 / 0** ✓ · `<title>` benar ✓ · kemunculan "staging" lain hanya URL aset lokal ✓.

## 3. Logo & header (terukur via CDP)
| Viewport | Logo | Toggle | Overflow | Elemen meluber |
|---|---|---|---|---|
| 360×800 | **48×48 px** | 41 px | **0 px** | 0 |
| 390×844 | **48×48 px** | 41 px | **0 px** | 0 |
| 412×915 | **48×48 px** | 41 px | **0 px** | 0 |
| 768×1024 | **48×48 px** | 43 px | **0 px** | 0 |
| 1366×768 | **72×72 px** | 0 | −15 px | 0 |
| 1920×1080 | **72×72 px** | 0 | −15 px | 0 |

Berkas logo tetap lampiran **1316** (aset Tambun) — **tidak** ditukar dengan logo Cibitung; rasio asli terjaga.

## 4. Kontainer Elementor `6c931a7`
`display: none` ✓ (beranda). Tidak ada bagian lain yang hilang; **tidak dihapus dari data** (menunggu evaluasi terpisah).

## 5. Hasil per viewport & halaman
Beranda (6 viewport) + Kontak + Pendaftaran + Sejarah (390 & 1366): **semua 200**, **overflow 0 px** (−15 px di desktop = lebar scrollbar), **0 elemen meluber**.

## 6. Gambar, tabel, iframe
- 21 `<img>`, 17 `loading="lazy"` (belum termuat karena di luar viewport) · **404 gambar = 0** ⇒ tidak ada gambar rusak.
- **Uji gambar nyata** (disuntikkan ke DOM, tidak disimpan): natural 740 px → dirender **320/350/372/728/740 px** (360/390/412/768/1366) ⇒ tidak meluber ✓.
- **Uji tabel lebar**: dibatasi 320–1200 px ⇒ tidak meluber ✓.
- Iframe: beranda 300–640 px; Kontak 350 px @390 ✓.

## 7. Console / network
**Console error kosong (`[]`) pada 12 kombinasi halaman×viewport** ✓ (hook `window.onerror` + `console.error` via CDP). Permintaan ke domain produksi: **0**.

## 8. Daftar perubahan
**Tidak ada** (verifikasi murni).

## 9. Bukti visual
`…\staging\tambun\qa6h\` — `beranda@390x844.png`, `beranda@1366x768.png`, `kontak@390x844.png`.

## 10. Masalah yang belum terselesaikan
1. Belum ada konten guru nyata untuk diuji (uji sintetis lulus).
2. Kontainer kosong `6c931a7` masih ada di data.
3. **Berita kosong** — sumber Tambun tidak memiliki berita sah (bukan masalah teknis; perlu keputusan konten sekolah).
4. Halaman lain tidak diuji satu per satu.

## 11. Keterbatasan
Emulasi viewport CDP (**bukan Android fisik**) · console via hook JS · 404 hanya dari log terakhir.

## 12. Konfirmasi
- **Produksi tidak disentuh**; 0 permintaan ke domain produksi; tidak ada deployment.
- Situs lain di luar cakupan tidak diubah oleh pekerjaan Tambun ini.
- **`goly1264_ams` tidak diakses, tidak di-query, tidak di-backup, tidak di-restore, tidak diubah.**
- **`Downloads\admin.php` dibiarkan persis** (84.070 B, mtime 2026-10-13 12:30:18). Berkas sumber logo tidak diubah.
